---
title: "Vision: cosa sa fare l'AI sulle immagini"
sidebar_position: 2
---

# Vision: cosa sa fare l'AI sulle immagini

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 2.2</span>
  <span>~10 min di lettura</span>
</div>

<p class="lesson-lead">Classificazione, object detection, OCR, document understanding, video: sei capacità distinte con requisiti tecnici distinti. Capire dove ogni approccio eccelle — e dove smette di funzionare — è la competenza che ti serve per scegliere la soluzione giusta senza dover testare tutto.</p>

Nella lezione 2.1 hai capito il meccanismo: le immagini diventano token visivi, il transformer ragiona su tutto insieme. Qui scendi nel pratico: quali task sa fare bene l'AI sulle immagini, quali tool esistono, dove i modelli moderni battono le pipeline classiche, e dove invece i vecchi approcci specializzati tengono ancora.

## Classificazione: "cosa c'è in questa immagine?"

La classificazione è il task più vecchio e più maturo del computer vision. Un modello vede un'immagine e assegna una (o più) etichette: "gatto", "documento fiscale", "immagine di qualità accettabile", "prodotto difettoso".

Per molti scenari di classificazione standard, i modelli multimodali di frontiera 2026 (GPT-5.4, Gemini 3.1 Pro, Claude Opus 4.7) fanno già un ottimo lavoro senza training aggiuntivo — descrivi in linguaggio naturale cosa distingue le categorie e il modello classifica. Zero dataset etichettati, zero training. Il salto di qualità rispetto alla generazione 2024 è netto soprattutto sulle immagini ad alta risoluzione: Opus 4.7 a fine 2025-26 ha portato la risoluzione gestita nativamente da ~1MP a oltre 3.7MP, rendendo affidabile la lettura di documenti, schermate e fotografie ad alta densità.

Il limite: su task di classificazione **fine-grained e domain-specific** — distinguere varianti di uno stesso prodotto industriale, classificare malattie rare su radiografie, riconoscere specie vegetali simili — i modelli generalisti spesso non bastano. Qui servono modelli specializzati addestrati su dati di dominio, o quanto meno il fine-tuning di un modello base.

## Object detection: "dove sono gli oggetti?"

La classificazione dice *cosa*; la **object detection** dice *dove*. Produce bounding box — rettangoli di coordinate — attorno a ogni oggetto rilevato nell'immagine. "Tre persone: [0.1, 0.2, 0.3, 0.5], [0.4, 0.1, 0.6, 0.4], [0.7, 0.3, 0.9, 0.6]."

I modelli multimodali generalisti non sono ottimizzati per questo: rispondono bene alla domanda "ci sono persone in questa immagine?" ma producono coordinate bounding box poco precise. Per task che richiedono localizzazione precisa e processing ad alta velocità su stream video, i modelli specializzati di detection (YOLO, DETR e derivati) sono ancora lo standard.

Quando usare cosa:
- **Task di comprensione** ("ci sono veicoli nel parcheggio?") → modello multimodale
- **Localizzazione precisa e conteggio** → modello specializzato di detection
- **Real-time su video** (>10 FPS) → modello specializzato, ottimizzato per throughput

## OCR e document understanding

**OCR — Optical Character Recognition** — trasforma testo nelle immagini in testo digitale. Ma il vero problema dei documenti non è estrarre il testo: è capire la *struttura*.

Una fattura ha un header, un footer, una tabella di voci con colonne. Un contratto ha sezioni numerate, clausole, paragrafi rientrati. Un form ha campi etichettati. Un OCR classico estrae tutti i caratteri in ordine di posizione — ma senza capire che quella stringa è un importo nella colonna "IVA", o una data di scadenza nel campo "Valid until".

Qui i modelli multimodali nativi marcano una differenza netta. Puoi mostrare a GPT-5.4 o Claude Opus 4.7 una foto di una fattura e chiedere di restituire un JSON con i campi strutturati — importo, data, partita IVA, voci di dettaglio. Il modello vede il layout, capisce la struttura, e produce l'output. Nessun template di estrazione per-documento. Nessun addestramento su dataset di fatture.

**Document understanding** è il termine che copre questo scenario: non solo estrarre il testo, ma capire il documento come struttura semantica — tabelle, relazioni tra campi, layout multi-colonna, testo misto a grafici.

<span class="badge-stato evoluzione">In evoluzione</span> Modelli dedicati al document understanding (come DocOwl, InternVL, o i modelli aziendali di Azure Document Intelligence) esistono per casi d'uso ad alto volume e alta precisione, e spesso battono i modelli generalisti su benchmark di estrazione strutturata. Il gap si stringe, ma su pipeline di produzione da milioni di documenti il confronto va misurato.

## VQA: visual question answering

**VQA — Visual Question Answering** — è il task che meglio rappresenta il multimodale moderno: poni una domanda sull'immagine in linguaggio naturale, il modello risponde.

"Quante persone ci sono in questa foto?" "Qual è il prodotto più costoso in questo scontrino?" "Questa radiografia mostra qualcosa di anomalo?" "Che stagione è rappresentata in questa scena?"

I modelli moderni hanno capacità di VQA molto solide su immagini naturali e documenti comuni. I limiti emergono su:

- **Conteggio preciso** su immagini dense (>10 oggetti simili → errori frequenti)
- **Ragionamento spaziale fine** ("qual è il terzo oggetto da sinistra nella seconda riga?")
- **Dominio molto specifico** senza contesto nel prompt

## Image captioning e generazione di descrizioni

L'altro verso del VQA: dato un'immagine, genera una descrizione testuale. Utile per accessibilità (descrizioni per non vedenti), per indicizzazione di archivi visivi, per arricchire dataset.

I modelli generalisti producono descrizioni di buona qualità su scene naturali. Su immagini tecniche — schemi elettrici, piantine architettoniche, grafici scientifici — la qualità cala perché richiede un vocabolario di dominio che potrebbe non essere nella distribuzione di training.

## Video: comprensione temporale

La comprensione video aggiunge una dimensione: il *tempo*. Il modello deve capire non solo cosa c'è in un frame, ma come la scena cambia, cosa succede, in che ordine.

I modelli multimodali moderni supportano video (sequenze di frame), ma con limitazioni importanti:

- **Costo**: ogni frame è un'immagine con i suoi token visivi. Un video di 1 minuto a 1 fps = 60 immagini.
- **Lunghezza**: la context window limita quanti frame puoi mandare in una singola chiamata.
- **Comprensione fine**: capire un'azione rapida o un gesto sottile richiede frame rate alto, che costa.

Per task di video understanding ad alto volume (sorveglianza, analisi di contenuti UGC — User Generated Content), esistono pipeline specializzate. Per task occasionali di comprensione video (es. "riassumi questo video tutorial"), i modelli generalisti vanno bene.

<span class="badge-stato evoluzione">In evoluzione</span> La comprensione video nativa (senza dover estrarre frame manualmente) è un'area in rapida evoluzione. I modelli Gemini hanno già capacità estese su questo fronte; altri provider stanno seguendo.

## Cosa NON è la vision AI

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Il modello vede come vediamo noi" | Vede sequenze di patch/vettori. Non ha percezione spaziale innata; errori su "destra/sinistra" e conteggio preciso sono comuni. |
| "OCR + LLM e document understanding sono la stessa cosa" | OCR estrae caratteri; il document understanding capisce la struttura. I modelli multimodali fanno entrambi in un solo passo. |
| "Per classificazione serve sempre un modello specializzato" | Per classificazione generale e zero-shot, i modelli generalisti spesso bastano. Il specializzato serve sul fine-grained di dominio. |
| "La comprensione video è risolta" | Su video brevi e scene semplici sì. Su video lunghi, azioni rapide o dominio specifico, ci sono ancora limiti importanti. |

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani.

1. Cosa distingue classificazione e object detection, e quando serve ciascuna?
2. Perché il document understanding è diverso dall'OCR classico?
3. Descrivi un task di VQA reale nel tuo contesto lavorativo (o in uno che immagini). Il modello generalista basterebbe?
4. Quali sono i due limiti principali della comprensione video con i modelli attuali?
5. Quando la classificazione fine-grained di dominio richiede un modello specializzato invece del generalista?

---

## Glossario

- **Object detection** — task che identifica e localizza oggetti in un'immagine, producendo bounding box con coordinate.
- **Bounding box** — rettangolo di coordinate che delimita la posizione di un oggetto nell'immagine.
- **OCR (Optical Character Recognition)** — estrazione del testo contenuto in un'immagine.
- **Document understanding** — comprensione della struttura semantica di un documento (campi, tabelle, layout) oltre alla semplice estrazione di testo.
- **VQA (Visual Question Answering)** — task in cui si pone una domanda in linguaggio naturale su un'immagine e il modello risponde.
- **Image captioning** — generazione automatica di una descrizione testuale di un'immagine.
- **Fine-grained classification** — classificazione tra categorie molto simili che richiedono distinzioni sottili (es. specie animali diverse, varianti di prodotto).
- **UGC (User Generated Content)** — contenuti creati dagli utenti (video, foto, testo) su piattaforme sociali o di condivisione.

---

## Per approfondire

- **YOLO e DETR** — i modelli specializzati di object detection più diffusi; cerca le versioni recenti nella documentazione ufficiale.
- **"Document AI"** di Google Cloud e **"Azure Document Intelligence"** — esempi di servizi specializzati per il document understanding ad alto volume.
- **Benchmark VQA** (VQAv2, MMBench, MMMU) — dataset che permettono di comparare i modelli sulle capacità visual; cercali su Papers With Code per i risultati aggiornati.

*Risorse indicate per la ricerca; per i link aggiornati conviene cercarli al momento.*

---

## Prossima lezione

**2.3 Audio e speech.** Dalla vision al suono. ASR, TTS, modelli audio-nativi: quando la trascrizione basta e quando serve il modello che capisce il tono — non solo le parole.
