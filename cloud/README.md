---
title: Indice Cloud
slug: /
sidebar_position: 1
---

# Indice — Cloud Playbook

Percorso di studio per passare da full-stack a **Cloud + AI engineer applicativo**. Stesso metodo della guida AI: organizzato per **problema da risolvere**, non per servizio (i servizi cambiano nome, i problemi restano). Strategia: prima i **concetti universali** (parti 0-3), poi **AWS** dove diventi spendibile (parti 4-7).

Va letto in alternanza con l'**[AI Playbook](pathname:///ai)**: il cloud è il pezzo che chiude la guida AI — un sistema AI gira *su* cloud, e i due percorsi convergono nel capstone finale.

## Come è fatta

Organizzata per problema, non per tecnologia. Per l'ordine di studio con i prerequisiti, vedi **[SYLLABUS](./SYLLABUS.md)**. Le regole di scrittura stanno in `AGENTS.md` alla root del repo (condiviso con l'AI Playbook).

## Come studiarla

Stesso metodo dell'AI Playbook: leggi una lezione, poi **chiudi la pagina e fai la "Verifica di comprensione" a memoria**; rivedi il giorno dopo le risposte incerte. Nel cloud i progetti contano ancora più che altrove — alterna sempre lezione + pratica.

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
- [2.6 Decision drill — Dove far girare la mia app](./compute/drill.md) — bozza

### 3 — IaC & automazione
- [3.1 Infrastructure as Code (Terraform)](./iac-automation/iac-terraform.md) — bozza
- [3.2 CI/CD nel cloud](./iac-automation/cicd.md) — bozza
- [3.3 Decision drill — Cosa automatizzare](./iac-automation/drill.md) — bozza

### 4 — AWS pratica
- [4.1 Orientarsi in AWS](./aws/orientarsi-aws.md) — bozza
- [4.2 Compute (EC2, ECS/Fargate, Lambda)](./aws/compute.md) — bozza
- [4.3 Storage e database](./aws/storage-database.md) — bozza
- [4.4 Networking su AWS](./aws/networking-aws.md) — bozza
- [4.5 IaC su AWS](./aws/iac-aws.md) — bozza
- [4.6 Observability (CloudWatch)](./aws/observability.md) — bozza
- [4.7 Decision drill — Architettura AWS serverless](./aws/drill.md) — bozza

### 5 — Cloud per l'AI
- [5.1 GPU nel cloud](./cloud-for-ai/gpu-cloud.md) — bozza
- [5.2 Servizi AI gestiti vs self-hosted](./cloud-for-ai/gestiti-vs-self-hosted.md) — bozza
- [5.3 Ottimizzare i costi di inferenza](./cloud-for-ai/costi-inferenza.md) — bozza
- [5.4 Caching e CDN strategies](./cloud-for-ai/caching-strategie.md) — bozza
- [5.5 Kubernetes per workload AI](./cloud-for-ai/k8s-per-ai.md) — bozza
- [5.6 Architettura di un sistema AI in produzione](./cloud-for-ai/architettura-sistema-ai.md) — bozza
- [5.7 Decision drill — Deploy RAG su AWS](./cloud-for-ai/drill.md) — bozza

### 6 — Operations
- [6.1 FinOps: il cloud costa in modi sorprendenti](./operations/finops.md) — bozza
- [6.2 Monitoring e incident response](./operations/monitoring-incident.md) — bozza
- [6.3 Distributed tracing e observability avanzata](./operations/distributed-tracing.md) — bozza
- [6.4 Sicurezza in produzione](./operations/sicurezza-produzione.md) — bozza
- [6.5 Decision drill — I costi sono raddoppiati](./operations/drill.md) — bozza

### 7 — Sintesi
- [7.1 Reference architecture cloud](./capstone/reference-architecture-cloud.md) — bozza
- [7.2 Capstone — sistema end-to-end su AWS](./capstone/capstone.md) — bozza
- [7.3 Il portfolio che fa assumere](./capstone/portfolio.md) — bozza

## Totale

**42 lezioni** distribuite su 8 parti. Da scrivere in ordine secondo i prerequisiti del [SYLLABUS](./SYLLABUS.md).
