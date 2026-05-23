---
title: Decision drill — Fine-tuning vs RAG vs prompt engineering vs context engineering
sidebar_label: "1.10 FT vs RAG — drill"
sidebar_position: 10
---

# Decision drill — Fine-tuning vs RAG vs prompt engineering vs context engineering

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 1.10</span>
  <span>~15 min (incluso il tempo di lavoro)</span>
</div>

<p class="lesson-lead">La domanda architetturale più frequente in assoluto. Quattro strumenti che sembrano sovrapposti e non lo sono: servono bisogni distinti, si combinano, e scegliere quello sbagliato costa tempo e denaro. La griglia non si recita a memoria: si capisce e si applica.</p>

Questo drill presuppone che tu abbia capito la meccanica di ciascuno strumento:

- **Prompt engineering** — lezione 0.5
- **Fine-tuning** — lezione 0.3
- **RAG** — lezione 1.1
- **Context engineering** — lezione 1.3

Se uno di questi è ancora nebuloso, torna alla lezione prima di usare la griglia: capire la griglia senza capire i mattoni è recitarla, non usarla.

---

## La griglia sinottica

| | Prompt engineering | RAG | Context engineering | Fine-tuning |
|---|---|---|---|---|
| **Cosa modifica** | Il segnale nel prompt | Il materiale nel contesto | Come è strutturato il contesto | I pesi del modello |
| **Problema che risolve** | Guidare il comportamento su un task | Portare fatti specifici aggiornabili | Ottimizzare costo e qualità del contesto | Cambiare stile, tono, formato in modo stabile |
| **Richiede dati?** | No (o pochi esempi) | Sì (i tuoi documenti) | No | Sì (esempi etichettati input-output) |
| **Si aggiorna facilmente?** | Sì (cambi il prompt) | Sì (aggiorni il DB) | Sì (cambi la struttura) | No (riaddestramento) |
| **Costo principale** | Nessuno (già ce l'hai) | Infrastruttura vector DB | Progettazione e testing | Compute + dati + iterazione |
| **Latenza aggiunta** | Nessuna | Retrieval (~200ms) | Dipende dall'ottimizzazione | Nessuna (il modello è lo stesso) |

## La regola d'oro

**Inizia sempre dal prompt engineering.** È gratis, è immediato, copre il 70-80% dei casi. Se non basta, aggiungi RAG. Se RAG non basta, considera context engineering per ottimizzare. Il fine-tuning è l'ultima mossa, non la prima.

Non perché il fine-tuning sia inferiore: è perché ha costi fissi alti (dati, compute, iterazione) e tempo di ciclo lento. Se il prompt già funziona, non c'è nessun motivo di pagare quei costi.

## Quando il prompt engineering basta (e quando no)

**Basta quando:**
- Il task è ben definibile in linguaggio naturale.
- Non hai bisogno di informazioni esterne al modello.
- Il formato di output non è rigidamente vincolato.
- La qualità è accettabile dopo qualche iterazione del prompt.

**Non basta quando:**
- Le informazioni necessarie non sono nel modello (post-cutoff, dominio specifico, dati aziendali).
- Il tasso di errore sul formato è inaccettabile anche con few-shot.
- Il comportamento che vuoi è così specifico che nessuna combinazione di istruzioni lo cattura in modo stabile.

## Quando aggiungere RAG

RAG è la mossa giusta quando **la risposta dipende da informazioni che il modello non può conoscere**: documenti aziendali, dati aggiornati frequentemente, basi di conoscenza specifiche al dominio.

RAG *non* risolve:
- Il tono sbagliato (quello è prompt o fine-tuning).
- Il formato instabile (quello è structured output + prompt).
- La qualità generale del ragionamento (quello è scelta del modello).

La domanda decisiva: "Il modello sa già la risposta, o gli mancano i fatti?" Se gli mancano i fatti → RAG. Se sa i fatti ma risponde male → prompt o fine-tuning.

## Quando il context engineering fa la differenza

Context engineering entra in gioco quando **il problema non è trovare le informazioni, ma organizzarle bene nel prompt**. Sintomi che indicano un problema di context engineering:

- Le risposte peggiorano quando il contesto è lungo.
- I chunk giusti vengono recuperati ma la risposta non li usa.
- Il costo per chiamata è alto e il ROI è basso.
- Le istruzioni vengono "dimenticate" su input lunghi.

Context engineering si applica sempre in combinazione con RAG o con prompt complessi — non è uno strumento alternativo, è un layer di ottimizzazione sopra.

## Quando il fine-tuning vale il costo

Il fine-tuning è la scelta giusta in queste situazioni:

**Stile e formato molto specifici e stabili.** Il modello deve rispondere sempre in un certo modo — formulazioni precise, terminologia fissa, struttura di output rigida — e le istruzioni nel prompt non sono abbastanza stabili. Il fine-tuning lo "memorizza" nei pesi.

**Volume altissimo con costo dei token come vincolo.** Un modello fine-tunato su un task specifico può essere più piccolo del modello generalista e dare la stessa qualità su quel task. A grande volume, la differenza di costo è significativa.

**Adattamento linguistico profondo.** Un dialetto, una lingua a bassa risorsa, o un dominio tecnico con terminologia molto specifica che i modelli generalisti non trattano bene.

**Fine-tuning non è la risposta giusta quando:**
- Le informazioni da insegnare cambiano frequentemente (usa RAG).
- Il problema è solo la qualità del retrieval (aggiusta il retrieval).
- Non hai abbastanza dati di qualità (il fine-tuning su dati scarsi peggiora il modello).
- Non hai budget per l'iterazione (un solo tentativo di fine-tuning raramente funziona subito).

---

## Scenari di applicazione

**Come usarli:** leggi lo scenario, decidi, poi apri la risposta.

---

**Scenario 1.** Un'azienda vuole che il chatbot risponda sempre in italiano formale, senza emoji, con frasi brevi. Qual è la prima mossa?

<details>
<summary>Risposta</summary>

**Prompt engineering.** È esattamente il tipo di vincolo che si specifica nel system prompt — "rispondi sempre in italiano formale, frasi brevi, niente emoji". Se non basta, si aggiungono few-shot examples che mostrano il formato atteso. Il fine-tuning è una mossa sproporzionata per un vincolo di stile che il prompt gestisce bene.
</details>

---

**Scenario 2.** Lo stesso chatbot deve rispondere su 5.000 prodotti del catalogo, con specifiche tecniche aggiornate ogni settimana.

<details>
<summary>Risposta</summary>

**RAG.** Le specifiche dei prodotti sono dati aggiornabili, specifici al dominio, che il modello non conosce. La soluzione è indicizzarli nel vector DB e recuperarli al momento della query. Il fine-tuning sarebbe da rifare ogni settimana — un costo insostenibile. Prompt engineering da solo non basta: il modello non ha i dati.
</details>

---

**Scenario 3.** Un sistema legale deve produrre contratti con una struttura precisa — 12 sezioni in ordine fisso, ciascuna con clausole standard — con poca variazione di stile. Volume: 1.000 contratti al giorno.

<details>
<summary>Risposta</summary>

**Fine-tuning + structured output.** Il formato è rigido, stabile, ad alto volume. Le istruzioni nel prompt non danno la stabilità necessaria sulla struttura a 12 sezioni. Il fine-tuning su esempi di contratti ben formati può produrre un modello più piccolo e più preciso su questo task, riducendo il costo a grande volume. Structured output garantisce la struttura dei campi. RAG potrebbe essere aggiunto per le clausole specifiche al cliente, in combinazione.
</details>

---

**Scenario 4.** Le risposte del RAG sono buone quando il contesto è corto, ma peggiorano su query che richiedono molti documenti. Costo per chiamata alto.

<details>
<summary>Risposta</summary>

**Context engineering.** Il retrieval funziona (trova i documenti giusti), ma l'organizzazione del contesto è il problema. Interventi: ridurre il numero di chunk passati (3-5 ben selezionati invece di 15), mettere i chunk più rilevanti all'inizio e alla fine del contesto, comprimere il system prompt. Misura il costo prima e dopo ogni modifica.
</details>

---

## La combinazione è normale

Nella pratica, i quattro strumenti si combinano. Un sistema tipico in produzione usa:
- Prompt engineering per il comportamento di base e il formato.
- RAG per la conoscenza specifica al dominio.
- Context engineering per tenere il costo sotto controllo.
- Fine-tuning (se il volume lo giustifica) per stile e riduzione del modello.

Non è "quale scelgo": è "quale problema sto risolvendo adesso".
