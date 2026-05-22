---
title: Radar
sidebar_position: 3
---

# Radar — cosa sta cambiando

> Non un news feed. Traccia solo i **cambiamenti strutturali**: quando qualcosa passa da curiosità a standard de facto e merita una pagina della guida.

Il principio è quello del SYLLABUS, sezione "Tenere viva la guida": un campo che si muove in fretta richiede una valvola di sfogo. Il Radar è quel posto. Si rivede ~una volta al mese.

**Stati:**
- `EMERGENTE` — promettente, troppo presto per una pagina. Aspetto.
- `IN ADOZIONE` — sta diventando comune. Da seguire.
- `STANDARD` — è il modo in cui si fanno le cose. Esce dal radar e diventa (o aggiorna) una pagina.
- `LEGACY` — citato solo per contesto storico. Esce dal radar verso il basso.

Aggiornamento: maggio 2026.

---

## EMERGENTE

### Diffusion language models
*(es. LLaDA, Mercury)* — modelli di linguaggio basati su diffusion invece che autoregressivi. Producono testo in modo non-sequenziale, potenzialmente molto più veloce. Stato 2026: paper interessanti, qualche prototipo, nessun deployment serio. **Quando crei una pagina:** se uno di questi modelli supera la qualità autoregressiva su task standard.

### Test-time training
Adattamento dei pesi del modello durante l'inference, su pochi esempi, per task specifici. Diversa dalla fine-tuning classica. **Quando crei una pagina:** se diventa una funzionalità esposta dai provider o un pattern adottato in framework agentici.

### Confidential computing per LLM inference
Esecuzione di modelli su enclave hardware (Intel TDX, AMD SEV-SNP, Nvidia Confidential Computing) per garanzia crittografica che il provider non possa vedere i dati. **Quando crei una pagina:** quando i provider major lo offrono come SKU mainstream e i clienti enterprise lo richiedono. Oggi ancora di nicchia (qualche offerta Azure, OCI, GCP, ma piccola).

### Embedding inversion attacks
Tecniche per ricostruire il testo originale da un embedding. Ricerca attiva nel 2025-26. **Quando crei una pagina:** non una pagina propria, ma probabilmente un aggiornamento alla **4.1** se gli attacchi diventano pratici contro vector store esposti.

---

## IN ADOZIONE

### Computer-use agents / browser agents
Agenti che operano direttamente su browser e desktop tramite screenshot + click coordinates. GPT-5.4 native computer use (apr 2026), Claude Mythos Preview (top di WebArena), Google Project Mariner. Cambia la natura dei tool nei sistemi agentici. **Quando crei una pagina:** è candidato forte per una nuova lezione 1.x o un'espansione della 1.4. Probabilmente promosso a STANDARD entro fine 2026.

### Memory layers per agenti
Architetture separate per "memoria a lungo termine" dell'agente (vector DB + retrieval su storia personale dell'utente, fatti appresi tra sessioni). MemGPT, Letta, le prime primitive native nei framework. **Quando crei una pagina:** se entra come pattern standard nei framework principali. Già menzionato di striscio in **1.2** e **3.4**, andrebbe espanso.

### Multi-agent A2A protocols
Comunicazione standardizzata agente-agente, non solo client-server. La roadmap MCP 2026 lo mette tra i tre fronti principali (RC spec 28 lug 2026). **Quando crei una pagina:** quando MCP A2A è ratificato e i primi sistemi multi-agent reali lo usano. Aggiornare **1.5**.

### Inference-time reasoning come scelta architetturale
La distinzione "modello istantaneo" vs "modello che pensa" (Claude extended thinking, GPT-5 Thinking, Gemini deep reasoning) è ora una decisione di design esplicita. **Quando crei una pagina:** già coperta di striscio in **5.3** (triangolo qualità-latenza-costo); merita un blocco dedicato o un'espansione.

### Sovrane EU / modelli europei
Mistral, Aleph Alpha, e altri si presentano come alternativa "data residency garantita per design". Rilevante con EU AI Act + Digital Omnibus + crescente pressione politica per AI sovrane. **Quando crei una pagina:** se diventano scelta default per il pubblico EU. Aggiornare **4.3** e **5.1** se sì.

### Vector DB embedded in standard DBs
pgvector è il caso di studio: il vector store come estensione del DB esistente sta vincendo per progetti piccoli e medi. Altri DB stanno aggiungendo capacità vettoriali (MongoDB, MySQL, SQLite via sqlite-vec). **Quando crei una pagina:** già menzionato in **1.1** e **5.4**. Aggiornare se sostituisce il vector DB dedicato come pattern dominante.

### LanceDB
Crescita ripida tra i vector DB 2025-26 grazie a serverless + multimodale nativo. **Quando crei una pagina:** se entra nella top 3 di adozione, da aggiungere alla lista in **1.1** e **5.4**.

---

## STANDARD (promossi — già nella guida)

| Tema | Quando promosso | Dove vive |
|---|---|---|
| **MCP — Model Context Protocol** | Dic 2025 (donazione Linux Foundation) | [1.5](./building/mcp.md) |
| **OWASP LLM Top 10 v2025** | Fine 2024 (pubblicato), 2026 (canonico) | [4.1](./security/prompt-injection.md) |
| **EU AI Act + Digital Omnibus** | Mag 2026 (accordo Council-Parliament) | [4.4](./security/eu-ai-act.md) |
| **Hybrid search (dense + sparse)** | 2024-25 | [1.1](./building/rag.md) |
| **GraphRAG production-ready** | 2025-26 | [1.1](./building/rag.md) |
| **GPT Image 2 (rimpiazza DALL-E 3)** | Apr 2026 | [2.4](./multimodal/generazione-immagini.md) |
| **Whisper large-v3-turbo** | 2024-25 (de facto), 2026 (mainstream) | [2.3](./multimodal/audio.md) |
| **Continuous batching + PagedAttention** | 2024 | [6.1](./production/serving.md) |
| **PromptArmor / PromptGuard / defense-in-depth** | ICLR 2026 + consenso NCSC | [4.1](./security/prompt-injection.md), [4.2](./security/sicurezza-agenti.md) |
| **Terminal-Bench, OSWorld, τ-Bench** | 2025-26 (eval agentica) | [3.4](./evaluation/valutare-agenti.md) |

---

## LEGACY (citato solo per contesto storico)

| Tema | Stato | Note |
|---|---|---|
| **DALL-E 2 / DALL-E 3** | API sunset maggio 2026 | Rimpiazzati da GPT Image 2 |
| **GPT-4 family (4o, 4.1, 4.5)** | Retirati febbraio 2026 | Sostituiti dalla famiglia GPT-5 |
| **o-series (o1, o3, o3-pro, o4-mini)** | Consolidati nella famiglia GPT-5 | Inference-time reasoning ora trasversale |
| **Claude 3 (Opus 3, Sonnet 3.5)** | Sostituiti dalla famiglia Claude 4.x | |
| **Gemini 1.5 / 2.x** | Sostituiti da Gemini 3.x | Gemini Ultra brand abbandonato |
| **OWASP LLM Top 10 v1.1 (2023)** | Sostituita dalla v2025 | Rinumerazione + 2 nuove categorie |
| **EU AI Act timeline pre-Omnibus** | Aug 2026 high-risk → Dec 2027 | Rinvio del 7 mag 2026 |

---

## Come si usa questo file (per me, in futuro)

- **Ogni 4-6 settimane**, riapro e mi chiedo: qualcosa qui sopra è cambiato di stato? Posso scrivere il follow-up del passaggio EMERGENTE → IN ADOZIONE → STANDARD?
- **Quando promuovo a STANDARD**, scrivo o aggiorno la pagina della guida, sposto la riga nella tabella STANDARD del Radar, e (se serve) sposto qualcosa di vecchio in LEGACY.
- **Non aggiungere news qui**. Il Radar non è un blog del settore. Va aggiunto solo ciò che, secondo *giudizio mio*, potrebbe meritare una pagina nei prossimi 6-12 mesi. Se passa anno e nulla è cambiato, va tolto.
- **Tre stati massimi per voce in EMERGENTE / IN ADOZIONE**. Se ho 15 cose "in adozione", non sto facendo il radar — sto facendo l'elenco. La selezione è il valore.
