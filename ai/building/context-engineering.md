---
title: "Context engineering: cosa mettere nel contesto, a scala"
sidebar_position: 3
---

# Context engineering: cosa mettere nel contesto, a scala

<div class="lesson-meta">
  <span class="badge-stato evoluzione">In evoluzione</span>
  <span>Lezione 1.3</span>
  <span>~11 min di lettura</span>
</div>

<p class="lesson-lead">Con context window da centinaia di migliaia di token, "metti tutto dentro" sembra la risposta ovvia. Non lo è: il costo cresce linearmente con i token, e l'attenzione del modello decade nel mezzo. Decidere cosa entra nel contesto, in che ordine e con quale budget è diventata una competenza a sé.</p>

Nella lezione 1.1 hai visto come recuperare i pezzi giusti di documento per rispondere a una query. RAG risolve il problema di *trovare* la conoscenza. Ma un problema adiacente si è ingrandito con l'arrivo delle context window enormi: anche trovato il pezzo giusto, **come costruisci il prompt finale?**

Context engineering è la risposta a questa domanda. Non è una tecnica sola: è la disciplina di orchestrare tutto ciò che entra nel contesto — system prompt, storia della conversazione, documenti recuperati, istruzioni specifiche, esempi — in modo da massimizzare la qualità dell'output mantenendo il costo sotto controllo.

## Context window grandi: strumento potente, non soluzione magica

I modelli moderni hanno context window enormi — 128k, 200k, anche 1 milione di token. Sembra che risolvano tutto: basta buttare dentro i documenti e via. In pratica, questo approccio ha due problemi seri.

**Il costo.** I provider addebitano a token: ogni token in input ha un prezzo. Una context window da 200k token piena a ogni chiamata può costare decine di volte più di una richiesta ottimizzata. Se il sistema riceve centinaia di richieste al secondo, la differenza è la sostenibilità economica del prodotto.

**Il degrado di attenzione: "lost in the middle".** Non tutto ciò che metti nel contesto viene "pesato" ugualmente dal modello. Studi empirici mostrano che i modelli prestano più attenzione all'inizio e alla fine del contesto, e perdono informazioni nel mezzo — anche quando quelle informazioni sono critiche per la risposta. Il fenomeno si chiama "lost in the middle" ed è stato documentato su modelli diversi e context window di diverse dimensioni.

<details>
<summary>Sotto il cofano: perché succede il "lost in the middle"</summary>

Il meccanismo di attenzione nei transformer non è uniforme. L'attenzione è pesata: ogni token produce un vettore di attenzione che punta verso le parti del contesto più rilevanti. Ma ci sono effetti di posizione: i token all'inizio e alla fine del testo ricevono sistematicamente attenzione maggiore, perché pattern simili erano dominanti nei dati di training (i testi umani mettono le cose importanti all'inizio o alla fine).

Risultato pratico: se il tuo chunk critico è sepolto tra token 20.000 e 21.000 in un contesto da 100k, ha meno probabilità di influenzare la risposta rispetto allo stesso chunk messo a inizio o fine contesto. Non è garantito che vada storta — ma il rischio è reale e misurabile.
</details>

## La gerarchia: cosa mettere dove

Context engineering pratica significa decidere la struttura del prompt in modo deliberato, non per default.

**Inizio del contesto — le istruzioni.** Il system prompt, il ruolo del modello, le regole fondamentali e i vincoli più importanti. I modelli rispettano meglio le istruzioni quando stanno all'inizio, dove l'attenzione è più forte.

**Fine del contesto — la domanda.** La query dell'utente va alla fine, appena prima che il modello generi. Messa in fondo, è anche l'ultimo "segnale" prima della generazione: pesa molto.

**Mezzo del contesto — i documenti recuperati.** I chunk del RAG, la storia della conversazione, il materiale di riferimento. Questa è la zona "lost in the middle": se hai un chunk critico, consideralo un'eccezione — mettilo all'inizio o in fondo, non nel mezzo.

**Ordine dei chunk recuperati.** Non mettere i chunk in ordine casuale. Se il reranker ha già ordinato per rilevanza, spesso vale la pena mettere i più rilevanti alla fine, vicino alla domanda — così il chunk migliore è il più "recente" nella finestra dell'attenzione.

## Il budget di contesto

Context engineering è anche gestione del budget — quanti token puoi usare, e come distribuirli.

Un sistema in produzione ha più componenti che competono per il contesto:

- **System prompt** — fisso, cambia raramente, ma può diventare lungo con le istruzioni.
- **Esempi few-shot** — aiutano la qualità ma consumano token.
- **Storia della conversazione** — in un chatbot, cresce a ogni turno.
- **Chunk recuperati da RAG** — il grosso, variabile per query.
- **Spazio per l'output** — il modello ha bisogno di token per rispondere.

La somma deve stare nel limite. In pratica, si fissa un budget per categoria: "system prompt max 500 token, storia max 2k token, chunk max 4k token". Quando la storia cresce troppo, si tronca o si comprime (riassunto degli ultimi N turni). Quando i chunk sono troppi, ne passi meno ma meglio selezionati.

<span class="badge-stato evoluzione">In evoluzione</span> **Prompt caching.** I principali provider offrono caching del prefix del prompt: se il system prompt o un documento di riferimento fisso è uguale tra richieste, il suo processing non viene riaddebitato. Riduce significativamente i costi per sistemi con molto contesto condiviso tra chiamate. Cerca "prompt caching" nella documentazione del provider che usi.

## Context engineering per gli agenti

In un sistema agentico (lezione 1.5), il contesto cresce a ogni passo: ogni tool call aggiunge risultati, ogni turno di ragionamento aggiunge testo. Senza gestione, dopo pochi passi il contesto è pieno o costosissimo.

Le strategie pratiche:

**Summarization progressiva.** Dopo ogni N passi, riassumi la storia dell'agente in un blocco compresso. L'agente continua con il riassunto, non con l'intera storia.

**Selezione mirata dei tool result.** Non mettere l'intera risposta di un tool nel contesto. Estrai solo ciò che è rilevante per il passo successivo — spesso è una riga su 100.

**Memory layer separato.** Alcune architetture separano la "memoria a lungo termine" dell'agente (un vector DB dove l'agente può recuperare fatti passati) dalla context window (la finestra di lavoro attuale). Il contesto rimane gestibile; le cose vecchie si recuperano se servono.

## Cosa NON è il context engineering

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Context window grande = posso ignorare il problema" | Il costo è reale e scala. E "lost in the middle" è documentato anche sulle window più grandi. |
| "L'ordine degli elementi nel prompt non conta" | Conta: inizio e fine hanno più peso nell'attenzione del modello. Le istruzioni critiche vanno all'inizio. |
| "Context engineering è solo RAG" | RAG è uno dei componenti. Context engineering è la disciplina di orchestrare tutto — system prompt, storia, chunk, esempi, spazio per l'output. |
| "Più contesto = sempre risposta migliore" | Oltre un certo punto, più contesto porta rumore, non segnale. E costa di più. |

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani. L'ultima anticipa una lezione futura.

1. Cos'è "lost in the middle" e perché succede?
2. Dove vanno le istruzioni più importanti nel contesto, e perché?
3. Hai un sistema RAG che recupera 10 chunk. Come decidi l'ordine in cui metterli nel prompt?
4. In un chatbot, la storia della conversazione cresce a ogni turno. Come gestiresti il budget del contesto?
5. Cos'è il prompt caching e quando conviene usarlo?
6. *(anticipazione)* In un agente multi-step, il contesto cresce a ogni passo. Quali sono le due strategie principali per tenerlo sotto controllo?

---

## Glossario

- **Context window** — il numero massimo di token che il modello può tenere in considerazione in una singola inferenza; include input e output.
- **Context engineering** — la disciplina di decidere cosa mettere nel contesto, in che ordine e con quale budget per massimizzare qualità e minimizzare costo.
- **Lost in the middle** — fenomeno per cui il modello presta meno attenzione alle informazioni nel mezzo del contesto rispetto all'inizio e alla fine.
- **Budget di contesto** — la distribuzione pianificata dei token disponibili tra i diversi componenti del prompt.
- **Prompt caching** — funzionalità dei provider che permette di non riaddebitare il processing di porzioni di prompt identiche tra richieste diverse.
- **Summarization progressiva** — tecnica usata negli agenti per comprimere la storia del ragionamento precedente e liberare spazio nel contesto.

---

## Per approfondire

- **"Lost in the Middle: How Language Models Use Long Contexts"** — il paper che ha documentato sistematicamente il fenomeno; cerca il titolo su arXiv.
- **Documentazione "Prompt Caching"** di Anthropic e OpenAI — spiega come il caching funziona in pratica e i requisiti per attivarlo.

*Risorse indicate per la ricerca; per i link aggiornati conviene cercarli al momento.*

---

## Prossima lezione

**1.4 Structured output e function calling.** Finora hai imparato come portare informazioni al modello. Il passo successivo è usare quello che il modello produce — non come testo libero, ma come struttura che il tuo sistema può consumare direttamente.
