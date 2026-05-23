---
title: Syllabus
sidebar_position: 2
---

# Syllabus — AI Moderna (generalista applicativa)

> Programma di studio per essere competenti sull'AI moderna in **qualsiasi ruolo** applicativo
> (AI Engineer, Solution Architect, e ruoli software che ormai toccano l'AI). Copre l'AI a base
> LLM, il multimodale, il system design dei sistemi AI e i concetti trasversali.
>
> **Fuori scope:** addestrare modelli da zero, matematica pesante del ML, infrastruttura cloud
> pura (Kubernetes, networking, IaC). Si studia il *minimo concettuale* di training per saper
> *decidere*, non per farlo.
>
> Ordine sequenziale: rispetta i prerequisiti (segnati con ⊳). Tempistiche indicative su
> ~15-20 h/settimana, da adattare.

---

## Traguardi e rientro

> Pensato per chi studia a sprazzi: il problema non è capire, è il **costo di rientro** dopo un vuoto.

**Due traguardi "uscita con dignità".** Se a un certo punto molli, questi sono i punti in cui hai
comunque qualcosa di reale in mano — non un percorso a metà, ma una competenza spendibile:
- **Fine Parte 1 = so costruire un RAG online.** Già spendibile, da solo.
- **Fine Parte 3 = so costruirlo E dimostrare che funziona.** Il salto da "funziona sul mio
  schermo" a "ho i numeri che lo provano".

Tutto il resto è di più — ma se ti fermi a uno di questi due, non hai sprecato niente.

**Log di rientro.** A fine di **ogni** sessione, scrivi due righe in un file dedicato: *dove sono
ora / la prossima cosa è X*. Al rientro apri quel file e nei primi 5 minuti sai cosa fare, senza
rileggere appunti né ricostruire il contesto. È la singola abitudine che abbassa di più il costo
di rientro.

**Rito di rientro dopo un vuoto lungo.** La prima sessione dopo una pausa di 1-2 settimane **non
è teoria**: è aggiungere una cosa piccola al Progetto 0. Riattivi il contesto con le *mani* prima
che con la testa — tocchi codice vivo, e il filo torna da solo. La teoria viene dalla sessione dopo.

---

## Introduzione

### Esiti di apprendimento
Al termine sarai in grado di: spiegare come funzionano LLM e modelli multimodali; progettare
sistemi AI end-to-end ragionando sui trade-off; valutarne affidabilità, costi e sicurezza —
incluso il comportamento degli agenti, non solo i singoli output; e sapere quando l'AI (o un
certo tipo di AI) è la scelta giusta o sbagliata.

### Prerequisiti
Programmazione intermedia (preferibilmente Python), familiarità con API/database/servizi web.
Nessun prerequisito di ML o statistica.

### Metodo (vincolante)
Apprendimento attivo: la verifica a memoria va fatta **dentro la sessione di studio**, a chiusura
di una lezione — non come cancello d'ingresso al rientro. Le risposte incerte si rivedono il
giorno dopo (spacing); si alternano gli argomenti (interleaving). I progetti pratici non sono
opzionali: è lì che si impara ciò che nessuna pagina insegna.

> **Al rientro dopo una pausa, l'ordine è: prima rimetti mano al progetto, poi semmai ripassi.**
> Non auto-bloccarti con una verifica a memoria appena rientri: falliresti per il tempo passato,
> non per non aver capito, e ti fermeresti sulla soglia. La verifica è uno strumento *dentro* lo
> studio, mai un esame d'ammissione per ricominciare.

### Tre assi da non confondere (ricorrono in tutta la guida)
La stessa parola — "costi", "monitoring", "serving" — torna in parti diverse perché vive su
**tre assi distinti**. Tenerli separati evita di scambiare un approfondimento per un ripasso:
- **Design** (Parte 5): *come decidi* l'architettura, sulla lavagna, prima di costruire.
- **Evaluation** (Parte 3): *come misuri* se il sistema è buono — qualità di output e comportamento.
- **Operations / LLMOps** (Parte 6): *come lo fai vivere* in produzione — tool concreti, giorno-2, incidenti.
Quando un tema riappare, riappare su un asse diverso, con domande diverse.

### Tenere viva la guida (abitudine parallela — non opzionale se studi lento)
Questo è un campo che si muove in fretta: parte di ciò che studi invecchia *mentre* lo studi —
soprattutto i tool concreti (Parte 6: vLLM, TGI, gateway) e le aree marcate `In evoluzione`
(3.5, 4.2). Un programma statico su un campo così non basta: serve una **valvola di sfogo**.
Il *Radar* vive in **[RADAR.md](./RADAR.md)** — un file dove annotare cosa sta diventando
standard (changelog dei provider, release dei tool che usi, qualche paper) con stati
`EMERGENTE → IN ADOZIONE → STANDARD → LEGACY`. Da rivedere ~una volta al mese. Quando
qualcosa è passato da curiosità a standard di fatto, aggiorna la pagina relativa e sposta
la voce nella tabella STANDARD del Radar. **Il rischio numero uno per chi va lento non è
dimenticare i concetti — è studiare tool già superati senza accorgersene.** I concetti
fondazionali (Parti 0-1-3-5) reggono per anni; i tool no.

---

## PARTE 0 — Fondamenta concettuali
*Necessarie a tutto il resto. Da fare per prime.*

- **0.1 Come funziona un LLM** — token, context window, sampling, temperature. Include perché i token sono l'unità di costo (paghi a token, non a parola): un chiodo da piantare presto. Include anche il **prompt caching**: i token ripetuti (istruzioni di sistema, few-shot) possono costare 10× meno con il caching del provider — primo gesto su qualsiasi sistema serio (dettaglio tecnico in 5.6).
- **0.2 Embedding e spazi vettoriali** ⊳ 0.1 — significato come numeri, similarità del coseno.
- **0.3 Concetti ML che servono a chiunque** — training, fine-tuning, overfitting, dati
  train/test, bias nei dati. Non solo vocabolario: serve l'**intuizione meccanica** di cosa il
  fine-tuning *cambia* nel modello (sposta i pesi → cambia comportamento e stile; NON aggiunge
  fatti recuperabili, quello lo fa RAG). Confine netto: abbastanza meccanica da *sentire* la
  differenza fine-tuning vs RAG (serve a 1.7), non abbastanza da addestrare qualcosa. Senza questo
  pezzo, la griglia di decisione 1.7 si recita a memoria invece di capirla.
- **0.4 LLM vs ML classico vs altri approcci: quando NON usare la GenAI** ⊳ 0.1, 0.3 — criteri
  di scelta; quando un modello tradizionale, una regola, o nessun modello è la risposta giusta.
- **0.5 Prompt engineering come disciplina** ⊳ 0.1 — zero-shot vs few-shot, chain-of-thought,
  structured prompting. La parte seria, non da blog: **prompt versioning e A/B testing dei prompt
  in produzione** (si rivede in Parte 3, valutazione). È il primo strumento che si tocca: va qui,
  presto, perché è prerequisito implicito di mezza Parte 1.
- **0.6 Reasoning models — quando il pensiero costa, e quando vale** ⊳ 0.1, 0.5 — modelli che
  *pensano prima di parlare* (o1/o3, Claude extended thinking, Gemini reasoning budget). La
  distinzione tra chain-of-thought via prompt e ragionamento *addestrato*. Latenza ×5-20 e costo
  ×3-10: quando il guadagno di qualità vale il prezzo, quando no. Pattern di cascata/router.

## PARTE 1 — Costruire sistemi LLM
*Il cuore. Subito dopo i fondamenti.*

- **1.1 RAG — il nucleo** ⊳ 0.2 — chunking, vector DB, retrieval base (ricerca semantica a coseno),
  pipeline dalla query al documento recuperato. Il ciclo completo nella versione essenziale.
- **1.2 Retrieval avanzato** ⊳ 1.1 — reranking, hybrid search (BM25 + vettoriale), agentic
  retrieval, GraphRAG per domini a grafo. Le ottimizzazioni si studiano dopo aver capito il nucleo.
- **1.3 Context engineering: cosa mettere nel contesto, a scala** ⊳ 1.1 — competenza distinta dal
  RAG classico. Con context window enormi, *decidere cosa entra nel contesto* è un mestiere a sé:
  il trade-off tra riempire il contesto e il degrado di attenzione nel mezzo ("lost in the
  middle"), più il costo che cresce col contesto. Aggancio al triangolo qualità-latenza-costo (5.3).
- **1.4 Structured output e function calling** ⊳ 1.1 — far produrre dati strutturati, collegare funzioni.
- **1.5 Agenti semplici — tool calling e ReAct** ⊳ 1.4 — tool calling, il loop ReAct (Reason + Act),
  l'agente come processo iterativo. *Principio chiave: il controllo di flusso sta nell'orchestratore,
  il giudizio nell'LLM. Mai delegare il flusso ai prompt (→ loop e costi fuori controllo).*
- **1.6 Orchestrazione multi-agent** ⊳ 1.5 — pattern supervisore-worker, agenti paralleli, stato
  condiviso, framework (LangGraph, CrewAI). *Costo e latenza: ogni passo è un'altra inferenza —
  un sistema multi-agent costa molto più di una singola chiamata. Vedi 5.3 prima di scegliere
  un'architettura agentica.*
- **1.7 MCP — Model Context Protocol** ⊳ 1.4 — lo standard per collegare modelli, tool e dati.
- **1.8 Decision drill — Costruire**
- **1.9 Fine-tuning operativo — LoRA, QLoRA, DPO** ⊳ 0.3 — come fai fine-tuning senza addestrare
  70B parametri. Tecniche PEFT (LoRA, QLoRA), preferenze (DPO/ORPO), dataset size realistico,
  forgetting catastrofico, managed vs self-hosted.
- **1.10 Decision drill — Fine-tuning vs RAG vs prompt engineering vs context engineering** ⊳ 0.3,
  0.5, 1.1, 1.3, 1.9 — la domanda architetturale più frequente in assoluto, *dopo* aver toccato
  con mano sia RAG (1.1-1.2) che fine-tuning (1.9). Griglia di quando l'uno batte l'altro: costo,
  dati disponibili, frequenza di aggiornamento, serve uno *stile* o una *conoscenza*.

## PARTE 2 — Multimodale e altri tipi di AI
*La fetta "AI generalista" oltre il testo. Conoscenza fondazionale di cosa sa fare l'AI oggi.*

- **2.1 Come funziona il multimodale** ⊳ 0.2 — l'idea che testo, immagini, audio diventano
  embedding in uno spazio condiviso. Architettura **nativa** (un solo modello per tutte le
  modalità) vs pipeline separate: oggi il nativo è lo standard di fatto.
- **2.2 Vision: cosa sa fare l'AI sulle immagini** ⊳ 2.1 — classificazione, object detection, OCR,
  document understanding, video. A livello di capacità e casi d'uso.
- **2.3 Multimodal RAG — retrieval su documenti complessi** ⊳ 1.1, 2.2 — ColPali e vision
  embeddings per PDF, immagini, documenti con layout complesso; quando il RAG testuale non basta
  e come combinare retrieval visivo e testuale.
- **2.4 Audio e speech** ⊳ 2.1 — speech-to-text e text-to-speech; quando usare un modello dedicato
  (solo trascrizione) vs un multimodale (trascrizione + ragionamento).
- **2.5 Generazione di immagini (diffusion models)** ⊳ 2.1 — l'idea di base, senza la matematica;
  casi d'uso e limiti.
- **2.6 Quando multimodale, quando pipeline separate** ⊳ 2.1 — criteri di scelta e trade-off.
- **2.7 Decision drill — Multimodale**
- **2.8 Voice agents in tempo reale** ⊳ 2.4, 1.5 —
  <span class="badge-stato evoluzione">In evoluzione</span> conversazione vocale fluida sotto la
  soglia degli ~800ms percepiti. Perché la pipeline STT→LLM→TTS non basta, modelli speech-to-speech
  nativi (Realtime API, Gemini Live), gestione delle interruzioni (barge-in, turn detection),
  vincoli di rete mobile. Quando NON usarlo.
- **2.9 Video generation: cosa sa fare, costi, quando usarlo** ⊳ 2.1 —
  <span class="badge-stato evoluzione">In evoluzione</span> Sora, Veo, Runway, Kling: capability
  reali al 2026, modello di costo (pagamento al secondo di output, latenza in minuti), casi d'uso
  seri (mockup, b-roll, formazione, advertising) vs theatre da demo. Limiti: coerenza temporale,
  controllo fine, fisica plausibile. Quando NON usarlo.

## PARTE 3 — Valutare e rendere affidabile
*Ciò che separa una demo da un sistema serio. In interleaving con Parte 1.*

- **3.1 Eval benchmarks e dataset — il vocabolario** ⊳ 0.2 — prima il vocabolario, poi gli
  strumenti. I benchmark pubblici che senti nominare (MMLU, GPQA, SWE-bench, AgentBench, MTEB)
  e cosa misurano davvero. Contamination e perché i leaderboard sono segnale rumoroso. Costruire
  un **golden dataset interno** (50-200 esempi curati > 10k sintetici random). Synthetic data:
  usi legittimi e trappole.
- **3.2 LLM-as-judge** ⊳ 3.1, 1.1 — golden dataset, criteri, eval offline/online, bias del
  giudice. Valuta il **singolo output**: la base, ma non basta per gli agenti (vedi 3.5).
- **3.3 Observability** ⊳ 3.2 — tracing, costi, latenza, cosa loggare per *misurare*. Include il
  tracking dei prompt versionati introdotti in 0.5.
- **3.4 Gestire le allucinazioni** ⊳ 1.1, 3.2 — perché succedono, grounding, mitigazione.
- **3.5 Valutare il comportamento agentico end-to-end** ⊳ 1.6, 3.2 —
  <span class="badge-stato evoluzione">In evoluzione</span> il buco che si sta allargando più in
  fretta nel settore. Un agente non è un singolo output, è una **traiettoria** di decisioni e tool.
  Può produrre ogni risposta plausibile e fallire *come processo*: tool sbagliato, loop, obiettivo
  raggiunto per la strada sbagliata, o mancato pur "suonando bene". Si valuta la traiettoria, la
  scelta dei tool, il completamento del task — non solo il token finale. Scritto sui **principi**
  (cosa significa valutare un processo, quali domande porsi), non sui tool del mese, perché è
  un'area giovane che cambia in fretta.
- **3.6 Decision drill — Valutazione** (incluso un caso di valutazione agentica)

## PARTE 4 — Sicurezza, privacy e governance
- **4.1 Prompt injection e OWASP LLM Top 10** ⊳ 1.5 — attacchi, esfiltrazione, abuso di tool sul
  *singolo input*, guardrail. La sicurezza va resa **strutturale** (policy engine tra LLM e azioni),
  non affidata ai prompt.
- **4.2 Sicurezza agentica: la superficie d'attacco comportamentale** ⊳ 4.1, 1.6 —
  <span class="badge-stato evoluzione">In evoluzione</span> un agente con tool non è vulnerabile
  solo sull'input, ma sulla **catena di azioni**: una sequenza di passi ciascuno legittimo che
  insieme producono un danno (esfiltrazione progressiva, azioni distruttive concatenate). Perché
  i guardrail vanno a livello di policy *sull'esecuzione di ogni azione*, non solo sull'input —
  estensione concreta del principio "sicurezza strutturale" della 4.1.
- **4.3 Privacy e data residency** ⊳ 4.1 — dove finiscono i dati quando usi un LLM di terzi,
  **GDPR** (cosa diversa e più ampia dell'AI Act), gestione PII, anonimizzazione. Per progetti
  enterprise è spesso *il primo blocco reale*: la domanda che ferma tutto al primo meeting.
- **4.4 EU AI Act e governance** ⊳ 4.3 — categorie di rischio, obblighi UE, audit, responsible AI.
- **4.5 Decision drill — Sicurezza e privacy**

## PARTE 5 — System design dei sistemi AI
*L'asse DESIGN: come decidi l'architettura. Oggi richiesto anche per ruoli software generali.*

- **5.1 Anatomia di un sistema AI** ⊳ 1.5 — i componenti (orchestratore, retrieval, tool,
  guardrail, observability) e come si parlano. L'LLM è ~20% di un sistema di produzione.
- **5.2 Pattern di sistema** ⊳ 5.1 — sincrono vs asincrono, batch vs real-time, code/queue,
  batching delle richieste, load balancing. Sono i pattern dei sistemi distribuiti applicati all'AI.
  *Qui si sceglie il pattern; il come gestirlo in produzione è Parte 6.*
- **5.3 Il triangolo qualità-latenza-costo** ⊳ 5.1 — il trade-off che governa ogni decisione
  (richiamato da 1.3 e 1.5); caching (incluso semantico e **prompt caching del provider**), rate limiting, fallback, ridondanza.
  *Qui si ragiona sul trade-off in fase di design; l'ottimizzazione dei costi reali a runtime è 6.2.*
- **5.4 I dati come spina dorsale** ⊳ 0.2 — qualità, preparazione, pipeline a livello
  concettuale; database vettoriali a fondo. Spesso il vero collo di bottiglia, sottovalutato.
- **5.5 Decision drill — System design** (es. "progetta un chatbot di supporto su LLM di terzi")
- **5.6 Caching semantico e model routing** ⊳ 5.1, 5.3, 0.2 — le due leve di costo/latenza che
  vivono in architettura, non in produzione. Cache esatta vs semantica (con la trappola della
  soglia di similarità), **prompt caching del provider** (cache_control su Anthropic, automatic
  caching su OpenAI: -80-90% sui token ripetuti in prefix fissi — istruzioni di sistema,
  few-shot stabili), router upfront vs cascade reattiva,
  LLM gateway (LiteLLM, Portkey, Helicone, OpenRouter). Risparmio reale tipico 50-65%.

## PARTE 6 — Produzione (LLMOps)
*L'asse OPERATIONS: far vivere in produzione un sistema che NON addestri. Non ripete la Parte 5
(design) né la 3.3 (misurare): qui si parla di **tool concreti e giorno-2** — cosa fai quando il
sistema è già live e qualcosa va storto.*

- **6.1 Serving e inference** ⊳ 5.1 — gli **strumenti reali**: gateway/router, motori (vLLM, TGI),
  come si espone e si scala un endpoint. (La 5.2 sceglie il pattern; qui lo si mette in opera.)
- **6.2 Costi e FinOps a runtime** ⊳ 6.1, 5.3 — non il *ragionamento* sul triangolo (5.3) ma la
  **gestione reale**: monitoraggio della spesa, alert, cosa fai quando i costi esplodono in
  produzione, ottimizzazione token a posteriori.
- **6.3 Monitoring e drift in produzione** ⊳ 3.2, 6.1 — non il *cosa loggare per valutare* (3.2)
  ma il **giorno-2**: il degrado che emerge dopo settimane, gli alert operativi, l'incident response
  quando la qualità cala in modo silenzioso.
- **6.4 Rollout sicuro: canary, A/B e shadow traffic per LLM** ⊳ 6.1, 6.3, 3.1 — cambiare modello,
  prompt o pipeline in produzione senza far scoppiare tutto. Canary (5%→25%→100%) con rollback
  automatico legato alle metriche di 6.3, A/B per scelte qualitative (quale prompt converte di più),
  shadow traffic per confronto a costo controllato. La "regressione" è qualitativa, non binaria.
- **6.5 Decision drill — Produzione** (es. "il sistema funziona ma i costi sono raddoppiati: cosa
  indaghi e in che ordine")

## PARTE 7 — Architettura e sintesi
- **7.1 Reference architecture** ⊳ Parti 1-6 — come si compone un sistema completo.
- **7.2 Build vs buy, proprietari vs open-weight (e SLM)** ⊳ 7.1 — criteri di decisione, costo,
  controllo. Include la terza opzione **SLM (Small Language Models)**: Phi, Gemma small, Llama
  3.2 1B/3B, Qwen small — quando privacy on-device / latenza ultra-bassa / costo a scala /
  edge li rendono la scelta giusta.
- **7.3 Decision drill — Architettura** ⊳ 7.1, 7.2 — due scenari realistici con vincoli concreti, da scelta dell'archetipo a build vs buy.
- **7.4 Capstone — progetto end-to-end** ⊳ tutto — dalla scelta architetturale a costi,
  sicurezza e monitoring, con un decision record che difende le scelte.

---

## Progetti pratici (non opzionali — qui si impara davvero)
- **Dopo Parte 0 — Progetto 0, il "giocattolo vivo":** brutto ma online. Chiama un LLM, fagli
  fare structured output, sbatti tutto su una pagina web banale. Scopo: avere SUBITO un artefatto
  a cui tornare. Se studi a sprazzi, è l'ancora che ti fa ritrovare il filo nei vuoti tra una
  sessione e l'altra — molto prima del primo progetto serio.
- **Dopo Parte 1:** un RAG end-to-end su documenti veri, messo online.
- **Dopo Parte 3:** aggiungi valutazione (golden dataset + LLM-as-judge), prompt versionati e
  tracciamento costi.
- **Dopo Parte 5:** un sistema agentico con tool reali, guardrail, observability — e una
  valutazione della *traiettoria*, non solo degli output (3.4).
- **Capstone:** il sistema completo della Parte 7.

---

## Sequenza consigliata

L'ordine numerico è l'ordine di studio. L'unica accortezza riguarda Parte 1 e Parte 3: NON
studiarle come due parti aperte in parallelo (da autodidatta discontinuo, tenere due fronti
aperti moltiplica i "riprendo da dove?"). Usa il **micro-interleaving**: prendi un argomento di
costruzione e la sua valutazione come **coppia chiusa**, finiscila, poi passa oltre. Esempio:
1.1 RAG → 3.1 + 3.3 (valutarlo) come blocco unico → poi 1.2 → ecc. Mischi i due argomenti (che è
ciò che li fissa) senza lasciare due parti spalancate insieme.

```
Parte 0 + Progetto 0  →  [1.1→3.1→3.3]  →  [1.2]  →  [1.4→3.4]  →  ...  coppie chiuse
                      →  Parte 2  →  Parte 4  →  Parte 5  →  Parte 6  →  Parte 7
```

Sul multimodale (Parte 2), per togliere ogni ambiguità a freddo: dipende **solo** dagli embedding
(0.2), quindi sei *libero* di collocarlo dove vuoi. La numerazione lo mette "2" perché è
conoscenza fondazionale; la sequenza qui sopra lo piazza dopo le coppie 1⇄3 solo per non aprire
troppi fronti insieme. **Scegli una volta e basta:** o lo fai subito dopo la Parte 0 (se vuoi
prima il quadro completo di cosa sa fare l'AI), o dopo aver chiuso le coppie 1⇄3 (se vuoi prima
saper costruire bene col testo). Entrambe vanno bene — l'importante è non lasciare la domanda
aperta. Regola generale: non passare oltre senza aver superato la verifica a memoria della parte precedente.

## Perché studi questo? (nota deliberatamente aperta)
Questo programma è **neutro per scelta**: copre "AI generalista applicativa, qualsiasi ruolo".
La neutralità ha un costo — ti fa studiare al 100% cose che per il *tuo* caso specifico potrebbero
valere il 40% (se punti a costruire prodotti tuoi, la Parte 4 governance è sovradimensionata e i
progetti vanno raddoppiati; se punti a un team enterprise, è il contrario). Ma tagliare il
programma ORA su un obiettivo sarebbe sbagliato, perché il bersaglio è ancora mobile — ed è
normale che lo sia. **La risposta a "perché lo studi" non è in cima a questo file: è a metà.**
Dopo la Parte 0 e il Progetto 0 saprai se ti diverte di più *costruire* o *progettare/decidere*,
e *allora* questo programma va ritagliato su quello. Promemoria: quando arrivi a quel punto,
fermati e taglia. Fino ad allora, neutro va bene.

## I confini di questa guida
**Coperto:** AI applicativa a base LLM, multimodale, system design AI, valutazione (output e
comportamento agentico), sicurezza e privacy, LLMOps, architettura, più il minimo concettuale di ML.
**Non coperto (di proposito):** addestrare modelli da zero, matematica del ML, MLOps di training
(model registry, feature store, retraining pipeline), infrastruttura cloud pura (Kubernetes,
networking, IaC). E l'esperienza pratica vera, che si prende solo costruendo e rompendo sistemi.
