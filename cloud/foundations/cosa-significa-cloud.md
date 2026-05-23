---
title: Cosa significa cloud
sidebar_position: 1
---

# Cosa significa cloud

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 0.1</span>
  <span>~10 min di lettura</span>
</div>

<p class="lesson-lead">Il cloud non è un nuovo tipo di server. È un cambio di contratto con l'infrastruttura — e capirlo cambia ogni decisione tecnica che viene dopo.</p>

Prima di costruire qualcosa sul cloud, vale la pena capire cos'è davvero. Non la definizione da dizionario, ma il cambio mentale che porta: smetti di pensare a hardware da possedere e inizi a pensare a capacità da noleggiare, accendere, spegnere e pagare al consumo.

L'**idea in una frase**: il cloud è un'**interfaccia programmabile** sopra un'infrastruttura di qualcun altro, dove paghi solo per quello che usi, quando lo usi.

Il "qualcun altro" è un provider — AWS, Google Cloud, Azure — che ha costruito data center enormi, li gestisce internamente, ti espone tutto via API. Non tocchi mai un server fisico. Lo accendi con una chiamata HTTP, lo spegni con un'altra, e a fine mese arriva una fattura proporzionale all'uso.

## Dal data center di proprietà all'API

Per capire il cambiamento, conviene partire da com'era prima.

Un'azienda che voleva un'applicazione web comprava server fisici, li installava in un data center (di proprietà o in colocation), configurava rete, storage, backup, cooling, sicurezza fisica. Investimento enorme in anticipo, spesso mesi di lead time prima che i server arrivassero, e tutto dimensionato per il picco — anche se il picco capitava tre giorni l'anno. Nei restanti 362 giorni quella capacità stava lì a consumare corrente senza produrre niente.

Questo schema si chiama **CapEx** — *Capital Expenditure*, spesa in conto capitale: compro un asset, lo ammortamento dura anni, i costi sono fissi indipendentemente dall'utilizzo.

Il cloud ribalta il modello. Non compri più hardware: **noleggi capacità** per il tempo che ti serve, nella quantità che ti serve. Hai bisogno di 100 server per il Black Friday e 5 il resto dell'anno? Accendi 100 macchine a novembre, le spegni a dicembre, paghi solo quelle settimane. Questo schema è **OpEx** — *Operational Expenditure*, spesa operativa: costi variabili che seguono l'utilizzo reale, niente ammortamenti, niente asset in bilancio.

Il cambio CapEx → OpEx non è solo contabile. Cambia il rischio: invece di scommettere su una previsione di crescita con un acquisto da un milione di euro, puoi crescere in piccolo e scalare quando il bisogno si dimostra reale. E cambia la velocità: un nuovo ambiente di test in cloud si crea in minuti, non in settimane.

## Elasticità: il superpotere vero

La parola che senti più spesso parlando di cloud è **elasticità**: la capacità di scalare risorse verso l'alto o verso il basso in risposta al carico, in modo automatico.

Non è solo "posso aggiungere server". È che il sistema può farlo **da solo**, in pochi minuti, senza che nessuno chiami il data center. Un'applicazione con traffico a picco — concerti, Black Friday, comunicati stampa virali — in cloud può passare da 5 a 500 istanze in meno di un minuto, reggere il picco, e tornare a 5 quando il traffico scende.

In un data center fisico, questo scenario porta a una scelta obbligata: dimensioni per il picco (sprechi enormi per il 99% del tempo) o dimensioni per la media (sistema che va in crash al picco). Il cloud toglie questo dilemma.

L'elasticità ha però un lato oscuro: **i costi possono esplodere se non li tieni d'occhio**. Una configurazione sbagliata, un loop che replica risorse, un bucket di storage che inizia a ricevere traffico inaspettato — e a fine mese la fattura può sorprenderti. La prima cosa da fare quando apri un account AWS è impostare un **budget alert**. Non è un'opzione, è una norma igienica.

## Il modello di responsabilità condivisa

C'è un equivoco che brucia le aziende: pensare che passare al cloud significhi delegare tutta la responsabilità operativa al provider. Non funziona così.

AWS, Google Cloud e Azure condividono tutti un principio chiamato **modello di responsabilità condivisa**: il provider gestisce la sicurezza *dell'infrastruttura* (hardware fisico, rete backbone, data center, hypervisor); tu gestisci la sicurezza *di ciò che costruisci sopra* (configurazione dei sistemi, chi ha accesso, come sono protetti i dati).

In pratica: se lasci un bucket di storage pubblico per sbaglio e ci finiscono dati sensibili, è colpa tua, non del provider. AWS ti ha dato tutti gli strumenti per proteggerlo — policy di accesso, cifratura, logging — ma la responsabilità di usarli è tua.

La divisione esatta varia col livello di servizio. Su una VM il provider gestisce l'hypervisor e tu gestisci tutto sopra, incluso il patching del sistema operativo. Su un database gestito il provider gestisce anche il patching del database; tu gestisci chi vi accede. Su una funzione serverless il provider gestisce praticamente tutto l'underlay — tu scrivi solo il codice e gestisci gli accessi.

**Più sali nello stack, più responsabilità ti togli di dosso.** Ma la responsabilità non sparisce: si sposta verso strati più vicini ai dati e alla logica applicativa. Questo è il punto di partenza per la lezione 0.2.

<details>
<summary>CapEx vs OpEx — il calcolo del TCO</summary>

Il confronto si analizza con il **TCO** — *Total Cost of Ownership*, costo totale di possesso. Il TCO on-premise include: costo hardware ammortizzato (3-5 anni), data center (rack, power, cooling, spazio), personale IT per maintenance, rete e connettività, e il costo nascosto del sovradimensionamento per il picco.

Il TCO cloud include: compute, storage, rete al consumo, costi di supporto, tempo engineering per configurare e gestire i servizi, e l'eventuale costo di lock-in (migrazione futura).

La verità scomoda: **per workload stabili e prevedibili ad alto utilizzo (>70% costante), il data center di proprietà può avere TCO inferiore al cloud**, a parità di competenza operativa. Il cloud vince quasi sempre su workload variabili o spiky, startup senza capitale, velocità di sperimentazione, e servizi senza equivalente on-premise (AI managed, serverless, servizi di ML gestiti).

Sapere questo è utile perché cambia come si giustifica la scelta cloud: non "è sempre più economico", ma "la flessibilità e la velocità valgono il premium per il nostro caso".
</details>

## Cosa il cloud non è

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "È solo un server remoto" | Un server remoto che gestisci tu è VPS o colocation. Il cloud è una **piattaforma di servizi gestiti via API**: la differenza è quanta tra affittare un'auto e comprare pezzi di ricambio. |
| "Il cloud è sempre più economico" | Dipende dal workload. Per picchi e crescita imprevedibile, spesso sì. Per sistemi stabili ad alta utilization, il data center può costare meno. Il cloud vince quasi sempre su flessibilità e velocità, non necessariamente su prezzo. |
| "Il provider gestisce anche la sicurezza dei miei dati" | No. Il **modello di responsabilità condivisa** divide chiaramente: il provider protegge l'infrastruttura fisica, tu proteggi ciò che ci costruisci sopra. Un bucket mal configurato è responsabilità tua. |
| "Il cloud elimina l'operatività" | Riduce l'ops di basso livello (hardware, data center), ma aggiunge complessità nuova: gestione identità e permessi, rete virtuale, monitoring dei costi, sicurezza della configurazione. Ops non sparisce, cambia forma. |

## Verifica di comprensione

> Rispondi a memoria, senza rileggere. Le risposte incerte rivedile domani, non subito.

1. Qual è la differenza concreta tra CapEx e OpEx applicata al cloud?
2. Cos'è l'elasticità, e perché risolve un problema che il data center fisico non può risolvere?
3. Chi gestisce la sicurezza del dato in un servizio cloud — solo il provider o anche te?
4. Perché "il cloud è sempre più economico" non è una risposta completa?
5. Cos'è il TCO, e perché è lo strumento giusto per confrontare cloud e on-premise?
6. Cosa fare per prima cosa quando apri un account cloud?
7. *(anticipazione)* Se il provider gestisce di più, cosa rimane sotto la tua responsabilità?

---

## Glossario della pagina

- **CapEx** (*Capital Expenditure*): spesa in conto capitale; acquisto di asset con costo fisso anticipato.
- **Cloud computing**: piattaforma di servizi infrastrutturali accessibili via API, a consumo, gestiti da terze parti.
- **Elasticità**: capacità di scalare risorse in modo automatico e proporzionale al carico.
- **Modello di responsabilità condivisa**: divisione delle responsabilità di sicurezza e gestione tra provider cloud e cliente.
- **OpEx** (*Operational Expenditure*): spesa operativa; costi variabili proporzionali all'uso.
- **TCO** (*Total Cost of Ownership*): costo totale di possesso, inclusi tutti i costi diretti e indiretti.

## Per approfondire

- "AWS Shared Responsibility Model" — su `docs.aws.amazon.com`: la versione ufficiale con diagrammi per ogni tipo di servizio.
- AWS Pricing Calculator su `calculator.aws` — per confrontare scenari di costo reali.
- AWS Well-Architected Framework, pilastro "Cost Optimization" — il framework formale per analizzare TCO e giustificare scelte cloud.

## Prossima lezione

Ora che sai cos'è il cloud e perché il modello cambia, la domanda naturale è: quanto del sistema gestisce il provider e quanto rimane a te? La lezione 0.2 risponde con la distinzione **IaaS / PaaS / SaaS** — il primo strumento concreto per scegliere il livello di astrazione giusto.
