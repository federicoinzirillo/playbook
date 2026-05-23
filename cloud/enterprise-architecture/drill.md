---
title: "Decision drill — Landing zone enterprise"
sidebar_position: 6
---

# Decision drill — Landing zone enterprise

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 9.6</span>
  <span>~15 min (drill)</span>
</div>

<p class="lesson-lead">Un'azienda con 50 developer, 3 team e requisiti SOC 2 Type II deve costruire la sua landing zone AWS da zero. Disegna la struttura account, scegli gli SCP, progetta il provisioning. Poi leggi la griglia.</p>

Questo drill mette insieme tutto quello che hai visto nella Parte 9: Well-Architected Framework, governance multi-account, migrazione, compliance. Non è un esercizio astratto — è il tipo di domanda che un cliente enterprise fa al SA prima di firmare qualsiasi contratto.

---

## Scenario

**Contesto:**
FinTrack è una startup serie B nel settore fintech italiano. Ha 50 developer distribuiti su 3 team:
- **Team Platform**: infrastruttura condivisa, CI/CD, sicurezza
- **Team Product**: l'applicazione principale — backend Python, PostgreSQL, code di messaggi
- **Team Data**: pipeline di analytics, ML, data warehouse

**Vincoli:**
- Certificazione **SOC 2 Type II** richiesta entro 18 mesi (prerequisito per i clienti enterprise)
- Dati dei clienti **classificati PII** — residenza obbligatoria in UE (GDPR)
- Team attualmente su un **singolo account AWS**, tutto mescolato
- Budget: €8.000/mese di infrastruttura attuale, €12.000 tollerati durante la transizione
- Nessun team dedicato di security — il Team Platform gestisce anche la sicurezza

**Stato attuale:**
- Backend deployato su EC2 con deploy manuale via SSH
- Database RDS in un singolo account senza separazione prod/dev
- Nessun CloudTrail abilitato
- Nessuna policy IAM strutturata — molti developer hanno `AdministratorAccess`
- Nessun processo formale di provisioning delle risorse

---

## Domande — decidi e giustifica

Rispondi a ognuna come se dovessi presentare la soluzione al CTO di FinTrack. Scrivi le decisioni e le ragioni prima di leggere la griglia.

**Domanda 1 — Struttura degli account:**
Quanti account AWS crei? Come li organizzi in OU? Disegna la struttura.
Quali account crei subito (giorno 1) e quali aggiungi in un secondo momento?

**Domanda 2 — SCP obbligatorie:**
Quali SCP applichi alla Root o alle OU? Elenca almeno 4 SCP concrete con la policy che implementeresti.
Qual è la differenza tra le SCP che applichi alla Root vs quelle alla OU Sandbox?

**Domanda 3 — Provisioning degli account:**
Usi Control Tower o configuri AWS Organizations a mano? Perché?
Come garantisci che ogni nuovo account rispetti la baseline di sicurezza SOC 2 senza configurazione manuale?

**Domanda 4 — Log e audit:**
Dove vanno i log di CloudTrail? Come li proteggi dalla modifica o cancellazione?
Chi ha accesso ai log? Con quali permessi?

**Domanda 5 — IAM e accesso degli sviluppatori:**
Come risolvi il problema dei developer con `AdministratorAccess`?
Come accedono gli sviluppatori agli account (federazione, IAM user, SSO)?

**Domanda 6 — Piano di migrazione:**
Usando le 7R, come classifichi i workload attuali (backend, database, pipeline analytics)?
Qual è la sequenza logica della migrazione — cosa muovi per primo?

**Domanda 7 — Well-Architected:**
Dopo il primo mese di cloud, conduci una WAR sul workload principale.
Quali HRI ti aspetti di trovare dato lo stato attuale? Elencane almeno 3.

---

## Griglia di valutazione

### Domanda 1 — Struttura degli account

**Risposta attesa:**

Struttura minima per SOC 2 (giorno 1):
- **Management Account**: solo billing e Organizations. Zero workload.
- **Security/Audit Account**: Security Hub, GuardDuty, Config aggregato.
- **Log Archive Account**: CloudTrail aggregato con S3 Object Lock (WORM — Write Once Read Many). Accesso read-only per audit.
- **Product Dev**: ambiente di sviluppo del Team Product.
- **Product Prod**: ambiente di produzione del Team Product.
- **Data Dev / Data Prod**: due account per il Team Data (pipeline ML e analytics spesso richiedono permessi ampi — isolare è critico).
- **Sandbox**: account usa-e-getta per sperimentazione. Budget cappato, si resetta periodicamente.

Il Shared Services Account (VPN, Active Directory) può arrivare in un secondo momento se non c'è connettività on-premises.

**OU suggerita:**
```
Root
├── OU: Security (Audit, Log Archive)
├── OU: Produzione (Product Prod, Data Prod)
├── OU: Sviluppo (Product Dev, Data Dev)
└── OU: Sandbox (Sandbox)
```

**Trappole comuni:**
- Un unico account "prod" con tutti i workload — aumenta il blast radius e non soddisfa la separazione dei log richiesta da SOC 2.
- Non separare Data da Product — le pipeline ML spesso usano permessi ampi su S3; tenerle insieme al backend di produzione è un rischio.
- Dimenticare il Management Account dedicato — avere workload nell'account management è un anti-pattern grave.

---

### Domanda 2 — SCP obbligatorie

**Risposta attesa (minimo 4):**

1. **Blocca l'uso di regioni fuori dall'UE** (GDPR + SOC 2): `Deny *` su `aws:RequestedRegion` con whitelist `eu-west-1`, `eu-central-1`. Applicata alla Root — vale per tutta l'organizzazione.

2. **Vieta la disabilitazione di CloudTrail**: `Deny cloudtrail:StopLogging` e `cloudtrail:DeleteTrail` su `*`. Applicata alla Root.

3. **Vieta la creazione di bucket S3 pubblici**: `Deny s3:PutBucketPublicAccessBlock` con `Condition` che verifica la rimozione dei blocchi. O più semplicemente: imposta il Public Access Block a livello account con SCP.

4. **Vieta la creazione di IAM User con accesso console** nelle OU Produzione (forza l'uso di ruoli federati): `Deny iam:CreateUser` nelle OU Prod.

5. **Cappa il budget della Sandbox**: non è una SCP ma un AWS Budget con alert e azione automatica di stop delle istanze.

**Differenza Root vs Sandbox:**
Le SCP alla Root si applicano a tutti gli account — sono i guardrail non negoziabili (sicurezza, compliance, data residency). Le SCP alla OU Sandbox possono essere più permissive sulla sperimentazione ma più restrittive su costo (es. `Deny ec2:RunInstances` per istanze più grandi di `m5.large`).

---

### Domanda 3 — Provisioning con Control Tower

**Risposta attesa:** Control Tower, non Organizations a mano.

Con SOC 2 in scope, ogni account deve nascere già configurato con CloudTrail abilitato, Config attivo, Security Hub enrollato e GuardDuty abilitato. Farlo a mano è lento e soggetto a errori — su 8-10 account il rischio è quasi certo che qualcosa venga saltato.

Control Tower con Account Factory garantisce che ogni account, dal primo giorno, rispetti la baseline. Il costo di setup di Control Tower è recuperato dalla prima volta che si crea un account senza dimenticare un guardrail.

**Trappola**: scegliere di configurare Organizations a mano "perché è più semplice". Lo è per i primi 3 account; non lo è al decimo, e non regge un audit SOC 2.

---

### Domanda 4 — Log e audit

**Risposta attesa:**

CloudTrail multi-account aggregato sul Log Archive Account. Il bucket S3 deve avere:
- **Object Lock in modalità Compliance** con retention di almeno 1 anno (SOC 2 richiede tipicamente 12 mesi)
- **KMS encryption** con chiave gestita nel Log Archive Account
- **Bucket policy** che nega l'eliminazione anche agli amministratori del Log Archive Account

Accesso ai log: solo il team security (read-only), tramite ruolo federato. L'account Log Archive non deve avere accesso dalla console di produzione — separazione netta.

**Trappola**: tenere i log nello stesso account del workload. Se un attaccante compromette l'account di produzione, può cancellare le tracce.

---

### Domanda 5 — IAM e accesso developer

**Risposta attesa:**

Eliminare tutti gli IAM User con `AdministratorAccess`. Usare **AWS IAM Identity Center** (ex-SSO) per federare l'accesso da un identity provider centrale (anche un semplice account Okta o l'Entra ID aziendale se esiste).

Ogni developer ottiene un ruolo con permessi proporzionali all'account e all'ambiente:
- Accesso a Product Dev: permessi ampi ma non root
- Accesso a Product Prod: solo CI/CD in modalità assume-role, nessun accesso console manuale (practice nota come "break-glass" solo per emergenze documentate)
- Nessun accesso diretto agli account Security e Log Archive

**Trappola**: "migriamo le policy IAM attuali e le sistemiamo dopo". Il SOC 2 valuterà l'accesso privilegiato come uno dei primi controlli — è tra le prime cose da sistemare, non l'ultima.

---

### Domanda 6 — Piano di migrazione con le 7R

**Risposta attesa:**

| Workload | Strategia | Motivazione |
|---|---|---|
| Backend Python su EC2 | Replatform → ECS Fargate | Rimuove la gestione OS, abilita scaling automatico, mantiene lo stesso codice |
| RDS PostgreSQL | Retain + Multi-AZ | È già un managed service — aggiungi Multi-AZ e automated backup, fine |
| Deploy manuale via SSH | Retire + sostituisci con CI/CD | Non è un workload — è un anti-pattern da eliminare |
| Pipeline analytics (batch) | Replatform → AWS Batch / Step Functions | Gestisci la scheduling senza EC2 sempre acceso |
| Pipeline ML | Replatform → SageMaker Pipelines o Managed Airflow | Isola nel Data account, usa servizi managed per training e inference |

**Sequenza logica**: prima i fondamenti di sicurezza (CloudTrail, IAM, multi-account), poi la separazione prod/dev, poi la migrazione dei workload uno per volta con il minor blast radius possibile.

**Trappola**: iniziare dalla migrazione del database — è il workload più rischioso e dovrebbe arrivare dopo che l'ambiente è stabile.

---

### Domanda 7 — HRI attesi dalla WAR

**Risposta attesa (almeno 3):**

1. **Security — IAM**: developer con `AdministratorAccess` in produzione. HRI nel pilastro Security, sezione "Identità e accesso".

2. **Reliability — Backup**: nessun backup testato su RDS. L'automated backup esiste se attivato, ma non è mai stato testato con un restore reale. HRI nel pilastro Reliability.

3. **Security — Logging**: CloudTrail non abilitato = nessuna audit trail. HRI gravissimo in ottica SOC 2.

4. **Operational Excellence — Deployment**: deploy manuale via SSH non è né riproducibile né rollbackabile. HRI nel pilastro Operational Excellence.

5. **Cost Optimization**: istanze EC2 probabilmente sovra-provisionate perché non c'è mai stato un rightsizing. MRI (non HRI, ma certamente emergerebbe).

---

## Nota finale

Un drill come questo non ha una risposta unica. Quello che valuta un senior SA non è la correttezza assoluta della struttura, ma la **coerenza tra vincoli e decisioni**. Se dici "uso un solo account per semplicità", la risposta è sbagliata non perché sia una scelta tecnica impraticabile, ma perché contraddice il vincolo SOC 2 esplicito nel problema. Se dici "uso 20 account", la risposta è sbagliata perché la complessità di gestione supera il valore per un team di 50 persone senza un security team dedicato.

Il giudizio è architetturale, non meccanico.
