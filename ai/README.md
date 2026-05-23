---
title: Indice AI
slug: /
sidebar_position: 1
---

# Indice — AI Playbook

Guida di studio sull'AI applicativa moderna. Niente laurea richiesta: questo campo premia chi capisce i concetti e i trade-off, non chi ha un titolo. L'obiettivo è portarti dal "sento parole che non capisco" al "so di cosa si parla, so decidere, so costruire l'essenziale".

Copre l'**AI moderna applicativa in senso generalista**: LLM, multimodale (immagini, audio, video), system design dei sistemi AI, valutazione, sicurezza, produzione. Fuori scope: addestrare modelli da zero, matematica pesante, cloud/infrastruttura pura.

## Come è fatta

Organizzata per **problema da risolvere**, non per tecnologia (i tool muoiono in fretta; i problemi restano). Per l'ordine di studio con i prerequisiti, vedi **[SYLLABUS.md](./SYLLABUS.md)**. Per scriverla/estenderla nel tempo, le regole stanno in `AGENTS.md` alla root del repo.

## Come studiarla

Leggere non basta e dà una falsa sicurezza. Il metodo: leggi una lezione, poi **chiudi la pagina e fai la "Verifica di comprensione" a memoria**; rivedi il giorno dopo le risposte incerte. È lo sforzo di recuperare a memoria che fa imparare davvero.

## Stato delle pagine

> La struttura completa e ordinata (con prerequisiti) è in **[SYLLABUS.md](./SYLLABUS.md)**. Qui sotto la mappa con lo stato: scritte nella voce definitiva o ancora bozza. Le bozze sono già scaffoldate (file con frontmatter, posizione e scope), pronte da riempire una alla volta.

### 0 — Fondamenta concettuali
- [0.1 Come funziona un LLM](./foundations/come-funziona-un-llm.md) — scritta
- [0.2 Embedding e spazi vettoriali](./foundations/embedding.md) — scritta
- [0.3 Concetti ML che servono a chiunque](./foundations/concetti-ml.md) — scritta
- [0.4 LLM vs ML classico: quando NON usare la GenAI](./foundations/llm-vs-ml-classico.md) — scritta
- [0.5 Prompt engineering come disciplina](./foundations/prompt-engineering.md) — scritta
- [0.6 Reasoning models — quando il pensiero costa, e quando vale](./foundations/reasoning-models.md) — scritta

### 1 — Costruire sistemi LLM
- [1.1 RAG — il nucleo](./building/rag.md) — scritta
- [1.2 Retrieval avanzato](./building/retrieval-avanzato.md) — bozza
- [1.3 Context engineering](./building/context-engineering.md) — scritta
- [1.4 Structured output e function calling](./building/structured-output.md) — scritta
- [1.5 Agenti semplici — tool calling e ReAct](./building/agenti.md) — scritta
- [1.6 Orchestrazione multi-agent](./building/agenti-orchestrazione.md) — bozza
- [1.7 MCP](./building/mcp.md) — scritta
- [1.8 Decision drill — Costruire](./building/drill.md) — scritta
- [1.9 Fine-tuning operativo — LoRA, QLoRA, DPO](./building/fine-tuning-operativo.md) — scritta
- [1.10 Decision drill — Fine-tuning vs RAG vs PE vs context engineering](./building/drill-fine-tuning-vs-rag.md) — scritta

### 2 — Multimodale e altri tipi di AI
- [2.1 Come funziona il multimodale](./multimodal/come-funziona-multimodale.md) — scritta
- [2.2 Vision](./multimodal/vision.md) — scritta
- [2.3 Multimodal RAG](./multimodal/multimodal-rag.md) — da scrivere
- [2.4 Audio e speech](./multimodal/audio.md) — scritta
- [2.5 Generazione di immagini](./multimodal/generazione-immagini.md) — scritta
- [2.6 Multimodale vs pipeline separate](./multimodal/multimodale-vs-pipeline.md) — scritta
- [2.7 Decision drill — Multimodale](./multimodal/drill.md) — scritta
- [2.8 Voice agents in tempo reale](./multimodal/voice-agents.md) — scritta
- [2.9 Video generation](./multimodal/video-generation.md) — bozza

### 3 — Valutare e rendere affidabile
- [3.1 Eval benchmarks e dataset — il vocabolario](./evaluation/eval-benchmarks.md) — scritta
- [3.2 LLM-as-judge](./evaluation/llm-as-judge.md) — scritta
- [3.3 Observability](./evaluation/observability.md) — scritta
- [3.4 Gestire le allucinazioni](./evaluation/hallucination.md) — scritta
- [3.5 Valutare il comportamento agentico](./evaluation/valutare-agenti.md) — scritta
- [3.6 Decision drill — Valutazione](./evaluation/drill.md) — scritta

### 4 — Sicurezza, privacy e governance
- [4.1 Prompt injection e OWASP LLM Top 10](./security/prompt-injection.md) — scritta
- [4.2 Sicurezza agentica](./security/sicurezza-agenti.md) — scritta
- [4.3 Privacy e data residency](./security/privacy.md) — scritta
- [4.4 EU AI Act e governance](./security/eu-ai-act.md) — scritta
- [4.5 Decision drill — Sicurezza e privacy](./security/drill.md) — scritta

### 5 — System design dei sistemi AI
- [5.1 Anatomia di un sistema AI](./system-design/anatomia.md) — scritta
- [5.2 Pattern di sistema](./system-design/pattern.md) — scritta
- [5.3 Il triangolo qualità-latenza-costo](./system-design/triangolo.md) — scritta
- [5.4 I dati come spina dorsale](./system-design/dati.md) — scritta
- [5.5 Decision drill — System design](./system-design/drill.md) — scritta
- [5.6 Caching semantico e model routing](./system-design/caching-routing.md) — scritta

### 6 — Produzione (LLMOps)
- [6.1 Serving e inference](./production/serving.md) — scritta
- [6.2 Costi e FinOps a runtime](./production/costi-finops.md) — scritta
- [6.3 Monitoring e drift in produzione](./production/monitoring.md) — scritta
- [6.4 Rollout sicuro: canary, A/B e shadow traffic per LLM](./production/rollout-canary-ab.md) — bozza
- [6.5 Decision drill — Produzione](./production/drill.md) — scritta

### 7 — Architettura e sintesi
- [7.1 Reference architecture](./architecture/reference-architecture.md) — scritta
- [7.2 Build vs buy, proprietari vs open-weight](./architecture/build-vs-buy.md) — scritta
- [7.3 Decision drill — Architettura](./architecture/drill.md) — scritta
- [7.4 Capstone — progetto end-to-end](./capstone/progetto-finale.md) — scritta

---

**Avanzamento:** 46 lezioni scritte nella voce definitiva, 5 in bozza (1.2, 1.6, 2.3, 2.9, 6.4). Totale: 51 lezioni. La guida riflette lo stato del settore al maggio 2026 — modelli (Claude 4.x, GPT-5.x, Gemini 3.x), OWASP LLM Top 10 v2025, EU AI Act post-Digital Omnibus, MCP donato alla Linux Foundation. Tracking dei cambiamenti strutturali in **[RADAR.md](./RADAR.md)**.

**Per riprendere:** apri `AGENTS.md` (root del repo), fai leggere al modello la 0.1 e la 0.2 come riferimento di voce, poi scrivi 1-2 pagine per sessione seguendo il SYLLABUS.
