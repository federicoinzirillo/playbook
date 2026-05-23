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

> **Parti 0, 1, 2, 3, 4, 5, 6 e 7 scritte** (49 lezioni). Le lezioni della Parte 8 sono placeholder: il sidebar mostra la struttura completa del syllabus, ma il contenuto va scritto una lezione per volta. La struttura ordinata con i prerequisiti è in **[SYLLABUS](./SYLLABUS.md)**.

### 0 — Fondamenta
- [0.1 Cosa significa cloud](./foundations/cosa-significa-cloud.md) — scritta
- [0.2 IaaS, PaaS, SaaS](./foundations/iaas-paas-saas.md) — scritta
- [0.3 Le risorse fondamentali](./foundations/risorse-fondamentali.md) — scritta
- [0.4 Regioni, zone, geografia](./foundations/regioni-zone-geografia.md) — scritta
- [0.5 Persistenza dei dati: SQL, NoSQL, oggetti, vector](./foundations/data-layer.md) — scritta
- [0.6 Linux essenziale per cloud](./foundations/linux-essenziale.md) — scritta

### 1 — Networking & sicurezza
- [1.1 Networking nel cloud](./networking-security/networking.md) — scritta
- [1.2 DNS, TLS, Load Balancing, CDN](./networking-security/dns-tls-lb-cdn.md) — scritta
- [1.3 Identity e permessi (IAM concettuale)](./networking-security/identity-iam.md) — scritta
- [1.4 Segreti, chiavi, dati sensibili](./networking-security/segreti-chiavi.md) — scritta
- [1.5 Decision drill — Networking & accessi](./networking-security/drill.md) — scritta

### 2 — Compute & runtime
- [2.1 Container](./compute/container.md) — scritta
- [2.2 Orchestrazione e Kubernetes (concetti)](./compute/kubernetes-concetti.md) — scritta
- [2.3 VM vs Container vs Serverless](./compute/vm-container-serverless.md) — scritta
- [2.4 Async ed event-driven](./compute/async-event-driven.md) — scritta
- [2.5 API design per servizi cloud](./compute/api-design.md) — scritta
- [2.6 Dati in movimento](./compute/dati-movimento.md) — scritta
- [2.7 Decision drill — Dove far girare la mia app](./compute/drill.md) — scritta

### 3 — IaC & automazione
- [3.1 Infrastructure as Code](./iac-automation/iac-terraform.md) — scritta
- [3.2 Policy-as-code e governance dell'infra](./iac-automation/policy-as-code.md) — scritta
- [3.3 CI/CD nel cloud](./iac-automation/cicd.md) — scritta
- [3.4 Decision drill — Cosa automatizzare](./iac-automation/drill.md) — scritta

### 4 — AWS pratica
- [4.1 Orientarsi in AWS](./aws/orientarsi-aws.md) — scritta
- [4.2 Compute (EC2, ECS/Fargate, Lambda)](./aws/compute.md) — scritta
- [4.3 Messaggistica ed eventi (SQS, SNS, EventBridge, Step Functions)](./aws/messaggistica-eventi.md) — scritta
- [4.4 Storage e database](./aws/storage-database.md) — scritta
- [4.5 Networking su AWS](./aws/networking-aws.md) — scritta
- [4.6 IaC su AWS](./aws/iac-aws.md) — scritta
- [4.7 Observability (CloudWatch)](./aws/observability.md) — scritta
- [4.8 Decision drill — URL shortener su AWS](./aws/drill.md) — scritta

### 5 — Cloud per l'AI
- [5.1 GPU nel cloud](./cloud-for-ai/gpu-cloud.md) — scritta
- [5.2 Servizi AI gestiti vs self-hosted](./cloud-for-ai/gestiti-vs-self-hosted.md) — scritta
- [5.3 Ottimizzare i costi di inferenza](./cloud-for-ai/costi-inferenza.md) — scritta
- [5.4 Caching e CDN strategies](./cloud-for-ai/caching-strategie.md) — scritta
- [5.5 Kubernetes per workload AI](./cloud-for-ai/k8s-per-ai.md) — scritta
- [5.6 Architettura di un sistema AI in produzione](./cloud-for-ai/architettura-sistema-ai.md) — scritta
- [5.7 Vector database managed vs self-hosted](./cloud-for-ai/vector-db.md) — scritta
- [5.8 Training pipeline gestiti](./cloud-for-ai/training-pipeline.md) — scritta
- [5.9 Decision drill — Deploy RAG su AWS](./cloud-for-ai/drill.md) — scritta

### 6 — Operations
- [6.1 FinOps: il cloud costa in modi sorprendenti](./operations/finops.md) — scritta
- [6.2 Monitoring e incident response](./operations/monitoring-incident.md) — scritta
- [6.3 Distributed tracing e observability avanzata](./operations/distributed-tracing.md) — scritta
- [6.4 Sicurezza in produzione](./operations/sicurezza-produzione.md) — scritta
- [6.5 Resilienza e disaster recovery](./operations/resilienza-dr.md) — scritta
- [6.6 Compliance enterprise](./operations/compliance-enterprise.md) — scritta
- [6.7 Decision drill — I costi sono raddoppiati](./operations/drill.md) — scritta

### 7 — Sintesi
- [7.1 Reference architecture cloud](./capstone/reference-architecture-cloud.md) — scritta
- [7.2 Capstone — sistema end-to-end su AWS](./capstone/capstone.md) — scritta
- [7.3 Il portfolio che fa assumere](./capstone/portfolio.md) — scritta

### 8 — Architettura enterprise (traccia SA)
- [8.1 AWS Well-Architected Framework](./enterprise-architecture/well-architected.md) — bozza
- [8.2 Governance multi-account](./enterprise-architecture/governance-multiacccount.md) — bozza
- [8.3 Migrazione e modernizzazione](./enterprise-architecture/migrazione-modernizzazione.md) — bozza
- [8.4 Multi-cloud: consapevolezza senza dispersione](./enterprise-architecture/multicloud-awareness.md) — bozza
- [8.5 Decision drill — Landing zone enterprise](./enterprise-architecture/drill.md) — bozza

## Totale

**49 lezioni** distribuite su 9 parti. **49 scritte**, 5 placeholder (Parte 8). Da scrivere in ordine secondo i prerequisiti del [SYLLABUS](./SYLLABUS.md). La Parte 8 è opzionale — traccia per chi punta ad AI Cloud Solution Architect.
