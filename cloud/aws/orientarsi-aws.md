---
title: Orientarsi in AWS
sidebar_position: 1
---

# Orientarsi in AWS

<div class="lesson-meta">
  <span class="badge-stato evoluzione">In evoluzione</span>
  <span>Lezione 4.1</span>
  <span>~12 min di lettura</span>
</div>

<p class="lesson-lead">AWS ha oltre 200 servizi. Il trucco non è impararli tutti — è capire come sono organizzati, dove trovare quello che cerchi, e impostare subito i guardrail che evitano la bolletta shock.</p>

Le Parti 0–3 ti hanno dato i concetti universali. Da qui in poi li ancoriamo ad AWS: i concetti hanno un nome, una console, un prezzo. Non cambiano le idee — cambiano le etichette.

Il **principio guida di questa lezione**: prima i fondamentali operativi (account, IAM, billing) — poi i servizi. Chi apre AWS per la prima volta e inizia a cliccare su EC2 ha già saltato i tre step che gli costeranno cari nelle prossime settimane.

## La console, le regioni, e dove ti trovi

Appena accedi ad AWS, la prima cosa che vede l'occhio è la **Console di gestione** (*AWS Management Console*) — un'interfaccia web da cui puoi accedere a tutti i servizi. È utile per esplorare e capire; non è lo strumento per lavorare davvero (per quello c'è la CLI e Terraform, vedi Parte 3).

In alto a destra c'è un selettore di regione — es. `eu-west-1` (Irlanda), `us-east-1` (Virginia del Nord). Questo è il concetto di **regione** che hai visto in 0.4: ogni risorsa che crei (una VM, un bucket, una coda) esiste in una regione specifica. Se crei un'istanza EC2 in `us-east-1` e poi selezioni `eu-west-1`, quella istanza non appare. **Il 90% dei "dov'è finita la mia risorsa?" dipende da questo.**

Regola pratica: scegli una regione vicina ai tuoi utenti e rimani lì. Per progetti europei, `eu-west-1` (Irlanda) o `eu-central-1` (Francoforte) sono le scelte standard. `us-east-1` è storicamente la più economica e la prima ad avere i servizi nuovi, ma dai dati europei hai latenza e implicazioni GDPR.

La **barra di ricerca** in alto è il modo più veloce per trovare un servizio: scrivi "Lambda", "SQS", "RDS" e ci arrivi in un colpo. Non serve memorizzare la gerarchia dei menu.

## IAM: chi può fare cosa

IAM — *Identity and Access Management* — è il sistema di identità e permessi di AWS. L'hai visto come concetto in 1.3; qui prende forma concreta.

Quando crei un account AWS ottieni l'**account root** — un'identità con permessi assoluti su tutto. **Non usarla per il lavoro quotidiano.** L'account root serve per le operazioni iniziali (impostare billing, creare il primo utente IAM) e poi va nel cassetto con MFA — *Multi-Factor Authentication* — attivato.

Per lavorare crei **utenti IAM** o, meglio oggi, usi **IAM Identity Center** (ex AWS SSO — *Single Sign-On*) per gestire accessi federati. Per i servizi che girano su AWS e devono chiamare altri servizi, non usi utenti IAM: usi **IAM Role** — identità temporanee che un servizio (es. Lambda, EC2) assume per ottenere permessi senza credenziali fisse nel codice.

```
Account root
  └── IAM User (tu, per la console)
  └── IAM Role (per Lambda, EC2, ECS task — permessi temporanei)
  └── IAM Policy (documento JSON che dice "può fare X su risorsa Y")
```

Il **principio del least privilege** qui è concreto: una Lambda che legge da S3 deve avere solo `s3:GetObject` su quel bucket specifico — non `s3:*` su `*`. Un errore di configurazione IAM è la causa #1 di data breach in AWS.

<details>
<summary>IAM Policy: come è fatta una policy JSON</summary>

Una policy IAM è un documento JSON con uno o più `Statement`. Ogni statement dice: chi (`Principal`, implicito se allegato a un utente/ruolo), su cosa (`Resource`), può fare cosa (`Action`), e con quale effetto (`Effect: Allow` o `Deny`).

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::mio-bucket-prod/*"
    }
  ]
}
```

L'`ARN` — *Amazon Resource Name* — è l'identificatore univoco di ogni risorsa in AWS. Formato: `arn:aws:servizio:regione:account-id:tipo/nome`. Impararla a memoria non serve: la console te la mostra ovunque, e Terraform la genera in automatico.

**Deny esplicito batte Always Allow**: se una policy dice `Allow` e un'altra dice `Deny`, vince il `Deny`. Questo è il comportamento sicuro per default.
</details>

## Billing e budget alert: fallo adesso

Questo è il punto che i tutorial saltano e che ti porta la bolletta da 300 euro a fine mese.

AWS è pay-as-you-go: ogni risorsa che accendi costa, ogni byte che trasferisci fuori dalla rete AWS costa, ogni API call a certi servizi costa. Non ci sono avvisi automatici quando superi una soglia — **a meno che tu non li imposti**.

Cosa fare entro i primi 10 minuti di un account nuovo:

1. **Attiva il billing alert**: Console → Billing → Billing preferences → abilita "Receive Free Tier Usage Alerts" e "Receive Billing Alerts".
2. **Crea un Budget**: Console → Billing → Budgets → crea un budget mensile (es. 20€) con notifica email al 80% e al 100%. Costa zero.
3. **Imposta un Cost Anomaly Detection**: Console → Billing → Cost Anomaly Detection → crea un monitor per l'account. Ti manda un alert se la spesa quotidiana schizza sopra una soglia anomala. Gratis fino a certi volumi.

Le sorprese più comuni nei primi 90 giorni:
- **NAT Gateway dimenticato acceso**: ~$32/mese fisso + $0.045/GB di traffico. Un solo NAT Gateway su una VPC con traffico moderato può fare $50-100/mese.
- **Istanza EC2 lasciata accesa**: un `t3.medium` costa ~$30/mese. Un `m5.xlarge` ~$140/mese. Se la crei per testare e dimentichi di fermarla, te la trovi in bolletta.
- **Snapshot EBS non eliminati**: gli snapshot di volumi EBS costano $0.05/GB/mese. Accumuli decine di snapshot da vecchi test senza accorgertene.
- **Traffico in uscita (egress)**: trasferire dati *fuori* da AWS costa $0.09/GB (prima fascia). Dentro AWS, tra servizi della stessa regione, è quasi gratuito o molto economico. I costi di egress sono la prima sorpresa di chi non li ha mai visti.

## AWS CLI: la console da terminale

La **CLI di AWS** (*Command Line Interface*) è il modo corretto di interagire con AWS nel lavoro quotidiano — non la console web, che è lenta, non riproducibile e non automatizzabile.

```bash
# Installazione (macOS)
brew install awscli

# Configurazione: inserisce access key, secret, regione default
aws configure

# Esempio: lista tutti i bucket S3
aws s3 ls

# Esempio: invoca una Lambda
aws lambda invoke --function-name mia-funzione output.json
```

Con Terraform (Parte 3) la usi indirettamente — Terraform parla con AWS tramite le stesse credenziali che configuri con `aws configure`. Per il lavoro quotidiano, `aws configure` con un utente IAM con permessi limitati (o meglio, con un profilo SSO) è il punto di partenza.

**Mai mettere access key e secret nel codice o in un repository.** Il bot che scansiona GitHub alla ricerca di chiavi AWS è reale e la bolletta arriva in ore. Le chiavi vanno in variabili d'ambiente o, meglio, si usa IAM Role con OIDC nelle pipeline CI/CD (vedi 3.3).

## La mappa dei servizi

AWS ha oltre 200 servizi. La maggior parte non ti serve mai — o non subito. Una mappa delle categorie più frequenti:

| Categoria | Servizi principali |
|---|---|
| Compute | EC2, Lambda, ECS/Fargate, EKS |
| Storage | S3, EBS, EFS |
| Database | RDS (SQL), DynamoDB (NoSQL), ElastiCache (Redis) |
| Messaggistica | SQS, SNS, EventBridge |
| Networking | VPC, Route 53, CloudFront, API Gateway, ALB |
| IaC | CloudFormation, CDK |
| Observability | CloudWatch, X-Ray |
| Sicurezza | IAM, Secrets Manager, KMS, WAF |
| AI/ML | Bedrock, SageMaker |

Le lezioni 4.2–4.7 coprono le categorie più importanti una per una. L'obiettivo non è memorizzare la lista — è che alla fine di ogni lezione, quando senti il nome del servizio, sai cosa fa e quando si usa.

## Cosa non è

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Devo imparare tutti i servizi AWS" | Ne esistono oltre 200, ma il 90% dei sistemi reali ne usa una ventina. Concentrati su quelli della mappa qui sopra. |
| "L'account root è il mio utente principale" | L'account root ha permessi assoluti e non può essere ristretto. Usarlo quotidianamente è un rischio di sicurezza grave. Crea subito utenti IAM separati. |
| "AWS mi avvisa se spendo troppo" | Non lo fa, a meno che tu non imposti esplicitamente budget alert e anomaly detection. Fallo nel primo accesso. |
| "I servizi AWS funzionano uguale in tutte le regioni" | La maggior parte sì, ma alcuni servizi nuovi sono disponibili prima in `us-east-1`. Alcuni servizi come IAM e Route 53 sono globali (non per regione). |

## Verifica di comprensione

> Rispondi a memoria. Le risposte incerte rivedile domani.

1. Cosa succede se selezioni la regione sbagliata nella console AWS e non trovi le tue risorse?
2. Perché non si usa l'account root per il lavoro quotidiano?
3. Qual è la differenza tra un IAM User e un IAM Role?
4. Nomina tre tipi di costi AWS che un principiante scopre con sorpresa nella bolletta.
5. Cos'è un ARN e a cosa serve?
6. Come configuri la CLI AWS perché usi le tue credenziali nel terminale?
7. *(anticipazione)* Stai scrivendo una funzione Lambda che deve leggere da un bucket S3. Come gli dai i permessi senza inserire credenziali nel codice?

## Glossario della lezione

- **Console di gestione AWS**: interfaccia web per accedere a tutti i servizi AWS. Utile per esplorare, non per lavorare in produzione.
- **Regione**: area geografica indipendente (es. `eu-west-1`). Le risorse esistono in una regione specifica.
- **IAM** (*Identity and Access Management*): sistema AWS per gestire identità e permessi.
- **Account root**: identità con permessi assoluti creata alla registrazione. Da non usare quotidianamente.
- **IAM User**: identità umana con credenziali permanenti per accedere ad AWS.
- **IAM Role**: identità temporanea assunta da servizi AWS (Lambda, EC2) per ottenere permessi senza credenziali fisse.
- **IAM Policy**: documento JSON che definisce permessi. Si allega a utenti, gruppi o ruoli.
- **ARN** (*Amazon Resource Name*): identificatore univoco di ogni risorsa in AWS.
- **Least privilege**: principio di sicurezza — dare solo i permessi strettamente necessari.
- **Egress**: traffico in uscita da AWS verso internet. Genera costi significativi.
- **CLI** (*Command Line Interface*): strumento da terminale per interagire con AWS in modo riproducibile.

## Per approfondire

- **AWS Free Tier**: cerca "AWS Free Tier" su `aws.amazon.com` — molti servizi hanno un livello gratuito nei primi 12 mesi. Leggi i limiti prima di usarli in produzione.
- **AWS Billing docs**: cerca "AWS Billing and Cost Management" su `docs.aws.amazon.com` — la sezione "Getting started" copre budget, alert e cost explorer.
- **"AWS IAM best practices"** su `docs.aws.amazon.com/IAM` — la checklist ufficiale: MFA, rotazione chiavi, least privilege.

## Prossima lezione

Sai come muoverti nella console, sai chi può fare cosa, e hai i guardrail di spesa attivi. Adesso è il momento di fare girare codice su AWS. La prossima lezione copre le tre scelte di compute — EC2, ECS/Fargate e Lambda — con i loro punti di forza, i costi reali, e la logica per scegliere tra loro.
