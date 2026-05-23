---
title: "LLM vs ML classico: quando NON usare la GenAI"
sidebar_label: "0.4 LLM vs ML classico"
sidebar_position: 4
---

# LLM vs ML classico: quando NON usare la GenAI

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 0.4</span>
  <span>~11 min di lettura</span>
</div>

<p class="lesson-lead">Non ogni problema ha bisogno di un LLM — e molti stanno meglio senza. Capire quando il ML classico, una regola deterministica, o nessun modello è la scelta giusta è la prima decisione architettuale di qualunque sistema AI.</p>

Nella lezione 0.3 hai visto che i modelli imparano dai dati, che il fine-tuning sposta i pesi, e che overfitting e bias sono trappole reali. Ma c'è una domanda ancora più a monte, che si pone prima di qualsiasi scelta tecnica: **devo usare un LLM? Devo usare un modello, in assoluto?**

La risposta ovvia sarebbe "beh, dipende dal problema". Giusto — ma in pratica si vede spesso il contrario: il LLM viene scelto per default, senza chiedersi se esiste qualcosa di più semplice che funziona meglio. Questa lezione è un vaccino contro quell'errore.

## Il problema del martello dorato

Quando hai un martello, tutto sembra un chiodo. Nella fase attuale dell'AI, il martello in questione è il LLM — e l'entusiasmo attorno a questi strumenti ha reso ancora più facile il riflesso condizionato: "facciamo una cosa con l'AI" → "usiamo un LLM".

Il problema non è che gli LLM siano cattivi strumenti. È che sono costosi, imprevedibili su task strutturati, e non interpretabili. Per una categoria intera di problemi esistono soluzioni più economiche, più veloci, più accurate e più semplici da mantenere. Scegliere un LLM lì è come demolire un muro con un bazooka quando bastava un martello.

**La regola pratica:** prima di scrivere una riga di codice con un LLM, chiediti: "C'è qualcosa di più semplice che risolve il 90% del problema?" Se la risposta è sì, inizia da lì.

## La mappa degli strumenti

Non esiste una tassonomia ufficiale, ma pensarci su quattro livelli aiuta a strutturare il ragionamento.

**Livello 1 — Regole deterministiche.** Se la logica è scrivibile a mano con if/else o espressioni regolari, fallo. Costo: quasi zero. Interpretabilità: massima. Manutenzione: facile. Esempio: "se il campo email non contiene @, rifiuta il form". Nessun modello necessario.

**Livello 2 — ML classico.** Regressione logistica, random forest, gradient boosting (XGBoost, LightGBM), SVM. Addestrati su dati etichettati, producono output strutturati — una classe, un numero, una probabilità. Veloci in inferenza, almeno parzialmente interpretabili, economici. Dominano su dati tabulari.

**Livello 3 — Deep learning specifico.** Reti convoluzionali per le immagini, modelli BERT-like per NLP classico (classificazione di testo, riconoscimento di entità, sentiment). Prima degli LLM generalisti, erano lo stato dell'arte su task specifici — e su quei task specifici restano competitivi, spesso superiori.

**Livello 4 — LLM.** Modelli linguistici grandi, costosi, potenti su task aperti. La scelta giusta per task che richiedono generazione di testo libero, ragionamento su input non strutturato, comprensione di istruzioni complesse in linguaggio naturale.

**Il punto chiave:** i livelli superiori non sostituiscono quelli inferiori. Sono strumenti diversi per problemi diversi. Un sistema sano spesso li combina.

## Quando il ML classico vince

Il ML classico e il deep learning specifico battono i LLM in diverse situazioni concrete.

**Dati tabulari strutturati.** Hai una tabella con colonne ben definite — età, reddito, storico acquisti, categoria — e vuoi prevedere un valore o classificare. Random forest o XGBoost sono lo standard: accurati, veloci, interpretabili, e non richiedono GPU da migliaia di euro al mese. Un LLM che "legge" una tabella in linguaggio naturale è un workaround inutilmente complicato quando esiste la strada diretta.

**Classificazione di testo con classi fisse.** Hai 10 categorie di ticket di supporto. Le vuoi classificare automaticamente. Un modello BERT fine-tunato su qualche migliaio di esempi etichettati supera spesso un LLM via API — e costa centesimi invece di dollari a scala. I LLM generalisti arrivano benissimo all'80-90% senza training, ma su certi casi di confine il modello specializzato prende quel delta finale che fa la differenza.

**Anomaly detection.** Rilevare frodi, log anomali, sensori fuori range. Qui servono modelli statistici (isolation forest, autoencoder) o regole su soglie, non generazione di testo.

**Latenza e costo a grande volume.** Un endpoint che risponde in 5 ms su 10.000 richieste al secondo non può chiamare un LLM per ogni richiesta. Il costo sarebbe proibitivo, la latenza inaccettabile. Un modello classico, una volta addestrato, fa inferenza in microsecondi su CPU commodity.

**Regolamentazione e interpretabilità.** In certi settori — finanza, sanità, assicurazioni — devi spiegare perché hai preso una decisione. Una regressione logistica o un albero decisionale ti danno i pesi espliciti, i feature importance, la traiettoria della decisione. Un LLM è una scatola nera, e "il modello ha detto così" non è una spiegazione accettabile per un regolatore.

## Quando una regola batte tutto

Prima ancora del ML, c'è lo scenario in cui non serve nessun modello.

**La logica è completamente deterministica.** "Blocca la richiesta se proviene da un IP in lista nera." "Invia la notifica se il balance scende sotto 100 euro." Non sono previsioni statistiche: sono regole. Un if/else gestisce il caso perfettamente, non sbaglia mai, è istantaneo e non richiede dati di training.

**Il pattern è semplice e stabile.** Estrarre un numero di telefono da un testo. Validare un codice fiscale. Formattare una data. Le espressioni regolari esistono per questo — e lo fanno meglio di qualsiasi modello.

**Hai bisogno di garanzie forti.** "Se accade A, deve sempre succedere B, senza eccezioni." I modelli producono probabilità, non certezze. Se hai bisogno di un comportamento garantito al 100%, hai bisogno di codice deterministico — o di un livello di validazione programmatica che trasformi l'output probabilistico del modello in una decisione vincolata.

Una pipeline ibrida è normale e sana: regole deterministiche per i casi facili e frequenti, modello per i casi ambigui che le regole non coprono. Non è un compromesso: è buona architettura.

## La griglia di decisione

Queste domande, nell'ordine, orientano la scelta:

1. **La logica è scrivibile a mano senza errori?** → Scrivi il codice. Non serve un modello.
2. **L'output è strutturato (classe, numero, sì/no) e hai dati etichettati?** → Considera prima ML classico.
3. **Il task riguarda testo, ma con categorie fisse e output strutturato?** → Considera deep learning specializzato (BERT, modelli encoder) prima di un LLM generalista.
4. **Il task richiede generazione, ragionamento, o comprensione di istruzioni complesse in linguaggio naturale?** → LLM è la scelta naturale.
5. **La latenza deve essere sotto i 100ms o il volume è sopra le 1.000 richieste al secondo?** → LLM in tempo reale è difficilmente sostenibile; valuta caching, batching, o modelli locali più piccoli.
6. **Il dominio richiede interpretabilità legale o regolamentare?** → LLM usabile solo come supporto, mai come decision-maker finale autonomo.

Non è una ricetta meccanica: è un modo di strutturare il ragionamento. Il punto 4 non dice che il LLM sia sempre migliore; dice che lì vale la pena considerarlo seriamente.

## Cosa NON è questa distinzione

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Se c'è AI nel nome del progetto, serve un LLM" | Un LLM è uno strumento tra tanti. Spesso un modello classico o una regola fa il lavoro meglio, più velocemente e a un decimo del costo. |
| "Un LLM è sempre più accurato di un modello classico" | Su dati tabulari strutturati e classificazioni con categorie fisse, il ML classico vince di regola. Il LLM brilla su task aperti e linguaggio naturale. |
| "Il ML classico non scala" | Scala benissimo — spesso meglio degli LLM in latenza e costo a grande volume. È un mito degli anni 2000, non del presente. |
| "Usare un LLM non richiede dati" | Vero per zero-shot su task generici. Per la qualità alta in produzione, i dati di valutazione (e spesso di fine-tuning) sono comunque indispensabili. |

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani. L'ultima anticipa la prossima lezione.

1. Cita due scenari concreti in cui un modello classico batte un LLM.
2. Quando ha senso usare una regola deterministica invece di qualsiasi modello?
3. Hai un dataset di transazioni (timestamp, importo, paese, tipo di carta) e vuoi rilevare le frodi. Cosa scegli, e perché?
4. Hai un chatbot aziendale che deve rispondere a domande libere in linguaggio naturale sulla documentazione interna. Cosa scegli, e perché?
5. Cosa significa che un modello è "interpretabile" e perché importa in certi settori regolamentati?
6. *(anticipazione)* Hai deciso che il tuo task richiede un LLM. La prossima domanda è: come gli parli in modo che capisca bene cosa vuoi? Come si chiama questa disciplina?

---

## Glossario

- **ML classico** — famiglia di algoritmi di machine learning non basati su reti neurali profonde: regressione logistica, random forest, XGBoost, SVM. Forti su dati strutturati.
- **Dati tabulari** — dati organizzati in righe e colonne con tipi ben definiti; il terreno di elezione del ML classico.
- **Regola deterministica** — logica if/else o pattern matching che produce sempre lo stesso output dato lo stesso input, senza approssimazione statistica.
- **Interpretabilità** — capacità di spiegare perché un modello ha prodotto un certo output; i modelli classici sono più interpretabili degli LLM.
- **Anomaly detection** — rilevamento di pattern inusuali in un dataset; tipicamente risolto con modelli statistici, non con LLM.
- **Latenza di inferenza** — il tempo che un modello impiega a produrre un output; gli LLM sono molto più lenti dei modelli classici.
- **Pipeline ibrida** — sistema che combina regole deterministiche e modelli ML, ciascuno usato per i casi a cui è più adatto.

---

## Per approfondire

- **Documentazione di scikit-learn** — la libreria Python che implementa la maggior parte del ML classico. Ottima per vedere cosa esisteva e cosa esiste ancora prima degli LLM.
- **"The Bitter Lesson" di Rich Sutton** — riflessione storica su come i sistemi che scalano sui dati battono quelli con conoscenza scritta a mano; utile per il contesto di lungo periodo.
- **"Machine Learning Engineering" di Andriy Burkov** — copre il ciclo di vita dei sistemi ML classici in produzione. Cerca il titolo su Google.

*Risorse indicate per la ricerca; per i link aggiornati conviene cercarli al momento.*

---

## Prossima lezione

**0.5 Prompt engineering come disciplina.** Hai deciso che il tuo task merita un LLM. Bene. Ma la qualità dell'output dipende da come gli parli — e non in modo vago. Il prompt engineering è una disciplina vera, misurabile, con principi che reggono in produzione. È il primo strumento pratico che usi con un LLM, ed è il prerequisito di tutto ciò che segue.
