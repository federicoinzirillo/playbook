---
title: IaaS, PaaS, SaaS
sidebar_position: 2
---

# IaaS, PaaS, SaaS

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 0.2</span>
  <span>~9 min di lettura</span>
</div>

<p class="lesson-lead">Il cloud non è un'unica cosa — è uno spettro di accordi. Dove ti posizioni su quello spettro è la prima scelta architettuale che fai, e decide tutto il resto.</p>

Nella lezione 0.1 il provider gestiva "più o meno" a seconda di come accedevi al cloud. Adesso ci interessa capire esattamente cosa: chi gestisce il sistema operativo, chi gestisce il runtime, chi fa i backup, chi si occupa dello scaling. La risposta cambia radicalmente a seconda che tu usi **IaaS**, **PaaS** o **SaaS** — tre livelli di astrazione con tre rapporti di lavoro completamente diversi con l'infrastruttura.

L'**idea in una frase**: più sali nello stack, più il provider si occupa dell'infrastruttura e meno ne sai — il che è un vantaggio finché non è un limite.

## Lo spettro di gestione

Immagina di dover costruire un'applicazione. C'è un continuum di scelte:

Nella versione più cruda (**on-premise**) possiedi tutto: hardware, rete, sistema operativo, runtime, applicazione, dati. Gestisci ogni strato.

**IaaS** — *Infrastructure as a Service* — ti toglie il fardello fisico: il provider gestisce hardware, rete fisica, hypervisor. Tu ricevi una **macchina virtuale** con un sistema operativo. Da lì in su è tutto tuo: installa quello che vuoi, configura come vuoi, scala quando vuoi (ma a mano o con i tuoi script). Esempi: **Amazon EC2** (Elastic Compute Cloud), Google Compute Engine, Azure Virtual Machines.

**PaaS** — *Platform as a Service* — il provider gestisce anche sistema operativo, runtime, scaling automatico, aggiornamenti di sicurezza dello strato inferiore. Tu porti solo il **codice applicativo** e la configurazione ad alto livello. Esempi: **AWS Elastic Beanstalk**, **AWS Lambda** (nella sua forma serverless, dove il provider gestisce anche i server sottostanti), Google App Engine, Heroku.

**SaaS** — *Software as a Service* — il provider gestisce tutto, inclusa l'applicazione. Tu usi il software via browser o API, senza occuparti di nulla sotto. Esempi: Gmail, Slack, Salesforce, GitHub. Dal punto di vista del costruttore, il SaaS di altri è spesso una *dipendenza* nel tuo sistema, non qualcosa che costruisci.

| | On-premise | IaaS | PaaS | SaaS |
|---|---|---|---|---|
| Hardware fisico | Tu | Provider | Provider | Provider |
| Rete backbone | Tu | Provider | Provider | Provider |
| Sistema operativo | Tu | Tu | Provider | Provider |
| Runtime / linguaggio | Tu | Tu | Provider | Provider |
| Scaling automatico | Tu | Tu* | Provider | Provider |
| Applicazione | Tu | Tu | Tu | Provider |
| Dati | Tu | Tu | Tu | Tu* |

*In IaaS lo scaling può essere automatico con auto-scaling groups, ma lo configuri tu. *Nei SaaS i dati tecnicamentec sono "tuoi" per contratto — ma vivono nell'infrastruttura altrui.

## IaaS: massimo controllo, massima responsabilità

Con una VM (macchina virtuale) EC2 hai una macchina Linux o Windows con cui fare letteralmente qualsiasi cosa. Installi il database che vuoi, il runtime che vuoi, configuri il firewall come ti serve.

Il vantaggio è il controllo: se hai requisiti particolari (sistema operativo personalizzato, software legacy, configurazioni di rete specifiche), IaaS è spesso l'unica opzione. Il prezzo da pagare è l'**overhead operativo**: sei tu a fare il patching del sistema operativo, tu a configurare il monitoring, tu a scrivere gli script di scaling, tu a gestire i backup.

IaaS è la scelta giusta quando:
- Hai requisiti stringenti su sistema operativo o configurazione di basso livello.
- Stai migrando un sistema legacy che non si può facilmente containerizzare o riscrivere.
- Hai un workload stabile e prevedibile dove il controllo fine vale l'overhead.

## PaaS: focus sul codice, meno sull'infrastruttura

Con un servizio PaaS ti occupi di quello che distingue davvero il tuo prodotto: il codice applicativo. Il provider gestisce tutto il resto.

Lambda è l'esempio estremo: scrivi una funzione, la carichi, e Lambda gestisce tutto — provisioning dei server, scaling (anche a zero), alta disponibilità, aggiornamenti del runtime. Non sai nemmeno su quante macchine gira il tuo codice. Non lo devi sapere.

Il vantaggio è la velocità: deploy in secondi, scaling automatico, nessun server da mantenere. Il prezzo è la **perdita di controllo**: non puoi installare librerie di sistema arbitrarie, non puoi aprire porte di rete, non puoi fare cose che la piattaforma non prevede. E c'è il **vendor lock-in**: un'applicazione Lambda molto dipendente dai servizi AWS è difficile da spostare su un altro provider.

PaaS è la scelta giusta quando:
- Il tuo vantaggio competitivo sta nel codice applicativo, non nella configurazione infrastrutturale.
- Il workload è variabile (serverless scala a zero e paga zero quando non c'è traffico).
- La velocità di sviluppo vale più del controllo massimo.

## FaaS e serverless: il PaaS portato all'estremo

Vale la pena nominare **FaaS** — *Function as a Service*, di cui Lambda è l'archetipo — come categoria separata. In un'architettura serverless non gestisci "servizi" nel senso tradizionale, ma funzioni che si attivano su eventi: una chiamata HTTP, un messaggio in una coda, un upload su storage. Il provider gestisce tutto il ciclo di vita del container che le esegue.

Il modello di costo è radicalmente diverso: paghi per invocazione e per millisecondo di esecuzione, non per ore di server acceso. Con traffico basso o nullo, il costo è letteralmente zero — nessun server acceso ad aspettare. Con traffico alto scala da solo, senza limiti pratici.

Il trade-off sono i **cold start**: quando la funzione non è stata invocata di recente, il provider deve inizializzare il container, il che aggiunge latenza (tipicamente 100ms-1s a seconda del runtime). Per applicazioni latency-sensitive questa latenza può essere inaccettabile, e si torna ai container sempre-on.

<details>
<summary>Il modello di costo serverless vs container</summary>

Confronto di costo schematico per un'API con 1 milione di richieste/mese, 100ms di execution time, 256MB di memoria:

**Lambda (2026, us-east-1):**
- ~$0.20 per 1M invocazioni
- ~$4.17 per 1M GB-secondi (100ms × 256MB × 1M = 25.600 GB-s)
- **Totale: ~$4.37/mese**

**EC2 t3.micro (sempre acceso):**
- ~$8.40/mese (1 istanza, prezzi on-demand)
- **Totale: ~$8.40/mese** — ma può gestire molti più di 1M req/mese se il carico è distribuito

La soglia si inverte quando le richieste aumentano: sopra ~10M req/mese con execution time sostenute, un EC2 c3.small dedicato diventa più economico di Lambda. Questa è la ragione per cui le architetture ibride (Lambda per i picchi/eventi rari, container per il traffico base continuo) sono comuni nei sistemi maturi.
</details>

## Come scegliere il livello giusto

Non esiste una risposta universale — ma ci sono domande che aiutano:

**Hai requisiti di sistema operativo o configurazione molto specifici?** → IaaS. Altrimenti considera PaaS prima.

**Il workload ha picchi spiky o è quasi nullo per lunghi periodi?** → FaaS/serverless. Il costo a zero con traffico zero è un vantaggio enorme.

**Hai bisogno di esecuzione continua (WebSocket, job long-running, stream processing)?** → Container o VM, non Lambda (limite: 15 minuti per invocazione).

**Quanto vale la tua velocità di sviluppo rispetto al controllo?** → Più alto è il valore della velocità, più PaaS è la risposta giusta.

In un sistema reale di solito usi tutti e tre i livelli: **Lambda** per gli endpoint HTTP leggeri, **EC2 o container** per il servizio di elaborazione pesante, **RDS** (database gestito PaaS) per la persistenza. Lo spettro non è una scelta singola, è una tavolozza.

## Cosa non è

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "PaaS è sempre meglio perché fa di più" | Fa di più ma **controlla di meno**. Per certi requisiti, IaaS è l'unica opzione. |
| "SaaS non è rilevante per un ingegnere" | Gli SaaS di terze parti sono spesso *dipendenze* nel tuo sistema (Stripe, Twilio, Datadog). Capire come funzionano e dove stanno i rischi è parte del lavoro. |
| "Serverless = niente server" | Ci sono sempre server. Serverless significa solo che **non li gestisci tu**: il provider li alloca e li rilascia in modo trasparente. |
| "IaaS vs PaaS è una scelta una-tantum per il sistema" | Si mixano. Un sistema tipico usa IaaS per alcuni componenti, PaaS per altri, servizi managed (a metà strada) per i database. |

## Verifica di comprensione

> Rispondi a memoria, senza rileggere. Le risposte incerte rivedile domani.

1. Qual è la differenza fondamentale tra IaaS e PaaS?
2. Cosa gestisce il provider in Lambda che non gestisce in EC2?
3. In quale scenario il serverless ha un costo quasi zero?
4. Cos'è un cold start e quando è un problema?
5. Perché il vendor lock-in è maggiore con PaaS rispetto a IaaS?
6. Fai un esempio di sistema che usa tutti e tre i livelli contemporaneamente.
7. *(anticipazione)* Se usi un database gestito (RDS), chi fa il patching del database engine?

---

## Glossario della pagina

- **FaaS** (*Function as a Service*): modello serverless dove si deployano singole funzioni attivate da eventi; Lambda è l'archetipo AWS.
- **IaaS** (*Infrastructure as a Service*): il provider gestisce l'hardware fisico, tu gestisci il sistema operativo e tutto sopra.
- **Macchina virtuale (VM)**: istanza di sistema operativo che gira su un hypervisor; il mattone base di IaaS.
- **PaaS** (*Platform as a Service*): il provider gestisce anche SO e runtime; tu porti solo il codice applicativo.
- **SaaS** (*Software as a Service*): il provider gestisce tutto, inclusa l'applicazione; tu la usi.
- **Serverless**: architettura in cui non si gestiscono server; la piattaforma alloca e dealloca risorse automaticamente.
- **Vendor lock-in**: dipendenza forte da un provider specifico che rende costoso o difficile migrare altrove.

## Per approfondire

- AWS: pagina "Types of Cloud Computing" su `aws.amazon.com/types-of-cloud-computing` — la tassonomia ufficiale con esempi di servizi per ogni livello.
- AWS Lambda documentation su `docs.aws.amazon.com/lambda` — per capire il modello di esecuzione serverless.

## Prossima lezione

Ora sai *quanto* ti occupa il provider a seconda del livello scelto. Ma finora abbiamo parlato di "compute" come di qualcosa di generico. La lezione 0.3 scende nei dettagli: **quali risorse fondamentali esistono sul cloud** — compute, storage, networking, database — e come si distinguono tra loro.
