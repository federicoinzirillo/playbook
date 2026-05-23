---
title: Reference architecture cloud
sidebar_label: "8.1 Reference architecture"
sidebar_position: 1
---

# Reference architecture cloud

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 8.1</span>
  <span>~12 min di lettura</span>
</div>

<p class="lesson-lead">Hai studiato i mattoni separatamente — VPC, Lambda, RDS, SQS, IAM, CloudWatch. Questa lezione li assembla in un sistema coerente e ti insegna a leggere, valutare e adattare qualsiasi architettura di riferimento.</p>

Hai toccato ogni pezzo nelle lezioni precedenti. Il networking in Parte 1, il compute in Parte 2, i dati in movimento in Parte 3, l'IaC in Parte 4, i servizi AWS in Parte 5, il cloud per l'AI in Parte 6, le operations in Parte 7. Ogni lezione ti ha dato un mattone. Adesso è il momento di vederli assemblati.

**Una reference architecture** è un blueprint collaudato che distilla decisioni già prese, trappole già toccate e pattern che funzionano su larga scala. Non è un dogma: è il punto di partenza di una conversazione. Un senior che disegna un sistema su una lavagna bianca non parte da zero — parte dalla reference che conosce, poi la piega ai vincoli reali del problema.

## I sette livelli del sistema

Prima del diagramma, il racconto. Un'applicazione che serve richieste utente — con un backend AI — si struttura invariabilmente intorno a sette livelli.

**Edge**: il traffico arriva dal client, attraversa **CloudFront** — la CDN (*Content Delivery Network*) di AWS che distribuisce il contenuto statico dai nodi più vicini all'utente — e **WAF** (*Web Application Firewall*), che blocca richieste malformate, bot e gli attacchi OWASP prima che raggiungano il backend. Questo livello abbassa la latenza percepita e alleggerisce il carico sui servizi interni.

**API tier**: il traffico che supera l'edge arriva a un **ALB** (*Application Load Balancer*, che distribuisce il traffico verso più istanze) oppure a **API Gateway** per gli endpoint serverless. Qui si termina il TLS (*Transport Layer Security*, cifratura del canale), si gestisce l'autenticazione (Cognito, JWT), si fa rate limiting e si fa routing verso i servizi downstream.

**Compute tier**: le richieste sono gestite da **Lambda** per i workload event-driven a vita breve, e da **ECS/Fargate** per i servizi long-running, o che richiedono più RAM e CPU controllata. I due convivono nello stesso sistema: Lambda gestisce le API leggere, ECS gestisce il retrieval e l'inference. Il confine si traccia sul bisogno, non sull'abitudine.

**Data tier**: **RDS** (*Relational Database Service*) per i dati relazionali con query complesse; **DynamoDB** per key-value ad alta velocità e scaling automatico; **S3** per oggetti, documenti e artefatti binari. **ElastiCache** davanti a RDS o come cache di risposte frequenti: abbassa la latenza di un fattore 10 sulle query calde.

**Async tier**: **SQS** (*Simple Queue Service*) disaccoppia il producer dal consumer. Quando una richiesta avvia un job pesante — indicizzazione, embedding, invio email, post-processing — viene messa in coda invece di bloccare il thread. Il worker su ECS o Lambda consuma la coda al proprio ritmo, con retry automatici e DLQ (*Dead Letter Queue*) per i messaggi irrecuperabili.

**Sicurezza trasversale**: **IAM** (*Identity and Access Management*) con ruoli per ogni servizio, nessuna access key statica nel codice. **KMS** (*Key Management Service*) per la cifratura at-rest dei dati sensibili. **GuardDuty** per la threat detection automatica, **Security Hub** per l'aggregazione dei risultati.

**Observability trasversale**: **CloudWatch** raccoglie metriche, log e alarm da ogni componente. **X-Ray** traccia le richieste end-to-end attraverso Lambda, ECS e API Gateway. **Cost Explorer** e **Budgets** vigilano sulla spesa.

## Il diagramma

```mermaid
architecture-beta
  group aws(logos:aws)[AWS]

  service users(internet)[Utenti]
  service cf(logos:aws-cloudfront)[CloudFront + WAF] in aws
  service apigw(logos:aws-api-gateway)[API Gateway] in aws
  service lambda(logos:aws-lambda)[Lambda] in aws
  service ecs(logos:aws-ecs)[ECS / Fargate] in aws
  service ddb(logos:aws-dynamodb)[DynamoDB] in aws
  service rds(logos:aws-rds)[RDS] in aws
  service s3(logos:aws-s3)[S3] in aws
  service sqs(logos:aws-sqs)[SQS] in aws
  service cw(logos:aws-cloudwatch)[CloudWatch + X-Ray] in aws

  users:R --> L:cf
  cf:R --> L:apigw
  apigw:B --> T:lambda
  apigw:R --> L:ecs
  lambda:B --> T:ddb
  ecs:B --> T:rds
  ecs:R --> L:s3
  lambda:R --> L:sqs
  sqs:B --> T:ecs
  lambda:T --> B:cw
  ecs:T --> B:cw
```

*Flusso: utente → CloudFront → API Gateway → Lambda (workload leggeri) o ECS (workload pesanti, AI) → data tier (DynamoDB, RDS, S3). SQS disaccoppia i job asincroni. CloudWatch raccoglie segnali da ogni strato.*

## Le decisioni strutturali

Alcune scelte architetturali sono **difficili da cambiare a sistema in produzione**. Il cosiddetto debito architetturale si accumula qui, e si paga con gli interessi.

**Sincrono vs asincrono**: se metti tutto su Lambda sincrono, stai scommettendo che le dipendenze downstream siano sempre veloci e disponibili. Un database lento o un servizio esterno in timeout fa aspettare ogni richiesta utente. La regola pratica: **tutto ciò che dura più di ~200 ms o non richiede risposta immediata va su SQS**. Non è ottimizzazione — è robustezza di base.

**Database per il workload giusto**: il pattern "un solo RDS per tutto" funziona su piccola scala, poi soffoca. DynamoDB per sessioni utente e dati key-value è quasi sempre la scelta giusta: scaling automatico, nessuna connessione da gestire, latenza predicibile. RDS per dati relazionali, transazioni ACID, query analitiche complesse. Cambiare database con dati in produzione è tra le operazioni più costose che esistano.

**IAM least privilege dalla prima riga**: non è un'operazione da fare "dopo il lancio". Una Lambda che deve leggere un bucket S3 specifico deve avere **solo** quel permesso, nient'altro. Il costo di farlo bene è zero; il costo di farlo male è una breach o un'audit fallita.

**Multi-AZ per il data tier**: RDS ed ElastiCache devono essere Multi-AZ fin dall'inizio. Il costo è il doppio dell'istanza; il costo di un outage non pianificato — dati inaccessibili, rollback di emergenza, post-mortem — è molto di più.

## Come adattare la reference ai vincoli reali

Una reference architecture non risponde mai esattamente al tuo caso. I vincoli che cambiano le carte più spesso:

**Budget piccolo**: Lambda + API Gateway + DynamoDB è il set-up a quasi zero quando non c'è traffico. Niente ECS, niente RDS, niente ALB. Scala automaticamente, ma ha cold start e limiti di runtime.

**Traffico uniforme alto**: Lambda conviene meno di ECS/Fargate quando le richieste sono continue e prevedibili. Il costo per richiesta di Lambda su traffico sostenuto sorpassa il costo orario di un container.

**Dati sensibili (HIPAA, GDPR)**: S3 con Object Lock e cifratura KMS, RDS con encryption at-rest abilitata, CloudTrail su tutto con retention adeguata, nessun dato sensibile nei log di CloudWatch. Il compliance si costruisce nell'architettura, non si applica sopra.

**Sistema AI intensivo**: se l'inference è il collo di bottiglia, ECS su istanze GPU (famiglia `g5`, `inf2`) o AWS Bedrock per i modelli Foundation. Un LLM gateway (LiteLLM, Portkey) davanti al provider gestisce routing multi-modello, caching semantico e budget enforcement senza dover toccare ogni singola Lambda.

## Cosa non è

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "La reference architecture è il punto di arrivo, basta copiarla" | È il punto di partenza. Ogni componente che aggiungi o togli ha una motivazione. Il valore è capire il perché, non copiare il cosa. |
| "Lambda è sempre più economica di ECS/Fargate" | Su traffico intermittente, sì. Su traffico sostenuto e prevedibile, Lambda costa di più e ha cold start. La scelta dipende dal profilo di traffico reale. |
| "Il database lo scelgo dopo, si può sempre migrare" | Migrare un database con dati in produzione è costoso, rischioso e richiede downtime. Si sceglie prima, ragionando sul modello di accesso dei dati. |
| "La sicurezza si aggiunge a fine sviluppo" | IAM, cifratura e logging non si aggiungono — si progettano dentro. Farlo retroattivamente significa toccare ogni risorsa, ogni policy, ogni deploy pipeline. |

## Verifica di comprensione

1. Qual è la differenza di ruolo tra Lambda e ECS/Fargate nella stessa architettura? Quando usi l'uno e quando l'altro?
2. Perché mettere SQS tra due servizi invece di una chiamata sincrona diretta?
3. Cosa succede a un'istanza RDS single-AZ se l'AZ ha un problema?
4. Elenca i sette livelli di un'architettura AWS standard e il ruolo di ciascuno.
5. Perché IAM least privilege si imposta all'inizio, non "dopo"?
6. In quale scenario conviene API Gateway + Lambda rispetto ad ALB + ECS?
7. Cosa fa CloudFront rispetto ad API Gateway? Sono alternativi o complementari?

## Glossario della pagina

- **Reference architecture**: blueprint di sistema basato su pattern collaudati; punto di partenza da adattare, non vincolo rigido.
- **ALB** — *Application Load Balancer*: distribuisce il traffico HTTP/HTTPS verso più target in base a regole di routing.
- **CDN** — *Content Delivery Network*: rete di nodi distribuiti che servono contenuto statico dalla posizione geograficamente più vicina all'utente.
- **WAF** — *Web Application Firewall*: blocca richieste malformate, bot e attacchi OWASP prima che raggiungano il backend.
- **ElastiCache**: servizio di cache in-memory gestito da AWS (Redis o Memcached).
- **KMS** — *Key Management Service*: servizio AWS per la creazione e gestione delle chiavi di cifratura.
- **DLQ** — *Dead Letter Queue*: coda SQS di fallback per i messaggi che non riescono a essere processati dopo N tentativi.
- **Multi-AZ**: configurazione in cui un servizio mantiene risorse ridondanti in almeno due Availability Zone distinte.
- **LLM gateway**: componente proxy davanti a uno o più provider LLM che gestisce routing, caching semantico, budget e logging centralizzato.

## Per approfondire

- **AWS Well-Architected Framework** (`docs.aws.amazon.com`): i sei pilastri per valutare architetture AWS — reliability, security, performance efficiency, cost optimization, operational excellence, sustainability. Include il Responsible AI Lens del 2025.
- **AWS Architecture Center** (`aws.amazon.com/architecture`): reference architecture ufficiali per decine di workload, con diagrammi scaricabili e note sulle scelte.
- **AWS Solutions Library** (`aws.amazon.com/solutions`): implementazioni di reference architecture con codice IaC pronto all'uso.
- **re:Invent** — cerca le sessioni "Building scalable web applications on AWS" e "Serverless reference architectures": i SA AWS spiegano direttamente le scelte dietro i pattern più usati.

## Prossima lezione

La reference architecture ti ha dato il quadro mentale. La lezione successiva ti fa **costruire il sistema per davvero** — dalla Lambda vuota fino a un RAG assistant deployato su AWS con Terraform, CI/CD e monitoring attivo. Il capstone chiude il cerchio tra i due playbook.
