---
title: Decision drill — Produzione
sidebar_position: 4
---

# Decision drill — Produzione

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 6.4</span>
  <span>~25 min (incluso il tempo di lavoro)</span>
</div>

<p class="lesson-lead">Due scenari di incident reali sui sistemi LLM in produzione. Per ognuno: come indaghi, in quale ordine, quali mitigation di breve termine e quali fix di medio. Scrivi prima, poi apri la griglia.</p>

---

## Scenario A — I costi sono raddoppiati in una settimana

**Contesto:** un assistente AI per il supporto interno di un'azienda è live da quattro mesi. Modello: Claude Sonnet 4.6 via API (sostituito a febbraio 2026 al precedente GPT-4o quando il provider ha retirato la famiglia). Budget mensile storico: $3.000. Lunedì arriva un alert: il costo dell'ultima settimana è stato $1.500 invece dei $750 abituali. Nessun deployment recente. Niente alert su latenza, uptime o error rate. Il PM chiede "cosa è successo?".

**Cosa ti viene chiesto:**
1. Sequenza di indagine: cosa guardi per primo, secondo, terzo?
2. Quali dimensioni del costo aggrega per trovare la causa?
3. Quali sono le tre cause più probabili date queste informazioni?
4. Cosa fai immediatamente (mitigation di breve termine)?
5. Cosa fai dopo (fix strutturali)?

*Scrivi la risposta — poi apri la griglia.*

<details>
<summary>Griglia di valutazione</summary>

**Sequenza di indagine:**

1. **Confermare che è il sistema, non il logging.** Verifica fattura del provider o costo "fonte di verità" coincida con i log interni. Se discrepano c'è un bug nel calcolo, non un costo vero esploso.
2. **Aggregare per dimensione.** Costo per tenant/team, per feature, per modello, per ora del giorno. Cercare la dimensione dove la concentrazione del costo è cambiata.
3. **Cercare il delta.** È un singolo tenant esploso? Una feature? Una fascia oraria nuova? La media è cresciuta uniformemente o c'è una coda lunga nuova?

**Dimensioni da aggregare:**
- Costo per tenant — il classico "un cliente sta consumando troppo".
- Costo per feature applicativa — magari una feature lanciata 10 giorni fa è quella incriminata.
- Costo per richiesta (P50, P95, P99) — se P99 è esploso ma P50 è stabile, la coda lunga è il problema (richieste molto costose, non più richieste).
- Token input vs token output — se i token input sono cresciuti molto, è il contesto che si è gonfiato (prompt più lungo, contesti RAG più grandi, conversazioni più lunghe).
- Retry rate — un picco di retry può raddoppiare il costo silenziosamente.

**Tre cause più probabili:**

1. **Una conversazione/feature ha cominciato a portarsi storia troppo lunga.** In un assistente conversazionale è frequente: a 20 turni il prompt è 10x il primo turno. Sintomo: token input medi cresciuti, costo per richiesta cresciuto a parità di traffico.
2. **Un singolo tenant o utente ha cominciato a usarlo male.** Bot, integrazione automatica che fa loop, utente che incolla PDF interi. Sintomo: concentrazione del costo su pochi tenant rispetto al baseline.
3. **Una nuova feature o cambio di prompt di sistema.** Anche se non c'è stato deployment del modello, qualcuno potrebbe aver cambiato un prompt di sistema, aggiunto un contesto RAG più grande, abilitato tool calling che si moltiplica. Sintomo: costo cresciuto da una data precisa, correlato con un cambiamento in repo.

**Mitigation di breve termine:**
- **Quota per tenant** — se è un tenant che è esploso, mettere immediatamente un cap (in attesa di capire se è legittimo o no).
- **Routing temporaneo a modello più piccolo** — abbassare il modello per le query semplici, mentre indaghi.
- **Truncate aggressivo della conversazione** — se è la storia che cresce, limitare il numero di turni passati al modello.

**Fix strutturali (dopo aver risolto):**
- Implementare cap automatici per tenant (non solo per il primo che esplode).
- Routing intelligente del modello come default, non come patch.
- Anomaly detection automatica sul costo per tenant.
- Logging granulare se mancava (lezione 6.2).

**Trappole:**
- Non confermare il dato e correre a "ottimizzare" — magari è un bug del logging.
- Cambiare il modello globalmente perché "costa troppo" senza capire dove va il costo — riduci il costo ma anche la qualità in modo cieco.
- Mettere cap rigidi sul tenant senza prima capire se è uso legittimo (e quindi avere una conversazione commerciale) o anomalo (quota immediata).

</details>

---

## Scenario B — La qualità sta scivolando giù in silenzio

**Contesto:** un sistema RAG su documentazione tecnica per i clienti enterprise è live da otto mesi. Uptime al 99.95%. Latenza P95 stabile. Costo stabile. Nessun alert da settimane. Ma il customer success manager segnala che negli ultimi due mesi: il numero di ticket "il bot non sa rispondere" è cresciuto del 40%; due clienti enterprise hanno chiesto di parlare con il team perché "le risposte sono diventate generiche"; i thumbs up sono passati dal 70% al 55%.

**Cosa ti viene chiesto:**
1. Sequenza di indagine: come isoli il problema?
2. Quali tipi di drift sospetti per primo? Come li verifichi?
3. Quali metriche di qualità dovresti avere ma probabilmente non hai?
4. Mitigation di breve termine vs fix strutturali.
5. Come previeni che si ripeta?

<details>
<summary>Griglia di valutazione</summary>

**Sequenza di indagine:**

1. **Quantificare il calo con dati interni.** I segnali del CSM sono qualitativi. Servono numeri: thumbs trend (l'unico dato hard menzionato), retention sul sistema, frequenza di riformulazioni, tasso di abbandono nelle conversazioni. Se non hai queste metriche, è già la prima cosa da fissare.
2. **Cercare il momento del cambio.** Il calo è graduale o c'è un ginocchio in una data precisa? Un ginocchio sospetta un cambiamento specifico (update del modello, nuova versione dell'indice). Un calo graduale sospetta drift.
3. **Confrontare lo stato attuale con un baseline.** Versione del modello cambiata? Indice cambiato (nuovi documenti, vecchi rimossi)? Prompt di sistema modificato? Pipeline di retrieval modificata?
4. **Campionamento qualitativo.** Prendere 50 conversazioni recenti negative e 50 positive di otto mesi fa. Leggerle. Spesso il pattern emerge a occhio in modo che le metriche non vedono.

**Tipi di drift probabili:**

- **Drift di modello.** Il provider ha aggiornato il modello sotto. Verifica: nei log, la versione del modello è cambiata? Se sì, quando? Se non hai pinato la versione, è altamente probabile.
- **Drift di corpus.** La documentazione è stata aggiornata (riorganizzata, riscritta, deprecate vecchie sezioni). Verifica: pipeline di re-indicizzazione, date delle ultime modifiche ai documenti, statistiche dei chunk recuperati (sono gli stessi di otto mesi fa, o no?).
- **Drift di query.** I clienti enterprise usano il sistema in modi diversi rispetto a quando è stato calibrato (più complesso, più tecnico, su feature nuove non coperte dalla documentazione).
- **Drift di obiettivo.** Le aspettative sono cresciute. Quello che otto mesi fa era "sorprendentemente buono" oggi è "il minimo accettabile".

**Metriche che dovresti avere ma probabilmente non hai:**

- **Faithfulness sui campioni** — LLM-as-judge che valuta se la risposta è coerente con i chunk recuperati, su sampling stratificato.
- **Retrieval recall** — su un eval set, i chunk corretti sono ancora tra quelli recuperati?
- **Cluster delle query** — su periodi diversi, sono raggruppate negli stessi cluster, o sono apparsi cluster nuovi non coperti?
- **Coverage del corpus** — quali documenti vengono recuperati? Quali mai? La distribuzione è cambiata?
- **Eval suite che gira a ogni update** — quando il modello viene aggiornato, fai partire automaticamente una suite di valutazione su un set di prompt fissi e confronti i risultati.

**Mitigation di breve termine:**
- **Pin alla versione del modello** se non c'è già — fissa il modello a una versione che funzionava, se ancora disponibile.
- **Audit dell'indice.** Verifica che i documenti rilevanti siano indicizzati. Forza una re-indicizzazione completa.
- **Conversazione con i clienti enterprise insoddisfatti.** Capire esempi concreti di domande mal risposte è la fonte più ricca di insight.

**Fix strutturali:**
- **Eval suite automatica** che gira settimanalmente con report.
- **Monitoring di qualità in continuo** sul sample (LLM-as-judge asincrono).
- **Pin esplicito delle versioni dei modelli** in produzione.
- **Dashboard di drift** su query e corpus.
- **Pipeline di feedback chiusa**: i casi negativi del CSM finiscono in un dataset che alimenta l'eval suite.

**Prevenzione futura:**
La domanda chiave per il post-mortem: "quale segnale avrebbe permesso di accorgersene due mesi fa invece che ora?". Probabilmente: monitoring di qualità su sample (LLM-as-judge anche solo settimanale) + eval suite triggered su ogni update annunciato dal provider. La lezione 3.1 e la 6.3 messe in pratica fin dall'inizio.

**Trappole:**
- Trattare i thumbs come unica metrica — coverage 1-3%, bias forte, segnale debole.
- Concludere "il modello è degenerato" senza verificare se è stato aggiornato di versione — magari sì, e il problema è banalmente quello.
- Cambiare modello/prompt senza eval suite — rischi di peggiorare ulteriormente senza accorgertene.
- Ignorare il drift di corpus perché "l'indice è automatico" — l'automazione può aver tolto documenti utili o lasciato dentro versioni obsolete.

</details>

---

## Cosa fare se hai mancato molti punti

- Serving, gateway, motori di inferenza → **6.1 Serving e inference**
- Driver di costo, leve di ottimizzazione, alert → **6.2 Costi e FinOps a runtime**
- Drift, monitoring di qualità, tracing → **6.3 Monitoring e drift in produzione**
- Cosa loggare e LLM-as-judge → **3.1 LLM-as-judge** e **3.2 Observability**
- Tradeoff design su modello/contesto → **5.3 Il triangolo qualità-latenza-costo**
