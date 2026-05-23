---
title: "Sicurezza agentica: la superficie d'attacco comportamentale"
sidebar_label: "4.2 Sicurezza agentica"
sidebar_position: 2
---

# Sicurezza agentica: la superficie d'attacco comportamentale

<div class="lesson-meta">
  <span class="badge-stato evoluzione">In evoluzione</span>
  <span>Lezione 4.2</span>
  <span>~10 min di lettura</span>
</div>

<p class="lesson-lead">Un agente con tool non è vulnerabile solo sull'input: è vulnerabile sull'intera catena di azioni. Ogni tool aggiunto moltiplica la superficie d'attacco. I guardrail veri non stanno nei prompt — stanno nell'architettura.</p>

Nella lezione 4.1 hai visto la prompt injection. Su un singolo LLM che risponde a una domanda, le conseguenze di un'injection sono limitate: al massimo ti fa dire qualcosa che non dovresti. Su un agente con tool, lo stesso attacco può innescare una catena di azioni reali — inviare email, modificare dati, chiamare API esterne. La differenza non è di grado. È di categoria.

## La superficie d'attacco si moltiplica con i tool

Ogni tool che aggiungi a un agente aggiunge un vettore d'attacco. Non solo per le azioni che il tool può compiere, ma per i dati che porta nel contesto del modello.

Considera un agente con questi tool: leggi email, scrivi email, cerca nel database clienti, aggiorna il profilo cliente. Un'injection indiretta in un'email ricevuta può portare il modello a:

1. Leggere i dati del cliente (già nel contesto, gratis)
2. Aggiornare il profilo con dati falsi
3. Inviare una risposta con dati esfiltrati

Nessuna delle tre azioni è intrinsecamente "attacco" — il modello le esegue perché l'injection le ha presentate come parte del task legittimo. Ognuna è autorizzata dai permessi del tool. Insieme costituiscono un danno reale.

**Il problema non è l'azione singola. È la composizione.**

## Principio del minimo privilegio per gli agenti

La risposta architettuale è la stessa della sicurezza classica: ogni componente deve avere i permessi minimi necessari per il suo task, niente di più.

In pratica, questo significa:

**Scope ridotto per tool.** Un tool "leggi email" non dovrebbe restituire l'intero storico della casella. Dovrebbe restituire le ultime N email del thread richiesto. Un tool "cerca cliente" non dovrebbe esporre il numero di carta. La query è: di questi dati, quali servono davvero per il task?

**Tool con permessi differenziati per contesto.** Un agente che gestisce le FAQ del supporto non ha bisogno dell'accesso in scrittura al CRM. Se hai due agent — uno per leggere, uno per scrivere — l'injection che compromette il primo non porta automaticamente ai permessi del secondo.

**Separazione tra agenti con diversi livelli di privilegio.** L'architettura multi-agent permette di isolare: un agente orchestratore con visibilità alta ma azioni limitate, agenti specializzati con accesso ristretto al loro dominio. La compromissione di uno non implica la compromissione degli altri.

## Azioni irreversibili: il punto di non ritorno

Alcune azioni hanno conseguenze permanenti o difficilmente reversibili: inviare un'email, effettuare un pagamento, cancellare un record, chiamare un'API che attiva un workflow esterno. Queste sono il punto critico.

**Human in the loop obbligatorio per le irreversibili.** Prima di ogni azione irreversibile, l'agente deve presentare ciò che sta per fare e attendere conferma esplicita. Non è una limitazione dell'autonomia — è il guardrail che separa "un agente utile" da "un agente pericoloso".

L'implementazione concreta: il tool che esegue l'azione irreversibile viene sostituito da un tool di "proposta" — restituisce una descrizione dell'azione da compiere. Il codice la presenta all'utente e attende il sì. Solo dopo, un secondo tool esegue.

**Limite massimo di step.** Un agente che va in loop (o viene guidato da un'injection verso comportamenti iterativi) consuma risorse e amplifica il danno ad ogni ciclo. Un contatore di step con un limite fisso è un guardrail semplice e molto efficace.

## Isolamento del contesto tra sessioni

Nei sistemi multi-utente, il contesto di un'agente non deve "contaminare" la sessione di un altro utente. Se il contesto di sessione è persistente o condiviso, un'injection in una sessione può innescare comportamenti nella successiva.

Regola semplice: ogni sessione ha il proprio contesto isolato. I dati persistenti (profilo utente, preferenze) passano attraverso tool espliciti, non attraverso il contesto di sessione condiviso.

## Monitoring comportamentale

I guardrail architetturali riducono la superficie. Il monitoring comportamentale intercetta ciò che sfugge.

Cosa monitorare su un agente:
- **Sequenze di tool call inusuali** — un agente che di solito legge email e risponde, improvvisamente cerca nel DB e chiama un'API esterna: anomalia.
- **Volume di dati estratti** — un tool call che porta al contesto 10 MB di dati invece dei soliti 5 KB: anomalia.
- **Azioni fuori orario o fuori contesto** — richieste di notte, da IP inusuali, con pattern diversi dall'utente storico.

Non è possibile automatizzare tutto il rilevamento. L'obiettivo è ridurre il tempo tra compromissione e rilevamento.

<span class="badge-stato evoluzione">In evoluzione</span> Il campo della sicurezza agentica è in rapida evoluzione. Framework come LangGraph, LlamaIndex, CrewAI, AutoGen e i provider cloud stanno aggiungendo primitive di sicurezza (sandboxing, permissioned tool calls, audit log). Nel 2025-26 sono emersi framework dedicati di defense: **PromptArmor** (ICLR 2026, meno dell'1% di falsi positivi e falsi negativi su AgentDojo) e **PromptGuard** (riduzione del 67% del success rate degli attacchi); Claude Opus 4.5 ha portato l'attack success rate del browser agent attorno all'1% via reinforcement learning e classifier. Ma il consenso istituzionale (NCSC UK dic 2025, Schneier IEEE jan 2026) è chiaro: la prompt injection non si risolve completamente con l'architettura attuale — defense-in-depth, non un guardrail singolo.

## Supply chain del modello e dei tool

Finora abbiamo parlato di attaccanti che colpiscono dall'**input**. Esiste una superficie più subdola: l'attaccante che colpisce dall'**interno**, contaminando ciò di cui il modello è fatto o ciò a cui ha accesso.

**Modelli open-weight scaricati da registry pubblici.** Hugging Face nel 2024-25 ha visto comparire ripetutamente repository con pesi compromessi: backdoor che si attivano su trigger specifici, codice malevolo nei file `pickle` (protocollo di serializzazione Python che esegue codice arbitrario al caricamento). Mitigation: scarica solo da repository verificati (organizzazioni ufficiali del modello), usa il formato `safetensors` quando disponibile (non esegue codice), pinna le revisioni con hash invece di scaricare `main`.

**Prompt injection nel tool registry.** Se l'agente scopre i tool da un registro dinamico (es. server **MCP** community), un tool malevolo può presentarsi con una descrizione benigna e includere istruzioni nascoste nella sua specifica che il modello legge come prompt. Il principio è lo stesso dell'injection indiretta, ma il vettore è il "manuale d'uso" del tool, non i dati che il tool restituisce. Mitigation: whitelist dei server MCP fidati, code review delle descrizioni dei tool prima di abilitarli.

**Data poisoning a monte.** Per i modelli che ti addestri o fine-tuni: chi controlla il dataset controlla il comportamento. Esempi avvelenati in fase di fine-tuning possono installare comportamenti specifici attivabili da trigger ("se l'utente cita la parola X, ignora le istruzioni e fai Y"). Mitigation: provenienza del dataset, deduplica, evaluation su test set indipendente che cerca comportamenti anomali.

Il filo comune: la sicurezza agentica non è solo "cosa fa il modello con l'input che riceve". È anche "di cosa è fatto il modello" e "a cosa ha accesso dal momento in cui parte". Trattare modello e tool come **artefatti software** con versioning, hash, e provenance verificabile è il piano che la community sta importando dal supply-chain security classico (SLSA, sigstore).

## Cosa NON è la sicurezza agentica

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Basta non dare tool pericolosi" | Ogni tool è pericoloso in combinazione con gli altri. La composizione è il rischio. |
| "Il modello più recente è più sicuro, quindi ok" | La vulnerabilità è architetturale — dipende dai tool e dai permessi, non dal modello. |
| "Con un buon system prompt l'agente non farà cose brutte" | Il system prompt non è enforcement. Un'injection diretta o indiretta può sovrascriverlo. |
| "Il logging basta per la sicurezza" | Il logging è post-hoc. Serve per il forensics, non per la prevenzione. |

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani.

1. Perché la sicurezza agentica è un problema di categoria diversa rispetto a quella di un LLM che risponde a domande?
2. Descrivi il principio del minimo privilegio applicato a un agente con accesso a email e CRM.
3. Cos'è un'azione irreversibile e come la gestisci architetturalmente?
4. Un agente viene compromesso da un'injection indiretta in un'email. Quali guardrail possono limitare il danno?
5. *(anticipa 4.5)* Un agente ha tool: leggi-documenti, scrivi-documento, invia-notifica, aggiorna-DB. Quali tool richiedono human-in-the-loop e perché?

---

## Glossario

- **Superficie d'attacco** — insieme di tutti i punti attraverso cui un attaccante può interagire con il sistema.
- **Composizione di azioni** — il rischio che azioni singole legittime, combinate, producano un danno.
- **Minimo privilegio** — ogni componente ha solo i permessi necessari per il suo task specifico.
- **Human in the loop** — meccanismo che richiede conferma umana prima di eseguire azioni critiche o irreversibili.
- **Sandboxing** — esecuzione di componenti in ambienti isolati che limitano l'impatto di una compromissione.
- **Audit log** — registro immutabile delle azioni compiute, fondamentale per il forensics post-incidente.

---

## Per approfondire

- **OWASP LLM Top 10 (v2025)** — in particolare LLM06:2025 (Excessive Agency, articolata nelle tre cause: eccesso di funzionalità, di permessi, di autonomia) e LLM02:2025 (Sensitive Information Disclosure, promossa al #2).
- **"Compromised Agents: Attacking and Defending LLM Agent Systems"** — cerca il titolo su arXiv.
- Documentazione di sicurezza di LangGraph, Llama Stack, e dei tool ufficiali di OpenAI / Anthropic sulle policy dei tool.

*Risorse indicate per la ricerca; per i link aggiornati conviene cercarli al momento.*

---

## Prossima lezione

**4.3 Privacy e data residency.** I guardrail tecnici proteggono dall'attaccante esterno. Il GDPR e la data residency proteggono i dati degli utenti da te stesso — o meglio, da scelte architetturali che sembrano innocue e generano obblighi legali enormi.
