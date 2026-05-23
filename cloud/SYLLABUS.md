---
title: Syllabus
sidebar_position: 2
---

# Syllabus — Concetti Universali e AWS

> Programma per passare da full-stack a **Cloud + AI engineer applicativo**. Stesso metodo della
> guida AI: apprendimento attivo (richiamo a memoria, spacing, micro-interleaving), badge di stato,
> decision drill, un progetto che evolve. Stessa voce (vedi AGENTS).
>
> Verificato su fonti di **maggio 2026**.
>
> **Strategia provider:** concetti **universali** prima (valgono su qualsiasi cloud), poi **AWS**
> dove diventi concreto e spendibile. AWS perché è il più richiesto e ha l'ecosistema AI più ampio.
> I concetti durano anni; console e nomi dei servizi cambiano → per quelli, badge `In evoluzione` e Radar.

---

## Traguardi e rientro

> Pensato per chi studia a sprazzi: il problema non è capire, è il **costo di rientro** dopo un vuoto.

**Due traguardi "uscita con dignità".** Se a un certo punto molli, questi sono i punti in cui hai
comunque qualcosa di reale in mano — non un percorso a metà, ma una competenza spendibile:
- **Fine Parte 1 = so come funziona il cloud e posso ragionare su un'architettura di base.** Già
  spendibile in qualsiasi team che usa cloud — sai leggere una VPC, sai perché il traffico non passa,
  sai chi ha accesso a cosa.
- **Fine Parte 5 = so deployare un sistema reale su AWS con IaC.** Il salto da "funziona dalla
  console" a "ho l'infrastruttura versionata, riproducibile, automatizzata". È il portfolio minimo
  che passa il filtro dei colloqui cloud.

Tutto il resto è di più — ma se ti fermi a uno di questi due, non hai sprecato niente.

**Log di rientro.** A fine di **ogni** sessione, scrivi due righe in un file dedicato: *dove sono
ora / la prossima cosa è X*. Al rientro apri quel file e nei primi 5 minuti sai cosa fare, senza
rileggere appunti né ricostruire il contesto. Nel cloud è ancora più importante che nell'AI:
terminali aperti, risorse accese, configurazioni a metà. Il log ti salva anche dalla bolletta.

**Rito di rientro dopo un vuoto lungo.** La prima sessione dopo una pausa di 1-2 settimane **non
è teoria**: è aggiungere una cosa piccola al progetto in corso. Rideploya, tocca la console,
fai girare la pipeline. Il filo torna dalle mani.

---

## Cosa evitare (verificato 2026)
I dati 2026 sono concordi su dove i principianti sprecano tempo. Evita questi, ti risparmi mesi:

- **Kubernetes troppo presto.** Il 70-80% dei ruoli K8s sono senior. Studiane i *concetti* (cosa
  fa, perché), ma NON diventare admin di cluster all'inizio: non è lì che entri. Impara prima
  serverless e container gestiti.
- **Più provider insieme.** Padroneggia **AWS** fino in fondo, poi semmai espandi. "So un po' di
  AWS, un po' di Azure, un po' di GCP" = non sai nessuno dei tre per un colloquio.
- **Certificazioni avanzate prima delle basi.** L'ordine giusto è uno solo (vedi sotto). La
  Professional prima dell'Associate è tempo perso.
- **LeetCode/algoritmi.** I colloqui cloud sono su *infrastruttura e architettura*, non su DSA.
  Non spendere lì il tempo che va sui progetti.
- **Over-engineering per moda.** L'errore #1 dell'industria: stack complessi (K8s, multi-region,
  service mesh) prima di avere un problema reale che li giustifichi. Scegli la tecnologia sul
  bisogno, non sull'hype. Questo è esso stesso una *skill* da dimostrare in un colloquio.
- **Cert senza progetti (e viceversa).** Una cert senza portfolio è prova debole; un portfolio
  senza cert non passa i filtri automatici dei recruiter. Servono **entrambi**, ma nella stanza
  del colloquio contano più i progetti.

## Cloud e AI in alternanza
Stai studiando Cloud e AI in alternanza. Funziona SOLO a blocchi: chiudi un'unità di un dominio
(lezione + pratica) prima di passare all'altro. **Mai due fronti aperti lo stesso giorno.**
Sinergia: il cloud è il pezzo che avevi tagliato dall'AI. Un sistema AI gira *su* cloud — i due
percorsi si chiudono nel capstone finale.

---

## Introduzione

### Esiti di apprendimento
Spiegare come funziona il cloud e progettare/deployare sistemi su cloud ragionando sui trade-off;
scegliere il giusto livello di astrazione (serverless vs container vs VM) sul bisogno reale;
gestire networking, identità, costi e monitoring; e far girare carichi AI in produzione sul cloud.

### Prerequisiti
Sviluppo full-stack (ce l'hai), Git, riga di comando di base, un linguaggio (Python utile).

### Metodo (vincolante)
Apprendimento attivo: verifica a memoria *dentro* la sessione (non come cancello al rientro);
spacing; micro-interleaving. Al rientro dopo una pausa: prima rimetti mano al progetto, poi ripassi.
I progetti non sono opzionali — nel cloud ancora più che altrove, perché è ciò che il colloquio guarda.

### Tre assi da non confondere
- **Concetti** (cosa è e perché) — durano anni, sono universali.
- **Pratica AWS** (come si fa davvero) — dove diventi spendibile.
- **Operations** (come sopravvive in produzione: costi, monitoring, sicurezza) — il giorno-2.

---

## PARTE 0 — Cosa è il cloud, davvero (universale)
- **0.1 Cosa significa "cloud"** — responsabilità condivisa, CapEx vs OpEx, il cambio mentale da
  "il mio server" a "risorse che accendo/spengo/pago a consumo".
- **0.2 IaaS / PaaS / SaaS** ⊳ 0.1 — chi gestisce cosa; scegliere il livello di astrazione. Primo decision drill.
- **0.3 Le risorse fondamentali** ⊳ 0.1 — compute, storage (oggetti/blocchi/file), networking, db gestiti.
- **0.4 Regioni, zone, geografia** ⊳ 0.3 — latenza, ridondanza, data residency (↔ privacy guida AI), disaster recovery.
- **0.5 Persistenza dei dati: SQL, NoSQL, oggetti, vector** ⊳ 0.3 — quando scegli quale famiglia di storage. Concetto universale, prerequisito di 5.3 e di RAG.
- **0.6 Linux essenziale per cloud** ⊳ 0.3 — file system, processi, permessi, SSH, log: il minimo per non essere ciechi su una VM o in un container.

## PARTE 1 — Networking e sicurezza di base (universale)
*Dove i full-stack hanno i buchi più grossi, e dove si fanno i danni più costosi.*
- **1.1 Networking nel cloud** ⊳ 0.3 — reti virtuali, subnet, IP pubblici/privati, gateway, firewall.
- **1.2 DNS, TLS, Load Balancing, CDN** ⊳ 1.1 — come gli utenti raggiungono il tuo sistema: nomi, certificati, distribuzione del traffico, cache di bordo. Concetti universali che precedono ogni servizio managed (Route 53, ALB, CloudFront).
- **1.3 Identity e permessi (IAM concettuale)** ⊳ 0.1 — "chi può fare cosa su quale risorsa".
  Concetto di sicurezza n.1 del cloud. Principio del **least privilege**. Il tema
  **Zero Trust / mTLS / service mesh** (autenticazione service-to-service) vive nella **7.6**:
  è materia di produzione, non di basi di IAM.
- **1.4 Segreti, chiavi, dati sensibili** ⊳ 1.3 — credenziali fuori dal codice (↔ sicurezza guida AI).
- **1.5 Decision drill — Networking & accessi**

## PARTE 2 — Container e dove far girare il codice (universale)
*Da full-stack parti avvantaggiato. Qui sta la decisione architetturale più frequente del cloud.*
- **2.1 Container** ⊳ 0.3 — Docker concettuale, immagini, il "sul mio computer funzionava" risolto.
- **2.2 Orchestrazione e Kubernetes (CONCETTI, non admin)** ⊳ 2.1 —
  <span class="badge-stato evoluzione">In evoluzione</span> cosa fa K8s e *perché* esiste,
  scaling/self-healing. **Solo concetti**: il 70-80% dei ruoli K8s è senior, non è lì che entri.
  Sapere *quando* serve (e quando no) vale più che saperlo amministrare.
- **2.3 VM vs Container vs Serverless** ⊳ 2.1 — lo spettro di astrazione. Il decision drill
  centrale: serverless per workload spiky/event-driven e poca ops; container/K8s per traffico
  sostenuto, stateful, controllo fine. Cold start, vendor lock-in, costi a confronto. Include
  **edge compute** (Cloudflare Workers, Lambda@Edge, CloudFront Functions): quando il deploy
  globale a bassa latenza vince, trade-off V8 isolate vs container, limiti di runtime.
- **2.4 Async ed event-driven** ⊳ 2.3 — quando una richiesta sincrona NON è la risposta giusta: code, pub/sub, event bus, idempotenza. Concetto universale prima dei nomi (SQS/SNS/EventBridge in 5.x).
- **2.5 API design per servizi cloud** ⊳ 2.3 — REST vs GraphQL, versioning, paginazione, rate limiting, idempotenza, autenticazione. L'interfaccia conta più del runtime.
- **2.6 Decision drill — Dove far girare la mia app** (caso reale con vincoli di traffico/budget)

## PARTE 3 — Dati in movimento (universale)
*Mini-modulo. La scelta di architettura dei dati che blocca se fatta tardi — capirla prima dell'IaC.*
- **3.1 Dati in movimento — batch, streaming, CDC** ⊳ 2.4 — quando il batch non basta: stream
  processing, event sourcing, Change Data Capture. Kafka e Kinesis come concetto.

## PARTE 4 — Infrastructure as Code & automazione (universale)
- **4.1 IaC: infrastruttura come codice** ⊳ 2.3 — perché non si clicca più nelle console:
  riproducibilità, versioning, revisione. **Terraform** come standard (multi-cloud). Le
  alternative consapevoli del 2026: **Pulumi** (IaC in linguaggi general-purpose), **OpenTofu**
  (fork OSS dopo il cambio di licenza Terraform 2023), **AWS CDK** (TypeScript/Python che compila
  a CloudFormation). Terraform resta standard, ma sapere quando le altre vincono è parte del lavoro.
- **4.2 Policy-as-code e governance dell'infra** ⊳ 4.1 — il giorno-2 dell'IaC. Le regole
  organizzative ("nessun bucket S3 pubblico", "tutti i volumi cifrati") espresse come codice
  eseguibile, non come PDF: **OPA/Conftest, Sentinel, Checkov, tfsec, Terrascan**. **Drift detection**
  (accorgersi che la console è stata toccata a mano). **GitOps** come pattern di chiusura del cerchio.
- **4.3 CI/CD nel cloud** ⊳ 4.1 — pipeline di deploy automatico, esteso all'infrastruttura.
- **4.4 Decision drill — Cosa automatizzare e cosa no**

## PARTE 5 — AWS: dal concetto alla pratica (specifico)
*Scegli il provider e diventi spendibile. I concetti 0-4 ora hanno un nome.*
*<span class="badge-stato evoluzione">In evoluzione</span> studia il mapping concetto→servizio, non i nomi a memoria.*
- **5.1 Orientarsi in AWS** — console, regioni, **IAM** (= 1.3 in AWS), **billing e budget alert**
  (impostali SUBITO: l'errore #1 dei principianti è la bolletta a sorpresa).
- **5.2 Compute** ⊳ 5.1, 2.3 — EC2 (VM), ECS/Fargate (container gestiti), **Lambda** (serverless).
- **5.3 Messaggistica ed eventi su AWS** ⊳ 5.1, 2.4 — **SQS** (code FIFO/standard, DLQ),
  **SNS** (pub/sub fan-out), **EventBridge** (event bus con regole di routing e schema registry),
  **Step Functions** (orchestrazione di workflow con stato). Pattern: fan-out SNS→SQS, idempotenza,
  retry con backoff, poison pill→DLQ. È il "secondo piano" della 2.4 universale.
- **5.4 Storage e database** ⊳ 5.1, 0.3 — S3 (oggetti), EBS (blocchi), RDS/DynamoDB.
- **5.5 Networking** ⊳ 5.1, 1.1 — VPC, subnet, security group, API Gateway.
- **5.6 IaC su AWS** ⊳ 5.1, 4.1 — Terraform (preferibile) o CloudFormation/CDK.
- **5.7 Observability** ⊳ 5.1 — CloudWatch: log, metriche, alert.
- **5.8 Decision drill — Progetta una piccola architettura AWS serverless**
  (es. URL shortener: Lambda + API Gateway + DynamoDB + IAM least-privilege + CloudWatch —
  il progetto-tipo che i colloqui 2026 vogliono vedere)

## PARTE 6 — Cloud per l'AI (la sinergia — il tuo bersaglio)
*Dove Cloud e AI si incontrano. È la competenza che cresce di più nel 2026 (MLOps job ~10× in 5 anni).*
- **6.1 Far girare carichi AI sul cloud** ⊳ 5.2 — **GPU nel cloud** (perché costano, come si
  provisiona), serving a bassa latenza, batch vs real-time inference.
- **6.2 Servizi AI gestiti vs self-hosted** ⊳ 5.2 — API gestita (es. AWS Bedrock / SageMaker) vs
  ospitare il proprio modello: il build-vs-buy della guida AI in versione infrastrutturale.
- **6.3 Ottimizzare i costi di inferenza** ⊳ 6.1, 5.1 — il costo AI in produzione esplode qui;
  caching, batching, scelta dell'istanza. (↔ FinOps guida AI.)
- **6.4 Caching e CDN strategies** ⊳ 6.3, 1.2 — semantic cache per LLM, response cache, edge caching di embedding e risposte. La leva di costo e latenza più sottovalutata.
- **6.5 K8s per workload AI** ⊳ 2.2, 6.1 —
  <span class="badge-stato evoluzione">In evoluzione</span> qui K8s torna rilevante anche per
  non-senior: GPU scheduling, batch job, inference scalabile. Concetti + quando serve.
- **6.6 Architettura di un sistema AI in produzione su cloud** ⊳ tutto + Parti 5-6 guida AI —
  la sintesi: dove vivono retrieval, modello, guardrail, observability su infra reale. Include
  il pattern **LLM gateway** (LiteLLM, Portkey, Helicone) come componente di sistema: dove vive
  (in front di Bedrock o di un endpoint self-hosted), cosa fa (routing multi-modello, caching,
  rate limit, budget), perché conta. Cross-ref a 5.6 AI (caching semantico + model routing).
- **6.7 Vector database managed vs self-hosted** ⊳ 0.5, 6.1 — Pinecone, OpenSearch KNN, pgvector,
  Weaviate: la decisione build/managed applicata allo storage vettoriale. Quando pgvector nel DB
  esistente batte il servizio dedicato e quando no.
- **6.8 Training pipeline gestiti** *(opzionale)* ⊳ 6.1, 6.2 — SageMaker Pipelines, model registry,
  feature store: il pezzo MLOps che cade tra le due guide. Concetti senza entrare nell'addestramento.
  Opzionale per chi punta a ruoli AI-applicativi puri; obbligatorio per chi mira a MLOps Engineer.
- **6.9 Decision drill — Deploya il RAG della guida AI su AWS**

## PARTE 7 — Operations, costi, sicurezza in produzione
- **7.1 FinOps: il cloud costa in modi sorprendenti** ⊳ 5.1 — controllo spesa, gli errori che
  svuotano la carta (istanze dimenticate accese, NAT Gateway, traffico in uscita). Include la
  dimensione **sostenibilità e carbon-aware FinOps**: emissioni come asse di costo (AWS Customer
  Carbon Footprint Tool, scheduling carbon-aware delle workload spostabili), è entrato nelle slide
  enterprise nel 2025-26.
- **7.2 Monitoring e incident response** ⊳ 5.7 — il giorno-2: alert sensati, cosa guardare quando
  qualcosa si rompe alle 3 di notte.
- **7.3 Distributed tracing e observability avanzata** ⊳ 7.2 — quando il sistema è fatto di tanti pezzi (microservizi, async, LLM call): tracing end-to-end, correlazione log/metriche/trace, SLO e error budget.
- **7.4 Sicurezza in produzione** ⊳ 1.3, 1.4 — least privilege applicato, superficie d'attacco,
  errori di configurazione comuni (bucket S3 pubblici per sbaglio, ecc.).
- **7.5 Resilienza e disaster recovery** ⊳ 0.4, 5.2 — RPO/RTO, backup, failover multi-AZ e
  multi-region, chaos engineering: i concetti che il business chiede e l'engineer deve saper
  ragionare prima che succeda qualcosa.
- **7.6 Zero Trust e mTLS service-to-service** ⊳ 1.3, 2.2 — *never trust, always verify*: ogni
  chiamata service-to-service autenticata indipendentemente dalla rete. mTLS, service mesh
  (Istio, App Mesh, Linkerd), IAM Auth + SigV4 come alternativa AWS-native. Materia di produzione
  che era impropriamente in 1.3 — qui ha il contesto giusto (microservizi, multi-account, compliance).
- **7.7 Decision drill — I costi sono raddoppiati: cosa indaghi e in che ordine**

## PARTE 8 — Sintesi e portfolio
- **8.1 Reference architecture cloud** ⊳ Parti 1-7.
- **8.2 Capstone — un sistema reale end-to-end su AWS, con IaC** ⊳ tutto — idealmente ci metti
  sopra un progetto della guida AI. Cloud + AI chiusi nello stesso artefatto.
- **8.3 Il portfolio che fa assumere** — i progetti 2026 che contano nel colloquio, messi su GitHub
  con IaC. Cert + progetti insieme (vedi sotto).

---

## PARTE 9 — Architettura enterprise (traccia SA)
*Opzionale ma strategica per chi punta ad AI Cloud Solution Architect. I 5 temi che la guida
base non copre e che nei colloqui SA chiedono sempre.*

- **9.1 AWS Well-Architected Framework** ⊳ Parti 5-7 — i 6 pilastri (operational excellence,
  security, reliability, performance efficiency, cost optimization, sustainability) + il
  **Responsible AI Lens** (nov 2025). Come si fa una WAF Review con un cliente, cosa produce,
  come si prioritizzano gli High-Risk Issues. Lo strumento che i SA AWS usano in ogni engagement.
- **9.2 Governance multi-account** ⊳ 1.3, 5.1 — AWS Organizations: OUs, Service Control
  Policies (SCP) e Resource Control Policies (RCP), come si separa produzione da dev da sandbox.
  **AWS Control Tower**: landing zone come "account vending machine", Account Factory, guardrail
  preventivi/detective/proattivi, drift detection. Il pattern che le enterprise chiedono prima di
  firmare qualsiasi contratto cloud.
- **9.3 Migrazione e modernizzazione** ⊳ Parti 0-5 — le **7R** (Retire, Retain, Rehost,
  Relocate, Repurchase, Replatform, Refactor): la griglia di decisione per ogni workload da
  migrare. **Strangler fig** per i monoliti. AWS Migration Hub come strumento di tracking.
  Come si presenta una business case di migrazione al C-level.
- **9.4 Compliance enterprise** ⊳ 7.4 — SOC 2, ISO 27001, HIPAA: cosa chiedono davvero, cosa
  devi predisporre nell'infrastruttura, audit trail. Non diventi un compliance officer, ma non
  ti blocchi al primo meeting enterprise. Qui sta perché si applica alla *governance* (multi-account,
  policy-as-code, audit), non al day-2 puro delle operations.
- **9.5 Multi-cloud: consapevolezza senza dispersione** ⊳ Parti 0-4 — Azure e GCP come
  equivalenti concettuali (il mapping: EC2 ↔ Azure VM ↔ Compute Engine; Lambda ↔ Azure Functions
  ↔ Cloud Run; S3 ↔ Blob Storage ↔ GCS). Quando la scelta multi-cloud è tecnica (ridurre
  vendor lock-in, latenza geografica) e quando è politica (il cliente ha già Azure). **FinOps
  Foundation FOCUS standard** (2026): schema unificato per costi cross-cloud. Non si diventa
  multi-cloud engineer — si impara a parlarne con i clienti.
- **9.6 Decision drill — Progetta una landing zone enterprise** — scenario reale: azienda da
  50 developer, 3 team, requisiti di compliance SOC 2. Disegna la struttura account, gli SCP,
  il processo di provisioning.

---

## Certificazioni (verificato 2026 — ordine e ROI)
Non sono l'obiettivo, ma passano i filtri dei recruiter. L'ordine giusto:
1. **Cloud Practitioner** (opzionale, per principianti assoluti; dà spesso un voucher sconto) — segnale debole da solo.
2. **Solutions Architect Associate (SAA-C03)** — la cert col **ROI più alto**, quella che compare
   in più job description e copre la gamma più ampia di pattern. Timeline tipica: 4-6 settimane. È *la* cert da avere.
3. **Machine Learning Engineer Associate (MLA-C01)** — nuova (lanciata 2024), costruita apposta
   per il profilo AI + cloud: deployment, MLOps, inferenza scalabile, SageMaker, Bedrock. Per chi
   punta all'AI Cloud SA è la terza cert naturale dopo SAA, molto più centrata del vecchio Developer
   Associate. Timeline tipica: 3-4 settimane da SAA.
Due-tre cert è il "sweet spot". Le Professional vengono molto dopo, se mai.

## Progetto che evolve (un solo sistema che cresce)
- **Fase 0:** una pagina/app statica su AWS via CLI (S3 + CloudFront). Tocca la console, vedi i costi.
- **Fase 1:** containerizzala (Docker).
- **Fase 2:** un'API serverless reale (Lambda + API Gateway + DynamoDB + IAM least-privilege + CloudWatch).
- **Fase 3:** riproduci TUTTO con Terraform (IaC). Stesso sistema, zero click.
- **Fase 4:** CI/CD per il deploy automatico.
- **Fase 5:** ci fai girare sopra il RAG della guida AI.
- **Fase 6:** monitoring, alert, ottimizzazione costi.

## Sequenza
Parti 0→4 universali, in ordine (con la Parte 3 come mini-modulo dati). Parte 5 (AWS) quando i
concetti reggono. Parte 6 (Cloud+AI) quando ANCHE la guida AI è a buon punto. Micro-interleaving
dentro ogni parte. Parti 8-9 alla fine.

## I confini
**Coperto:** concetti cloud universali, networking/sicurezza, container, dati in movimento, IaC,
AWS pratico sui servizi che contano (~15, non 200), Cloud+AI, FinOps, operations in produzione,
certificazioni. Parte 9 (opzionale): Well-Architected, governance enterprise, migrazione,
compliance, multi-cloud a livello di consapevolezza.
**Non coperto (di proposito):** amministrazione K8s avanzata (senior), networking di basso livello
da sysadmin, ogni servizio AWS. E l'esperienza vera, che si prende deployando e rompendo cose reali.
