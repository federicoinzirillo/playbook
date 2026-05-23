---
title: Gestire le allucinazioni
sidebar_label: "3.4 Allucinazioni"
sidebar_position: 4
---

# Gestire le allucinazioni

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 3.4</span>
  <span>~13 min di lettura</span>
</div>

<p class="lesson-lead">Un LLM ti dice con la stessa sicurezza che "Roma è la capitale d'Italia" e che un articolo di legge inventato esiste. Non sta mentendo — sta facendo benissimo il suo unico lavoro: prevedere parole plausibili. La trappola è che plausibile non vuol dire vero, e nessun prompt da blog risolve il problema. Capire il meccanismo è la condizione per mitigarlo davvero.</p>

Nella lezione 0.1 te l'avevamo lasciato come monito: un LLM non recupera fatti, predice testo plausibile. È il punto da cui partono le **allucinazioni** — risposte sbagliate ma confezionate con assoluta convinzione. Sono il fallimento più tipico dei sistemi LLM, e quello che fa più male in produzione: il sistema sembra funzionare, fino al momento in cui qualcuno scopre che la citazione legale che ha usato non esiste, o il prezzo che ha dato al cliente era inventato. Questa lezione spiega perché succede, come riconoscerle, e quali contromisure reggono davvero — non quelle che ti dice il primo tutorial.

## Perché succedono (è il meccanismo, non un bug)

Un LLM non ha un database in testa. Genera token a probabilità (lezione 0.1): dato un contesto, calcola quale token è più probabile dopo, lo sceglie, lo aggiunge, ricomincia. La frase "Il decreto legislativo 231 del 2001 disciplina la responsabilità amministrativa degli enti" è, per il modello, una sequenza con probabilità alta perché ne ha viste tante simili in addestramento. Anche "Il decreto legislativo 412 del 1998 disciplina X" può avere probabilità alta — se la struttura della frase, le parole tipiche di un testo legale, l'apparenza di precisione sono presenti — *anche se quel decreto non esiste*.

**Il modello non distingue tra plausibile e vero**. Distingue tra plausibile e implausibile. La verità non è una dimensione del suo spazio probabilistico; lo è la coerenza statistica con il testo su cui è stato addestrato.

Da qui due categorie di allucinazione, diverse per causa e mitigazione.

**Fabricazione fattuale.** Il modello inventa fatti specifici: numeri, nomi, date, citazioni, riferimenti. Una "URL fonte" che sembra reale e non esiste. Una statistica precisa che nessuno ha mai pubblicato. È la forma più pericolosa perché il pacchetto è convincente.

**Inconsistenza interna.** Il modello contraddice se stesso nella stessa risposta, o contraddice il contesto che gli hai dato. "Il numero è 42 (vedi sopra: 47)". Emerge soprattutto su risposte lunghe o ragionamenti multi-step. È più facile da catturare programmaticamente.

> **Nota** — Non chiamarla "bugia". La bugia presuppone intenzione, e il modello non ne ha. Chiamarla "errore" sottostima il fenomeno (suona come un bug occasionale). "Allucinazione" è imperfetta ma cattura il punto: il modello *vede* qualcosa che non c'è, in modo coerente con la sua percezione statistica.

## Grounding: il principio che le riduce

La leva più potente per ridurre le allucinazioni si chiama **grounding** — ancorare la risposta del modello a *fonti verificabili* presenti nel contesto. Quando il modello ha una fonte sotto mano nel prompt, non deve "ricordare" il fatto: lo legge.

Non elimina le allucinazioni, ma le rende:

1. **Più rare** — è meno probabile inventare quando hai il riferimento esatto da copiare.
2. **Verificabili** — la fonte è lì, qualcuno (o un controllo automatico) può verificarla.
3. **Diagnosticabili** — se il modello sbaglia, vedi se è perché la fonte era sbagliata o perché non l'ha rispettata.

Il sistema più comune di grounding lo conosci: si chiama **RAG** (lezione 1.1). I documenti rilevanti nel contesto + istruzione esplicita di basarsi solo su quelli. Ma il grounding va oltre RAG — vale ogni volta che fornisci al modello una fonte di verità: un risultato di un tool (lezione 1.4), una tabella di dati, una specifica tecnica passata nel prompt.

## Le quattro contromisure che reggono

In ordine di impatto, da quello che fa la differenza più grande a quello che è raffinatura.

### 1. RAG con citazione obbligatoria

RAG da solo non basta: il modello può ricevere i chunk giusti e produrre comunque una sintesi inventata. La mossa che chiude il buco: **citazione obbligatoria**, strutturata, verificabile.

In pratica:

- Output con structured output (lezione 1.4): la risposta è un oggetto con `risposta` + `citazioni: [chunk_id, ...]`
- Il prompt dice al modello: "Per ogni affermazione, indica il chunk_id da cui viene. Se l'informazione non è nei chunk, scrivi 'Non disponibile nelle fonti'."
- Il codice valida che ogni chunk_id citato sia effettivamente tra quelli passati nel prompt — se il modello inventa un chunk_id che non esiste, lo scarti.

Questo trasforma il problema da "il modello è affidabile?" a "il modello cita le fonti che ha visto?" — una proprietà molto più facile da controllare.

### 2. Istruzioni esplicite a "non rispondere se non sai"

I modelli hanno un bias verso la produzione di testo: continuano a generare anche quando dovrebbero astenersi. Il prompt deve contrastare quel bias esplicitamente.

Non basta scrivere "non inventare". Funziona meglio una formulazione concreta che dia un'uscita pulita: *"Se l'informazione non è chiaramente presente nel contesto fornito, rispondi esattamente 'Non trovo questa informazione nei documenti'. Non integrare con conoscenza generale."*

La frase fissa è importante: ti permette di rilevare programmaticamente quando il modello si è astenuto, separando i casi "ha provato" da "ha riconosciuto di non sapere".

### 3. Verifica nel codice

Dove il fatto è verificabile in modo strutturale, **verificarlo nel codice è meglio che sperare nel prompt**.

- Il modello produce un importo? Confrontalo con la somma calcolata dal codice sulle voci di dettaglio (vedi il drill della 2.6).
- Il modello cita un URL? Controlla che risponda 200, e che il dominio sia in una whitelist.
- Il modello cita un articolo di legge? Confrontalo con un indice deterministico degli articoli esistenti.
- Il modello produce un JSON? Validazione di schema, range check sui campi numerici.

La regola: ogni volta che esiste una funzione deterministica che può dire "questo è impossibile/inconsistente", chiamala. Non chiedere al modello di fare il check su se stesso (il "self-check" funziona molto meno di quanto dovrebbe in teoria).

### 4. Chain-of-thought e verifica esplicita

Per task di ragionamento, chiedere al modello di **mostrare i passaggi prima della risposta finale** (chain-of-thought, lezione 0.5) riduce certi tipi di allucinazione: i passaggi intermedi vincolano la risposta finale a una catena coerente, più difficile da rompere con un'invenzione casuale.

Variante più forte: **self-consistency**. Generi la risposta più volte (con temperature > 0) e prendi la maggioranza. Se il modello sa la risposta, le N generazioni convergono. Se sta allucinando, divergono. Costa N volte di più ma su decisioni critiche vale.

## Cosa NON funziona (anche se sembra)

Tre interventi che girano nei tutorial e in produzione tengono molto meno di quello che promettono.

**"Temperature a 0 elimina le allucinazioni."** Riduce la varianza, non l'allucinazione. Se la risposta più probabile è già un'invenzione (perché è un pattern statisticamente plausibile), temperature 0 te la dà più volte di fila. Aumenta la *riproducibilità* di una risposta sbagliata, non la sua correttezza.

**"Cambio modello con uno più grande / più recente."** I modelli più grandi allucinano *meno* su task generali, ma **non smettono di allucinare**. Su domini specifici, su numeri precisi, su citazioni legali, il modello più nuovo può allucinare in modi diversi — non meno. È raramente la mossa giusta come prima contromossa.

**"Aggiungo nel prompt 'non inventare informazioni'."** Aiuta marginalmente, non risolve. Il modello è statistico: una frase di vincolo entra nella distribuzione, ma non è un guardrail strutturale. Funziona meglio se combinato con l'istruzione esplicita di astenersi (contromossa #2 sopra).

**"Self-check: chiedo al modello di verificare la propria risposta."** Funziona meno di quanto dovrebbe. Il modello che ha appena inventato tende a confermare la propria invenzione — è coerente con se stesso, anche quando si sbaglia. Il self-check è utile come segnale debole, non come guardrail.

## Misurare le allucinazioni

Una mitigazione che non puoi misurare non sai se funziona. La pratica standard:

1. **Golden dataset** (lezione 3.2) con casi noti di "trabocchetto" — domande dove la risposta corretta è "non lo so" o richiede informazioni specifiche che non sono nel contesto.
2. **Criterio "fedeltà" nel giudice** — il giudice valuta se la risposta è ancorata al contesto o ha inventato qualcosa.
3. **Metrica dedicata: hallucination rate** = (risposte che il giudice marca come "infedeli al contesto" o "fattualmente sbagliate") / (totale risposte).

Trackare questa metrica nel tempo, agganciata al tracing (lezione 3.3), è il modo per accorgersi prima che lo veda un cliente.

<details>
<summary>Sotto il cofano: perché il "self-check" del modello funziona poco</summary>

Quando il modello ha generato una risposta sbagliata, il testo di quella risposta è ora nel suo contesto. Per il meccanismo della generazione (lezione 0.1), il modello tratta il proprio output come "qualcosa di plausibile che è già stato scritto" — e dunque la sua valutazione successiva tende a essere consistente con esso.

Risultato pratico: se chiedi "controlla che quanto hai scritto sia corretto", il modello cerca conferme della propria risposta più di quanto cerchi contraddizioni. Il self-check funziona un po' di più quando lo fai in una **sessione separata** (senza la risposta precedente nel contesto), ma anche lì è un segnale debole.

Per questo le contromisure forti sono *esterne* al modello: validazione nel codice, RAG con verifica delle citazioni, golden dataset con casi trabocchetto. Il modello da solo non si autocorregge in modo affidabile.
</details>

## Cosa NON sono le allucinazioni

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Sono un bug che verrà risolto con la prossima versione del modello" | Sono una proprietà del meccanismo (predizione probabilistica), non un difetto di implementazione. Si riducono, non si eliminano. |
| "RAG elimina le allucinazioni" | Le riduce molto. Ma il modello può ricevere i chunk giusti e produrre comunque una sintesi inventata, o citare chunk che non gli sono stati passati. Servono citation enforcement e validazione. |
| "Temperature 0 risolve il problema" | Riduce la varianza, non l'invenzione. Una risposta sbagliata diventa solo più riproducibile. |
| "Chiedo al modello di verificare se stesso ed evito l'allucinazione" | Il self-check è un segnale debole: il modello tende a confermare la propria risposta. Le contromisure forti sono esterne: codice, citazioni verificabili, golden dataset. |

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani. L'ultima anticipa una lezione futura.

1. Perché un'allucinazione è una conseguenza del meccanismo, non un bug?
2. Cos'è il grounding e perché RAG ne è l'esempio principale?
3. Come trasformi il problema "il modello è affidabile?" in "il modello cita le fonti che ha visto?" — passo a passo.
4. Tre interventi che *sembrano* risolvere le allucinazioni ma in pratica funzionano meno del previsto. Quali e perché.
5. Come misureresti il tasso di allucinazione di un sistema in produzione, sfruttando quello che hai visto nelle lezioni 3.2 e 3.3?
6. Un assistente legale produce una citazione di legge inesistente. Quali contromisure metti, in che ordine?
7. *(anticipazione)* Hai un agente che usa tool. Un'allucinazione in un agente non è solo "una frase sbagliata" — può tradursi in un'azione sbagliata. Come cambia la natura del problema?

---

## Glossario

- **Allucinazione** — risposta del modello fattualmente sbagliata ma presentata con sicurezza; conseguenza del meccanismo probabilistico, non un bug.
- **Fabricazione fattuale** — il tipo di allucinazione in cui il modello inventa fatti specifici (date, numeri, citazioni, URL).
- **Inconsistenza interna** — il tipo di allucinazione in cui il modello contraddice se stesso o il contesto fornito nella stessa risposta.
- **Grounding** — ancorare la risposta del modello a fonti verificabili presenti nel contesto, riducendo lo spazio in cui il modello deve "inventare".
- **Citation enforcement** — pratica di forzare il modello a citare fonti strutturate (chunk_id, ID di documento) e validarle nel codice.
- **Self-consistency** — generare la stessa risposta più volte e prendere la maggioranza; utile per task di ragionamento critico, ma costoso.
- **Hallucination rate** — la metrica che misura il tasso di risposte allucinate sul totale, calcolata via LLM-as-judge + criterio di fedeltà.
- **Self-check** — pratica di chiedere al modello di verificare la propria risposta; segnale debole, non sostituisce le verifiche esterne.

---

## Per approfondire

- **"Survey of Hallucination in Natural Language Generation"** — survey accademico che classifica i tipi di allucinazione e le contromisure principali; cerca il titolo su arXiv.
- **Documentazione "Reducing hallucinations"** di Anthropic e OpenAI — i provider documentano le best practice per i loro modelli; cerca "hallucinations" sui rispettivi siti tecnici.
- **Ricerca sul "faithfulness" nei sistemi RAG** — letteratura specifica sul grounding e la fedeltà dei sistemi retrieval-augmented; cerca "RAG faithfulness" o "retrieval faithfulness" su arXiv.

*Risorse indicate per la ricerca; per i link aggiornati conviene cercarli al momento.*

---

## Prossima lezione

**3.4 Valutare il comportamento agentico end-to-end.** Una risposta sbagliata di un chatbot è fastidiosa. Una decisione sbagliata di un agente che può eseguire tool — scrivere file, chiamare API, modificare un database — è un *evento*. Valutare un agente non è valutare l'ultimo token: è valutare la *traiettoria* di decisioni, la scelta dei tool, il completamento del task. È il buco che si sta allargando più in fretta nel settore.
