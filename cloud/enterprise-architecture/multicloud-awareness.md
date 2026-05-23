---
title: Multi-cloud awareness
sidebar_label: "9.5 Multi-cloud awareness"
sidebar_position: 5
---

# Multi-cloud awareness

<div class="lesson-meta">
  <span class="badge-stato evoluzione">In evoluzione</span>
  <span>Lezione 9.5</span>
  <span>~10 min di lettura</span>
</div>

<p class="lesson-lead">Il multi-cloud è reale nelle grandi enterprise, ma spesso per ragioni che non hanno niente a che fare con la tecnologia. Prima di sapere come gestirlo, devi capire perché esiste — e quando è una scelta tecnica invece di un fatto politico.</p>

Entra in qualsiasi grande azienda e troverai AWS, Azure e magari GCP coesistere nello stesso budget IT. Non perché qualcuno abbia fatto una comparazione tecnica sistematica. Più spesso: il team di data science usa Azure ML perché l'azienda ha già un contratto Microsoft Enterprise; il team mobile usa Firebase (GCP); il backend principale gira su AWS dove si è sempre girato. Il multi-cloud è già lì — non è stata una scelta, è successo.

La **multi-cloud awareness** non significa saper usare tre cloud contemporaneamente. Significa capire come i concetti che conosci su AWS si mappano sugli altri provider, come parlarne in modo sensato con un cliente che ha quel mix, e quando il multi-cloud è una soluzione tecnica vera invece di un argomento di marketing.

## Il mapping concettuale

I tre provider grandi risolvono gli stessi problemi con prodotti diversi. Il mapping non è mai perfetto — ogni servizio ha sfumature proprie — ma la struttura concettuale è la stessa.

| Categoria | AWS | Azure | GCP |
|---|---|---|---|
| **VM compute** | EC2 | Azure Virtual Machines | Compute Engine |
| **Container serverless** | AWS Fargate / ECS | Azure Container Apps | Cloud Run |
| **Functions** | Lambda | Azure Functions | Cloud Functions |
| **Object storage** | S3 | Azure Blob Storage | Google Cloud Storage (GCS) |
| **Managed Kubernetes** | EKS | AKS | GKE |
| **Relational DB managed** | RDS | Azure Database (Postgres/MySQL) | Cloud SQL |
| **NoSQL managed** | DynamoDB | Cosmos DB | Firestore / Bigtable |
| **Message queue** | SQS | Azure Service Bus | Pub/Sub |
| **Event streaming** | Kinesis | Azure Event Hubs | Pub/Sub |
| **CDN** | CloudFront | Azure CDN / Front Door | Cloud CDN |
| **IAM** | IAM + STS | Microsoft Entra ID (ex-Azure AD) | Cloud IAM |
| **Infrastruttura as code** | CloudFormation | ARM / Bicep | Deployment Manager |
| **LLM platform** | Amazon Bedrock | Azure OpenAI Service | Vertex AI |

Qualche differenza che conta:

- **Azure Active Directory / Entra ID**: Azure è nativo nell'ecosistema Microsoft. Se l'azienda usa Microsoft 365 e Teams, l'identità è già su Entra ID. L'integrazione con Azure è diretta; con AWS si usa federation tramite SAML o OIDC.
- **GKE vs EKS vs AKS**: GKE è considerato il managed Kubernetes più maturo tecnicamente — Google ha inventato Kubernetes. EKS integra meglio con l'ecosistema AWS (IAM, Load Balancer, EBS). AKS è la scelta naturale se si è già su Azure.
- **DynamoDB vs Cosmos DB**: entrambi NoSQL managed multi-regione, ma con modelli di pricing e modelli di consistenza diversi. Cosmos DB offre più API compatibili (MongoDB, Cassandra, Gremlin). DynamoDB ha pricing per operazione più prevedibile.

## Quando il multi-cloud è tecnico

Casi reali in cui la scelta multi-cloud ha una giustificazione tecnica:

**Latenza geografica**: AWS non ha una region in certi paesi (es. alcuni mercati emergenti) dove Azure o GCP sono presenti. Un'azienda globale può usare il provider con la copertura geografica migliore per zona.

**Servizi best-in-class**: Google Maps, Firebase, i modelli Gemini sono su GCP. Azure OpenAI Service aveva (per un periodo) accesso prioritario ai modelli OpenAI. Se il servizio che ti serve esiste solo su un provider, usi quel provider per quel servizio.

**Disaster recovery cross-provider**: teoricamente, failover su un provider diverso elimina il rischio di outage di un singolo provider. In pratica è costoso da mantenere e pochi team lo implementano davvero.

**Evitare vendor lock-in**: usando tecnologie che astraggono il provider (Kubernetes, Terraform, Kafka), si mantiene la portabilità. Ma la portabilità ha un costo: i servizi managed più potenti (Lambda, Fargate, Aurora Serverless) sono specifici del provider.

## Quando il multi-cloud è politico

Molto più spesso di quanto si voglia ammettere:

**Negoziazione contrattuale**: avere un contratto con due provider aumenta il potere negoziale. Molte enterprise usano Azure come alternativa credibile per negoziare i prezzi con AWS, senza mai intenzione di migrare davvero.

**Silos organizzativi**: team diversi scelgono provider diversi. Nessuno ha l'autorità per standardizzare. Il multi-cloud è un sintomo di governance assente.

**Acquisizioni**: un'azienda compra una startup che gira su GCP. Ora ha multi-cloud. Non è una strategia — è un'eredità.

La multi-cloud awareness significa riconoscere queste dinamiche e non trattare il multi-cloud come un obiettivo tecnico di per sé. **Un sistema distribuito su due cloud è più complesso da gestire, non meno.** Reti, IAM, logging, osservabilità — tutto va duplicato o astratto.

## FinOps Foundation e il FOCUS standard

Quando si hanno costi su più provider, il primo problema è confrontarli: AWS usa una terminologia di billing diversa da Azure, diversa da GCP. Il **FOCUS standard** (*FinOps Open Cost and Usage Specification*), sviluppato dalla FinOps Foundation e in fase di adozione nel 2026, definisce uno schema comune per i dati di costo cloud.

FOCUS specifica colonne e valori standardizzati per concetti come: tipo di risorsa, regione, servizio, periodo di billing, costo effettivo, costo ammortizzato. L'obiettivo è avere un dataset unificato che possa essere ingerito da strumenti FinOps (CloudHealth, Apptio, Grafana) senza trasformazioni custom per ogni provider.

Per un engineer, il punto pratico è: se costruisci una pipeline di cost analytics che deve coprire più provider, usa FOCUS come schema target invece di trattare il CSV di AWS e il CSV di Azure come silos separati.

## Cosa non è

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Multi-cloud = più resiliente per definizione" | Un sistema su due provider è più resiliente solo se la dipendenza da un singolo provider è davvero il rischio principale. Nella maggior parte dei casi, i guasti sono applicativi o di rete, non outage di provider. La complessità multi-cloud può introdurre più guasti di quanti ne eviti. |
| "Bisogna scegliere un provider e restare su quello" | Non necessariamente. Un sistema con il backend su AWS e Firebase come push notification su GCP è perfettamente ragionevole. L'obiettivo è ridurre la complessità non necessaria, non il dogma del singolo provider. |
| "Kubernetes mi rende cloud-agnostic" | Kubernetes astrae il layer di orchestrazione dei container. Ma sotto c'è ancora il networking del provider, il managed storage del provider, i load balancer del provider, le policy IAM del provider. La portabilità è parziale, non totale. |
| "FOCUS risolverà il problema del billing multi-cloud" | FOCUS standardizza il formato dei dati, non il modello di pricing. AWS e Azure calcolano il costo in modo diverso (per es. pricing del traffico uscente). Il confronto diretto richiede ancora interpretazione. |

## Verifica di comprensione

1. Qual è l'equivalente di Lambda su Azure e su GCP?
2. In quali scenari la scelta multi-cloud ha una giustificazione tecnica reale?
3. Perché Kubernetes non garantisce vera cloud-agnosticism?
4. Cos'è il FOCUS standard e a quale problema risponde?
5. Qual è la differenza principale tra DynamoDB e Cosmos DB dal punto di vista delle API supportate?
6. Come si integra un'azienda Microsoft-centric (Active Directory) con AWS?
7. Quali sono i tre motivi politici più comuni per cui le enterprise si trovano in multi-cloud senza averlo pianificato?

## Glossario della pagina

- **Multi-cloud**: utilizzo di servizi cloud di più provider (AWS, Azure, GCP) da parte della stessa organizzazione.
- **FOCUS** — *FinOps Open Cost and Usage Specification*: schema aperto della FinOps Foundation per standardizzare i dati di costo tra provider cloud diversi.
- **FinOps Foundation**: organizzazione no-profit che definisce pratiche e standard per la gestione finanziaria del cloud.
- **Vendor lock-in**: dipendenza da un singolo provider al punto da rendere costoso o difficile il cambiamento.
- **Microsoft Entra ID** (ex-Azure Active Directory): il servizio di identità e accesso di Microsoft, integrato nativamente con Azure e usabile via federation su altri provider.
- **GKE / EKS / AKS**: i servizi Kubernetes managed di GCP, AWS e Azure rispettivamente.
- **Federation (IAM)**: meccanismo che permette di usare un'identità gestita da un sistema esterno (es. Entra ID) per autenticarsi su un altro sistema (es. AWS IAM), tramite standard come SAML o OIDC.

## Per approfondire

- **FinOps Foundation FOCUS** (`focus.finops.org`): la specifica ufficiale e i tool compatibili.
- **AWS to Azure services comparison** (`learn.microsoft.com`): mapping ufficiale mantenuto da Microsoft.
- **Google Cloud for AWS professionals** (`cloud.google.com`): guida ufficiale per chi conosce AWS e vuole capire GCP.
- **"The Multi-Cloud Trap"** — cerca questo tema su InfoQ o ACM Queue per letture critiche sull'approccio multi-cloud in enterprise reali.

## Prossima lezione

Hai tutto il quadro della Parte 9. L'ultima lezione è un **decision drill** su scenario enterprise reale: un'azienda con 50 developer, 3 team, e requisiti SOC 2. Disegna la struttura account, scegli gli SCP, progetta il processo di provisioning. Poi confronta con la griglia di valutazione.
