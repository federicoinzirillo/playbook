---
title: Decision drill — Dove far girare la mia app
sidebar_label: "2.6 Decision drill"
sidebar_position: 6
---

# Decision drill — Dove far girare la mia app

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 2.6</span>
  <span>~10 min di lettura</span>
</div>

<p class="lesson-lead">Quattro scenari reali con vincoli di traffico, budget e team. Ogni scenario ha una risposta difendibile — e una trappola comune in cui i junior cadono.</p>

Questa lezione non introduce nuovi concetti. Prende quelli delle lezioni 2.1–2.5 e li mette sotto pressione con scenari realistici: budget, traffico, struttura del team, requisiti di latenza. L'obiettivo è allenare il giudizio, non la memoria.

Prima di guardare le griglie di valutazione, prova a rispondere da solo. La verifica attiva fissa più di qualsiasi rilettura.

---

## Scenario A — Startup SaaS con traffico imprevedibile

**Situazione**: sei il solo sviluppatore di una startup SaaS B2B. L'applicazione è un dashboard analytics per PMI — richieste HTTP sincrone, sessioni utente di 20-40 minuti. Il traffico è imprevedibile: 0 richieste di notte, 100-500 richieste/secondo durante gli orari lavorativi europei. Budget infrastrutturale: €200/mese. Non hai un DevOps team.

**Vincoli aggiuntivi**: l'applicazione fa query su un database PostgreSQL con JOINs complesse (risposta media 200ms). Non esiste logica di elaborazione asincrona.

**Domande**:
1. Scegli l'approccio di deployment (VM, container, serverless) e giustifica.
2. Lambda è adatta a questo caso? Perché sì o no?
3. Quale servizio gestito AWS usi per il PostgreSQL?

<details>
<summary>Griglia di valutazione</summary>

**Risposta difendibile**: ECS Fargate + RDS PostgreSQL.

- Il traffico ha profilo di picco diurno con drop notturno — non è spiky/event-driven (non è il caso ideale di Lambda). Le sessioni durano 20-40 minuti con query continue: Lambda non è adatta a sessioni utente sostenute e ha cold start che peggiora l'esperienza.
- ECS Fargate con scaling basato su CPU/richieste gestisce il drop notturno scalando a zero task (o a 1 minimo), riducendo i costi fuori orario.
- RDS PostgreSQL: managed, backup automatici, patch gestite da AWS. Non gestire PostgreSQL su EC2 da solo quando sei un dev solo è la scelta ovvia.

**Budget check**: Fargate da 0,5 vCPU / 1 GB RAM, 8 ore/giorno di picco × 20 giorni lavorativi = ~€20/mese compute. RDS t3.micro ~$13/mese. CloudWatch, ALB: ~€30/mese. Totale: ~€65/mese — dentro budget.

**Trappola comune**: scegliere Lambda perché "serverless scala da solo". Lambda scala, ma ha cold start su query PostgreSQL (connessioni TCP non persistenti tra invocazioni), e il modello a funzione non si adatta bene a sessioni utente con stato. RDS Proxy mitiga il problema delle connessioni, ma aggiunge costo e complessità.
</details>

---

## Scenario B — Elaborazione batch notturna con picchi

**Situazione**: un e-commerce che ogni notte alle 2:00 elabora 500.000 ordini del giorno per generare report di magazzino, aggiornare i prezzi dinamici, e inviare 50.000 email di conferma ai clienti. L'elaborazione richiede ~45 minuti totali. Di giorno il sistema è praticamente idle.

**Vincoli aggiuntivi**: le email vanno inviate in ordine (prima gli ordini prioritari), ma i report di magazzino e l'aggiornamento prezzi sono indipendenti tra loro.

**Domande**:
1. Quale modello di esecuzione usi per il job di 45 minuti? (Lambda ha un timeout di 15 minuti.)
2. Come parallelizzi l'elaborazione degli ordini in modo controllato?
3. Le email devono essere idempotenti? Perché?

<details>
<summary>Griglia di valutazione</summary>

**Risposta difendibile**: AWS Batch (o ECS task schedulato da EventBridge) per il job principale. SQS per l'invio email con idempotency key.

- Lambda va fuori dai 15 minuti per un job di 45 minuti — out. ECS task schedulato (triggato da EventBridge Scheduler alle 2:00) è il pattern corretto per batch overnight: container che parte, gira fino a completamento, si ferma. Costo: zero quando non gira.
- La parallelizzazione degli ordini si fa con SQS: il job principale suddivide gli ordini in messaggi e una fleet di Lambda (o container) li consuma in parallelo. L'ordine di priorità si gestisce con una SQS FIFO per le email prioritarie.
- Le email **devono** essere idempotenti: se il job crasha a metà, al retry alcuni ordini già processati verrebbero reprocessati. Senza idempotenza → email duplicate ai clienti. L'idempotency key è l'ID ordine.

**Trappola comune**: progettare il job come un monolite sequenziale. Se qualcosa crasha al minuto 40 di un job sequenziale, ricomincia da capo. La parallelizzazione con SQS dà resilienza gratuita.
</details>

---

## Scenario C — Microservizi con latenza critica

**Situazione**: un'applicazione fintech con SLA contrattuale di 200ms p99 su ogni richiesta API. Il sistema ha 8 microservizi che si chiamano in cascata: ogni richiesta dell'utente attiva mediamente 3 chiamate a servizi downstream. Il team ha 10 sviluppatori con esperienza K8s.

**Vincoli aggiuntivi**: alcune chiamate downstream sono idempotenti e ripetibili, altre no. Il traffico è stabile (10.000 req/sec costanti, con picchi 3× a Natale). Hai 3 mesi per il deployment iniziale.

**Domande**:
1. Container orchestrati o serverless? Quale servizio AWS?
2. Le chiamate tra microservizi in cascata potrebbero creare problemi di latenza anche sotto i 200ms — cosa monitori?
3. Come gestisci i picchi 3× a Natale senza over-provisioning tutto l'anno?

<details>
<summary>Griglia di valutazione</summary>

**Risposta difendibile**: EKS (K8s su AWS) con HPA per scaling + X-Ray per distributed tracing.

- Traffico stabile a 10.000 req/sec + team K8s fluent + 8 microservizi con cicli di vita diversi: è esattamente il caso d'uso di K8s. Lambda sarebbe sbagliata: cold start romperebbe il SLA 200ms p99 durante i picchi (istanze non warm al momento del picco natalizio).
- **Latency budget**: con 3 hop in cascata e SLA totale 200ms, ogni hop ha un budget di ~60ms. Monitori latenza p50/p95/p99 per ogni servizio con AWS X-Ray (distributed tracing) o un equivalente. Senza visibilità su dove va il tempo, non puoi ottimizzare.
- **Picchi natalizi**: HPA scala automaticamente le repliche. Pre-scalare manualmente la settimana prima (Predictive Scaling) per evitare cold start K8s durante il picco reale. I Kubernetes cluster autoscaler aggiunge nodi se necessario.

**Trappola comune**: non considerare il **tail latency amplification** nelle chiamate in cascata. Se il servizio A chiama B, C, D in sequenza e ognuno ha p99=50ms, il p99 totale non è 150ms ma molto di più (la probabilità di incontrare un evento lento si moltiplica). Il pattern circuit breaker (es. con Istio o una libreria come Resilience4j) previene che un servizio lento degradi l'intera cascata.
</details>

---

## Scenario D — Pipeline ML con stream di eventi

**Situazione**: una piattaforma di streaming video vuole aggiornare le raccomandazioni utente in tempo reale basandosi sui video guardati nelle ultime 2 ore. Il sistema elabora 500.000 eventi di "video completato" all'ora. Il modello ML di raccomandazione impiega 80ms per produrre una lista personalizzata.

**Vincoli aggiuntivi**: l'aggiornamento delle raccomandazioni non deve essere visibile all'utente durante la riproduzione (asincrono), ma deve essere disponibile entro 30 secondi dalla fine del video. Il budget mensile è flessibile ma va ottimizzato.

**Domande**:
1. Quale sistema di streaming usi per gli eventi "video completato"?
2. Lambda o container per il job di inferenza ML da 80ms?
3. Come garantisci la finestra di 2 ore per le feature ML?

<details>
<summary>Griglia di valutazione</summary>

**Risposta difendibile**: Kinesis Data Streams per gli eventi + Lambda per inferenza + DynamoDB per feature store.

- Kinesis Data Streams regge 500.000 eventi/ora con shard multipli. Alternativa: SQS con Lambda trigger — più semplice da gestire, adeguato se non serve replay o analisi dello stream con Kinesis Data Analytics.
- Lambda per il job di inferenza a 80ms: il profilo è esattamente il caso Lambda — event-driven, breve durata, spike imprevedibili. Il cold start (tipicamente 100-200ms per Python con dipendenze leggere) è accettabile perché la finestra è 30 secondi, non 30ms. Provisioned Concurrency se il cold start diventa critico.
- **Feature store temporale**: le feature degli ultimi 2 ore richiedono uno store che supporti query temporali veloci. DynamoDB con TTL di 2 ore per le feature; oppure Redis ElastiCache per latenza sub-millisecondo se il volume lo richiede. Kinesis Data Analytics (Apache Flink) può calcolare feature aggregate (media view time per categoria nelle ultime 2 ore) in modo continuo.

**Trappola comune**: usare un database SQL per le feature temporali. Query "tutte le feature dell'utente X nelle ultime 2 ore" su PostgreSQL con 500.000 inserimenti/ora è un problema di performance non banale. DynamoDB o Redis con strutture dati appropriate (TTL automatico) sono le scelte corrette.
</details>

---

## Riepilogo dei pattern

| Scenario | Pattern vincente | Trappola principale |
|---|---|---|
| A — SaaS B2B con profilo diurno | ECS Fargate + RDS | Lambda per sessioni utente sostenute |
| B — Batch notturno con fan-out | ECS Task + SQS | Job monolitico senza idempotenza |
| C — Microservizi con SLA stretto | EKS + HPA + X-Ray | Ignorare la tail latency amplification in cascata |
| D — Stream ML real-time | Kinesis + Lambda + DynamoDB TTL | SQL per feature store ad alta velocità |

## Prossima parte

Hai completato la Parte 2: compute, container, K8s, serverless, async, API design. La **Parte 3** apre il capitolo *Dati in movimento* — batch vs streaming, log distribuiti, event sourcing, CDC: una scelta di architettura dei dati che blocca se fatta tardi, da capire prima di passare all'IaC.
