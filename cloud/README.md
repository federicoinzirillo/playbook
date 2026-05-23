---
title: Indice Cloud
slug: /
sidebar_position: 1
---

# Indice — Cloud Playbook

Guida di studio sul cloud applicativo moderno. Organizzata per **problema da risolvere**, non per servizio: i servizi cambiano nome, i problemi restano.

Va letto in parallelo con l'**[AI Playbook](pathname:///ai)** — stessa voce, stessa struttura.

## Come è fatta

Per l'ordine di studio con i prerequisiti, vedi **[SYLLABUS](./SYLLABUS.md)**. Le regole di scrittura stanno in `AGENTS.md` alla root del repo.

## Come studiarla

Leggi una lezione, poi **chiudi la pagina e fai la "Verifica di comprensione" a memoria**; rivedi il giorno dopo le risposte incerte. È lo sforzo di recuperare a memoria che fa imparare davvero.

## Stato delle pagine

> Tutte le pagine sono attualmente **bozze placeholder**: il sidebar mostra la struttura completa del syllabus, ma il contenuto va scritto una lezione per volta. La struttura ordinata con i prerequisiti è in **[SYLLABUS](./SYLLABUS.md)**.

### 0 — Fondamenta
- [0.1 Cosa significa cloud](./foundations/cosa-significa-cloud.md) — bozza
- [0.2 IaaS, PaaS, SaaS](./foundations/iaas-paas-saas.md) — bozza
- [0.3 Le risorse fondamentali](./foundations/risorse-fondamentali.md) — bozza
- [0.4 Regioni, zone, geografia](./foundations/regioni-zone-geografia.md) — bozza
- [0.5 Persistenza dei dati: SQL, NoSQL, oggetti, vector](./foundations/data-layer.md) — bozza
- [0.6 Linux essenziale per cloud](./foundations/linux-essenziale.md) — bozza

### 1 — Networking & sicurezza
- [1.1 Networking nel cloud](./networking-security/networking.md) — bozza
- [1.2 DNS, TLS, Load Balancing, CDN](./networking-security/dns-tls-lb-cdn.md) — bozza
- [1.3 Identity e permessi (IAM concettuale)](./networking-security/identity-iam.md) — bozza
- [1.4 Segreti, chiavi, dati sensibili](./networking-security/segreti-chiavi.md) — bozza
- [1.5 Decision drill — Networking & accessi](./networking-security/drill.md) — bozza

### 2 — Compute & runtime
- [2.1 Container](./compute/container.md) — bozza
- [2.2 Orchestrazione e Kubernetes (concetti)](./compute/kubernetes-concetti.md) — bozza
- [2.3 VM vs Container vs Serverless](./compute/vm-container-serverless.md) — bozza
- [2.4 Async ed event-driven](./compute/async-event-driven.md) — bozza
- [2.5 API design per servizi cloud](./compute/api-design.md) — bozza
- [2.6 Dati in movimento — batch, streaming, CDC](./compute/dati-movimento.md) — bozza
- [2.7 Decision drill — Dove far girare la mia app](./compute/drill.md) — bozza

### 3 — IaC & automazione
- [3.1 Infrastructure as Code (Terraform)](./iac-automation/iac-terraform.md) — bozza
- [3.2 Policy-as-code e governance dell'infra](./iac-automation/policy-as-code.md) — bozza
- [3.3 CI/CD nel cloud](./iac-automation/cicd.md) — bozza
- [3.4 Decision drill — Cosa automatizzare](./iac-automation/drill.md) — bozza

### 4 — AWS pratica
- [4.1 Orientarsi in AWS](./aws/orientarsi-aws.md) — bozza
- [4.2 Compute (EC2, ECS/Fargate, Lambda)](./aws/compute.md) — bozza
- [4.3 Messaggistica ed eventi (SQS, SNS, EventBridge, Step Functions)](./aws/messaggistica-eventi.md) — bozza
- [4.4 Storage e database](./aws/storage-database.md) — bozza
- [4.5 Networking su AWS](./aws/networking-aws.md) — bozza
- [4.6 IaC su AWS](./aws/iac-aws.md) — bozza
- [4.7 Observability (CloudWatch)](./aws/observability.md) — bozza
- [4.8 Decision drill — Architettura AWS serverless](./aws/drill.md) — bozza

### 5 — Cloud per l'AI
- [5.1 GPU nel cloud](./cloud-for-ai/gpu-cloud.md) — bozza
- [5.2 Servizi AI gestiti vs self-hosted](./cloud-for-ai/gestiti-vs-self-hosted.md) — bozza
- [5.3 Ottimizzare i costi di inferenza](./cloud-for-ai/costi-inferenza.md) — bozza
- [5.4 Caching e CDN strategies](./cloud-for-ai/caching-strategie.md) — bozza
- [5.5 Kubernetes per workload AI](./cloud-for-ai/k8s-per-ai.md) — bozza
- [5.6 Architettura di un sistema AI in produzione](./cloud-for-ai/architettura-sistema-ai.md) — bozza
- [5.7 Vector database managed vs self-hosted](./cloud-for-ai/vector-db.md) — bozza
- [5.8 Training pipeline gestiti](./cloud-for-ai/training-pipeline.md) — bozza
- [5.9 Decision drill — Deploy RAG su AWS](./cloud-for-ai/drill.md) — bozza

### 6 — Operations
- [6.1 FinOps: il cloud costa in modi sorprendenti](./operations/finops.md) — bozza
- [6.2 Monitoring e incident response](./operations/monitoring-incident.md) — bozza
- [6.3 Distributed tracing e observability avanzata](./operations/distributed-tracing.md) — bozza
- [6.4 Sicurezza in produzione](./operations/sicurezza-produzione.md) — bozza
- [6.5 Resilienza e disaster recovery](./operations/resilienza-dr.md) — bozza
- [6.6 Compliance enterprise](./operations/compliance-enterprise.md) — bozza
- [6.7 Decision drill — I costi sono raddoppiati](./operations/drill.md) — bozza

### 7 — Sintesi
- [7.1 Reference architecture cloud](./capstone/reference-architecture-cloud.md) — bozza
- [7.2 Capstone — sistema end-to-end su AWS](./capstone/capstone.md) — bozza
- [7.3 Il portfolio che fa assumere](./capstone/portfolio.md) — bozza

## Totale

**44 lezioni** distribuite su 8 parti. Da scrivere in ordine secondo i prerequisiti del [SYLLABUS](./SYLLABUS.md).
