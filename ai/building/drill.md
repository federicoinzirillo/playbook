---
title: Decision Drill — Costruire sistemi LLM
sidebar_position: 8
---

# Decision drill — Costruire sistemi LLM

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 1.8</span>
  <span>~20 min (incluso il tempo di lavoro)</span>
</div>

<p class="lesson-lead">Allenamento da architetto: dato un scenario reale con vincoli concreti, decidi l'architettura e giustifica ogni scelta. Lo scopo non è la risposta giusta — è nominare i trade-off che un senior nominerebbe.</p>

Hai costruito il vocabolario della Parte 1: RAG, context engineering, structured output, agenti, MCP. Il drill non verifica che tu li ricordi — verifica che tu sappia usarli per ragionare. Ogni scenario ha più soluzioni valide; la valutazione è sul giudizio, non sulla risposta esatta.

**Come usarlo:** leggi lo scenario, scrivi la tua risposta completa *prima* di aprire la griglia. La griglia non è la soluzione: è quello che avrebbe nominato un senior. Se ne hai mancati alcuni, non significa che la tua risposta fosse sbagliata — significa che ci sono angoli da espandere.

---

## Scenario A — Il motore di ricerca documentale

**Contesto:** una società legale ha 80.000 documenti regolatori (PDF, DOC, pagine web). Gli utenti — avvocati — pongono domande precise dove i codici normativi esatti, i numeri di articolo e i riferimenti incrociati contano. Il modello deve citare la fonte. Budget contenuto (nessun team dedicato all'AI), latenza massima 4 secondi, aggiornamento dei documenti settimanale.

**Cosa ti viene chiesto:** progetta il sistema di retrieval end-to-end. Scegli le componenti, motiva ogni scelta, nomina i trade-off principali e le trappole da evitare.

*Scrivi la tua risposta completa qui — poi apri la griglia.*

<details>
<summary>Griglia di valutazione — apri solo dopo aver scritto la tua risposta</summary>

**Cosa avrebbe nominato un senior:**

**Chunking.** Documenti legali non si spezzano a taglia fissa: si rispetta la struttura naturale (articoli, commi, sezioni). Overlap importante: un articolo può iniziare a metà di un chunk. Chunk gerarchici se le domande sono a volte sul singolo articolo (chunk piccoli), a volte sul contesto di una sezione intera (chunk più grandi).

**Hybrid search, non solo semantico.** Avvocati cercano spesso codici esatti ("articolo 34 GDPR", "d.lgs. 231/2001"). Il dense retrieval da solo fallirebbe su questi match esatti: senza le parole giuste, la similarità semantica non aiuta. Sparse retrieval (BM25) gestisce il match esatto. Hybrid = entrambi con Reciprocal Rank Fusion.

**Reranker.** Con 80k documenti tecnici, la qualità del ranking dopo il primo retrieval è critica. Un reranker cross-encoder valuta la coppia (domanda, chunk) in modo più preciso. Latenza extra ma gestibile: recuperi 50 candidati veloce, rerankate i migliori 10, passi i top 3-5 al modello.

**Citazione forzata.** Structured output: il modello deve produrre risposta + lista di reference esatti (documento, sezione, chunk ID). Non lasciarlo libero di "citare" in modo vago. Valida che i chunk citati siano effettivamente tra quelli passati nel prompt.

**Vector DB.** Con 80k documenti, non serve infrastruttura costosa: pgvector su Postgres è sufficiente e riduce la complessità. Upgrade a un DB dedicato solo se la scala lo richiede.

**Aggiornamento settimanale.** Processo di re-indicizzazione programmato: i nuovi documenti vengono chunkati, embeddati e aggiunti. I documenti modificati richiedono delete + re-ingest. Fondamentale: un ID stabile per ogni chunk, per permettere la ricerca esatta della source alla citazione.

**Trade-off da nominare:**

- *Dimensione del chunk*: chunk troppo piccoli perdono il contesto di un articolo; troppo grandi portano rumore e consumano contesto. Punto di equilibrio tipico: 400-600 token per i paragrafi normativi, con overlap di 50-100 token.
- *Reranker vs latenza*: ogni reranker aggiunge ~500ms-1s. Con latenza max 4s, il budget è stretto. Soluzione: reranker leggero (cross-encoder piccolo) o retrieval più selettivo.
- *Context window e numero di chunk*: passare 10 chunk al modello è rischioso ("lost in the middle"). 3-5 chunk ben selezionati battono 10 mediocri.

**Trappole da evitare:**

- Retrieval solo semantico su documenti con codici normativi precisi → fallisce sui match esatti.
- Non testare il retrieval separatamente dal modello → non sai dove sta il problema.
- Chunk a taglia fissa su PDF → rompe gli articoli a metà, produce chunk privi di senso.
- Lasciare il modello libero di citare senza structured output → le citazioni saranno vaghe o inventate.
</details>

---

## Scenario B — L'assistente agentico per report finanziari

**Contesto:** una società di consulenza vuole un assistente che, data una domanda di analisi (es. "confronta la marginalità di questi tre clienti nell'ultimo trimestre"), recuperi autonomamente i dati dal CRM e dal database finanziario, faccia i calcoli, e produca un report strutturato. I dati sono sensibili. Il team ha 2 sviluppatori e vuole un MVP in 4 settimane.

**Cosa ti viene chiesto:** scegli l'architettura (agente singolo o multi-agent?), i tool necessari, come gestire la sicurezza e cosa mostrare nel MVP vs. cosa posticipare.

<details>
<summary>Griglia di valutazione</summary>

**Cosa avrebbe nominato un senior:**

**Architettura.** Per un MVP, agente singolo. Multi-agent aggiunge complessità — orchestrazione, debug, latenza additiva — che in 4 settimane non si giustifica. L'agente singolo con 3-4 tool ben definiti è il modo più veloce per avere qualcosa in produzione.

**Tool necessari:**
- `query_crm(cliente_id, periodo)` — restituisce dati del cliente
- `query_finanziario(cliente_id, periodo, metrica)` — restituisce marginalità e dati economici
- `calcola_statistiche(dati)` — calcoli aggregati (non affidarli all'LLM per i numeri precisi)
- `genera_report(template, dati)` — structured output per il formato finale

**Sicurezza.** Dati sensibili = accesso ai tool con autenticazione e autorizzazione per utente. L'agente non deve poter accedere ai dati di un cliente se l'utente non è autorizzato. Il controllo non va nel prompt: va nel codice del tool, prima di ogni query. Log immutabile di ogni azione dell'agente (chi ha chiesto cosa, quando).

**Structured output.** Il report finale deve avere un formato fisso che si integra nel sistema esistente. Definisci lo schema e forza l'output.

**MVP vs. posticipare:**
- **MVP:** agente singolo, 4 tool, report strutturato, log delle azioni, autorizzazione base.
- **Posticipare:** multi-agent, confronto automatico multi-periodo, integrazioni aggiuntive, dashboard.

**Trade-off da nominare:**

- *Agente singolo vs. multi-agent*: multi-agent è più parallelizzabile ma molto più complesso. Su 4 settimane e 2 sviluppatori, l'agente singolo è la scelta giusta.
- *LLM per i calcoli numerici*: i modelli fanno errori di calcolo. Non fare i calcoli nell'LLM: passa i numeri raw al tool `calcola_statistiche`, poi dai i risultati all'LLM per il commento.
- *Latenza vs. accuratezza*: un agente che fa 5-6 tool call + reasoning può durare 15-30 secondi. L'utente deve saperlo e avere feedback visivo ("sto recuperando i dati...").

**Trappole da evitare:**

- Autorizzazione nei prompt ("rispondi solo sui clienti del tuo team") → non è sicurezza, è un suggerimento che un utente malevolo può aggirare.
- Calcoli numerici nell'LLM senza verifica → i report finanziari con errori di calcolo sono un disastro.
- Multi-agent da subito → triplica il tempo di debug e sviluppo per un MVP.
</details>

---

## Cosa fare se hai mancato molti punti

Normale: i drill allenano il giudizio, non lo valutano. Torna alla lezione dove il punto mancante è trattato:

- Chunking e retrieval → **1.1 RAG**
- Budget contesto e "lost in the middle" → **1.3 Context engineering**
- Structured output e output verificabile → **1.4 Structured output**
- Scelte architetturali agente singolo vs multi-agent → **1.5-1.6 Agenti**
- Sicurezza del tool calling → **4.1 Prompt injection**, **4.2 Sicurezza agentica**
