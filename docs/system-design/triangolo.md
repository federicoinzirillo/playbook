---
title: Il triangolo qualità-latenza-costo
sidebar_position: 3
---

# Il triangolo qualità-latenza-costo

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 5.3</span>
  <span>~10 min di lettura</span>
</div>

<p class="lesson-lead">Ogni decisione su un sistema AI — quale modello, quanto contesto, caching sì o no, agente o pipeline — sposta il cursore su tre assi: qualità della risposta, latenza, costo per richiesta. Non puoi massimizzarli tutti e tre. La bravura sta nel capire quale ottimizzare per il tuo caso e quali sacrificare.</p>

Il triangolo qualità-latenza-costo non è una metafora vaga. È il frame che applichi a ogni decisione architettuale. Il modello migliore è più lento e più caro. Il caching riduce il costo ma sacrifica la freschezza delle risposte. Un agente con 10 iterazioni produce risultati migliori ma la latenza è 10 volte quella di una singola chiamata. Capire esplicitamente cosa stai ottimizzando ti fa fare scelte consapevoli invece di procedere per tentativi.

## I tre assi

**Qualità** — quanto la risposta è corretta, utile, aderente all'intenzione dell'utente. Si misura con eval (lezione 3.1), non a occhio in produzione. La qualità dipende principalmente da: scelta del modello, ricchezza del contesto, qualità del retrieval, prompt engineering.

**Latenza** — il tempo che l'utente aspetta. Si misura in due punti: TTFT (time to first token — quando arriva la prima parola) e latenza totale (quando la risposta è completa). Dipende da: modello (i modelli grandi sono più lenti), lunghezza dell'output, numero di iterazioni agentiche, overhead dell'orchestratore, latenza di rete.

**Costo** — il costo per richiesta, tipicamente in token. Dipende da: modello scelto, lunghezza del contesto (input tokens), lunghezza dell'output (output tokens), numero di chiamate. I token di output costano tipicamente 3-5 volte di più dei token di input.

**La relazione tra i tre:** non è un triangolo equilatero. Qualità e costo sono spesso correlati — modelli migliori costano di più. Ma qualità e latenza non lo sono necessariamente: con il caching puoi avere qualità alta e latenza bassa sulle richieste ripetitive, a costo di freshness. La compressione del contesto può ridurre costo e latenza con impatto limitato sulla qualità, se fatta bene.

## Le leve che sposti

### Scelta del modello

La leva con più impatto su tutti e tre gli assi. Modelli più grandi (GPT-4o, Claude Opus, Gemini Ultra) danno qualità superiore su task complessi, ma costano di più e sono più lenti. Modelli più piccoli (GPT-4o mini, Claude Haiku, Gemini Flash) sono veloci ed economici, con qualità sufficiente per molti task.

La domanda corretta non è "quale modello è il migliore". È: **quale modello è sufficiente per il mio task specifico?** L'overkill è un costo reale.

Un pattern comune: modello piccolo per il routing e le domande semplici, modello grande solo per i task complessi che lo richiedono davvero. Il routing intelligente può ridurre il costo del 60-70% con impatto minimo sulla qualità percepita.

### Lunghezza del contesto

Ogni token nel contesto è un costo e contribuisce alla latenza. Un contesto di 10.000 token costa 10 volte di più di uno da 1.000 token. Strategie per gestirlo:

**Compressione del contesto** — riassumere le parti della conversazione che non servono più in dettaglio. Una conversazione da 50 messaggi si comprime in un riassunto da 200 token + gli ultimi 5 messaggi integrali.

**Retrieval selettivo** — invece di mettere tutti i documenti nel contesto, recuperare solo le parti rilevanti (RAG). Il retrieval ben fatto porta meno token al modello, con qualità uguale o migliore.

**Prompt engineering** — un prompt più corto che dice la stessa cosa. Non banale, ma ogni token risparmiato nel system prompt si moltiplica per ogni richiesta.

### Caching

Il caching è la leva più potente per ridurre costo e latenza senza sacrificare qualità — ma ha un costo: la freschezza dei dati.

**Cache esatta** — stessa stringa di input → stesso output. Funziona bene per task altamente ripetitivi (FAQ, query standard). Il costo della cache hit è quasi zero: latenza di ms, costo di storage.

**Cache semantica** — input simili → risposta già calcolata, recuperata per similarità vettoriale. Più flessibile, copre più casi, ma richiede una soglia di similarità ben calibrata: troppo bassa restituisce risposte scorrette, troppo alta non fa cache di nulla.

**Prefix caching (KV cache)** — funzionalità offerta da alcuni provider: se il system prompt è identico tra molte richieste, il costo del calcolo di quei token viene ridotto. Rilevante per system prompt lunghi usati in produzione.

**Il trade-off freschezza:** una risposta in cache potrebbe essere obsoleta se il contesto è cambiato. Per dati che cambiano (prezzi, inventory, notizie), la cache va usata con TTL (time to live) coerente con la frequenza di aggiornamento dei dati.

### Agente vs pipeline singola

Un agente con N iterazioni ha latenza N volte quella di una singola chiamata, e costo proporzionale. La qualità può essere superiore — può correggere errori, recuperare informazioni aggiuntive, pianificare meglio.

La domanda: questo task richiede davvero il ragionamento iterativo, o una pipeline ben strutturata con una singola chiamata funziona altrettanto bene?

Spesso la risposta è: la pipeline funziona per il 90% dei casi, l'agente serve per il 10% più complesso. Progettare il sistema per gestire entrambi — pipeline di default, escalation all'agente su segnale — è un pattern comune.

## Il punto di lavoro: dove si trova il tuo sistema

Ogni sistema ha un "punto di lavoro" sul triangolo — la combinazione di qualità, latenza e costo che stai ottenendo oggi. L'ottimizzazione è spostare quel punto nella direzione che conta per il tuo caso.

Le domande da porsi:
1. **Qual è il vincolo stringente?** Se l'utente aspetta già 3 secondi di troppo, la latenza è il problema da risolvere, non la qualità.
2. **Dove stai sprecando?** Se il 40% delle query sono identiche, il caching ti dà un risparmio immediato.
3. **Qual è la qualità minima accettabile?** Se il modello più piccolo la raggiunge, non serve quello grande.
4. **Il costo scala linearmente col volume?** Se sì, il margine si restringe con la crescita — ottimizza prima di scalare, non dopo.

## Cosa NON è il triangolo

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Uso sempre il modello migliore per sicurezza" | Overkill sistematico moltiplica i costi senza migliorare la qualità su task semplici. |
| "Il caching è per i pigri" | Il caching è una scelta di design consapevole con trade-off espliciti. Su workload ripetitivi è obbligatorio. |
| "Latenza e qualità sono sempre in contrasto" | Con il caching puoi avere entrambi sulle richieste ripetitive. Il contrasto è vero solo senza cache. |
| "Ottimizziamo dopo" | L'architettura determina l'ordine di grandezza del costo. Cambiare pattern dopo il deployment è molto più costoso. |

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani.

1. Descrivi i tre assi del triangolo e la principale leva che agisce su ciascuno.
2. Qual è la differenza tra cache esatta e cache semantica? Quando usi l'una e quando l'altra?
3. Un sistema ha costo per richiesta troppo alto. Nomina tre leve che puoi tirare, con il trade-off di ciascuna.
4. Perché il routing intelligente (modello piccolo + modello grande) può ridurre il costo del 60-70%?
5. *(anticipa 5.5)* Il tuo sistema di supporto ha il 35% di richieste identiche (stesso testo). Quale componente aggiungi e con quale configurazione?

---

## Glossario

- **TTFT** — Time to First Token; tempo dalla richiesta all'arrivo del primo token in streaming.
- **Cache esatta** — riutilizzo della risposta per input identici; latenza quasi zero, costo di storage.
- **Cache semantica** — riutilizzo per input simili tramite similarità vettoriale; richiede calibrazione della soglia.
- **Prefix caching / KV cache** — ottimizzazione del provider che riduce il costo di calcolo per system prompt identici.
- **TTL** — Time to Live; durata massima di una voce in cache prima che venga considerata obsoleta.
- **Routing** — meccanismo che indirizza le richieste verso modelli diversi in base alla complessità del task.

---

## Per approfondire

- **Pricing pages dei provider** (OpenAI, Anthropic, Google) — i listini token/modello rendono il triangolo concreto con numeri reali; aggiornarsi ogni trimestre.
- **"The Economics of LLM APIs"** — vari post su blog tecnici analizzano il costo reale a scala; cerca questo tema su blog di Hamel Husain, Eugene Yan, e simili.

*Risorse indicate per la ricerca; per i link aggiornati conviene cercarli al momento.*

---

## Prossima lezione

**5.4 I dati come spina dorsale.** Hai il sistema, hai il pattern, hai il triangolo. Ma il sistema è buono quanto i dati che gli dai. Qualità dei dati, pipeline di preparazione, database vettoriali a fondo — il collo di bottiglia più sottovalutato.
