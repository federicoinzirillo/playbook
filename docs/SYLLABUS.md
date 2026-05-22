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
(3.4, 4.2). Un programma statico su un campo così non basta: serve una **valvola di sfogo**.
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

- **0.1 Come funziona un LLM** — token, context window, sampling, temperature. Include perché i token sono l'unità di costo (paghi a token, non a parola): un chiodo da piantare presto.
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

## PARTE 1 — Costruire sistemi LLM
*Il cuore. Subito dopo i fondamenti.*

- **1.1 RAG** ⊳ 0.2 — chunking, vector DB, retrieval, reranking, hybrid search, GraphRAG.
- **1.2 Context engineering: cosa mettere nel contesto, a scala** ⊳ 1.1 — competenza distinta dal
  RAG classico. Con context window enormi, *decidere cosa entra nel contesto* è un mestiere a sé:
  il trade-off tra riempire il contesto e il degrado di attenzione nel mezzo ("lost in the
  middle"), più il costo che cresce col contesto. Aggancio al triangolo qualità-latenza-costo (5.3).
- **1.3 Structured output e function calling** ⊳ 1.1 — far produrre dati strutturati, collegare funzioni.
- **1.4 Agenti e orchestrazione** ⊳ 1.3 — tool calling, ReAct, multi-agent, orchestratore-worker.
  *Principio chiave: ciò che va garantito sta nell'orchestratore, ciò che richiede giudizio
  nell'LLM. Mai mettere il controllo di flusso nei prompt (→ loop e costi fuori controllo).*
  *Costo e latenza: un sistema multi-agent costa e impiega molto più di una singola chiamata —
  ogni passo è un'altra inferenza. Vedi il triangolo in 5.3 prima di scegliere un'architettura agentica.*
- **1.5 MCP — Model Context Protocol** ⊳ 1.3 — lo standard per collegare modelli, tool e dati.
- **1.6 Decision drill — Costruire**
- **1.7 Decision drill — Fine-tuning vs RAG vs prompt engineering vs context engineering** ⊳ 0.3,
  0.5, 1.1, 1.2 — la domanda architetturale più frequente in assoluto. Griglia di quando l'uno
  batte l'altro: costo, dati disponibili, frequenza di aggiornamento, serve uno *stile* o una
  *conoscenza*.

## PARTE 2 — Multimodale e altri tipi di AI
*La fetta "AI generalista" oltre il testo. Conoscenza fondazionale di cosa sa fare l'AI oggi.*

- **2.1 Come funziona il multimodale** ⊳ 0.2 — l'idea che testo, immagini, audio diventano
  embedding in uno spazio condiviso. Architettura **nativa** (un solo modello per tutte le
  modalità) vs pipeline separate: oggi il nativo è lo standard di fatto.
- **2.2 Vision: cosa sa fare l'AI sulle immagini** ⊳ 2.1 — classificazione, object detection, OCR,
  document understanding, video. A livello di capacità e casi d'uso.
- **2.3 Audio e speech** ⊳ 2.1 — speech-to-text e text-to-speech; quando usare un modello dedicato
  (solo trascrizione) vs un multimodale (trascrizione + ragionamento).
- **2.4 Generazione di immagini (diffusion models)** ⊳ 2.1 — l'idea di base, senza la matematica;
  casi d'uso e limiti.
- **2.5 Quando multimodale, quando pipeline separate** ⊳ 2.1 — criteri di scelta e trade-off.
- **2.6 Decision drill — Multimodale**

## PARTE 3 — Valutare e rendere affidabile
*Ciò che separa una demo da un sistema serio. In interleaving con Parte 1.*

- **3.1 LLM-as-judge** ⊳ 1.1 — golden dataset, criteri, eval offline/online, bias del giudice.
  Valuta il **singolo output**: la base, ma non basta per gli agenti (vedi 3.4).
- **3.2 Observability** ⊳ 3.1 — tracing, costi, latenza, cosa loggare per *misurare*. Include il
  tracking dei prompt versionati introdotti in 0.5.
- **3.3 Gestire le allucinazioni** ⊳ 1.1, 3.1 — perché succedono, grounding, mitigazione.
- **3.4 Valutare il comportamento agentico end-to-end** ⊳ 1.4, 3.1 —
  <span class="badge-stato evoluzione">In evoluzione</span> il buco che si sta allargando più in
  fretta nel settore. Un agente non è un singolo output, è una **traiettoria** di decisioni e tool.
  Può produrre ogni risposta plausibile e fallire *come processo*: tool sbagliato, loop, obiettivo
  raggiunto per la strada sbagliata, o mancato pur "suonando bene". Si valuta la traiettoria, la
  scelta dei tool, il completamento del task — non solo il token finale. Scritto sui **principi**
  (cosa significa valutare un processo, quali domande porsi), non sui tool del mese, perché è
  un'area giovane che cambia in fretta.
- **3.5 Decision drill — Valutazione** (incluso un caso di valutazione agentica)

## PARTE 4 — Sicurezza, privacy e governance
- **4.1 Prompt injection e OWASP LLM Top 10** ⊳ 1.4 — attacchi, esfiltrazione, abuso di tool sul
  *singolo input*, guardrail. La sicurezza va resa **strutturale** (policy engine tra LLM e azioni),
  non affidata ai prompt.
- **4.2 Sicurezza agentica: la superficie d'attacco comportamentale** ⊳ 4.1, 1.4 —
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

- **5.1 Anatomia di un sistema AI** ⊳ 1.4 — i componenti (orchestratore, retrieval, tool,
  guardrail, observability) e come si parlano. L'LLM è ~20% di un sistema di produzione.
- **5.2 Pattern di sistema** ⊳ 5.1 — sincrono vs asincrono, batch vs real-time, code/queue,
  batching delle richieste, load balancing. Sono i pattern dei sistemi distribuiti applicati all'AI.
  *Qui si sceglie il pattern; il come gestirlo in produzione è Parte 6.*
- **5.3 Il triangolo qualità-latenza-costo** ⊳ 5.1 — il trade-off che governa ogni decisione
  (richiamato da 1.2 e 1.4); caching (incluso semantico), rate limiting, fallback, ridondanza.
  *Qui si ragiona sul trade-off in fase di design; l'ottimizzazione dei costi reali a runtime è 6.2.*
- **5.4 I dati come spina dorsale** ⊳ 0.2 — qualità, preparazione, pipeline a livello
  concettuale; database vettoriali a fondo. Spesso il vero collo di bottiglia, sottovalutato.
- **5.5 Decision drill — System design** (es. "progetta un chatbot di supporto su LLM di terzi")

## PARTE 6 — Produzione (LLMOps)
*L'asse OPERATIONS: far vivere in produzione un sistema che NON addestri. Non ripete la Parte 5
(design) né la 3.2 (misurare): qui si parla di **tool concreti e giorno-2** — cosa fai quando il
sistema è già live e qualcosa va storto.*

- **6.1 Serving e inference** ⊳ 5.1 — gli **strumenti reali**: gateway/router, motori (vLLM, TGI),
  come si espone e si scala un endpoint. (La 5.2 sceglie il pattern; qui lo si mette in opera.)
- **6.2 Costi e FinOps a runtime** ⊳ 6.1, 5.3 — non il *ragionamento* sul triangolo (5.3) ma la
  **gestione reale**: monitoraggio della spesa, alert, cosa fai quando i costi esplodono in
  produzione, ottimizzazione token a posteriori.
- **6.3 Monitoring e drift in produzione** ⊳ 3.2, 6.1 — non il *cosa loggare per valutare* (3.2)
  ma il **giorno-2**: il degrado che emerge dopo settimane, gli alert operativi, l'incident response
  quando la qualità cala in modo silenzioso.
- **6.4 Decision drill — Produzione** (es. "il sistema funziona ma i costi sono raddoppiati: cosa
  indaghi e in che ordine")

## PARTE 7 — Architettura e sintesi
- **7.1 Reference architecture** ⊳ Parti 1-6 — come si compone un sistema completo.
- **7.2 Build vs buy, proprietari vs open-weight** ⊳ 7.1 — criteri di decisione, costo, controllo.
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
