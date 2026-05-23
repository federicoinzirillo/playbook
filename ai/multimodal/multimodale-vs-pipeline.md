---
title: Quando multimodale, quando pipeline separate
sidebar_label: "2.6 Multimodale vs pipeline"
sidebar_position: 6
---

# Quando multimodale, quando pipeline separate

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 2.6</span>
  <span>~9 min di lettura</span>
</div>

<p class="lesson-lead">La scelta tra un modello multimodale nativo e una pipeline di modelli specializzati non è tecnica — è economica, organizzativa, e dipende dal task specifico. La griglia qui ti dà i criteri per decidere senza dover testare tutto.</p>

Nelle lezioni 2.1-2.4 hai visto le capacità di ciascuna modalità. Ora la domanda che si pone in ogni progetto reale: costruisco con un modello multimodale nativo (un'unica API che riceve immagini, audio, testo) oppure con una pipeline di modelli specializzati (OCR → LLM, ASR → LLM, ecc.)?

La risposta non è "il nativo è sempre meglio perché è più moderno" — né "la pipeline è sempre meglio perché è controllabile". Dipende da cinque assi.

## I cinque assi di decisione

### 1. Il task richiede ragionamento cross-modale?

Questa è la domanda più importante. Se la risposta corretta dipende dall'interazione tra modalità — non da ciascuna separatamente — il multimodale nativo ha un vantaggio che la pipeline non può replicare.

**Esempi che richiedono cross-modal reasoning:**
- "Questo grafico è coerente con i dati nella tabella accanto?" → serve vedere entrambi insieme
- "Il tono della voce del cliente è coerente con le parole che dice?" → serve l'audio grezzo, non la trascrizione
- "Estrai le informazioni da questo documento con layout a due colonne dove il testo scorre tra una colonna e l'altra" → il layout visivo è inseparabile dal contenuto

**Esempi che NON richiedono cross-modal:**
- "Trascrivi questa chiamata e riassumila" → ASR + LLM funziona perfettamente
- "Estrai il testo da questa fattura e controlla che i totali siano corretti" → OCR + LLM funziona
- "Classifica questi 1000 prodotti da foto" → classificatore dedicato + eventuale post-processing

Quando il task è sequenziale — modalità A produce output, output va a modello B — la pipeline è quasi sempre sufficiente e più economica.

### 2. Costo e volume

Questo asse spesso decide da solo.

I modelli multimodali nativi addebitano i token visivi e audio come token aggiuntivi — e li addebitano ogni volta. Un'immagine ad alta risoluzione può costare 800-1500 token. A 100.000 richieste al giorno con un'immagine ciascuna, il conto si nota.

Una pipeline Whisper turbo (on-premise o quasi gratuito) + un modello testo della generazione 2026 (GPT-5.3 Instant, Claude Haiku 4.5, Gemini 3 Flash) può costare 10-20 volte meno della modalità audio-nativa dei modelli di frontiera su grandi volumi.

**Regola pratica:** se il volume è alto e il task è separabile in step, fai il conto prima di scegliere l'architettura. La differenza può essere di un ordine di grandezza.

### 3. Best-in-class per singola modalità

Per alcuni task specializzati, i modelli dedicati battono ancora i generalisti.

- **OCR su manoscritti storici** → modelli addestrati su dati specifici
- **Classificazione medica su radiografie** → modelli fine-tunati su dataset clinici
- **Trascrizione di accenti regionali forti** → Whisper con fine-tuning specifico
- **Object detection in real-time su video** → YOLO e derivati, ottimizzati per throughput

Se stai costruendo un sistema dove la qualità su quella modalità specifica è critica, confronta i benchmark prima di assumere che il generalista basti.

### 4. Controllo, debugging e manutenzione

Le pipeline separate hanno un vantaggio spesso sottovalutato: **ogni step è ispezionabile**. Quando qualcosa va storto, puoi vedere esattamente cosa ha prodotto l'OCR, cosa ha ricevuto il LLM, dove è entrato l'errore.

Con un modello multimodale nativo, l'errore è un black box: hai dato immagine + testo, hai ricevuto risposta sbagliata, non sai dove nel processo interno è andata storta la comprensione.

**Le pipeline valgono quando:**
- Hai SLA di qualità per singola componente
- Vuoi sostituire un componente senza toccare il resto
- Il debugging è critico (es. sistemi mission-critical)
- Hai team separati che si occupano di parti diverse del sistema

### 5. Latenza

Le pipeline aggiungono latenza per ogni step: OCR richiede tempo, poi il LLM richiede tempo. Se la latenza end-to-end è critica (real-time, meno di 1 secondo), un singolo modello multimodale può essere più veloce — una sola richiesta API invece di due o tre in sequenza.

D'altro canto, alcuni step della pipeline si parallelizzano: se devi processare immagine e audio contemporaneamente, puoi mandarli in parallelo ai rispettivi modelli specializzati e poi combinare i risultati, riducendo la latenza complessiva.

## La griglia decisionale

| | Multimodale nativo | Pipeline separata |
|---|---|---|
| **Task cross-modale** | ✓ Vantaggio netto | ✗ Perde informazione tra step |
| **Alto volume** | ✗ Costo token visivi/audio | ✓ Modelli dedicati molto più economici |
| **Qualità massima su task specifico** | ~ Generalista, buono ma non sempre il migliore | ✓ Fine-tuned models vincono spesso |
| **Debugging e controllo** | ✗ Black box | ✓ Step ispezionabili |
| **Latenza end-to-end** | ✓ Un solo step | ~ Dipende da parallelismo |
| **Semplicità implementativa** | ✓ Un'API | ✗ Orchestrazione multi-modello |
| **On-premise possibile** | ✗ Spesso solo API | ✓ Whisper, YOLO, ecc. girano on-prem |

## Il pattern ibrido

La scelta non è binaria. Un sistema maturo spesso combina:

- **Modello multimodale per l'understanding** — capisce il documento, risponde alle domande
- **Modelli specializzati per la trasformazione** — ASR per la trascrizione, TTS per la risposta vocale, diffusion per la generazione di immagini

Esempio classico: un assistente per documenti che usa un modello multimodale per rispondere alle domande su immagini e testo misto, ma usa un TTS dedicato per sintetizzare la risposta audio (qualità migliore, costo molto più basso di un audio-native model).

<span class="badge-stato evoluzione">In evoluzione</span> I modelli multimodali nativi migliorano velocemente e il divario con i modelli specializzati si stringe. Il confronto economico va rifatto periodicamente: il punto di pareggio si sposta con ogni nuova versione dei modelli.

## Cosa NON è questa scelta

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Il multimodale nativo è sempre meglio perché è più recente" | Non se il task è separabile. La pipeline è spesso 10x più economica a parità di qualità. |
| "La pipeline è obsoleta" | Ancora la scelta giusta per task separabili ad alto volume, per on-premise, e per debugging critico. |
| "Scelgo uno e non cambio mai" | L'architettura va rievaluata quando cambiano i modelli, il volume o il task. |
| "Il multimodale risolve la qualità dell'OCR" | Su documenti comuni sì. Su manoscritti, layout inusuali, font rari, i modelli specializzati tengono il vantaggio. |

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani. L'ultima anticipa il drill.

1. Qual è la domanda più importante da porsi prima di scegliere nativo vs pipeline?
2. In quale scenario la pipeline è quasi sempre preferibile per ragioni di costo?
3. Perché il debugging è più facile in una pipeline che con un modello nativo?
4. Descrivi un sistema ibrido realistico e spiega perché ha senso.
5. *(anticipazione)* Un'azienda farmaceutica vuole un sistema che analizza radiografie e produce un report testuale. Quale architettura sceglieresti, e perché non il multimodale nativo generico?

---

## Glossario

- **Cross-modal reasoning** — capacità di fare inferenze che richiedono informazioni da più modalità contemporaneamente; impossibile in una pipeline sequenziale senza perdita di informazione.
- **Pipeline separata** — architettura in cui modelli specializzati per ogni modalità si passano output in sequenza.
- **Token visivi** — la rappresentazione di un'immagine in token per il modello multimodale; il loro numero (e costo) dipende dalla risoluzione.
- **On-premise** — esecuzione del modello su infrastruttura propria invece che su API di terze parti; rilevante per dati sensibili e per controllo del costo.
- **Best-in-class** — il modello migliore per una singola modalità specifica, spesso specializzato e fine-tunato su dati di dominio.

---

## Per approfondire

- **Benchmark comparativi** (OpenLLM Leaderboard, MMMU, DocVQA) — confrontano i modelli multimodali su task specifici; utili per capire dove i generalisti tengono e dove no.
- **Prezzi aggiornati dei modelli** — i provider pubblicano i listini token/immagine; confronta prima di scegliere l'architettura.

*Risorse indicate per la ricerca; per i link aggiornati conviene cercarli al momento.*

---

## Prossima lezione

**2.6 Decision drill — Multimodale.** Hai tutti i mattoni. Due scenari realistici con vincoli concreti: sistema di estrazione dati da documenti e customer support vocale. Decidi l'architettura, poi apri la griglia.
