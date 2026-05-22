---
title: Build vs buy, proprietari vs open-weight
sidebar_position: 2
---

# Build vs buy, proprietari vs open-weight

<div class="lesson-meta">
  <span class="badge-stato evoluzione">In evoluzione</span>
  <span>Lezione 7.2</span>
  <span>~13 min di lettura</span>
</div>

<p class="lesson-lead">Due decisioni che attraversano ogni progetto AI serio: cosa costruire in casa e cosa comprare; quando un modello proprietario chiuso (GPT, Claude, Gemini) vince e quando un open-weight (Llama, Mistral, Qwen) batte la mano. Non ci sono risposte universali — ci sono criteri di decisione che, applicati bene, ti fanno scegliere oggi senza pentirti tra sei mesi.</p>

Queste due decisioni vengono spesso discusse come fossero ideologiche: "noi facciamo tutto in casa" oppure "noi compriamo tutto". Entrambe sono posizioni sbagliate. La realtà è che ogni componente della reference architecture (lezione 7.1) ha la sua risposta, e la risposta giusta dipende da: maturità del componente sul mercato, criticità per il tuo business, controllo richiesto, costo a volume atteso, competenze del team. Cinque criteri, non uno.

## Build vs buy — criterio per criterio

**1. Maturità del componente sul mercato.** Più il componente è commoditizzato, più "buy" diventa la risposta giusta. LLM gateway? Esistono cinque soluzioni mature, comprare. Vector store? Stesso discorso. Pipeline di ingestion documenti? Più frammentato, dipende. Quando esiste una soluzione mainstream usata da migliaia di aziende, ricostruirla è quasi sempre uno spreco — a meno che il prossimo criterio non lo richieda.

**2. Criticità e differenziazione per il tuo business.** Se il componente è quello che ti distingue dai concorrenti, lo costruisci. Se è infrastruttura, lo compri. Il prompt di sistema che codifica il tuo dominio? Build, è il tuo know-how. La pipeline di evaluation specifica per il tuo caso d'uso? Build, perché definisce cosa "qualità" significa per te. Il vector store sotto? Buy, è un dettaglio tecnico.

**3. Controllo richiesto.** Servono garanzie particolari (data residency, certificazioni, audit)? Buy può non essere possibile, o l'unica opzione "buy" disponibile costa troppo. Spesso il "build" qui è in realtà "self-host di open-source" — non costruisci da zero, ma gestisci tu.

**4. Costo a volume atteso.** Le soluzioni managed pagano a uso. A volumi alti, il "build" (anche solo come self-host) può diventare drasticamente più economico. Il punto di pareggio dipende dal componente — per il vector store è di solito molto alto (managed conviene fino a milioni di vettori), per il gateway è più basso (LiteLLM self-hosted è gratis e gestibile presto).

**5. Competenze del team.** "Build" significa anche "manutenere per sempre". Se il team non ha le competenze (e non vuole costruirle), il TCO del "build" esplode silenziosamente. È il fattore più sottovalutato — si calcola il costo del "buy" come prezzo della licenza e quello del "build" come "tempo di scrittura iniziale", ignorando l'orizzonte di manutenzione.

<details>
<summary>La regola pragmatica</summary>

In ordine di preferenza, per ogni componente:

1. **Buy managed** se esiste, costa accettabile, soddisfa i vincoli (controllo, privacy).
2. **Self-host di open-source** se "buy" non è possibile o conveniente al tuo volume, e hai (o sei disposto ad acquisire) le competenze.
3. **Build interno** solo se è davvero il pezzo che ti differenzia, o se non esistono alternative mature.

L'80% dei sistemi AI ben fatti che vedi in giro sono al livello 1 o 2 per la maggior parte dei componenti, e al livello 3 solo per la business logic specifica e il prompt engineering del dominio.

</details>

## Cosa praticamente non si costruisce in casa

Una lista breve di componenti che, salvo casi molto particolari, comprare o usare open-source è sempre la scelta migliore:

- **Modelli foundation.** Addestrare un LLM da zero costa decine di milioni di dollari per essere "appena competitivo". Fine-tuning di modelli esistenti è un'altra storia (lezione 1.7), ma il base model si compra o si scarica.
- **Vector store.** Sei vendor maturi, due eccellenti open-source. Costruirne uno è esercizio accademico.
- **Motori di inferenza.** vLLM, TGI, TensorRT-LLM sono enormi sforzi di engineering — non li ri-fai.
- **Observability tooling.** Langfuse, LangSmith, Phoenix coprono il 95% dei casi.

E cosa praticamente non si compra mai pronto (perché *è* il sistema):

- **L'orchestrazione applicativa.**
- **I prompt e i criteri di qualità.**
- **L'eval suite specifica per il dominio.**
- **Le integrazioni con i sistemi interni.**

## Modelli proprietari vs open-weight

Ortogonale al build vs buy: anche scegliendo "buy del modello", c'è da decidere se proprietario chiuso (GPT-4o, Claude, Gemini) o open-weight (Llama, Mistral, Qwen, DeepSeek). Non è una scelta religiosa.

**A favore dei modelli proprietari:**

- **Frontier capabilities.** I migliori modelli su task complessi (ragionamento, multi-step, codice) restano i proprietari delle frontier labs. Il gap si chiude lentamente ma esiste.
- **Zero ops.** Niente GPU, niente serving, niente versioning manuale. Si paga a token e si pensa al business.
- **Capacità multimodali avanzate.** I modelli proprietari sono spesso avanti sull'integrazione visione/audio/testo.
- **Tooling integrato.** Structured output, function calling, prompt caching, batch API — i provider offrono ergonomia che con open-weight devi costruire.

**A favore degli open-weight:**

- **Data residency.** I dati non lasciano la tua infrastruttura. Per casi con vincoli stringenti (legale, sanità, finance highly regulated), è spesso l'unica opzione praticabile.
- **Costo a volume.** Sopra una certa soglia di volume, il costo per token degli open-weight self-hosted è significativamente inferiore. Il punto di pareggio dipende dal modello e dal hardware ma esiste.
- **Controllo del modello.** Pin alla versione assoluto, nessuna deprecation imposta, possibilità di fine-tuning specifico.
- **Niente dipendenza strategica.** Non sei legato al destino commerciale e ai prezzi di un singolo provider.
- **Qualità in rapida convergenza.** Il gap di qualità si è ridotto enormemente. Per la maggior parte dei task applicativi non frontier, gli open-weight oggi sono "abbastanza buoni" — termine che andava sostenuto due anni fa, oggi è ovvio. <span class="badge-stato evoluzione">In evoluzione</span>

<details>
<summary>L'ibrido come default emergente</summary>

Molti sistemi seri oggi usano entrambi. Il modello proprietario per le query complesse o per le funzionalità frontier (visione, audio, ragionamento difficile). Il modello open-weight per il grosso del traffico (query semplici, classificazione, summarization). Il routing tra i due (lezione 6.2) è gestito dal gateway. Questa configurazione spesso ottimizza simultaneamente costo, latenza media, e copertura — meglio di entrambe le opzioni pure.

</details>

## Il ragionamento sul vendor lock-in

Un argomento spesso usato a favore dell'open-weight è "non vogliamo dipendere da un singolo provider". È un argomento sensato ma sopravvalutato se non hai un gateway. Con un gateway ben fatto (lezione 6.1), cambiare provider proprietario (da Anthropic a OpenAI, o aggiungere un secondo provider come fallback) è una configurazione. Il lock-in vero non è sull'API, è su:

- **Capacità specifiche del modello** (es. una particolare modalità multimodale, un determinato stile di output)
- **Pricing che cambia** sotto i piedi
- **Versioni deprecate** senza un equivalente nel catalogo
- **Disservizio** che ti tira giù il sistema

Per tutti e quattro, la difesa giusta è il gateway + un fallback configurato, non necessariamente l'open-weight. L'open-weight è la difesa giusta quando hai vincoli di privacy/controllo, o quando il costo a volume lo rende strettamente conveniente.

## La matrice di decisione, da usare davvero

Quando devi scegliere build vs buy per un componente, o proprietario vs open-weight per il modello, scrivi una matrice come questa e compilala. Forza a esplicitare i trade-off:

| Criterio | Peso | Buy / Proprietario | Build / Open-weight |
|---|---|---|---|
| Costo iniziale | | | |
| Costo a regime (volume X) | | | |
| Time-to-market | | | |
| Controllo / privacy | | | |
| Differenziazione | | | |
| Manutenibilità nel tempo | | | |
| Rischio vendor lock-in | | | |

Il peso lo decidi tu in base al business. Non c'è un peso universale. Ma scriverlo costringe a una conversazione esplicita e produce un decision record (sezione successiva) che tra sei mesi ti ringrazi di aver lasciato dietro.

## Il Decision Record: il documento che difende le scelte

Ogni decisione architetturale importante (componente, modello, archetipo) merita un **ADR — Architecture Decision Record**. Una pagina che dice:

1. **Contesto.** Cosa stiamo decidendo e perché lo facciamo ora.
2. **Opzioni considerate.** Almeno tre. "Buy X", "Self-host Y", "Build Z" — con sintesi pro/contro.
3. **Decisione.** Cosa abbiamo scelto.
4. **Conseguenze.** Cosa diventa più facile, cosa più difficile, cosa accettiamo di rischiare.
5. **Data e revisione.** Quando l'abbiamo deciso, quando rivedere.

Sembra burocrazia, è oro. Tra un anno, quando entra un membro nuovo del team o quando il PM chiede "perché abbiamo scelto X?", la risposta esiste scritta. Senza ADR, le scelte diventano leggende — "credo che l'abbia deciso Marco prima che andasse via" — e cambiarle costa il triplo.

## Cosa NON è build vs buy

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Più build = più professionale / più controllo" | Build = più manutenzione per sempre. Solo dove ti differenzia. |
| "Proprietario = qualità superiore" | Era vero due anni fa, oggi vale solo per le frontier capabilities. |
| "Open-weight = sempre più economico" | Dipende dal volume e dal costo del compute. Sotto soglia, managed costa meno. |
| "Cambieremo provider quando serve" | Senza gateway, "cambiare provider" tocca tutto il codice. La possibilità va costruita prima. |

## Cosa dura, cosa evitare

Dura: **i cinque criteri** (maturità, criticità, controllo, costo, competenze), **il gateway come prerequisito di mobilità**, **gli ADR come pratica**.

Cambia: **chi vince in quale fascia** — la classifica dei modelli (proprietari e open-weight) si aggiorna ogni pochi mesi; **i prezzi**; **il gap di qualità** tra proprietari frontier e open-weight (in chiusura, ma a velocità variabile).

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani.

1. Cita i cinque criteri di decisione per build vs buy e fai un esempio di componente per cui ciascuno è decisivo.
2. Quando il "build interno" è davvero giustificato, e quando è una trappola?
3. Quali sono i pro principali dei modelli proprietari, e quali degli open-weight?
4. Perché il vendor lock-in è un argomento "sensato ma sopravvalutato" se hai un gateway?
5. Cosa contiene un ADR e perché vale la pena scriverlo anche se sembra burocrazia?

---

## Glossario

- **Build vs buy** — decisione su se costruire un componente in casa o acquistare/usare uno esistente.
- **Self-host di open-source** — variante di "build" in cui non costruisci da zero ma gestisci tu un componente open-source.
- **Modello proprietario / closed-weight** — modello accessibile solo via API del provider, pesi non distribuiti (GPT-4o, Claude, Gemini).
- **Modello open-weight** — modello i cui pesi sono distribuiti pubblicamente e possono essere serviti in proprio (Llama, Mistral, Qwen, DeepSeek).
- **Frontier capability** — capacità di punta che solo i modelli proprietari più avanzati hanno (al netto del momento).
- **ADR — Architecture Decision Record** — documento breve che cattura una decisione architettonica, le alternative considerate e le conseguenze.
- **TCO — Total Cost of Ownership** — costo totale incluso sviluppo iniziale, manutenzione, ops, deperimento.

---

## Per approfondire

- **"Architecture Decision Records"** — cerca il pattern di Michael Nygard (l'originale è del 2011, ancora il riferimento).
- **"The Tao of Pragmatic Architecture"** — letture sui pattern di decisione architetturale; non specifico AI, ma applicabile.
- **Open LLM Leaderboard di Hugging Face** — riferimento aggiornato sulla classifica degli open-weight; va guardato per "dove siamo oggi", non come scrittura.

---

## Prossima lezione

**Decision drill — Architettura.** Tre scenari per testare i criteri di decisione di questa lezione e della 7.1. Una PMI che vuole un assistente sui dati, un'enterprise che valuta proprietario vs open-weight, una scelta di componenti su un sistema esistente.
