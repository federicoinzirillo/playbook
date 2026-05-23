---
title: "Video generation: cosa sa fare, costi, quando usarlo"
sidebar_label: "2.9 Video generation"
sidebar_position: 9
---

# Video generation: cosa sa fare, costi, quando usarlo

<div class="lesson-meta">
  <span class="badge-stato evoluzione">In evoluzione</span>
  <span>Lezione 2.9</span>
  <span>~11 min di lettura</span>
</div>

<p class="lesson-lead">Il video generativo è l'area che nel 2025-26 ha fatto i salti più spettacolari — e quella dove è più facile farsi illudere da una demo. Questa lezione separa le capability reali dai limiti concreti, e ti dà i criteri per decidere se e quando usarlo in un progetto serio.</p>

La generazione di immagini (lezione 2.5) è diventata matura: modelli affidabili, costi bassi, casi d'uso consolidati. Il video è un passo avanti più lungo di quanto sembri. Un'immagine è un singolo frame; un video è una sequenza di frame temporalmente coerente, con movimenti plausibili, fisica credibile e continuità di identità tra i frame. Sono vincoli molto più stringenti.

I modelli di video generation del 2026 sono impressionanti. Ci sono anche cose che ancora non sanno fare, costi che sorprendono, e latenze che rendono alcuni casi d'uso impossibili. Partiamo dal quadro reale.

## Lo stato del 2026: cosa sanno fare

I modelli principali al 2026 sono **Sora** (OpenAI), **Veo 2 e Veo 3** (Google DeepMind), **Runway Gen-4**, **Kling** (Kuaishou), **LTX Video** (Lightricks, open-source). Ogni mese esce qualcosa di nuovo o vengono aggiornati, quindi i numeri specifici invecchiano in fretta — i principi restano.

Cosa sanno fare bene, collettivamente:
- **Testo-a-video**: dato un prompt descrittivo, generare un clip da 5-30 secondi con qualità visiva alta su soggetti comuni (persone, paesaggi, oggetti, animali)
- **Immagine-a-video**: animare una foto di partenza con movimenti specificati ("la persona sorride", "la scena dissolve verso destra")
- **Style transfer e coerenza visiva**: mantenere uno stile grafico coerente su più clip
- **Movimenti di camera** specificati nel prompt: zoom, pan, dolly, aerial shot
- **Transizioni fluidità**: dissolvenze, morph tra scene

Veo 3 (mid-2025) ha introdotto la generazione di audio sincronizzato con il video — musica e sound effects generati insieme al visual. È un salto qualitativo: non più solo immagini in movimento, ma un media completo.

## I limiti: dove ancora si vede la cuciture

Nonostante i progressi, ci sono aree dove i modelli del 2026 mostrano ancora i loro limiti:

**Coerenza temporale.** Su clip lunghi (>10-15 secondi) il modello perde il filo: personaggi che cambiano leggermente aspetto da un secondo all'altro, oggetti che appaiono e spariscono, mani con sei dita che diventano cinque. La coerenza identitaria su clip lunghi è ancora un problema aperto.

**Fisica plausibile.** Liquidi che scorrono in modo strano, fuoco che si comporta come nebbia, oggetti che attraversano altri oggetti. Per scene semplici funziona bene; per scene con fisica complessa si notano le anomalie.

**Controllo fine.** "Il personaggio a sinistra si gira verso destra e indica l'edificio rosso sullo sfondo" — i modelli attuali seguono le istruzioni generali ma non garantiscono che ogni dettaglio sia esatto. Più il prompt è specifico, più è probabile che qualcosa non torni.

**Volti e espressioni.** I volti umani sono migliorati molto, ma le micro-espressioni su primer piani lunghi possono sembrare artificiali. Nelle demo si vede sempre l'inquadratura più riuscita; in produzione sistematica la varianza è alta.

## Il modello di costo: non come l'immagine

Con la generazione di immagini, il costo è per immagine — qualche centesimo, e la latenza è 2-5 secondi. Con il video è tutto diverso.

**Costo**: si paga **al secondo di video generato**, non per query. I prezzi variano per modello e qualità, ma un ordine di grandezza realistico al 2026 è $0.50-2.00 per secondo di video a 720p-1080p. Un clip da 15 secondi costa quindi $7-30. Su produzione sistematica, i budget si esauriscono in fretta.

**Latenza**: generare 10 secondi di video richiede **minuti**, non secondi. Le pipeline cloud più veloci (Runway Gen-4 con accelerazione) scendono a 1-2 minuti per 10 secondi; i modelli più lenti possono richiedere 5-10 minuti. Questo rende il video generation **non adatto a uso real-time** o on-demand durante una sessione utente. Si pre-genera, si cachea, si usa.

**Coda e throttling**: i provider cloud di video generation hanno limiti di concurrency significativamente più restrittivi rispetto ai modelli di testo. Burst di richieste parallele vengono accodate. Per pipeline batch, pianifica tempi di completamento nell'ordine delle ore, non dei minuti.

## Casi d'uso seri (con ROI reale)

**Mockup di prodotto e visualizzazione.** Un brand che vuole mostrare un prodotto non ancora fisicamente disponibile, in diversi ambienti e contesti. Il costo di un video generativo è una frazione del costo di un set fotografico. Funziona bene su oggetti (prodotti, veicoli, architettura); meno su persone che interagiscono con il prodotto.

**B-roll per contenuti video.** Un producer che ha girato un'intervista ha bisogno di immagini di supporto (panoramiche, close-up, transizioni). Generare b-roll su tema specifico è molto più veloce e meno costoso che girarlo o acquistarlo. Qualità sufficiente per la maggior parte delle produzioni non-broadcast.

**Formazione interna e e-learning.** Scenari simulati (situazioni di lavoro, procedure di sicurezza, onboarding) generati su specifiche. Il ciclo di revisione è molto più veloce che con riprese fisiche — il prompt si modifica, si rigenera.

**Advertising e contenuto social.** Varianti di un annuncio video su diversi stili visivi o messaggi. A/B test di creatività senza costi di produzione proporzionali al numero di varianti.

**Prototipazione di storyboard.** Trasformare uno storyboard in un animatic video prima di investire in produzione. Veloce, economico per uso interno.

## Quando NON usarlo

**Real-time o on-demand durante l'interazione utente.** La latenza di minuti esclude qualunque scenario dove l'utente aspetta il video generato in sessione.

**Quando la coerenza dell'identità è critica.** Se hai bisogno dello stesso personaggio con lo stesso aspetto in più clip di una storia lunga, le tecniche attuali (IP-Adapter, face control) sono fragili. Una persona reale filmata è ancora molto più affidabile.

**Quando la precisione del dettaglio conta.** Istruzioni di sicurezza, dimostrazioni di prodotto dove ogni gesto deve essere esatto, contenuti educativi dove la fisica deve essere corretta. Un errore non rilevato in un video di formazione può creare comportamenti sbagliati.

**Su scala senza budget dedicato.** Se hai bisogno di 1.000 clip da 15 secondi, a $15 medio per clip stai parlando di $15.000. Fai il conto prima.

**Quando la qualità broadcast è richiesta.** I modelli del 2026 producono risultati adatti al web e al mobile. Per trasmissione televisiva o cinema, la qualità attuale non regge ancora il confronto con riprese reali.

<span class="badge-stato rischio">A rischio</span> I prezzi e i provider cambiano rapidamente. I numeri in questa lezione riflettono lo stato di maggio 2026; controlla i pricing attuali prima di costruire un business case.

## Cosa NON è

| Pensiero sbagliato | Come stanno le cose |
|---|---|
| "Stesso funzionamento della generazione di immagini, solo più frame" | La coerenza temporale è un problema radicalmente diverso. Un modello che genera immagini ottime può generare video mediocri perché il video richiede coerenza tra frame, non solo qualità di frame singoli. |
| "Posso usarlo per generare video di training sicuro" | "Sicuro" per cosa? Se il video mostra procedure fisiche (sicurezza sul lavoro, emergenze), errori non rilevati possono creare comportamenti sbagliati. Qualunque video di formazione va validato da esperti di dominio. |
| "I modelli open-source sono equivalenti ai cloud" | Modelli open-source come LTX Video o CogVideoX sono notevolmente più economici da eseguire, ma richiedono GPU potente e producono qualità inferiore ai modelli frontier cloud su scene complesse. Il trade-off è reale. |
| "La latenza migliorerà presto, meglio costruirci sopra ora" | Forse. Ma costruire oggi un'architettura che assume latenza di secondi su qualcosa che oggi richiede minuti è prematura ottimizzazione. Progetta per la realtà attuale con interfacce che possono cambiare. |

## Verifica di comprensione

1. Perché la generazione di video è intrinsecamente più difficile della generazione di immagini?
2. Un cliente vuole un sistema che generi video personalizzati on-demand per ogni utente al login. Quali problemi tecnici sollevi?
3. Qual è il modello di costo tipico della video generation (al 2026) e come si confronta con la generazione di immagini?
4. Nomina due casi d'uso dove il video generativo ha ROI reale e due dove non è adatto allo stato attuale.
5. *Domanda di scenario*: hai un budget di €2.000 per creare contenuti video di formazione su 5 scenari di sicurezza, ciascuno lungo circa 60 secondi. Il video generativo è fattibile? Fai un conto approssimativo.

## Glossario della pagina

**Coerenza temporale** — proprietà di un video per cui i soggetti mantengono aspetto, posizione e identità coerenti da un frame all'altro nel tempo.

**Immagine-a-video** (image-to-video) — generazione di video a partire da un'immagine statica di partenza, con movimenti specificati nel prompt.

**B-roll** — filmato di supporto usato per coprire voiceover o transizioni in un video; non è il contenuto principale ma arricchisce il montaggio.

**IP-Adapter** — tecnica per mantenere l'identità visiva di un soggetto (volto, stile) coerente tra più generazioni. Fragile sui video rispetto alle immagini.

**Latenza di generazione** — tempo necessario per produrre il video dopo l'invio del prompt. Attualmente nell'ordine dei minuti per clip da 10-30 secondi.

## Per approfondire

- Tieni aggiornati i prezzi controllando direttamente le pagine pricing di Runway, Kling, e Google Veo.
- Cerca "video generation benchmark 2025 2026" per confronti aggiornati su qualità e latenza tra modelli.
- La community di Civitai e Reddit r/StableDiffusion documenta casi d'uso pratici con modelli open-source.

## Prossima lezione

Con questa lezione chiudi il quadro completo delle modalità: testo, vision, audio, immagini, video. La Parte 2 è completata. Il passo successivo è la Parte 3 — **valutare e rendere affidabile** ciò che hai costruito. Si inizia con il vocabolario dell'evaluation: benchmark, dataset e metriche (lezione 3.1).
