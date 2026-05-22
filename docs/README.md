---
title: Guida AI — Inizia da qui
slug: /
sidebar_position: 1
---

# Guida AI — capire di cosa parlano tutti, ed esserne competitivi

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

### 1 — Costruire sistemi LLM
- [1.1 RAG](./building/rag.md) — scritta
- [1.2 Context engineering](./building/context-engineering.md) — scritta
- [1.3 Structured output e function calling](./building/structured-output.md) — scritta
- [1.4 Agenti e orchestrazione](./building/agenti.md) — scritta
- [1.5 MCP](./building/mcp.md) — scritta
- [1.6 Decision drill — Costruire](./building/drill.md) — scritta
- [1.7 Decision drill — Fine-tuning vs RAG vs PE vs context engineering](./building/drill-fine-tuning-vs-rag.md) — scritta

### 2 — Multimodale e altri tipi di AI
- [2.1 Come funziona il multimodale](./multimodal/come-funziona-multimodale.md) — scritta
- [2.2 Vision](./multimodal/vision.md) — scritta
- [2.3 Audio e speech](./multimodal/audio.md) — scritta
- [2.4 Generazione di immagini](./multimodal/generazione-immagini.md) — scritta
- [2.5 Multimodale vs pipeline separate](./multimodal/multimodale-vs-pipeline.md) — scritta
- [2.6 Decision drill — Multimodale](./multimodal/drill.md) — scritta

### 3 — Valutare e rendere affidabile
- [3.1 LLM-as-judge](./evaluation/llm-as-judge.md) — scritta
- [3.2 Observability](./evaluation/observability.md) — scritta
- [3.3 Gestire le allucinazioni](./evaluation/hallucination.md) — scritta
- [3.4 Valutare il comportamento agentico](./evaluation/valutare-agenti.md) — scritta
- [3.5 Decision drill — Valutazione](./evaluation/drill.md) — scritta

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

### 6 — Produzione (LLMOps)
- [6.1 Serving e inference](./production/serving.md) — scritta
- [6.2 Costi e FinOps a runtime](./production/costi-finops.md) — scritta
- [6.3 Monitoring e drift in produzione](./production/monitoring.md) — scritta
- [6.4 Decision drill — Produzione](./production/drill.md) — scritta

### 7 — Architettura e sintesi
- [7.1 Reference architecture](./architecture/reference-architecture.md) — da riscrivere
- [7.2 Build vs buy, proprietari vs open-weight](./architecture/build-vs-buy.md) — da riscrivere
- [Decision drill — Architettura](./architecture/drill.md) — da riscrivere
- [7.3 Capstone — progetto end-to-end](./capstone/progetto-finale.md) — da riscrivere

---

**Avanzamento:** 2 lezioni complete nella voce definitiva (0.1, 0.2). Tutte le altre sono scaffoldate ma da scrivere. Le pagine "da riscrivere" erano state buttate giù prima di definire il registro: vanno rifatte col gold standard.

**Per riprendere:** apri `AGENTS.md` (root del repo), fai leggere al modello la 0.1 e la 0.2 come riferimento di voce, poi scrivi 1-2 pagine per sessione seguendo il SYLLABUS.
