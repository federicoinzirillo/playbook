---
title: Decision drill — System design
sidebar_label: "5.5 Decision drill"
sidebar_position: 5
---

# Decision drill — System design

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 5.5</span>
  <span>~25 min (incluso il tempo di lavoro)</span>
</div>

<p class="lesson-lead">Due scenari da progettare dall'inizio. Per ognuno: architettura dei componenti, pattern di sistema, scelta del modello, pipeline di dati, e dove si trova il punto di lavoro sul triangolo. Scrivi prima, poi apri la griglia.</p>

---

## Scenario A — Chatbot di supporto enterprise

**Contesto:** un'azienda SaaS con 5.000 clienti business vuole un assistente AI per il supporto tecnico. La knowledge base: 3.000 articoli di documentazione (aggiornati frequentemente), 200.000 ticket storici risolti (aggiornati ogni giorno), changelog di prodotto (settimanale). Gli utenti sono sviluppatori che integrano l'API. Richiedono risposte precise e citano la fonte. Il volume stimato: 2.000 richieste al giorno, con picchi a 300 richieste all'ora. Budget: \$5.000 al mese per i costi AI. Latenza accettabile: 8 secondi.

**Cosa ti viene chiesto:**
1. Disegna l'architettura dei componenti (non serve essere precisi sul codice; bastano i componenti e come si collegano).
2. Quale pattern di sistema? Sincrono, asincrono, streaming?
3. Quale modello e quale strategia di retrieval? Come gestisci le tre sorgenti diverse?
4. Dove si trova il tuo sistema sul triangolo qualità-latenza-costo? Quali leve hai per stare nel budget?
5. Come aggiorni l'indice quando la documentazione cambia?

*Scrivi la risposta completa — poi apri la griglia.*

<details>
<summary>Griglia di valutazione</summary>

**Architettura dei componenti:**

- **Gateway** — autenticazione (API key per cliente), rate limiting (non permettere a un singolo cliente di saturare il budget), routing.
- **Orchestratore** — pipeline: query → retrieval → re-ranking → contesto → modello → risposta con citation.
- **Retrieval ibrido** — ricerca semantica (vector store) + ricerca lessicale (BM25 o simile). L'ibrido funziona meglio della sola semantica per query tecniche con termini specifici ("errore 429 su endpoint /v2/send").
- **Re-ranking** — dopo il retrieval, un modello di re-ranking (cross-encoder) ordina i risultati per rilevanza reale. Migliora significativamente la qualità su knowledge base grandi.
- **Tre indici separati** per le tre sorgenti — con metadati distinti: `source_type` (docs, ticket, changelog), `updated_at`, `product_version`. Questo permette di filtrare ("cerca solo nei docs aggiornati") e di gestire freshness differenziata.
- **Cache semantica** — con 2.000 richieste/giorno e documentazione tecnica, molte query sono simili. La cache semantica con soglia alta (0.95+) riduce il costo del 20-40%.
- **Observability** — log strutturati per ogni richiesta: query, chunk recuperati, modello usato, latenza, costo stimato, feedback utente (thumbs up/down).

**Pattern:** sincrono con streaming. Latenza accettabile di 8 secondi è gestibile con streaming — l'utente vede la risposta che cresce, non aspetta in silenzio. Nessun motivo per asincrono su una query interattiva.

**Modello:** un modello di fascia veloce (Claude Haiku 4.5, GPT-5.3 Instant, Gemini 3 Flash) per la maggior parte delle query; modello grande (Claude Opus 4.7, GPT-5.4, Gemini 3.1 Pro) solo per query complesse rilevate dall'orchestratore (query lunghe, multiple entità, "spiegami come funziona X"). Il routing intelligente riduce significativamente il costo medio.

**Aggiornamento dell'indice:**
- Documentazione: webhook o polling ogni ora; aggiornamento incrementale basato su hash del contenuto.
- Ticket storici: pipeline batch notturna; non serve intra-day freshness.
- Changelog: aggiornamento manuale o trigger sul commit/release.

**Budget:** \$5.000/mese su 2.000 richieste × 30 giorni = 60.000 richieste/mese. Budget per richiesta: ~\$0.083. Con modello medio, contesto di 3.000 token input + 500 token output, il costo è nell'ordine di \$0.002-0.010 per richiesta — abbondantemente dentro budget. La cache riduce ulteriormente.

**Trappole:**
- Non separare le sorgenti porta a contesto mescolato e rende impossibile l'attributione della fonte.
- Non fare re-ranking su knowledge base grandi produce retrieval impreciso — i chunk più simili semanticamente non sono sempre i più rilevanti.
- Non mettere rate limiting per cliente permette a un singolo cliente di esaurire il budget.
</details>

---

## Scenario B — Analisi di contratti legali

**Contesto:** uno studio legale vuole un sistema per analizzare contratti in PDF (NDA, contratti di fornitura, accordi di licensing). Per ogni contratto, il sistema deve estrarre le clausole chiave, identificare clausole potenzialmente problematiche, e rispondere a domande specifiche dell'avvocato ("questa clausola limita il diritto di rescissione?"). I contratti sono da 5 a 200 pagine. I dati sono riservati — non possono uscire dall'infrastruttura controllata. Latenza: non è real-time, 5 minuti sono accettabili. Volume: 20-50 contratti al giorno.

**Cosa ti viene chiesto:**
1. Architettura e pattern: sincrono, asincrono, batch?
2. Come gestisci il vincolo di data residency (dati che non escono)?
3. Come gestisci contratti da 200 pagine che superano la context window?
4. Quale pipeline di dati per i PDF? Quali problemi ti aspetti?
5. Come progetta il sistema per rispondere sia all'estrazione strutturata ("dammi tutte le clausole di penale") che alle domande libere dell'avvocato?

<details>
<summary>Griglia di valutazione</summary>

**Architettura e pattern:**

Pattern **asincrono con job queue** — non c'è nessun motivo per fare sincrono su un task che può durare minuti. L'avvocato carica il contratto, riceve un job ID, può fare altre cose e torna a vedere il risultato.

Flusso:
1. Upload PDF → API → job enqueued → job ID restituito
2. Worker processa in background: parsing → chunking → analisi
3. Risultati salvati nel DB interno
4. Notifica (email o webhook) quando pronto
5. L'avvocato apre l'interfaccia e vede il report; può fare domande free-form sopra

**Data residency:**

I contratti legali non possono uscire dall'infrastruttura controllata. Soluzione: **modello on-premise o cloud privato**.

- **Opzione 1 — Open-weight model on-premise:** Llama 4, Mistral o Qwen 3 su hardware GPU proprio (o in cloud con dedicated compute). Il dato non lascia l'infrastruttura. Richiede team con competenze di deployment.
- **Opzione 2 — Azure OpenAI / AWS Bedrock in modalità data isolation:** i modelli proprietari in esecuzione su infrastruttura cloud dedicata al cliente. Il provider del cloud ha accesso all'infrastruttura, ma i dati non sono usati per training e la data residency è garantita contrattualmente. Più semplice dell'on-premise puro.

Non usare l'API pubblica di OpenAI/Anthropic per dati che non devono uscire — le garanzie di data isolation sono diverse.

**Contratti da 200 pagine:**

200 pagine ≈ 100.000-150.000 token. Non entrano nella context window di molti modelli, e anche se entrassero, il costo sarebbe enorme.

Strategia a due livelli:
1. **Estrazione strutturata con pipeline** — non si manda il contratto intero al modello. Si parsano le sezioni (clausole identificate per titolo, header, struttura), si processano chunk per chunk con prompt specifici per ogni tipo di clausola (penali, rescissione, limitazioni di responsabilità). Output strutturato per ogni sezione.
2. **RAG per le domande free-form** — il contratto viene indicizzato nel vector store (locale, non in cloud esterno). Le domande dell'avvocato usano il retrieval per trovare le sezioni rilevanti, poi il modello risponde con citazione del paragrafo specifico.

**Pipeline PDF:**

I PDF legali sono notoriamente difficili: layout multi-colonna, tabelle, riferimenti incrociati, scan di documenti fisici firmati, PDF protetti. Problemi attesi:
- PDF di scan → serve OCR (Azure Document Intelligence, Google Document AI, o open-source come Tesseract ma qualità inferiore).
- Layout multi-colonna → parser naive mescola il testo. Serve un parser layout-aware.
- Tabelle → la struttura tabulare va preservata, non appiattita in testo lineare.
- Numerazione delle clausole → va estratta come metadato per permettere referenze precise ("clausola 7.3").

Strumenti da valutare: Azure Document Intelligence (se si usa Azure), `pdfplumber` per PDF nativi, `camelot` per tabelle, servizi specializzati per legal document parsing.

**Estrazione strutturata vs domande free-form:**

Sono due task diversi che richiedono approcci diversi, ma condividono la stessa base dati.

- **Estrazione strutturata** → output JSON con schema predefinito. Prompt specifici per categoria di clausola, validati contro lo schema. Esempio: `{tipo: "penale", testo: "...", parti: [...], importo: "...", condizioni: [...]}`.
- **Domande free-form** → RAG sul vector store del contratto. L'avvocato chiede in linguaggio naturale, il sistema recupera le sezioni rilevanti e risponde con citazione.

Il risultato dell'estrazione strutturata può essere usato sia per il report iniziale che come metadato aggiuntivo nel vector store (migliora il retrieval sulle domande successive).

**Trappole:**
- Fare tutto con un singolo prompt "analizza questo contratto" → il modello produce output inconsistente e non strutturato.
- Non gestire i PDF di scan → il 20-30% dei contratti legali sono scan, non PDF nativi.
- Non citare le fonti → l'avvocato deve poter verificare il paragrafo esatto, non fidarsi della risposta.
- Mettere dati riservati su API cloud pubbliche → violazione dei NDA firmati dallo studio con i clienti.
</details>

---

## Cosa fare se hai mancato molti punti

- Architettura dei componenti → **5.1 Anatomia di un sistema AI**
- Pattern sincrono/asincrono/batch → **5.2 Pattern di sistema**
- Triangolo qualità-latenza-costo e caching → **5.3 Il triangolo**
- Pipeline di dati, chunking, vector store → **5.4 I dati come spina dorsale**
- Data residency e modelli on-premise → **4.3 Privacy e data residency**
- Structured output per estrazione → **1.3 Structured output e function calling**
