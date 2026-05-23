---
title: Privacy e data residency
sidebar_position: 3
---

# Privacy e data residency

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 4.3</span>
  <span>~10 min di lettura</span>
</div>

<p class="lesson-lead">Quando mandi un dato a un'API LLM di terzi, dove finisce? Chi può leggerlo? Per quanto? Queste non sono domande filosofiche — sono obblighi legali. Il GDPR si applica a ogni dato personale, indipendentemente da chi elabora il testo: il provider LLM è il tuo responsabile del trattamento, e devi saperlo gestire prima di andare in produzione.</p>

La privacy in un sistema LLM non è un tema da affrontare a fine progetto come "aggiungiamo la privacy policy". Spesso è il primo blocco reale in un contesto enterprise: il primo meeting con il legal termina con "ma questi dati dove vanno?", e se non hai la risposta, il progetto si ferma.

## Dove finiscono i dati che mandi a un'API LLM

Quando chiami un'API come quella di OpenAI, Anthropic, o Google, il testo che invii — incluso il contesto, i documenti recuperati da RAG, i messaggi dell'utente — transita sui server del provider e viene elaborato lì.

Il modello, di default, **non addestra su quei dati** — i principali provider hanno politiche esplicite su questo, e per le API di produzione l'addestramento sui prompt è tipicamente disabilitato per impostazione predefinita. Ma il dato transita, viene loggato per ragioni operative, e risiede nei loro sistemi per un certo periodo.

Tre domande che il legal farà sicuramente:

1. **Dove si trovano fisicamente i server** che elaborano i dati? (data residency)
2. **Chi può accedervi** e in che circostanze? (data access policy del provider)
3. **Per quanto tempo** vengono conservati i log? (retention policy)

Le risposte variano tra provider e tra tier di servizio. Azure OpenAI, per esempio, offre opzioni di data residency europea esplicita. L'API di OpenAI.com ha policy diverse. Verificare prima di scegliere il provider è parte del design del sistema.

## GDPR e trattamento dei dati personali

Il **GDPR** — *General Data Protection Regulation*, il regolamento europeo sulla protezione dei dati — si applica ogni volta che tratti dati personali di residenti nell'UE, indipendentemente da dove si trova la tua azienda.

**Dato personale** è qualsiasi informazione che può identificare una persona, direttamente o indirettamente: nome, email, numero di telefono, ma anche IP, cookie, identificativi di sessione, e — attenzione — conversazioni che contengono queste informazioni.

Quando un utente chatta con il tuo assistente AI e ti racconta un problema di salute, un'informazione sul suo datore di lavoro, o semplicemente usa il suo nome, hai dati personali nel contesto del modello. Quel testo finisce nell'API del provider. Il provider diventa il tuo **responsabile del trattamento** (*data processor*), e tu il **titolare del trattamento** (*data controller*).

Questo richiede:
- Un **Data Processing Agreement (DPA)** firmato con il provider.
- La garanzia che il provider operi in modo compatibile col GDPR (adeguatezza o standard contractual clauses per trasferimenti extra-UE).
- Una base legale per il trattamento (consenso, contratto, legittimo interesse — non basta voler usare l'AI).

<details>
<summary>Trasferimenti extra-UE e lo scudo UE-USA</summary>

Mandare dati personali EU su server negli USA richiede una base legale per il trasferimento internazionale. Per anni il quadro è stato instabile (Privacy Shield invalidato nel 2020). Il **Data Privacy Framework** UE-USA del 2023 ha ripristinato un meccanismo di adeguatezza, ma la situazione è stata volatile. I principali provider americani aderiscono al DPF e offrono anche le Standard Contractual Clauses (SCC) come alternativa. Verifica lo stato aggiornato col tuo consulente legale prima di progettare l'architettura.
</details>

## PII e anonimizzazione

**PII** — *Personally Identifiable Information*, informazioni identificabili — è il sottoinsieme dei dati personali più diretto: nomi, email, codici fiscali, numeri di telefono, coordinate bancarie.

La strategia più sicura: **non mandare PII all'API LLM se non è necessario**.

In molti casi, l'LLM non ha bisogno del nome reale dell'utente per fare il suo lavoro. Può lavorare su un identificativo anonimo. I dati sensibili possono essere pseudonimizzati — sostituiti con un token — prima di entrare nel contesto, e de-pseudonimizzati nel codice dopo che il modello ha elaborato.

```
Input utente: "Voglio sapere lo stato dell'ordine di Mario Rossi, ordine #12345"
          ↓
Pre-processing nel codice:
  "Mario Rossi" → sostituito con "[CLIENTE_A]"
  "#12345" → mantenuto (non è PII da solo)
          ↓
Inviato al LLM: "Voglio sapere lo stato dell'ordine di [CLIENTE_A], ordine #12345"
          ↓
LLM risponde con "[CLIENTE_A]"
          ↓
Post-processing: "[CLIENTE_A]" → "Mario Rossi" nella risposta finale
```

Questo schema non elimina tutti i rischi (il contesto può contenere altri dati identificabili), ma riduce significativamente l'esposizione.

**Anonimizzazione vs pseudonimizzazione:** sono concetti distinti nel GDPR. L'anonimizzazione vera (impossibile de-pseudonimizzare) rimuove i dati dall'ambito del GDPR. La pseudonimizzazione (come nell'esempio sopra) riduce il rischio ma il dato rimane personale.

## On-premise come risposta

Quando i dati sono troppo sensibili per uscire dall'infrastruttura controllata — dati medici, dati legali, segreti industriali — la risposta è eseguire il modello on-premise.

**Open-weight models** (Llama, Mistral, Qwen) si installano su hardware proprio o su cloud privato. Il dato non esce mai: viene elaborato dentro il perimetro controllato.

Il trade-off è chiaro: i modelli open-weight migliori sono oggi quasi comparabili ai top proprietari su molti task, ma richiedono hardware dedicato (GPU), team con competenze di deployment, e manutenzione. Per molte aziende enterprise con dati sensibili, il costo è giustificato.

I principali cloud provider (Azure, AWS, GCP) offrono anche soluzioni ibride: modelli proprietari in esecuzione su infrastruttura cloud dedicata al cliente, con data residency garantita e senza condivisione del dato con il provider del modello. Il costo è più alto del multi-tenant, ma più basso dell'on-premise puro.

## La checklist pratica prima di andare in produzione

Prima di mettere in produzione qualsiasi sistema che elabora dati utente con un LLM esterno:

- [ ] Hai identificato quali dati personali transitano nel contesto del modello?
- [ ] Il provider ha un DPA disponibile e l'hai firmato?
- [ ] Hai verificato la data residency e l'adeguatezza del trasferimento internazionale?
- [ ] Hai una base legale per il trattamento (e l'hai documentata)?
- [ ] Hai una retention policy definita (quanto tieni i log delle conversazioni)?
- [ ] Il tuo sistema ha un meccanismo per rispondere alle richieste DSAR (*Data Subject Access Request* — il diritto dell'utente di vedere e cancellare i propri dati)?
- [ ] L'anonimizzazione è applicata dove il modello non ha bisogno dei dati identificativi?

<span class="badge-stato stabile">Stabile</span> Il GDPR è regolamento del 2018 con aggiornamenti interpretativi continui. Le linee guida applicative per l'AI vengono dal EDPB (*European Data Protection Board*) e dagli autorità nazionali (per l'Italia, il Garante). Le nozioni base sono stabili; le applicazioni specifiche all'AI si evolvono.

<span class="badge-stato evoluzione">In evoluzione</span> Da tenere d'occhio per il 2027: il **Cyber Resilience Act** (CRA, Regulation EU 2024/2847) entra pienamente in vigore l'11 dicembre 2027 e impone requisiti di cybersecurity, vulnerability handling e SBOM (Software Bill of Materials) ai "prodotti con elementi digitali" — categoria che include sistemi AI distribuiti in EU. Si applica in parallelo a GDPR e AI Act, non in alternativa. Per chi distribuisce modelli o sistemi AI come componenti software (specialmente B2B o open-source non-personale), è il terzo livello di compliance da pianificare.

## Cosa NON è la privacy nell'AI

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Il modello non addestra sui miei dati, quindi sono a posto" | Il dato transita comunque; GDPR si applica al trattamento, non solo all'addestramento. |
| "La privacy policy basta" | La privacy policy informa gli utenti ma non sostituisce gli obblighi verso il provider. |
| "I dati sono solo testo, non sono dati personali" | Il testo può contenere PII. La forma non determina la categoria. |
| "Con il consenso dell'utente posso fare tutto" | Il consenso è una delle basi legali, ma ha requisiti precisi (libero, informato, specifico, revocabile). |

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani.

1. Qual è la differenza tra titolare del trattamento e responsabile del trattamento? Chi è chi in un sistema con LLM di terzi?
2. Cos'è un DPA e perché è necessario?
3. Descrivi la differenza tra anonimizzazione e pseudonimizzazione, con implicazioni GDPR.
4. Quando ha senso scegliere un modello on-premise rispetto a un'API esterna?
5. Un utente ti scrive chiedendo di vedere ed eliminare tutti i suoi dati. Quali componenti del tuo sistema devono essere in grado di rispondere a questa richiesta?

---

## Glossario

- **GDPR** — General Data Protection Regulation; regolamento UE 2016/679 sulla protezione dei dati personali.
- **Dato personale** — qualsiasi informazione che può identificare una persona fisica, direttamente o indirettamente.
- **PII** — Personally Identifiable Information; il sottoinsieme più diretto dei dati personali (nome, email, CF, ecc.).
- **Titolare del trattamento** — chi determina le finalità e i mezzi del trattamento dei dati.
- **Responsabile del trattamento** — chi tratta i dati per conto del titolare (es. il provider LLM).
- **DPA** — Data Processing Agreement; contratto che regola il trattamento dei dati da parte del responsabile.
- **Data residency** — requisito che i dati risiedano fisicamente in una specifica giurisdizione.
- **DSAR** — Data Subject Access Request; richiesta dell'interessato di accedere ai propri dati o cancellarli.
- **Pseudonimizzazione** — sostituzione degli identificativi diretti con token, mantenendo la possibilità di re-identificazione.

---

## Per approfondire

- **Sito del Garante per la protezione dei dati personali** (garante.privacy.it) — guide pratiche e provvedimenti, anche su AI.
- **EDPB Guidelines on AI** — l'autorità europea pubblica linee guida interpretative; cerca su edpb.europa.eu.
- **DPA dei principali provider** — OpenAI, Anthropic, Google, Microsoft Azure hanno i DPA disponibili sui loro siti legal.

*Risorse indicate per la ricerca; per i link aggiornati conviene cercarli al momento.*

---

## Prossima lezione

**4.4 EU AI Act e governance.** Il GDPR copre i dati. L'AI Act copre i sistemi AI in quanto tali — classifica il rischio del tuo sistema e impone obblighi diversi a seconda di dove cade. La mappa delle categorie e cosa significa in pratica per un AI engineer.
