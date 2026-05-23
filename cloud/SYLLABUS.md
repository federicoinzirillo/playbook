---
title: Syllabus
sidebar_position: 2
---

# Syllabus — Cloud + AI Engineer (concetti universali → AWS)

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

## ⚠️ LEGGI PRIMA: cosa EVITARE (verificato 2026)
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

## ⚠️ Cloud + AI in alternanza
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

## PARTE 0 — Cosa è il cloud, davvero 🟢 (universale)
- **0.1 Cosa significa "cloud"** — responsabilità condivisa, CapEx vs OpEx, il cambio mentale da
  "il mio server" a "risorse che accendo/spengo/pago a consumo".
- **0.2 IaaS / PaaS / SaaS** ⊳ 0.1 — chi gestisce cosa; scegliere il livello di astrazione. Primo decision drill.
- **0.3 Le risorse fondamentali** ⊳ 0.1 — compute, storage (oggetti/blocchi/file), networking, db gestiti.
- **0.4 Regioni, zone, geografia** ⊳ 0.3 — latenza, ridondanza, data residency (↔ privacy guida AI), disaster recovery.

## PARTE 1 — Networking e sicurezza di base 🔵 (universale)
*Dove i full-stack hanno i buchi più grossi, e dove si fanno i danni più costosi.*
- **1.1 Networking nel cloud** ⊳ 0.3 — reti virtuali, subnet, IP pubblici/privati, gateway, firewall.
- **1.2 Identity e permessi (IAM concettuale)** ⊳ 0.1 — "chi può fare cosa su quale risorsa".
  Concetto di sicurezza n.1 del cloud. Principio del **least privilege**.
- **1.3 Segreti, chiavi, dati sensibili** ⊳ 1.2 — credenziali fuori dal codice (↔ sicurezza guida AI).
- **1.4 Decision drill — Networking & accessi**

## PARTE 2 — Container e dove far girare il codice 🔵 (universale)
*Da full-stack parti avvantaggiato. Qui sta la decisione architetturale più frequente del cloud.*
- **2.1 Container** ⊳ 0.3 — Docker concettuale, immagini, il "sul mio computer funzionava" risolto.
- **2.2 Orchestrazione e Kubernetes (CONCETTI, non admin)** ⊳ 2.1 —
  <span class="badge-stato evoluzione">In evoluzione</span> cosa fa K8s e *perché* esiste,
  scaling/self-healing. **Solo concetti**: il 70-80% dei ruoli K8s è senior, non è lì che entri.
  Sapere *quando* serve (e quando no) vale più che saperlo amministrare.
- **2.3 VM vs Container vs Serverless** ⊳ 2.1 — lo spettro di astrazione. Il decision drill
  centrale: serverless per workload spiky/event-driven e poca ops; container/K8s per traffico
  sostenuto, stateful, controllo fine. Cold start, vendor lock-in, costi a confronto.
- **2.4 Decision drill — Dove far girare la mia app** (caso reale con vincoli di traffico/budget)

## PARTE 3 — Infrastructure as Code & automazione 🔵 (universale)
- **3.1 IaC: infrastruttura come codice** ⊳ 2.3 — perché non si clicca più nelle console:
  riproducibilità, versioning, revisione. **Terraform** come standard (multi-cloud).
- **3.2 CI/CD nel cloud** ⊳ 3.1 — pipeline di deploy automatico, esteso all'infrastruttura.
- **3.3 Decision drill — Cosa automatizzare e cosa no**

## PARTE 4 — AWS: dal concetto alla pratica 🟡 (specifico)
*Scegli il provider e diventi spendibile. I concetti 0-3 ora hanno un nome.*
*<span class="badge-stato evoluzione">In evoluzione</span> studia il mapping concetto→servizio, non i nomi a memoria.*
- **4.1 Orientarsi in AWS** — console, regioni, **IAM** (= 1.2 in AWS), **billing e budget alert**
  (impostali SUBITO: l'errore #1 dei principianti è la bolletta a sorpresa).
- **4.2 Compute** ⊳ 4.1, 2.3 — EC2 (VM), ECS/Fargate (container gestiti), **Lambda** (serverless).
- **4.3 Storage e database** ⊳ 4.1, 0.3 — S3 (oggetti), EBS (blocchi), RDS/DynamoDB.
- **4.4 Networking** ⊳ 4.1, 1.1 — VPC, subnet, security group, API Gateway.
- **4.5 IaC su AWS** ⊳ 4.1, 3.1 — Terraform (preferibile) o CloudFormation/CDK.
- **4.6 Observability** ⊳ 4.1 — CloudWatch: log, metriche, alert.
- **4.7 Decision drill — Progetta una piccola architettura AWS serverless**
  (es. URL shortener: Lambda + API Gateway + DynamoDB + IAM least-privilege + CloudWatch —
  il progetto-tipo che i colloqui 2026 vogliono vedere)

## PARTE 5 — Cloud per l'AI 🔵 (la sinergia — il tuo bersaglio)
*Dove Cloud e AI si incontrano. È la competenza che cresce di più nel 2026 (MLOps job ~10× in 5 anni).*
- **5.1 Far girare carichi AI sul cloud** ⊳ 4.2 — **GPU nel cloud** (perché costano, come si
  provisiona), serving a bassa latenza, batch vs real-time inference.
- **5.2 Servizi AI gestiti vs self-hosted** ⊳ 4.2 — API gestita (es. AWS Bedrock / SageMaker) vs
  ospitare il proprio modello: il build-vs-buy della guida AI in versione infrastrutturale.
- **5.3 Ottimizzare i costi di inferenza** ⊳ 5.1, 4.1 — il costo AI in produzione esplode qui;
  caching, batching, scelta dell'istanza. (↔ FinOps guida AI.)
- **5.4 K8s per workload AI** ⊳ 2.2, 5.1 —
  <span class="badge-stato evoluzione">In evoluzione</span> qui K8s torna rilevante anche per
  non-senior: GPU scheduling, batch job, inference scalabile. Concetti + quando serve.
- **5.5 Architettura di un sistema AI in produzione su cloud** ⊳ tutto + Parti 5-6 guida AI —
  la sintesi: dove vivono retrieval, modello, guardrail, observability su infra reale.
- **5.6 Decision drill — Deploya il RAG della guida AI su AWS**

## PARTE 6 — Operations, costi, sicurezza in produzione 🟡
- **6.1 FinOps: il cloud costa in modi sorprendenti** ⊳ 4.1 — controllo spesa, gli errori che
  svuotano la carta (istanze dimenticate accese, NAT Gateway, traffico in uscita).
- **6.2 Monitoring e incident response** ⊳ 4.6 — il giorno-2: alert sensati, cosa guardare quando
  qualcosa si rompe alle 3 di notte.
- **6.3 Sicurezza in produzione** ⊳ 1.2, 1.3 — least privilege applicato, superficie d'attacco,
  errori di configurazione comuni (bucket S3 pubblici per sbaglio, ecc.).
- **6.4 Decision drill — I costi sono raddoppiati: cosa indaghi e in che ordine**

## PARTE 7 — Sintesi e portfolio 🟡
- **7.1 Reference architecture cloud** ⊳ Parti 1-6.
- **7.2 Capstone — un sistema reale end-to-end su AWS, con IaC** ⊳ tutto — idealmente ci metti
  sopra un progetto della guida AI. Cloud + AI chiusi nello stesso artefatto.
- **7.3 Il portfolio che fa assumere** — i progetti 2026 che contano nel colloquio, messi su GitHub
  con IaC. Cert + progetti insieme (vedi sotto).

---

## Certificazioni (verificato 2026 — ordine e ROI)
Non sono l'obiettivo, ma passano i filtri dei recruiter. L'ordine giusto:
1. **Cloud Practitioner** (opzionale, per principianti assoluti; dà spesso un voucher sconto) — segnale debole da solo.
2. **Solutions Architect Associate (SAA-C03)** — la cert col **ROI più alto**, quella che compare
   in più job description e copre la gamma più ampia di pattern. Timeline tipica: 4-6 settimane. È *la* cert da avere.
3. **Developer Associate (DVA-C02)** — per il tuo profilo (sviluppatore): serverless, CI/CD,
   Lambda, API Gateway, DynamoDB. Mappa diretta a job da developer.
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
Parti 0→3 universali, in ordine. Parte 4 (AWS) quando i concetti reggono. Parte 5 (Cloud+AI)
quando ANCHE la guida AI è a buon punto. Micro-interleaving dentro ogni parte.

## I confini
**Coperto:** concetti cloud universali, networking/sicurezza, container, IaC, AWS pratico sui
servizi che contano (~15, non 200), Cloud+AI, FinOps, certificazioni.
**Non coperto (di proposito):** amministrazione K8s avanzata (senior), networking di basso livello
da sysadmin, multi-cloud, ogni servizio AWS. E l'esperienza vera, che si prende deployando e
rompendo cose reali.
