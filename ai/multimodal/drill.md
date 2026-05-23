---
title: Decision drill — Multimodale
sidebar_label: "2.7 Decision drill"
sidebar_position: 7
---

# Decision drill — Multimodale

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 2.7</span>
  <span>~20 min (incluso il tempo di lavoro)</span>
</div>

<p class="lesson-lead">Due scenari con vincoli reali. Decidi l'architettura — nativo o pipeline, quale modello, perché — e giustifica ogni scelta prima di aprire la griglia.</p>

Hai le basi di tutta la Parte 2: meccanismo multimodale, capacità per modalità, griglia nativo vs pipeline. Il drill verifica che tu sappia applicarle su scenari con vincoli concreti — non che tu le reciti.

**Come usarlo:** leggi lo scenario, scrivi la tua risposta completa *prima* di aprire la griglia. La griglia non è la soluzione: è quello che avrebbe nominato un senior. Se ne hai mancati alcuni, torna alla lezione dove quel punto è trattato.

---

## Scenario A — Estrazione dati da fatture

**Contesto:** un'azienda di logistica riceve 15.000 fatture al mese da fornitori diversi — formati diversi, layout diversi, alcuni PDF nativi, molti scan di qualità variabile. Deve estrarre automaticamente: fornitore, data, numero fattura, importo totale, importo IVA, voci di dettaglio (descrizione + quantità + prezzo unitario). I dati estratti devono finire in un database ERP. L'errore tollerato sull'importo totale: 0%. Budget: contenuto (nessun team ML dedicato). Latenza: non è real-time, un'ora di processing notturno va bene.

**Cosa ti viene chiesto:** scegli l'architettura (multimodale nativo, pipeline OCR + LLM, altro?), giustifica ogni componente, nomina i trade-off e le trappole. Cosa fai per garantire che l'importo totale estratto sia corretto?

*Scrivi la tua risposta completa — poi apri la griglia.*

<details>
<summary>Griglia di valutazione</summary>

**Cosa avrebbe nominato un senior:**

**Architettura: multimodale nativo per l'estrazione, con validazione nel codice.**

Un modello multimodale nativo (Claude Opus 4.7, GPT-5.4, Gemini 3.1 Pro) è la scelta migliore qui perché le fatture hanno layout variabili — il layout visivo è inseparabile dai dati. Un OCR classico estrae testo in ordine di posizione, perdendo le relazioni tra colonne e campi. Il modello multimodale vede la pagina come un documento strutturato.

**Structured output obbligatorio.** Non lasciare che il modello risponda in prosa. Definisci uno schema Pydantic preciso con tutti i campi richiesti. Il modello produce JSON strutturato, il codice valida prima di inserire nel DB.

**Validazione critica sull'importo totale.** Il 0% di errore tollerato non si ottiene fidandosi del modello. Dopo l'estrazione:
1. Il codice calcola la somma delle voci di dettaglio.
2. La confronta con il totale estratto.
3. Se c'è discrepanza → alert per revisione umana, non inserimento automatico.

Il modello può sbagliare a leggere un numero, trasporre cifre, perdere una riga. La validazione nel codice è il guardrail obbligatorio.

**Gestione dei scan di bassa qualità.** Non tutti gli scan sono leggibili. Prima della chiamata al modello, una pre-verifica della qualità dell'immagine (contrasto, risoluzione, orientamento) permette di inviare per revisione umana i documenti con qualità sotto soglia, invece di sprecare chiamate API su input irrecuperabili.

**Volume e costo.** 15.000 fatture al mese × costo per immagine. Fai il conto: a ~800-1500 token per immagine media (dipende dalla risoluzione del modello), a un certo prezzo per token, il budget mensile deve essere sostenibile. Considera se un modello più piccolo (Gemini 3 Flash, Claude Haiku 4.5) con qualità leggermente inferiore è accettabile sul tuo use case prima di scegliere il modello top.

**Latenza.** Batch notturno → nessun vincolo real-time. Puoi processare in parallelo (pool di chiamate API) per finire in tempo.

**Trade-off da nominare:**

- *OCR classico + LLM vs multimodale nativo*: OCR classico è molto più economico ma perde struttura su layout complessi. Se le fatture hanno struttura uniforme, OCR + LLM può bastare. Se i layout variano molto, il nativo vince.
- *Costo per errore vs costo per revisione umana*: la validazione automatica riduce gli errori ma non li azzera. Quantifica quante fatture andranno in revisione umana e il costo di quel processo.

**Trappole da evitare:**

- Inserire nel DB senza validazione dell'importo → garantito che prima o poi ci sarà un errore.
- Non gestire la bassa qualità degli scan → sprechi chiamate API e ottieni estrazioni inutilizzabili.
- Chiedere al modello di calcolare i totali invece di farlo nel codice → i modelli fanno errori aritmetici.
</details>

---

## Scenario B — Customer support vocale in tempo reale

**Contesto:** un operatore telecom vuole sostituire il primo livello del customer support con un sistema AI vocale. Gli utenti chiamano, descrivono il problema, il sistema capisce e risponde vocalmente. Principali tipi di richiesta: problemi di connettività, richieste di fattura, attivazione/disattivazione servizi. Il sistema deve capire l'italiano colloquiale (accenti regionali inclusi), rispondere in meno di 2 secondi, trasferire a un operatore umano quando non è sicuro. Volume: 500 chiamate simultanee nei picchi.

**Cosa ti viene chiesto:** scegli l'architettura audio, giustifica latenza e costo, decidi quando il modello deve trasferire a un umano (e come implementarlo), nomina i rischi principali.

<details>
<summary>Griglia di valutazione</summary>

**Cosa avrebbe nominato un senior:**

**Architettura: pipeline ASR + LLM + TTS, non audio-nativo.**

Il ragionamento economico è decisivo. 500 chiamate simultanee × costo audio-nativo = costi probabilmente insostenibili. Una pipeline Whisper turbo + un modello testo veloce (GPT-5.3 Instant, Claude Haiku 4.5, Gemini 3 Flash) + TTS è un ordine di grandezza più economica e ha latenza comparabile se ottimizzata.

**Whisper per ASR.** Per customer support vocale, la comprensione del *tono* non è il focus primario — capisci l'intenzione dalle parole. Il tono diventa rilevante per identificare un cliente frustrato; ma puoi farlo anche con sentiment analysis sul testo, non necessariamente con audio-nativo. Whisper gestisce bene l'italiano colloquiale e ha varianti ottimizzate per velocità (la **large-v3-turbo** è lo standard di fatto: ~6x più veloce di large-v3 a qualità sostanzialmente uguale).

**LLM per il reasoning.** Classifica il tipo di richiesta, decide la risposta, decide se trasferire a umano. Structured output per la risposta: forza il modello a produrre `{risposta: "...", trasferisci: bool, motivo: "..."}`.

**TTS per la risposta vocale.** Un modello TTS dedicato (ElevenLabs, Azure Speech, Google TTS) produce qualità alta a costo basso rispetto all'audio-nativo.

**Latenza target di 2 secondi.** Il budget si distribuisce:
- ASR (Whisper streaming): ~300-500ms
- LLM (risposta breve): ~500-800ms
- TTS (primi chunk): ~300-500ms (con streaming, non aspetti la risposta completa)

Il trucco critico: **streaming**. Non aspettare che Whisper finisca prima di mandare al LLM, e non aspettare che il LLM finisca prima di iniziare il TTS. Pipeline parzialmente sovrapposta, non sequenziale.

**Handoff a umano.** Il modello deve trasferire quando:
- Confidence bassa sulla classificazione della richiesta
- Il cliente ha già chiesto di parlare con un umano
- Richiesta fuori scope (es. dispute legali, reclami formali)
- Più di 2 tentativi falliti di comprensione

Implementazione: la condizione `trasferisci: true` nell'output strutturato. Il codice lo rileva e gestisce il trasferimento. Non delegare al modello la logica di quando trasferire — il codice la controlla.

**Rischi principali:**

- *Accenti regionali forti*: Whisper gestisce bene il mainstream italiano; su dialetti marcati la qualità cala. Misura il WER (Word Error Rate — la percentuale di parole trascritte male) sui tuoi audio reali prima del deployment.
- *Latenza di rete*: 500 chiamate simultanee richiedono infrastruttura ottimizzata. La latenza percepita dal cliente è network + processing — ottimizza entrambe.
- *Loop di non-comprensione*: senza un limite esplicito di tentativi, un cliente che il sistema non capisce rimane bloccato. Il limite di 2 tentativi prima del trasferimento è il guardrail.
- *Compliance GDPR*: le chiamate vengono registrate? Serve consenso esplicito e policy chiara sulla retention dei dati audio.

**Trade-off da nominare:**

- *Pipeline ASR+LLM+TTS vs audio-nativo*: il nativo capirebbe il tono (frustrazione, urgenza) senza passare per il testo. Vale il costo a 500 chiamate simultanee? Probabilmente no, a meno che la detection del tono non sia il core differenziante del prodotto.
- *Velocità vs qualità ASR*: Whisper large-v3 dà qualità massima ma è lento; **large-v3-turbo** è ~6x più veloce con qualità solo leggermente inferiore. Per real-time, il turbo praticamente sempre vince.
</details>

---

## Cosa fare se hai mancato molti punti

- Cross-modal reasoning vs pipeline separata → **2.5 Quando multimodale, quando pipeline**
- Validazione degli output strutturati → **1.3 Structured output**
- Costo token per immagine/audio → **2.5** + documentazione del provider che usi
- Streaming ASR e latenza → **2.3 Audio e speech**
- Controllo di flusso fuori dai prompt (handoff) → **1.4 Agenti**
