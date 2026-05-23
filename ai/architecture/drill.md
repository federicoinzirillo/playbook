---
title: Decision drill — Architettura
sidebar_position: 3
---

# Decision drill — Architettura

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 7.3</span>
  <span>~30 min (incluso il tempo di lavoro)</span>
</div>

<p class="lesson-lead">Due scenari di scelta architetturale: una PMI che vuole un assistente sui propri dati, una multinazionale che valuta proprietario vs open-weight. Allenamento da architetto — non c'è una risposta giusta, c'è il saper nominare i trade-off come li nominerebbe un senior.</p>

---

## Scenario A — La PMI che vuole un assistente sui dati

**Contesto:** uno studio di consulenza fiscale con 40 dipendenti vuole un assistente AI che risponda a domande sui propri documenti interni (circolari fiscali archiviate, contratti di consulenza, manualistica interna, deliberazioni). Volume: ~50 utenti attivi, ~200 query al giorno. Budget: vogliono spendere "ragionevolmente" — diciamo \$1.500/mese inclusi tutti i costi infrastrutturali e API. Privacy: i dati sono sensibili ma non soggetti a vincoli specifici di data residency (no sanità, no PA). Team: due sviluppatori senior generalisti, nessuna esperienza precedente di AI. Tempi: vorrebbero qualcosa di usabile entro 8 settimane.

**Cosa ti viene chiesto:**
1. Build vs buy: per ogni componente principale della reference architecture, motiva la scelta.
2. Modello: proprietario o open-weight? Perché?
3. Archetipo architetturale: pipeline, RAG conversazionale o agente?
4. Quali rischi vedi e come li mitighi?
5. Roadmap a 8 settimane: cosa va a regime, cosa rimandi a una seconda fase.

*Scrivi la risposta completa — poi apri la griglia.*

<details>
<summary>Griglia di valutazione</summary>

**Build vs buy:**

- **Modello:** **buy** (API esterna). Volume basso, costi a token irrisori, team senza esperienza GPU/serving. Self-host sarebbe sovra-ingegnerizzazione.
- **Vector store:** **buy managed** (Pinecone, Weaviate Cloud) o **pgvector** se hanno già un Postgres. Sotto i 100.000 chunk, pgvector è la scelta più semplice e più economica.
- **Gateway LLM:** **self-host di LiteLLM** o equivalente. Costa quasi nulla, mette in casa il routing e la cache, dà telemetria. Investimento di una giornata di un senior, evita lock-in.
- **Orchestrazione:** **build** (è il sistema). Framework leggero (LangChain è un'opzione, ma anche FastAPI + chiamate dirette al gateway funziona bene a questa scala).
- **Observability:** **buy managed** (Langfuse Cloud o Helicone). A questo volume, costo è basso e UI immediata vale più del self-host.
- **Pipeline di ingestion:** **build** (è specifica al dominio — circolari fiscali hanno struttura particolare). Usano parser open-source (pdfplumber per i PDF) e scrivono la logica di chunking sopra.

**Modello:** **proprietario** (Claude Sonnet 4.6, GPT-5.4 o Gemini 3.1 Pro tramite API). Non ci sono vincoli di data residency che richiedano open-weight, il volume non giustifica self-host, il team non ha le competenze. Più avanti possono valutare di aggiungere un modello più piccolo (anche open-weight via Bedrock/Azure) per le query semplici, ma è una micro-ottimizzazione.

**Archetipo:** **RAG conversazionale**. Il task è Q&A su corpus interno, non azioni — non serve un agente. Pipeline pura sarebbe troppo rigida (gli utenti faranno follow-up). RAG conversazionale con storia limitata è la scelta giusta.

**Rischi:**
- **Qualità delle risposte su documenti fiscali tecnici.** Mitigazione: golden dataset di 30-50 domande tipiche scritte con un fiscalista, eval suite (lezione 3.2) prima di lanciare.
- **Allucinazione su normativa.** Mitigazione: faithfulness check (lezione 3.4), citazione obbligatoria delle fonti, prompt di sistema che istruisce "se non sei sicuro, dillo".
- **Documentazione che cambia.** Le circolari fiscali si aggiornano. Mitigazione: pipeline di re-indicizzazione, almeno settimanale.
- **Adozione bassa.** Spesso il rischio più grosso. Mitigazione: workshop di introduzione, raccolta feedback dalla prima settimana.

**Roadmap a 8 settimane:**

Settimane 1-2: setup base — gateway, vector store, pipeline di ingestion sui documenti più importanti (non tutti), prompt iniziale, UI minima.
Settimane 3-4: RAG funzionante, golden dataset costruito col fiscalista, prima eval suite.
Settimane 5-6: iterazione su prompt e retrieval sulla base dei risultati dell'eval, ampliamento del corpus.
Settimana 7: pilota con 5 utenti interni, raccolta feedback strutturato.
Settimana 8: rollout su tutti i dipendenti, monitoring impostato.

In seconda fase (mesi 3-6): cache semantica, eventuale routing su modello più piccolo, dashboard per il PM, espansione del corpus a documenti meno strutturati.

**Trappole:**
- Sovra-ingegnerizzare al primo giro (microservizi, Kubernetes, vector store self-host): a 200 query/giorno non serve niente di tutto questo.
- Non scrivere l'eval suite dall'inizio: senza, ogni modifica al prompt o al retrieval è "sembra meglio?", impossibile decidere.
- Sottovalutare la pipeline di ingestion: i PDF fiscali italiani hanno tabelle, riferimenti incrociati, struttura — parser naive porta a retrieval scadente.

</details>

---

## Scenario B — La multinazionale che valuta proprietario vs open-weight

**Contesto:** una multinazionale farmaceutica vuole un sistema AI per supportare gli scientific affairs nella ricerca su letteratura scientifica e documentazione regolatoria interna. Volume previsto: 5.000 utenti, 100.000 query al giorno a regime. Vincoli: i documenti interni sono altamente riservati (proprietà intellettuale su farmaci in sviluppo); soggetti a GDPR e a controlli regolatori (FDA, EMA). Budget AI dedicato: ~\$200.000/mese a regime. Team: due squadre dedicate, una di MLOps con esperienza GPU, una applicativa con esperienza AI. Tempi: 12-18 mesi per arrivare a regime, MVP a 6 mesi.

**Cosa ti viene chiesto:**
1. Modello: proprietario, open-weight, o ibrido? Motiva con i criteri.
2. Topology di deployment: cloud pubblico, cloud privato, ibrido on-prem?
3. Quali componenti costruiscono in casa che la PMI non costruirebbe?
4. Come gestiscono il vendor lock-in?
5. Cosa cambierebbe la decisione se uno dei vincoli (privacy, budget, team) fosse diverso?

<details>
<summary>Griglia di valutazione</summary>

**Modello: ibrido.**

- **Open-weight self-hosted** (Llama 4 Scout o Maverick, Mistral Large, Qwen 3 o DeepSeek v3, a seconda del benchmark sul dominio scientifico) per il grosso del traffico. Motivi: (1) i dati di IP farmaceutica non possono uscire dall'infrastruttura controllata; (2) a 100.000 query/giorno, il costo di self-host diventa drasticamente più conveniente del pay-per-token; (3) hanno il team per gestirlo.
- **Modello proprietario tramite Azure OpenAI / AWS Bedrock con data isolation** per le query che richiedono frontier capabilities (ragionamento complesso su letteratura scientifica, multimodale se serve analizzare immagini di studi clinici). I provider cloud offrono contratti enterprise con data residency garantita, accettabili per la maggior parte dei dati.
- **I documenti altamente riservati** (formule di farmaci in sviluppo, IP non pubblica) restano confinati al modello open-weight self-hosted.

Il routing tra i due passa dal gateway, basato su sensibilità del documento + complessità della query.

**Topology:** **cloud privato** (Azure dedicated o AWS GovCloud-style) per il serving degli open-weight + gli indici vettoriali; **on-prem** per i dati più riservati (proprietà intellettuale attiva). Cloud pubblico solo per servizi accessori che non vedono dati sensibili.

**Cosa costruiscono in casa che la PMI non costruirebbe:**

- **Serving di open-weight** (vLLM/TensorRT-LLM su GPU dedicate). La PMI non lo farebbe mai.
- **Eval suite specifica per il dominio scientifico/regolatorio**, con annotazione da esperti interni. Per la farma è critico — gli errori in questo dominio possono significare ritardi regolatori o IP esposta.
- **Pipeline di ingestion sofisticata** per articoli scientifici (parsing di formule, riferimenti, figure). Servizi commerciali esistono ma non coprono tutto bene.
- **Layer di audit logging** rinforzato per soddisfare i requisiti regolatori (chi ha chiesto cosa, quale risposta è stata data, su quali documenti).
- **Pipeline di re-training/fine-tuning** del modello open-weight su corpus interno (fattibile a questo livello di investimento; impensabile per la PMI).

**Vendor lock-in:**

- **Gateway LLM** (probabilmente custom o LiteLLM enterprise) come confine astratto.
- **Almeno due provider proprietari abilitati** in parallelo per le query "frontier" — Azure OpenAI come primario e Bedrock con Claude come secondario, ad esempio. Tradeoff: due contratti, due telemetrie, due profili di costo da gestire.
- **Open-weight self-hosted** dà l'opzione di rimanere "fermi" su una versione del modello che funziona, indipendentemente da deprecation imposte.

**Cosa cambierebbe la decisione se i vincoli fossero diversi:**

- **Senza il vincolo IP:** si potrebbe usare maggiormente proprietario via API, riducendo il self-host. Probabilmente comunque ibrido per ragioni di costo a volume, ma il bilancio si sposterebbe.
- **Budget ridotto a \$20.000/mese:** self-host di modelli grandi non più giustificato, si tornerebbe verso proprietari + modelli piccoli open-weight su volumi limitati.
- **Team senza competenze GPU:** Bedrock/Azure managed open-weight diventerebbe la scelta per gli open-weight, evitando il self-host puro. Compromesso accettabile.
- **Volume 10x più basso:** quasi tutto proprietario, l'open-weight self-host non si paga.

**Trappole:**
- Costruire tutto in casa per "sovranità" senza valutare il costo di manutenzione decennale.
- Sottovalutare la complessità della governance regolatoria: il sistema AI deve essere documentato come parte dei controlli FDA/EMA — l'audit logging è non-negoziabile.
- Pensare che self-host = niente lock-in: ti leghi alla famiglia di modelli open-weight scelta (Llama vs Mistral vs altro), alle GPU che hai comprato, agli strumenti specifici del tuo stack.
- Ignorare il EU AI Act (lezione 4.4): un sistema che supporta decisioni regolatorie può ricadere in classificazioni più stringenti.

</details>

---

## Cosa fare se hai mancato molti punti

- Componenti, strati e confini → **7.1 Reference architecture**
- Criteri build vs buy, proprietario vs open-weight, ADR → **7.2 Build vs buy**
- Pattern di sistema (sincrono/asincrono/batch) → **5.2 Pattern di sistema**
- Triangolo costo-latenza-qualità → **5.3 Il triangolo**
- Pipeline dati, vector store → **5.4 I dati come spina dorsale**
- Privacy, EU AI Act → **4.3 Privacy** e **4.4 EU AI Act**
- FinOps a runtime → **6.2 Costi e FinOps**
- Eval suite → **3.1 LLM-as-judge** e **3.5 Decision drill — Valutazione**

---

## Prossima lezione

**7.4 Capstone — progetto end-to-end.** L'ultimo esercizio: un caso completo che attraversa tutte le aree. Non c'è una griglia che ti dice "hai vinto", c'è un sistema che devi saper progettare e difendere davanti a uno stakeholder. È il pezzo che porti a un colloquio.
