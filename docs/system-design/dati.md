---
title: I dati come spina dorsale
sidebar_position: 4
---

# I dati come spina dorsale

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 5.4</span>
  <span>~11 min di lettura</span>
</div>

<p class="lesson-lead">Un LLM eccellente su dati scadenti produce risposte scadenti. I dati — qualità, struttura, freshness, pipeline di aggiornamento — sono il collo di bottiglia più sottovalutato nei sistemi AI. E il database vettoriale, che in molti considerano un dettaglio tecnico, è in realtà un componente con trade-off architetturali precisi.</p>

C'è un'abitudine diffusa nei progetti AI: si sceglie il modello con cura e si trattano i dati come un problema da risolvere dopo. È esattamente il contrario di quello che serve. La qualità del modello impatta il ragionamento; la qualità dei dati impatta le premesse su cui il ragionamento avviene. Premesse sbagliate producono conclusioni sbagliate, qualunque sia il modello.

## Il problema con i dati grezzi

I dati che hai in azienda raramente sono pronti per un sistema AI. Provengono da fonti diverse, hanno formati diversi, contengono ridondanze, errori, informazioni obsolete.

**Qualità.** Un documento con informazioni contraddittorie, o obsoleto rispetto al prodotto attuale, ingiganterà il problema se il sistema lo usa come fonte di verità. Il garbage-in-garbage-out vale più con i LLM che con qualsiasi altro sistema — perché il LLM *sembra* convincente anche quando sbaglia.

**Formato.** PDF, Word, HTML, database relazionali, fogli Excel: ognuno richiede un parser diverso. I parser non sono tutti uguali — un PDF con layout a colonne estratto con un parser generico diventa testo mescolato, perdendo la struttura del documento.

**Granularità.** Un documento da 50 pagine non può entrare integralmente nel contesto del modello. Deve essere spezzato in **chunk** — frammenti — di dimensione appropriata. Come si spezza il documento non è indifferente: uno chunk troppo piccolo perde contesto; uno troppo grande riduce la precisione del retrieval.

**Freshness.** I dati cambiano nel tempo. Un sistema RAG che usa un indice vecchio di sei mesi risponde su informazioni obsolete. La pipeline di aggiornamento — quanto spesso si re-indicizza, cosa si aggiorna in modo incrementale — è parte del design.

## Il chunking: come non sbagliarlo

Il chunking è la scelta di come spezzare i documenti in frammenti da indicizzare. Impatta direttamente la qualità del retrieval.

**Fixed-size chunking** — si spezza ogni N token con M token di overlap. Semplice, ma rompe spesso le frasi a metà e separa concetti correlati. Funziona sui testi brevi e uniformi.

**Semantic chunking** — si spezza sui confini semantici (paragrafi, sezioni, cambi di argomento rilevati con un modello). Produce chunk più coerenti, migliora il retrieval, ma richiede più compute in fase di indicizzazione.

**Chunk gerarchici** — si mantiene sia una versione fine (frasi, paragrafi) che una versione grossolana (sezione, documento). La ricerca avviene sul fine; il contesto viene allargato al grossolano per avere più contesto intorno al risultato rilevante. Tecnica chiamata **parent document retrieval**.

La dimensione giusta dipende dal task. Domande fattuali ("qual è la deadline?") funzionano meglio con chunk piccoli e precisi. Domande di comprensione ("spiegami come funziona il processo X") funzionano meglio con chunk più grandi che danno contesto.

## Il database vettoriale: non è un dettaglio

Un **database vettoriale** — *vector store* — indicizza gli embedding dei chunk e permette di recuperare quelli con similarità più alta a una query. È il cuore del RAG.

<details>
<summary>Come funziona l'indicizzazione vettoriale</summary>

Quando aggiungi un documento al vector store, un modello di embedding converte ogni chunk in un vettore — un array di numeri che rappresenta il significato semantico nel embedding space (lezione 0.2). Questi vettori vengono indicizzati con strutture dati specializzate (HNSW — *Hierarchical Navigable Small World*, IVF — *Inverted File Index*) che permettono la ricerca del vicino approssimativo in modo efficiente. Al momento della query, la query stessa viene convertita in vettore con lo stesso modello di embedding, e si cercano i vettori più simili. La similarità è tipicamente calcolata con cosine similarity o dot product.
</details>

**I trade-off del vector store** non sono tutti uguali:

| Dimensione | Trade-off |
|---|---|
| **Velocità di query** | Indici approssimati (HNSW) sono veloci ma non garantiscono il risultato esatto; indici esatti sono lenti su grandi dataset |
| **Scalabilità** | Quanti vettori puoi indicizzare prima che la query rallenti? |
| **Aggiornamento** | Alcuni store sono ottimizzati per lettura (ri-build dell'indice costoso); altri supportano aggiornamenti incrementali |
| **Metadata filtering** | Puoi filtrare per attributi (data, categoria, fonte) prima della ricerca semantica? |
| **Hosting** | Cloud-managed (Pinecone, Weaviate Cloud) vs self-hosted (Qdrant, Milvus, pgvector) |

**pgvector** merita menzione separata: è un'estensione di PostgreSQL che aggiunge il tipo vettoriale e la ricerca per similarità. Per dataset < 1 milione di vettori, è spesso la scelta più semplice — non introduce un nuovo sistema da gestire, si usa il database già presente.

## La pipeline di dati

Un sistema RAG in produzione non è un'importazione una tantum. È una pipeline continua:

```
Sorgenti → Estrazione → Pulizia → Chunking → Embedding → Indicizzazione → Vector store
                                                                           ↑
                                                              Aggiornamento incrementale
                                                              (trigger: documento modificato)
```

**Estrazione** — parser per ogni formato sorgente. PDF (PyMuPDF, pdfplumber, servizi come Azure Document Intelligence per layout complessi), HTML (BeautifulSoup, trafilatura), database (query su tabelle specifiche), Word/Excel (python-docx, openpyxl).

**Pulizia** — rimozione di header/footer ripetuti, boilerplate, HTML spazzatura, contenuto duplicato. Un passaggio spesso ignorato che impatta significativamente la qualità.

**Deduplicazione** — contenuto identico o quasi identico da fonti diverse gonfia l'indice e distorce il retrieval verso le fonti più rappresentate.

**Aggiornamento incrementale** — invece di re-indicizzare tutto ogni volta, si tracciano le modifiche (hash del contenuto, data di modifica) e si aggiornano solo i chunk cambiati. Fondamentale per dataset grandi e che cambiano frequentemente.

## I metadati sono dati

Ogni chunk nell'indice vettoriale dovrebbe avere metadati associati: fonte (documento, URL, ID), data di creazione/modifica, categoria, autore, versione. I metadati permettono:

- **Filtering pre-retrieval** — "cerca solo nei documenti aggiornati negli ultimi 30 giorni"
- **Attributione nella risposta** — "questa informazione viene da [documento X]"
- **Gestione della freshness** — escludere automaticamente documenti obsoleti
- **Debugging** — capire da dove viene una risposta quando è sbagliata

Non aggiungere metadati dopo il fatto è doloroso. Vanno progettati nello schema dell'indice dall'inizio.

## Il vero collo di bottiglia

In quasi tutti i progetti AI che partono da dati aziendali reali, il problema non è il modello. Il problema è:
- Documenti in formati difficili da parsare (PDF di scan, documenti Word con formattazione complessa)
- Dati obsoleti o contraddittori nelle sorgenti
- Nessun processo di aggiornamento — il sistema diventa stale in settimane
- Chunking mal calibrato che produce retrieval scadente
- Nessun metadato → nessuna possibilità di filtrare o attribuire

Il modello eccellente su dati scadenti produce risposte scadenti. Il modello medio su dati ottimi produce risposte sorprendentemente buone. Vale sempre la pena investire nella pipeline di dati prima di passare al modello successivo.

## Cosa NON sono i dati in un sistema AI

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Importiamo i documenti e siamo pronti" | L'importazione è il 10% del lavoro. Parsing, pulizia, chunking, aggiornamento sono il resto. |
| "Il vector store è un dettaglio implementativo" | La scelta del vector store e del suo schema impatta scalabilità, query latency e costo. |
| "Chunk grandi = più contesto = meglio" | Chunk troppo grandi riducono la precisione del retrieval. La granularità va calibrata sul task. |
| "I metadati li aggiungiamo dopo" | Lo schema dei metadati va progettato insieme all'indice. Aggiungerli dopo richiede re-indicizzazione. |

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani.

1. Cos'è il chunking e perché la dimensione del chunk impatta il retrieval?
2. Qual è la differenza tra fixed-size chunking e semantic chunking?
3. Descrivi tre trade-off nella scelta di un vector store.
4. Perché i metadati vanno progettati dall'inizio e non aggiunti dopo?
5. Un sistema RAG su documentazione tecnica risponde in modo inaccurato su feature recenti. Quali sono le cause più probabili nella pipeline di dati?

---

## Glossario

- **Chunking** — processo di suddivisione dei documenti in frammenti di dimensione appropriata per l'indicizzazione e il retrieval.
- **Overlap** — sovrapposizione tra chunk consecutivi per preservare il contesto al confine tra due frammenti.
- **Parent document retrieval** — tecnica che usa chunk piccoli per la ricerca e chunk più grandi per fornire contesto al modello.
- **Vector store / database vettoriale** — sistema che indicizza embedding e supporta ricerca per similarità semantica.
- **HNSW** — Hierarchical Navigable Small World; struttura dati per ricerca approssimativa del vicino più simile, standard nei vector store moderni.
- **pgvector** — estensione PostgreSQL che aggiunge supporto nativo ai vettori e alla ricerca per similarità.
- **Freshness** — quanto recente è l'informazione nell'indice rispetto alla sorgente originale.
- **Deduplicazione** — rimozione di contenuto duplicato o quasi-duplicato prima dell'indicizzazione.

---

## Per approfondire

- **Documentazione di Qdrant, Weaviate, Pinecone, pgvector** — confronta le capacità e i trade-off di ogni vector store dalle fonti primarie.
- **"RAG From Scratch"** di LangChain — serie di notebook che mostra concretamente chunking, retrieval, e re-ranking; cerca su GitHub o YouTube.

*Risorse indicate per la ricerca; per i link aggiornati conviene cercarli al momento.*

---

## Prossima lezione

**5.5 Decision drill — System design.** Hai tutti i componenti, i pattern, il triangolo e la pipeline di dati. Il drill ti mette davanti a due scenari da progettare dall'inizio: un chatbot di supporto enterprise e un sistema di analisi documenti legali. Decidi e giustifica.
