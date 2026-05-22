---
title: Capstone — progetto end-to-end
sidebar_position: 1
---

# Capstone — un sistema AI end-to-end

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 7.4 — Capstone</span>
  <span>~più sessioni (non si legge, si fa)</span>
</div>

<p class="lesson-lead">L'esame finale. Un caso che attraversa <strong>tutte</strong> le aree della guida: foundation, costruzione, valutazione, sicurezza, design, produzione, architettura. Non si supera leggendo — si supera progettando, scrivendo un decision record, e sapendo difendere ogni scelta davanti a uno stakeholder che chiede "perché?". È il pezzo da mostrare a un colloquio o a un cliente.</p>

Le lezioni precedenti ti hanno dato i mattoni. Il drill di ogni parte ti ha allenato sulle decisioni. Il capstone è il salto finale: nessuno ti dice quale archetipo usare, nessuno ti dice quale modello, nessuno ti dice cosa loggare. Devi progettare il sistema *e* difendere ogni scelta nominando l'alternativa che hai scartato.

## Lo scenario

Un'**azienda media del settore assicurativo italiano** (1.500 dipendenti, 800.000 clienti) vuole introdurre un assistente AI usabile dai propri agenti di filiale. L'agente in filiale, durante una conversazione con il cliente, deve poter chiedere all'assistente:

- Domande sulla normativa interna ("posso applicare lo sconto X a questo profilo?")
- Domande sulle polizze del cliente specifico ("quali coperture ha attive?", "qual è la sua storia sinistri?")
- Spiegazioni in linguaggio semplice di clausole di polizza ("cosa significa esattamente la clausola 7.3 di questa polizza?")
- Suggerimenti per cross-selling ("dato il profilo del cliente, quale altra polizza ha senso proporre?")

**Vincoli:**
- **Dati riservati** — informazioni anagrafiche, sanitarie (assicurazioni vita), finanziarie. GDPR + normativa di settore.
- **Volume previsto** — 2.000 agenti attivi, ~30.000 query al giorno a regime, picchi di 8.000/ora nelle ore di sportello.
- **Latenza** — gli agenti sono al telefono o sportello col cliente: risposta in 5-8 secondi è il massimo accettabile; più di 10 secondi è inutilizzabile.
- **Budget** — €30.000/mese a regime per tutto (modelli, infrastruttura, servizi).
- **Team disponibile** — quattro sviluppatori senior generalisti, un ML engineer junior, un team ops esistente che non ha mai gestito GPU.
- **Compliance** — devono superare audit interni di sicurezza e privacy. Il sistema può supportare decisioni commerciali (cross-selling) ma non decisioni autonome.
- **Timeline** — MVP in 4 mesi, regime in 12 mesi.

## Cosa devi produrre

Un **decision record** di 4-8 pagine che copra le sette aree elencate sotto. Per ogni scelta: cosa scegli, quali alternative hai scartato, perché. È il documento che porteresti davanti a uno stakeholder (Comitato IT, audit di sicurezza, capo dell'innovazione).

### 1. Inquadramento e classificazione (Parte 4, EU AI Act)

- A che classificazione di rischio ricade il sistema secondo l'EU AI Act? Perché?
- Quali sono gli obblighi che ne derivano?
- Quali dati personali tratta e con quale base giuridica?

### 2. Architettura logica (Parte 5, Parte 7)

- Quale archetipo: pipeline, RAG conversazionale, agente o ibrido? Per quali sotto-task?
- Mappa i sei strati della reference architecture (lezione 7.1) per questo sistema specifico. Quali componenti hai in ognuno?
- Quale pattern di sistema (sincrono/asincrono/batch)? Per quali parti?
- Disegna almeno uno schema (Mermaid o a parole) che mostri il flusso di una query tipica.

### 3. Modello e serving (Parte 6, Parte 7)

- Modello proprietario, open-weight o ibrido? Perché?
- Servizi managed o serving in proprio? Considerando il team senza esperienza GPU.
- Come gestisci il routing tra modelli diversi (se ibrido).
- Quale gateway LLM, quali fallback.

### 4. Dati e knowledge (Parte 1, Parte 5)

- Quali sono le sorgenti? Come gestisci la diversità tra documentazione strutturata (normativa) e dati transazionali (polizze del cliente)?
- Quale chunking, quale vector store, quale strategia di retrieval?
- Come gestisci la freshness dei dati transazionali (polizze, sinistri che cambiano)?
- Come previeni che un agente veda dati di clienti che non sono suoi?

### 5. Sicurezza, guardrail, privacy (Parte 4)

- Quali guardrail di input metti, quali di output?
- Come previeni il prompt injection (lezione 4.1)?
- Come gestisci dati sanitari (assicurazioni vita) — pseudonimizzazione? Confinamento?
- Quali rischi specifici del cross-selling automatizzato e come li gestisci?
- Chi può fare cosa (autorizzazioni a grana fine)?

### 6. Valutazione e monitoring (Parte 3, Parte 6)

- Quale golden dataset costruisci e con chi?
- Quali metriche di qualità? Faithfulness su normativa, accuratezza su polizze, qualità del cross-selling.
- Quale eval suite gira automaticamente? Quando?
- Quali tre livelli di monitoring (operativo, qualità, drift) implementi e con quali strumenti?
- Cosa succede quando il provider del modello rilascia un update?

### 7. Costi, ops, governance (Parte 6, Parte 7)

- Stima del costo a regime con i numeri dello scenario. Sta dentro i €30.000/mese?
- Quali sono le tre leve principali di ottimizzazione del costo se sfori?
- Roadmap a 4 mesi (MVP) e a 12 mesi (regime). Cosa lasci fuori dall'MVP intenzionalmente?
- Chi è responsabile di cosa lato ops? Cosa fa il team esistente, cosa va costruito.
- Come gestisci un incident di qualità che cala? Un incident di costo che esplode? Una segnalazione del cliente di output discriminatorio?

## Come ti valuti

Non c'è una soluzione "giusta". Ci sono soluzioni che un senior difenderebbe e soluzioni che farebbero alzare un sopracciglio. La tua risposta è solida se:

- **Per ogni scelta hai nominato almeno un'alternativa scartata e il motivo.** Senza questo, è ricetta, non architettura.
- **Hai esplicitato i trade-off.** "Faccio X, accetto questo rischio in cambio di questo beneficio."
- **Hai distinto MVP da regime.** Cosa va fatto subito vs cosa si rimanda. È competenza, non rinuncia.
- **Hai identificato i rischi specifici di questo dominio** (assicurativo, sanitario, finanziario) e non solo i rischi generici dell'AI.
- **Le tue stime di costo reggono uno scrutinio elementare** (numero di query × costo per query a livello di ordine di grandezza).
- **Hai un piano per quando qualcosa va storto** — quale incident response, chi è responsabile.

Se tutto questo c'è, hai un decision record che difende un sistema. È quello che porti a un colloquio per dimostrare di saper fare il mestiere.

## Suggerimenti pratici

**Non scriverlo tutto di getto.** Usa più sessioni: una per il modello, una per i dati, una per la sicurezza. Tornaci sopra. Riscrivilo. La prima versione è quasi sempre lacunosa nei rischi.

**Usa gli ADR come unità.** Ogni decisione importante è un mini-ADR (contesto, alternative, decisione, conseguenze). Il decision record finale è un montaggio di ADR + sezioni di sintesi.

**Chiediti per ogni componente: "cosa succede se questo si rompe alle 3 di notte?"** Spesso fa emergere pezzi mancanti (failover, alert, manual override).

**Fa' leggere il documento a qualcuno con poca esperienza AI.** Se non capisce le scelte, il documento è scritto male. Se le capisce e ti fa domande imbarazzanti, te le farà anche lo stakeholder.

**Non aver paura di dire "non lo so ancora, da decidere alla settimana 8 con questi criteri".** Un decision record che ammette le incertezze esplicitamente è più professionale di uno che finge certezza dove non c'è.

## Cosa NON è il capstone

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Devo scrivere codice per dimostrare di saper fare" | Il capstone è un decision record. Il codice viene dopo, e dipende dalle scelte. |
| "Devo includere tutto quello che ho imparato" | Devi includere quello che il sistema richiede. Il resto è rumore. |
| "Più diagrammi, più professionale" | Un diagramma che chiarisce vale dieci diagrammi che riempiono. |
| "Se non so qualcosa, evito di parlarne" | Esplicitare un'incertezza ben caratterizzata è più forte di mascherarla con sicurezza vuota. |

## La tabella di marcia consigliata

1. **Sessione 1 — leggi lo scenario, scrivi a mano (o in chat con un collega) cosa tu, di getto, faresti.** Non vincolarti. Annota tutto.
2. **Sessione 2 — 1. Inquadramento e classificazione.** Apri la 4.4, rispondi pulito.
3. **Sessione 3 — 2. Architettura logica + 3. Modello e serving.** Apri 7.1, 7.2, 5.1, 6.1.
4. **Sessione 4 — 4. Dati + 5. Sicurezza.** Apri 5.4, 1.1, 4.1, 4.3.
5. **Sessione 5 — 6. Valutazione + 7. Costi e ops.** Apri 3.1, 3.5, 6.2, 6.3, 6.4.
6. **Sessione 6 — Revisione finale.** Rileggi tutto. Chiediti per ogni sezione: ho nominato l'alternativa scartata? Ho un piano per il fallimento?
7. **Sessione 7 — Lettura da fuori.** Falo leggere a qualcuno. Aggiorna in base al feedback.

---

## Per approfondire (dopo il capstone)

Quando il capstone è scritto e ti soddisfa, sei nella zona "AI Engineer applicativo solido". I prossimi passi per evolvere verso AI Solution Architect:

- **Ripeti il capstone su un altro dominio** (sanità, manifattura, retail). Cambia tutto, anche se i pattern restano simili.
- **Tieni il radar acceso** — il campo si muove. Le decisioni che oggi sono giuste possono non esserlo tra un anno. La pratica di scrivere ADR datati ti dà la disciplina di rivisitare.
- **Lavora sul racconto.** Saper progettare è metà; saperlo difendere a un C-level che chiede "perché stiamo spendendo X?" è l'altra metà. Pratica le presentazioni di architettura, non solo i diagrammi.

---

## Chiusura

Se sei arrivato qui e hai scritto il capstone, hai attraversato sette parti — fondamenta, costruire, multimodale, valutare, sicurezza, design, produzione, architettura — e hai un documento che difende un sistema reale. Questo è il portfolio. Buon lavoro.
