---
title: Monitoring e drift in produzione
sidebar_position: 3
---

# Monitoring e drift in produzione

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 6.3</span>
  <span>~11 min di lettura</span>
</div>

<p class="lesson-lead">La 3.2 ti ha mostrato cosa loggare per valutare. Qui parliamo del giorno-2: il sistema è live da settimane, niente errori, niente alert — eppure la qualità sta scivolando giù. Il monitoring di produzione non è solo "il server è su"; per un sistema LLM, il fallimento silenzioso è la norma, non l'eccezione.</p>

Un'API REST tradizionale fallisce in modo rumoroso: errore 500, latenza che esplode, dashboard rossa. Un sistema LLM fallisce in modo educato: la risposta arriva, è grammaticalmente perfetta, sembra ragionevole — solo che è sempre meno utile, sempre più imprecisa, sempre più fuori target. Nessun alert tradizionale scatta. Gli utenti notano prima dei sistemi di monitoring, e quando lo fanno hanno già smesso di fidarsi.

## Tre livelli di monitoring, tre tipi di problema

Servono tre livelli sovrapposti. Coprono problemi di natura diversa, con strumenti diversi.

**Monitoring operativo.** Il classico: uptime, latenza, error rate, throughput, utilizzo GPU. Questo strato risponde a "il sistema sta rispondendo?". Lo gestisci con gli stessi strumenti del resto del software (Prometheus, Grafana, Datadog, OpenTelemetry). Non è specifico dell'AI, ma per i sistemi LLM serve aggiungere metriche su TTFT (lezione 6.1), token consumati, hit rate della cache.

**Monitoring di qualità.** L'output continua ad avere il livello di qualità atteso? Questo strato non risponde a "è giù?" ma a "fa ancora bene il suo lavoro?". Si appoggia su LLM-as-judge (lezione 3.1) e su segnali utente. È lo strato che cattura il degrado silenzioso.

**Monitoring del drift.** Cosa sta cambiando — nelle query degli utenti, nei dati recuperati, nei risultati? Il drift è il cambiamento graduale delle distribuzioni rispetto al baseline. Anche se ogni singola richiesta sembra ok, lo spostamento aggregato segnala che il sistema sta operando in un regime diverso da quello per cui era stato calibrato.

Senza tutti e tre, vedi solo una parte del problema.

## Il monitoring di qualità: come si fa in pratica

Il dilemma centrale: valutare la qualità in produzione costa. Mandare ogni risposta a un LLM-as-judge raddoppia (almeno) i costi e aggiunge latenza. La soluzione è il **sampling**.

**Sampling stratificato.** Non valutare tutto, ma garantire copertura sui segmenti che contano: per feature, per tenant, per tipo di query. Una percentuale piccola (1-5%) campionata in modo intelligente è sufficiente per stimare la qualità per segmento.

**Asincrono, non bloccante.** La valutazione avviene dopo che la risposta è stata data all'utente. Logghi la (query, risposta), un job in background la valuta, salva il risultato. Nessun impatto su latenza.

**Metriche che si guardano:**
- **Faithfulness** (su sistemi RAG) — la risposta è coerente con i documenti recuperati? Un calo qui significa che il modello sta "andando per conto suo" più del solito.
- **Relevance** — la risposta affronta davvero la domanda? Cala quando le query degli utenti si spostano fuori dal dominio coperto.
- **Hallucination rate** — frequenza di affermazioni non supportate dai documenti recuperati o dalla ground truth.
- **Format compliance** — quando l'output deve avere uno schema (lezione 1.3), quante volte lo rispetta?

**Segnali utente impliciti.** Spesso più affidabili di quelli espliciti. Thumbs up/down sono usati raramente. Ma "l'utente ha riformulato la stessa domanda entro 30 secondi" è un segnale forte che la risposta non andava bene. "L'utente ha chiuso la conversazione subito dopo la risposta" lo stesso. "L'utente ha copiato negli appunti l'output strutturato" è un segnale positivo. Vanno loggati e aggregati.

<details>
<summary>Perché i thumbs up/down non bastano</summary>

Tasso di utilizzo dei thumbs su sistemi reali: 1-3% delle conversazioni. E con forte bias verso il negativo (la gente lascia feedback quando è arrabbiata, non quando è soddisfatta). Sono utili come segnale aneddotico, terribili come metrica. I segnali impliciti hanno copertura molto maggiore (100% delle interazioni, non solo quelle dove l'utente clicca) e meno bias. La pratica è: log esplicito quando c'è, segnali impliciti sempre, LLM-as-judge su campione.
</details>

## Il drift: il cambiamento che ti scappa di mano

Quattro tipi di drift, ognuno con sintomi e cause diverse.

**Drift di input (query drift).** Le domande che ti fanno gli utenti stanno cambiando. Sintomo: il distribuzione delle lunghezze, dei topic, delle lingue, delle entità menzionate si sposta. Causa tipica: il prodotto è andato in un nuovo mercato, una nuova feature ha cambiato il pattern d'uso, sei diventato virale su una nicchia. Come si misura: clustering degli embedding delle query, distribuzione dei topic estratti, statistiche di base (lunghezza, lingua).

**Drift di documento (corpus drift).** I dati su cui fai retrieval (lezione 5.4) stanno cambiando, ma l'indice no — o viceversa, l'indice è cambiato molto rispetto a quando hai calibrato i prompt. Sintomo: faithfulness che cala, citazioni a documenti vecchi anche quando ce ne sono di nuovi. Come si misura: frequenza dei chunk recuperati, distribuzione delle date di update dei documenti nelle risposte.

**Drift del modello.** Il provider ha aggiornato il modello sotto il tuo naso. Non è teorico — è successo con ogni provider major più volte. Sintomo: cambiano lo stile, la lunghezza media, il comportamento sui prompt borderline. Come si difendi: pin sulla versione del modello quando possibile (`gpt-4o-2024-08-06` invece di `gpt-4o`), eval suite che gira a ogni update annunciato.

**Drift di obiettivo.** Il business vuole che il sistema faccia ora cose leggermente diverse da quelle per cui è stato calibrato. Sintomo: il PM dice "perché non risponde mai a X?". Non è un bug, è un drift nei requisiti. Come si gestisce: review periodica dei criteri di valutazione (lezione 3.1), non solo del sistema.

I primi tre si rilevano con metriche; il quarto solo con conversazioni con chi usa il sistema.

## Tracing distribuito: senza, non capisci niente

Quando una risposta è sbagliata, hai bisogno di ricostruire tutto: cosa è arrivato, quali chunk sono stati recuperati, quali tool sono stati chiamati, quale prompt è andato al modello, quale risposta è tornata, quali transformation sono avvenute prima di mostrarla all'utente.

Senza tracing distribuito, è ricostruzione manuale dai log. Con tracing, è un click. La differenza tra "ci metto un giorno a capire" e "ci metto 5 minuti".

Tool concreti: **OpenTelemetry** è lo standard generico; **LangSmith**, **Langfuse**, **Phoenix (Arize)** sono specifici per LLM e capiscono il modello dati dell'AI (chain, tool call, retrieval). La scelta dipende dallo stack esistente — se hai già OpenTelemetry, conviene integrare; se parti da zero su un sistema AI, gli strumenti specifici danno UI migliore out-of-the-box.

## Incident response: cosa fare quando la qualità cala

L'incident sul degrado di qualità è diverso dall'incident sul downtime. Non c'è un singolo evento da contenere — c'è una distribuzione che si è spostata. La sequenza:

1. **Confermare il segnale.** Cala su una metrica o su più? Su un segmento o globalmente? Il drift può essere reale o un artefatto del cambio della distribuzione di chi misura.
2. **Cercare il delta.** Cosa è cambiato di recente? Update del modello? Modifica al prompt di sistema? Nuova versione dell'indice? Cambio nella pipeline di retrieval? Le release recenti sono il primo sospetto.
3. **Isolare con tracing.** Su un campione di casi negativi, ricostruire il path completo. Spesso il problema si manifesta in uno stadio specifico (retrieval, generazione, post-processing).
4. **Rollback se possibile.** Se il colpevole è un cambio recente, rollback. Se il colpevole è esterno (cambio del modello dal provider), serve mitigazione (cambio di provider, downgrade alla versione precedente se ancora disponibile, modifica del prompt per compensare).
5. **Post-mortem.** Cosa avrebbe permesso di accorgersene prima? Quale metrica era assente, quale alert non scattato. La risposta diventa lavoro per i prossimi sprint.

## Cosa NON è il monitoring AI

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "L'uptime è verde, tutto bene" | L'uptime non vede il degrado di qualità. È necessario ma insufficiente. |
| "I thumbs up/down sono la nostra metrica" | Utili come segnale, terribili come metrica primaria. Coverage troppo bassa e bias. |
| "Valutiamo tutte le risposte con LLM-as-judge" | Costa il doppio del sistema e non aggiunge molto rispetto a un sampling stratificato. |
| "Il drift lo rileviamo quando un utente si lamenta" | È esattamente il momento in cui hai già perso fiducia. Va rilevato prima, sui dati. |

## Cosa dura, cosa evitare

Dura: **il framework a tre livelli** (operativo, qualità, drift), il **tracing distribuito**, l'**incident response come pratica**.

Cambia: gli **strumenti specifici** (Langfuse, LangSmith, Arize si evolvono velocemente; alcuni spariranno). I **modelli di costo** del monitoring (oggi LLM-as-judge è caro, in due anni potrebbe non esserlo più).

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani.

1. Quali sono i tre livelli di monitoring e quale problema copre ognuno?
2. Perché i segnali utente impliciti sono spesso più affidabili dei thumbs up/down?
3. Cita quattro tipi di drift e un sintomo per ciascuno.
4. Cosa permette il tracing distribuito che il logging tradizionale non permette?
5. Qual è la sequenza di incident response su un calo di qualità?

---

## Glossario

- **Drift** — cambiamento graduale della distribuzione (di input, output, dati) rispetto al baseline su cui il sistema è stato calibrato.
- **Query drift** — cambiamento nella distribuzione delle domande degli utenti.
- **Corpus drift** — cambiamento nei dati su cui si fa retrieval rispetto allo stato in cui era stato calibrato il sistema.
- **Sampling stratificato** — campionamento di una percentuale piccola garantendo copertura per segmento (feature, tenant, tipo).
- **Tracing distribuito** — strumentazione che ricostruisce tutto il path di una richiesta attraverso i componenti del sistema.
- **Segnali utente impliciti** — comportamenti che indicano qualità (riformulazione, abbandono, copia negli appunti) senza feedback esplicito.
- **TTFT — Time to First Token** — vedi lezione 6.1; metrica primaria nello streaming.

---

## Per approfondire

- **OpenTelemetry per GenAI** — leggi le semantic convention per AI; lo standard si sta consolidando.
- **Langfuse** e **Phoenix (Arize)** — entrambi open-source, vale la pena vedere il modello dati che usano per sistemi LLM.
- **"Monitoring Machine Learning Models in Production"** — testo classico (Chip Huyen e altri) — molti concetti sul drift sono pre-LLM ma si applicano direttamente.

---

## Prossima lezione

**6.4 Decision drill — Produzione.** Hai serving, costi, monitoring. Il drill ti mette davanti a due scenari di incident: i costi che esplodono e la qualità che cala in silenzio. Per ognuno, sequenza di indagine e mitigation.
