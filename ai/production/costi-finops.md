---
title: Costi e FinOps a runtime
sidebar_label: "6.2 Costi e FinOps"
sidebar_position: 2
---

# Costi e FinOps a runtime

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 6.2</span>
  <span>~11 min di lettura</span>
</div>

<p class="lesson-lead">La 5.3 ti ha fatto ragionare sul triangolo qualità-latenza-costo in fase di design. Qui parliamo del giorno-2: la fattura che arriva, il dashboard che lampeggia rosso, il PM che chiede "perché stiamo spendendo il triplo rispetto al mese scorso?". FinOps per l'AI è il mestiere di mantenere prevedibili i costi di un sistema il cui prezzo dipende da cosa fanno gli utenti.</p>

Una delle differenze più sgradevoli tra software tradizionale e sistemi LLM è che il costo marginale per richiesta è grosso, variabile e fuori dal tuo controllo diretto. Un utente che fa una query di 50 token costa frazioni di centesimo; un utente che incolla un PDF intero costa un dollaro. Stessa interfaccia, stessa "richiesta" dal punto di vista del business, costi 100x diversi. Il problema FinOps comincia qui.

## I tre driver del costo a runtime

Tutto il costo di un sistema LLM in produzione si riconduce a tre voci:

**Token consumati** — il driver principale per chi usa API esterne. Costo = (token input + token output × moltiplicatore) × prezzo per token × richieste. Sembra banale, ma dietro ognuna di queste variabili c'è una leva.

**Compute riservato** — il driver principale per chi serve modelli in proprio. GPU costano per ora di running, non per token effettivo. Una GPU al 20% di utilizzo costa quanto una al 90%. Il costo si abbatte usandola, non spegnendola (che è raramente possibile per la latenza richiesta).

**Storage e infrastruttura accessoria** — vector store, log, observability, banda. Singolarmente piccoli, sommati significativi. Spesso ignorati nei conti iniziali e poi sorpresa a fine mese.

La struttura del costo dipende dalla scelta architetturale di base. API esterna = costo proporzionale al volume. Serving in proprio = costo fisso del compute + storage. Sistemi ibridi hanno entrambe le strutture sovrapposte.

## L'osservabilità del costo: prima di ottimizzare, misurare

Il primo errore in FinOps AI è ottimizzare prima di sapere dove vanno i soldi. La domanda corretta non è "come riduciamo i costi" ma "quale operazione, quale utente, quale path costa di più".

**Cosa devi loggare per ogni chiamata al modello:**

- Modello usato (con versione)
- Token input e token output
- Costo stimato (calcolato al volo dal pricing del provider)
- Tenant / utente / feature applicativa
- Cache hit (se la cache ha servito la richiesta, costo = 0)
- Tempo di esecuzione

Aggregando questi log per dimensione (per tenant, per feature, per modello) ottieni le risposte vere: "il 70% del costo viene dal 5% degli utenti", "la feature X costa il triplo della Y nonostante metà del traffico", "le richieste con allegato PDF costano 50x la media".

Senza questi dati, ogni discussione FinOps è opinione.

<details>
<summary>Dove vive questo logging</summary>

Idealmente nel **gateway LLM** (lezione 6.1): è il punto di passaggio di tutte le chiamate, vede modello, token, latenza. Tutti i gateway moderni (LiteLLM, Portkey, Helicone) hanno questa istrumentazione di base. Da lì il dato finisce in un sistema di analytics (BigQuery, ClickHouse, anche un Postgres per volumi modesti) dove costruisci le viste per tenant/feature/modello. La visualizzazione la fai dove preferisci (Grafana, Metabase, una dashboard custom). Il punto chiave: il logging è in linea con la chiamata, non un'integrazione che puoi rimandare.
</details>

## Le leve concrete sul costo

Quando sai dove vanno i soldi, hai cinque leve principali. In ordine di impatto tipico:

**Caching.** È sempre la prima leva e quella con miglior rapporto sforzo/risultato. Cache esatta (stessa query → risposta cached) elimina costo e latenza. Cache semantica (query simile sopra soglia di similarità) richiede embedding ma copre molti più casi. Su sistemi reali si vedono hit rate del 20-50% — significa abbattere il costo della stessa percentuale.

**Routing intelligente del modello.** Non tutte le query meritano il top di gamma. Un classificatore (anche semplice, anche basato su regole) instrada le query semplici su un modello piccolo (Haiku 4.5, GPT-5.3 Instant, Gemini 3 Flash, o un modello open-weight locale) e solo quelle complesse sul grande (Opus 4.7, GPT-5.4, Gemini 3.1 Pro). Riduzione del costo medio del 40-70% su workload reali, senza perdita percepita di qualità se il routing è ben calibrato.

**Ottimizzazione del prompt.** Prompt più corti = meno token input. Spesso i prompt di sistema crescono per accrescimento — ogni volta che qualcosa va storto si aggiunge una riga di "non fare X". Dopo un anno il prompt di sistema è 2000 token, di cui metà non servono più. Revisione periodica obbligatoria. Stesso discorso per il contesto recuperato in RAG — se ne metti 8000 token quando ne bastano 2000, paghi 4x senza vantaggio.

**Compressione del contesto.** Quando il prompt è grande per natura (documenti lunghi), tecniche di sintesi (riassumere chunk meno rilevanti) o di **prompt compression** (modelli specializzati che riducono i token mantenendo il significato, come LLMLingua) riducono i token input significativamente. Trade-off: complessità in più, qualche perdita di fedeltà.

**Limiti di consumo per tenant.** Rate limiting e quota per utente/cliente. Protegge da utenti malevoli o malfunzionamenti applicativi che si moltiplicano (il classico bug che fa scattare retry infiniti). Non riduce il costo "buono", taglia quello "cattivo".

## Il costo che non vedi: le strutture nascoste

Tre voci che spesso non finiscono nelle stime iniziali e diventano problemi in produzione.

**Retry e fallimenti.** Ogni retry costa quanto la chiamata originale. Un sistema con retry aggressivi su errori transitori può pagare 2-3x il costo "previsto". Il logging dei retry separati dalle chiamate iniziali è essenziale per scoprirlo.

**Conversazioni che crescono.** In un sistema agentico o conversazionale, il contesto accumulato cresce a ogni turno. Una conversazione a 20 turni con storia completa nel prompt costa molto più della somma dei singoli messaggi. Strategie: sommarizzazione della storia, sliding window, retrieval sulla storia invece di passarla tutta.

**Tool calling che si moltiplica.** Un agente che chiama tool in loop può fare 10-20 chiamate al modello per una singola richiesta utente. Se non monitorate, queste richieste "interne" possono superare il costo delle richieste "esterne". Vanno loggate e contate.

## La gestione operativa: alert, budget, incident

Un sistema LLM in produzione ha bisogno di **alert sul costo** come ne ha sulla latenza. Configurazione tipica:

- **Budget mensile per ambiente** (dev, staging, prod) — alert al 50%, 80%, 95% del budget.
- **Anomaly detection sul costo orario** — se il costo dell'ultima ora è 3x la media delle ultime 24, alert.
- **Cost per request P95** — se il 95° percentile del costo per request raddoppia, qualcosa è cambiato (utenti che incollano contenuti enormi, prompt che è cresciuto, retry che si moltiplicano).
- **Tenant top spender** — chi sta spendendo di più questa settimana, e perché.

Quando scatta un alert da "costi raddoppiati", l'incident response ha una sequenza precisa: (1) confermare l'anomalia non è un bug del logging; (2) identificare la dimensione che è esplosa (tenant, feature, modello); (3) capire se è traffico legittimo (cresciuto naturalmente) o anomalo (bug, abuso); (4) decidere se mettere un cap temporaneo e indagare, o lasciar andare se è crescita sana.

## Cosa NON è il FinOps AI

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Cambieremo modello quando i costi salgono" | Il cambio modello è la leva più lenta e rischiosa. Caching e routing vengono prima. |
| "Una richiesta costa X" | Il costo per richiesta ha distribuzione lunga: la mediana è bassa, il P99 può essere 100x. La media è ingannevole. |
| "Tanto è cloud, scaliamo per natura" | Senza limiti per tenant, un cliente o un bug può triplicare il costo in un giorno. |
| "Lo monitoreremo dopo il lancio" | I costi che esplodono sono difficili da diagnosticare a posteriori senza il logging giusto fin dall'inizio. |

## Cosa dura, cosa evitare

Dura: **il logging granulare per dimensione** (tenant, feature, modello), **caching e routing come prime leve**, **alert sul costo trattati come alert operativi**.

Cambia velocemente: i **prezzi assoluti dei modelli** (in calo costante da anni — non costruire il business case sul prezzo di oggi se il sistema vivrà più di un anno) e **quale modello è migliore per quale fascia di costo** (la frontiera si sposta ogni pochi mesi).

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani.

1. Quali sono i tre driver principali del costo a runtime in un sistema LLM?
2. Perché il logging granulare è prerequisito di qualunque ottimizzazione?
3. In che ordine valuti le leve di ottimizzazione, e perché?
4. Quali sono tre voci di costo "nascoste" che spesso non finiscono nelle stime iniziali?
5. Riceve un alert "il costo orario è 3x la media". Qual è la sequenza di indagine?

---

## Glossario

- **FinOps** — disciplina di gestione finanziaria del cloud (e dell'AI) che combina engineering, finance e operations.
- **Cache hit rate** — percentuale di richieste servite dalla cache invece che chiamando il modello.
- **Routing intelligente del modello** — instradamento della richiesta a un modello più piccolo o più grande in base a regole o classificatore.
- **Prompt compression** — tecniche di riduzione dei token del prompt mantenendo il significato (es. LLMLingua).
- **Anomaly detection sul costo** — alert quando la metrica di costo devia significativamente dal baseline storico.
- **Cap per tenant** — limite di consumo (in richieste o costo) applicato per cliente/utente.

---

## Per approfondire

- **Cloud FinOps Foundation** — riferimento generale al FinOps; il framework si applica adattato all'AI.
- **Documentazione di pricing dei principali provider** (OpenAI, Anthropic, Google) — leggi attentamente. I dettagli (input vs output, prompt caching, batch API) cambiano l'aritmetica reale.
- **LLMLingua** su GitHub — strumento concreto di prompt compression, utile per capire il trade-off.

---

## Prossima lezione

**6.3 Monitoring e drift in produzione.** Hai i costi sotto controllo. Adesso il problema più subdolo: la qualità che degrada lentamente, senza che nessuno se ne accorga. Nessun errore, nessun alert — solo gli utenti che pian piano si fidano meno. È il giorno-2 più cattivo da affrontare.
