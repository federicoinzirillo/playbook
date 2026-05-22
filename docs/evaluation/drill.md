---
title: "Decision drill — Valutazione"
sidebar_position: 5
---

# Decision drill — Valutazione

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 3.5</span>
  <span>~20 min (incluso il tempo di lavoro)</span>
</div>

<p class="lesson-lead">Allenamento da architetto della valutazione. Due scenari realistici con vincoli concreti — un chatbot di supporto e un agente con tool — su cui devi impostare la pipeline di eval end-to-end. Lo scopo non è la risposta giusta: è nominare i trade-off che un senior nominerebbe, e non saltare i pezzi che fanno la differenza tra "ho una demo" e "ho un sistema misurabile".</p>

Hai costruito il vocabolario della Parte 3: LLM-as-judge sui singoli output, observability per vedere dentro, contromisure per le allucinazioni, eval di traiettoria per gli agenti. Il drill verifica che tu sappia *assemblare* questi mattoni in un sistema di valutazione coerente — non solo che li riconosca a memoria.

**Come usarlo:** leggi lo scenario, scrivi la tua risposta completa *prima* di aprire la griglia. La griglia non è la soluzione: è quello che avrebbe nominato un senior. Se ne hai mancati alcuni, non significa che la tua risposta fosse sbagliata — significa che ci sono angoli da espandere. Torna alla lezione dove il punto mancante è trattato.

---

## Scenario A — Due versioni di un chatbot di supporto

**Contesto.** Una società di e-commerce ha un chatbot di customer support su un RAG di documenti aziendali (politiche di reso, FAQ, manuali prodotto). Hai due versioni del sistema: la **v1** in produzione da 3 mesi, e la **v2** sperimentale dove hai cambiato il prompt e aumentato il numero di chunk recuperati. Devi decidere se promuovere la v2 in produzione, con dati a supporto. La direzione vuole capire non solo "è migliorata" ma "di quanto" e "su cosa". Volume in produzione: 8.000 conversazioni al giorno. Budget per la valutazione: contenuto.

**Cosa ti viene chiesto.** Progetta la pipeline di valutazione completa: come misuri il salto v1 → v2, quali criteri usi, come gestisci i bias del giudice, come decidi la soglia per promuovere, cosa monitori dopo il rilascio.

*Scrivi la tua risposta completa qui — poi apri la griglia.*

<details>
<summary>Griglia di valutazione — apri solo dopo aver scritto la tua risposta</summary>

**Cosa avrebbe nominato un senior:**

**Golden dataset come primo passo.** Niente decisione senza un golden dataset rappresentativo. Costruirlo da: log reali di v1 (campiona 200 conversazioni stratificate per tipo di richiesta — reso, fattura, prodotto, generico), edge case noti (richieste ambigue, dati mancanti, fuori scope), casi avversariali (utenti che provano a far rispondere su cose fuori scope). Includere casi "trabocchetto" dove la risposta corretta è "non ho questa informazione" — fondamentali per misurare le allucinazioni.

**Criteri del giudice scelti per il task.** Per un RAG di customer support:
- *Fedeltà*: la risposta usa solo i chunk recuperati?
- *Pertinenza*: risponde alla domanda dell'utente?
- *Completezza*: copre i punti rilevanti, senza buchi?
- *Azionabilità*: l'utente può fare qualcosa con la risposta, o resta sospeso?
- *Tono*: è coerente con il brand (formale ma cordiale)?

Massimo 5 criteri. Sopra i 6 il giudice perde il filo e i punteggi diventano correlati.

**Confronto pair-wise, non singolo.** Per ridurre il bias di lunghezza e calibrazione, fai *confronto a coppie*: per ogni caso del golden dataset, mostra al giudice la risposta v1 e v2 affiancate e chiedi "quale è migliore, e perché?". Invertire la posizione (A=v1 vs B=v2 e A=v2 vs B=v1) e mediare elimina il bias di posizione. Risultato: % win rate di v2 su v1, per criterio.

**Calibrazione umana.** Su un sottoinsieme (30-50 casi) il giudizio del giudice va confrontato con un giudizio umano (un operatore di supporto, non chi ha scritto il prompt). Se la correlazione è alta (Pearson >0.7), il giudice è affidabile per le iterazioni rapide. Se no, il giudice va prima sistemato.

**Soglia di promozione esplicita.** Non "v2 sembra meglio": criteri quantitativi decisi *prima* di guardare i risultati. Esempio: v2 promossa se il win rate è ≥55% su almeno 3 dei 5 criteri E non peggiora di >5% su nessun criterio. Decidere la soglia *prima* evita il classico bias del "vedo i risultati e razionalizzo".

**A/B test in produzione.** Eval offline ha confermato il salto. Prima di full rollout, A/B test in produzione: 10% del traffico su v2, 90% su v1, per 1-2 settimane. Confronta hallucination rate, escalation a umano, durata media conversazione, e — se possibile — soddisfazione utente esplicita (feedback con pollice su/giù).

**Monitoring continuo dopo il rilascio.** Aggancia il giudice all'observability: campiona 2-5% delle conversazioni di produzione, fai valutare dal giudice, dashboard con punteggi nel tempo. Se la media scende del 10% in una settimana → alert. Trace + versione del prompt + chunk recuperati per ogni conversazione: senza questo, una caduta di qualità non sai da dove viene.

**Trade-off da nominare:**

- *Pair-wise vs scoring assoluto*: il pair-wise è più affidabile per confronti A/B (riduce i bias del giudice), lo scoring assoluto è meglio per tracking nel tempo (puoi confrontare oggi con un mese fa). In pratica si usano entrambi.
- *Costo del giudice*: 200 casi × 5 criteri × 2 versioni × confronto invertito = 4000 chiamate al giudice. Fai il conto del costo prima — un giudice GPT-4 su 4000 casi non è gratis.
- *Velocità di rilascio vs robustezza*: la soglia conservativa (es. 55% win + nessun peggioramento) è più sicura ma rilascia meno spesso. Un'azienda che sperimenta molto può accettare soglie meno strette.

**Trappole da evitare:**

- Valutare solo sul caso che ti viene in mente → non rappresenta i casi reali. Stratifica dal traffico vero.
- Saltare la calibrazione umana del giudice → costruisci sopra un metro che non sai come misura.
- Decidere la soglia dopo aver visto i risultati → confirmation bias garantito.
- Promuovere senza A/B test in produzione → l'eval offline e la realtà divergono più di quanto si pensi.
- Non agganciare la valutazione all'observability → al primo segnale di degradamento non sai cosa è cambiato.
</details>

---

## Scenario B — Agente con tool per analisi finanziaria

**Contesto.** Stessa società del drill 1.6 (assistente finanziario): un agente che, data una domanda di analisi, recupera dati da CRM e DB finanziario, fa calcoli, e produce un report. È in pre-rilascio. Il problema: il team ha provato 30 query "tipiche" e le risposte finali sembrano corrette, ma quando si guardano i log, l'agente a volte usa il tool sbagliato o chiama lo stesso tool 4-5 volte di fila prima di azzeccare i parametri. Il responsabile vuole capire se l'agente è pronto, e cosa monitorare in produzione. Volume previsto: 200 analisi al giorno, ciascuna con potenzialmente decine di tool call.

**Cosa ti viene chiesto.** Progetta la pipeline di eval agentica. Quali metriche misuri (oltre task completion), come costruisci gli scenari di test, come usi un giudice di traiettoria, cosa scatta in produzione. Cosa fai del problema "i loop sui parametri" che il team ha già notato.

<details>
<summary>Griglia di valutazione</summary>

**Cosa avrebbe nominato un senior:**

**Le metriche di traiettoria sono essenziali.** Task completion da sola non basta: il team ha già visto che le risposte finali "sembrano corrette" mentre la traiettoria è inefficiente. Le metriche multi-dimensionali della 3.4:
- *Task completion* (obiettivo raggiunto, sì/no/parziale)
- *Tool selection accuracy* (a ogni step, il tool scelto era il più adatto?)
- *Tool call correctness* (i parametri erano giusti dato lo stato?)
- *Step efficiency* (numero di step vs minimo necessario)
- *Trajectory safety* (effetti collaterali indesiderati? Per un agente di analisi finanziaria di sola lettura il rischio è minore, ma vale comunque sui tool che possono modificare stato del CRM)

**Test scenari scriptati come spina dorsale.** Costruire 40-60 scenari coprenti:
- Casi normali: query frequenti (top 10 per volume)
- Edge case: dati mancanti (cliente non in CRM), periodo senza transazioni, richieste ambigue
- Adversarial: domande fuori scope ("analizza la concorrenza", che il sistema non può fare), tentativi di prompt injection nei dati del CRM

Per ogni scenario: traiettoria attesa (sequenza minima di tool call) e anti-traiettoria (cose che non deve fare — es. non interrogare l'API del CRM per un cliente di cui non ha bisogno).

**Sandbox obbligatoria.** Replica del CRM e del DB finanziario con dati anonimizzati ma realistici. L'agente non deve poter scrivere niente in produzione durante i test. Permette di testare scenari distruttivi (es. "cancella il record X") senza danni — anche se l'agente non *dovrebbe* poter cancellare, è proprio per i casi in cui scopri che *può*.

**Giudice di traiettoria per i casi non scriptabili.** Le analisi su query libere ("trova trend nei clienti enterprise dell'ultimo anno") non hanno una traiettoria attesa univoca. Lì usi un giudice LLM con criteri di traiettoria (logica della sequenza, pertinenza dei tool, gestione errori, completamento). Pattern pratico: riassumi i risultati dei tool prima di passarli al giudice — i raw output del DB sono megabyte; al giudice basta il summary rilevante.

**Indagine specifica sui loop sui parametri.** Il team l'ha già notato. Diagnosi: probabilmente l'agente non ha abbastanza esempi few-shot di chiamate corrette, o la descrizione dei tool è ambigua sui parametri. Contromisure:
- Validazione dei parametri *prima* della chiamata, nel codice (lezione 1.3): se l'agente passa un periodo invalido, intercetti e restituisci un errore strutturato all'agente prima di chiamare il tool
- Limit esplicito al numero di retry per tool (es. max 2 retry sullo stesso tool prima di trasferire a umano)
- Logging del numero di retry per tool nel trace, alert se >2 in produzione

**Continuous evaluation in produzione.** Ogni run produce un trace completo (sequenza di tool call, parametri, risultati, risposta finale). Su un campione (5-10%) gira il giudice di traiettoria. Trace con punteggio basso o con loop sopra-soglia vanno in *coda di review*: diventano nuovi scenari del golden dataset. Sistema che impara dai suoi fallimenti senza aspettare incidenti.

**Soglia di rilascio.** Criterio quantitativo prima di guardare i risultati: rilascio se task completion ≥85% sugli scenari, tool selection accuracy ≥90%, step efficiency ≤2x del minimo nel 95% dei casi, zero violazioni di anti-traiettoria sui casi avversariali.

**Trade-off da nominare:**

- *Scenari scriptati vs giudice di traiettoria*: i primi sono affidabili e ripetibili ma costano tempo a definirli. Il secondo è scalabile ma più rumoroso. Approccio combinato: scriptati per il core, giudice per il long tail.
- *Sandbox fedele vs costo*: una sandbox identica alla produzione (stesso DB, stessi tool, dati anonimizzati) costa molto. Una sandbox semplificata (mock dei tool) è economica ma può nascondere problemi che emergono solo con dati reali. Trovare il punto di equilibrio.
- *Velocità di rilascio vs metriche multi-dimensionali*: misurare 5 dimensioni è più ricco ma più lento del solo task completion. Per un MVP forse parti da 2-3 metriche; man mano che maturi, espandi.

**Trappole da evitare:**

- Valutare solo task completion → l'agente che risolve un task in 12 step inutili passa il test, in produzione costa 6x.
- Saltare la sandbox → primo bug serio in produzione e hai un incidente, non una telemetria.
- Non distinguere errori del tool (es. timeout) da errori dell'agente (es. parametro sbagliato) → diagnosi sbagliata, fix nel posto sbagliato.
- Non costruire la coda di review per la production → l'agente fallisce in modi nuovi nel tempo (utenti reali, casi non previsti) e tu te ne accorgi dopo l'incidente.
- Far validare la sicurezza al modello stesso ("non fare azioni distruttive") → il modello segue le istruzioni in modo probabilistico. La sicurezza va nei guardrail di codice (lezione 4.2).
</details>

---

## Cosa fare se hai mancato molti punti

I drill allenano il giudizio, non lo valutano. Torna alla lezione dove il punto mancante è trattato:

- Costruzione del golden dataset, criteri, calibrazione del giudice → **3.1 LLM-as-judge**
- Trace, metriche di latenza/costo, prompt versioning, eval online → **3.2 Observability**
- Hallucination rate, citation enforcement, contromisure che reggono e quelle che no → **3.3 Gestire le allucinazioni**
- Metriche di traiettoria, test scriptati, sandbox, giudice di traiettoria → **3.4 Valutare agenti**
- Structured output per validazione → **1.3 Structured output**
- Controllo di flusso e guardrail fuori dai prompt → **1.4 Agenti**
