---
title: IaC su AWS
sidebar_position: 6
---

# IaC su AWS

<div class="lesson-meta">
  <span class="badge-stato evoluzione">In evoluzione</span>
  <span>Lezione 5.6</span>
  <span>~11 min di lettura</span>
</div>

<p class="lesson-lead">Terraform, CloudFormation, CDK — tre modi per descrivere l'infrastruttura AWS come codice. I principi li hai visti in 4.1. Qui vedi come si traducono in pratica su AWS, con le differenze reali che contano quando scegli.</p>

Sei già arrivato qui avendo visto Terraform in modo generale (4.1). Questa lezione non ripete i concetti — li ancora su AWS: come si configura il provider AWS, quali risorse Terraform gestisce meglio, dove CloudFormation è inevitabile, e quando CDK è la scelta giusta.

## Il provider AWS di Terraform

Terraform parla con AWS tramite il **provider AWS** — un plugin mantenuto da HashiCorp e dalla community che mappa ogni risorsa Terraform in API AWS. È il provider più maturo e più usato dell'ecosistema Terraform.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "eu-west-1"
}
```

Le credenziali vengono dall'ambiente: Terraform legge le stesse variabili di `aws configure` (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) o, meglio, usa **IAM Role with Instance Profile** quando gira su EC2/ECS, o OIDC nelle pipeline CI/CD (vedi 3.3).

**Backend S3 per lo stato**: il pattern standard su AWS è tfstate su S3 con locking via DynamoDB — già visto in 3.1. In AWS la config è:

```hcl
terraform {
  backend "s3" {
    bucket         = "mio-progetto-tfstate"
    key            = "prod/terraform.tfstate"
    region         = "eu-west-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

Il bucket S3 e la tabella DynamoDB vanno creati manualmente (prima volta — il problema bootstrap). Poi tutto il resto si gestisce con Terraform.

**Moduli AWS validati**: il Terraform Registry (`registry.terraform.io/modules/terraform-aws-modules`) ha moduli verificati per VPC, EKS, RDS, ALB, Lambda. Usarli riduce drasticamente il codice da scrivere e incorpora le best practice. Es. il modulo `terraform-aws-modules/vpc/aws` crea l'intera struttura VPC (subnet pubbliche/private, IGW, NAT Gateway, route table) con ~15 righe di configurazione.

## CloudFormation — lo strumento nativo AWS

**CloudFormation** è il servizio IaC nativo di AWS. Template YAML o JSON che descrivono risorse AWS; AWS gestisce la creazione, l'aggiornamento e la cancellazione in uno **stack** (gruppo di risorse trattate come unità).

CloudFormation ha un vantaggio reale su Terraform in un caso: risorse AWS di nuova introduzione. Ogni volta che AWS lancia un nuovo servizio, CloudFormation lo supporta quasi subito (è dello stesso vendor). Il provider Terraform può avere un ritardo di settimane o mesi. Se lavori con servizi AWS sperimentali o di nicchia, CloudFormation può essere l'unica opzione IaC.

L'altro vantaggio: CloudFormation è free — non hai costi di state file storage o licenze. Terraform con backend S3 + DynamoDB è quasi gratis (~pochi centesimi al mese), ma non zero.

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: mio-progetto-prod
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
```

Il limite di CloudFormation: la sintassi YAML/JSON è verbosa, non ha la potenza espressiva di Terraform HCL (niente `for_each` pulito, logica condizionale limitata, moduli meno eleganti). Per infrastrutture grandi e complesse, Terraform scala meglio.

## AWS CDK — infrastruttura in TypeScript o Python

**CDK** (*Cloud Development Kit*) è l'approccio di AWS all'IaC in linguaggio general-purpose. Scrivi l'infrastruttura in TypeScript, Python, Java o Go; CDK compila a CloudFormation.

```python
from aws_cdk import (
    aws_lambda as _lambda,
    aws_apigateway as apigw,
    Stack
)

class MioStack(Stack):
    def __init__(self, scope, id, **kwargs):
        super().__init__(scope, id, **kwargs)

        fn = _lambda.Function(
            self, "MiaFunzione",
            runtime=_lambda.Runtime.PYTHON_3_12,
            handler="handler.main",
            code=_lambda.Code.from_asset("lambda/")
        )

        apigw.LambdaRestApi(self, "MioApi", handler=fn)
```

Il punto di forza del CDK: **astrazioni di alto livello** (*Constructs* L2 e L3). Un `LambdaRestApi` crea automaticamente la Lambda, l'API Gateway, i permessi IAM necessari, il deployment. Con Terraform avresti 4-5 risorse separate da definire e collegare; con CDK è una riga.

Il punto debole: compila a CloudFormation, quindi il codice generato è spesso verboso e difficile da debuggare quando qualcosa va storto. E CDK è AWS-only — se mai migri a multi-cloud, lo riscrivi da zero.

**Quando CDK vince su Terraform**: team che preferiscono TypeScript/Python, infrastrutture fortemente condizionate dalla logica applicativa (es. "crea una Lambda per ogni entry nel database"), pattern che CDK Constructs gestisce con poche righe.

**Quando Terraform vince su CDK**: infrastruttura multi-cloud o multi-provider, team che già conoscono HCL, necessità di state granulare e moduli riusabili tra progetti diversi, stabilità a lungo termine.

<details>
<summary>SAM — Serverless Application Model</summary>

**SAM** (*Serverless Application Model*) è un superset di CloudFormation specifico per applicazioni serverless (Lambda + API Gateway + DynamoDB). Riduce la verbosità di CloudFormation per i pattern serverless più comuni.

```yaml
Resources:
  MiaFunzione:
    Type: AWS::Serverless::Function
    Properties:
      Handler: handler.main
      Runtime: python3.12
      Events:
        Api:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

SAM si usa con `sam build`, `sam deploy`, `sam local invoke` (emula Lambda in locale). È mantenuto da AWS ed è la scelta più rapida per sviluppare e testare applicazioni Lambda in locale prima del deploy su AWS.

Il limite: SAM è per serverless. Per infrastrutture ibride (Lambda + ECS + RDS + VPC), Terraform o CDK scalano meglio.
</details>

## Quale scegliere nel 2026

La risposta dipende dal contesto, ma c'è una mappa ragionevole:

| Situazione | Scelta consigliata |
|---|---|
| Nuovo progetto su AWS, team medio | Terraform con provider AWS + moduli validati |
| Team Python/TS che ama il proprio linguaggio | CDK |
| Nuovo servizio AWS appena uscito non ancora su Terraform | CloudFormation (o aspetta il provider Terraform) |
| Solo Lambda + API Gateway + DynamoDB | SAM per sviluppo locale, Terraform/CDK per produzione |
| Multi-cloud o multi-provider | Terraform, senza alternative reali |
| Già in CloudFormation e funziona | Non migrare solo per moda |

Il segnale più forte del 2026: Terraform (o OpenTofu) resta lo standard de facto per team con infrastrutture complesse. CDK è in forte crescita nei team applicativi che vogliono esprimere l'infra nello stesso linguaggio del codice. CloudFormation è il fallback quando gli altri non supportano ancora il servizio.

## Cosa non è

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "CDK è meglio di Terraform perché uso Python" | CDK compila a CloudFormation — errori in produzione sono errori di CloudFormation. Il debugging è meno diretto. Il vantaggio delle astrazioni L2/L3 è reale, ma il trade-off esiste. |
| "CloudFormation è legacy" | È il layer su cui CDK è costruito. Comprende ogni servizio AWS. Non è legacy — è lo strato infrastrutturale AWS. Usarlo direttamente ha i suoi casi. |
| "Terraform non funziona bene con AWS" | Il provider AWS di Terraform è uno dei più maturi e completi. Copre il 95%+ dei servizi AWS. Il ritardo su nuovi servizi è reale ma raro nella pratica quotidiana. |
| "SAM sostituisce Terraform per serverless" | SAM è ottimo per sviluppo locale e deploy semplici. Per infrastrutture che includono VPC, RDS, ECS, IAM complesso, Terraform gestisce la complessità in modo più scalabile. |

## Verifica di comprensione

> Rispondi a memoria. Le risposte incerte rivedile domani.

1. Dove conserva lo stato Terraform quando lavori su AWS? Quale servizio AWS usi per il locking?
2. Qual è il vantaggio principale di CloudFormation rispetto a Terraform su AWS?
3. Cosa fa AWS CDK con il codice TypeScript/Python che scrivi?
4. In quale scenario CDK ha un chiaro vantaggio su Terraform?
5. Cos'è SAM e per cosa è ottimizzato?
6. Hai un progetto con Lambda + API Gateway + DynamoDB + VPC + RDS. Quale tool IaC sceglieresti e perché?
7. *(anticipazione)* Stai scrivendo un modulo Terraform che crea una Lambda con il suo IAM Role e i permessi per leggere da S3. Come struttureresti le risorse nel modulo?

## Glossario della lezione

- **Provider AWS** (Terraform): plugin che traduce le risorse Terraform in API AWS.
- **Backend S3**: configurazione Terraform per conservare lo stato su S3 con locking DynamoDB.
- **CloudFormation**: servizio IaC nativo AWS — template YAML/JSON → stack di risorse.
- **Stack** (CloudFormation): gruppo di risorse AWS gestite come unità da CloudFormation.
- **CDK** (*Cloud Development Kit*): framework AWS per scrivere infrastruttura in TypeScript/Python, compilata a CloudFormation.
- **Constructs L2/L3** (CDK): astrazioni di alto livello che creano più risorse AWS con poca configurazione.
- **SAM** (*Serverless Application Model*): superset CloudFormation per serverless, con tooling locale per Lambda.
- **Terraform Registry**: repository di moduli e provider Terraform, inclusi i moduli ufficiali AWS.

## Per approfondire

- **Terraform AWS provider**: `registry.terraform.io/providers/hashicorp/aws` — documentazione completa di ogni risorsa AWS supportata.
- **AWS CDK Workshop**: cerca "AWS CDK Workshop" su `cdkworkshop.com` — tutorial hands-on ufficiale.
- **terraform-aws-modules**: `github.com/terraform-aws-modules` — moduli verificati per VPC, EKS, RDS, Lambda.

## Prossima lezione

Hai l'infrastruttura deployata con IaC. L'ultima lezione tecnica della Parte 5 copre CloudWatch — come sai se il sistema funziona, come ricevi alert quando si rompe, e come navighi i log a 3 di notte durante un incident.
