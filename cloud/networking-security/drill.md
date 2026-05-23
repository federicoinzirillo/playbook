---
title: Decision drill — Networking & accessi
sidebar_position: 5
---

# Decision drill — Networking & accessi

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 1.5</span>
  <span>~20 min (con riflessione)</span>
</div>

<p class="lesson-lead">Quattro scenari, vincoli reali. Devi decidere, giustificare, e trovare i buchi — come farebbe un senior durante una code review o un design review.</p>

Questo drill copre le lezioni 1.1–1.4. Per ogni scenario: leggi i vincoli, scrivi la tua risposta prima di guardare la griglia. Non ci sono risposte perfette — ci sono trade-off da riconoscere e trappole da evitare.

---

## Scenario A — API pubblica con backend privato

**Contesto**: stai progettando un'API REST per un'applicazione mobile con 50.000 utenti attivi. Backend su EC2 con autoscaling. Database PostgreSQL su RDS. Budget limitato. SLA richiesto: 99,9% (circa 8,7 ore di downtime/anno).

**Vincoli**:
- I server applicativi non devono essere direttamente raggiungibili da internet
- Il database non deve mai essere raggiungibile da nessuno tranne l'applicazione
- Il team di 4 sviluppatori deve poter fare SSH agli app server per debug emergenziale

**Domanda**: disegna a parole l'architettura di rete e di accesso. Quali componenti, in quali subnet, con quali Security Group?

<details>
<summary>Griglia di valutazione</summary>

**Quello che un senior nominerebbe:**

*Architettura di rete*:
- VPC con almeno due AZ (per il 99,9% SLA — una AZ sola non basta)
- Subnet pubblica in ogni AZ: solo ALB e NAT Gateway
- Subnet privata in ogni AZ: EC2 (autoscaling group), RDS (Multi-AZ)
- Internet Gateway attaccato al VPC
- NAT Gateway nella subnet pubblica (per aggiornamenti outbound dai server privati)

*Security Group*:
- SG-ALB: inbound 443 da 0.0.0.0/0; outbound verso SG-App sulla porta applicativa (es. 8080)
- SG-App: inbound 8080 da SG-ALB; inbound 22 da SG-Bastion; outbound verso SG-DB su 5432; outbound HTTPS (443) verso 0.0.0.0/0 (per chiamate API esterne via NAT)
- SG-DB: inbound 5432 solo da SG-App. Nient'altro.
- SG-Bastion: inbound 22 da IP fissi del team. Outbound 22 verso SG-App.

*Accesso SSH emergenziale*:
- Bastion host (o **AWS Systems Manager Session Manager** che è meglio — niente porte SSH aperte, tutto loggato)
- Mai aprire SSH direttamente da 0.0.0.0/0

**Trade-off attesi da riconoscere:**
- NAT Gateway in una sola AZ è un single point of failure per il traffico outbound (e costa ~$32/mese). Due NAT Gateway (uno per AZ) costano il doppio ma garantiscono HA.
- Session Manager vs Bastion: Session Manager elimina SG-Bastion e la gestione delle chiavi SSH, preferibile per team moderni.
- Il 99,9% SLA richiede Multi-AZ su RDS e almeno due istanze EC2 su AZ diverse.

**Trappole comuni:**
- Mettere RDS in subnet pubblica "per comodità"
- Un Security Group aperto su 0.0.0.0/0 per porta 22
- Una sola AZ (nessuna ridondanza)
- Non considerare il costo del NAT Gateway nel "budget limitato"
</details>

---

## Scenario B — Lambda + RDS in VPC: connettività

**Contesto**: hai una Lambda Python che esegue query su un'istanza RDS PostgreSQL in una subnet privata. Devi anche fare una chiamata a un'API esterna (servizio di geolocalizzazione) ad ogni invocazione. Il budget è molto limitato.

**Domanda**: come configuri la Lambda per accedere sia a RDS (in VPC) sia all'API esterna? Quali sono i costi che rischi di non considerare?

<details>
<summary>Griglia di valutazione</summary>

**Il problema**: una Lambda in VPC può accedere a risorse nel VPC (RDS), ma perde l'accesso internet diretto. Per raggiungere l'esterno ha bisogno del NAT Gateway — che costa.

**Opzioni:**

*Opzione 1 — Lambda in VPC con NAT Gateway*:
- Lambda associata a subnet privata
- Security Group Lambda: outbound 5432 verso SG-RDS, outbound 443 verso 0.0.0.0/0
- NAT Gateway per il traffico verso l'API esterna
- Costo: ~$32/mese (NAT fissi) + costo per GB processato
- Pro: soluzione semplice, tutto in un posto

*Opzione 2 — Lambda in VPC + VPC Endpoint per l'API esterna (se è AWS)*:
- Se l'API esterna è un servizio AWS (es. Bedrock, S3), usa VPC Endpoint Interface
- Elimina il NAT Gateway per quel traffico
- Costo inferiore se il volume di dati è alto

*Opzione 3 — Lambda in VPC (per RDS) + chiamata API tramite Event Bridge o Step Functions*:
- Architetturalmente più complessa, raramente giustificata solo per questo

**Con "budget molto limitato" la risposta onesta è**: NAT Gateway costa ~$32/mese fissi anche a zero traffico — è spesso il costo sorpresa più citato nei post-mortem FinOps. Se il budget è davvero stretto e la Lambda ha poche invocazioni, valuta alternative come **RDS Proxy** (connessione pooling, riduce le connessioni) e VPC Endpoint per i servizi AWS che usi.

**Trappole comuni:**
- Pensare che mettere Lambda in VPC sia gratuito per il networking (non lo è se serve NAT)
- Non stimare il volume di traffico in uscita per il costo del NAT Gateway
- Dimenticare che Lambda in VPC ha cold start più lento (ENI provisioning — risolto con provisioned concurrency)
</details>

---

## Scenario C — Credenziali hardcoded scoperte dall'audit

**Contesto**: l'audit di sicurezza ha trovato la stringa `db_password = "Pr0d_P@ssw0rd2024!"` in `config.py`, committata 8 mesi fa. Il branch è stato rimosso ma il commit esiste ancora nella history del repo privato. Il database RDS è in produzione e riceve traffico. Il team ha 6 persone.

**Domanda**: qual è il piano di remediation immediato e permanente? Ordina le azioni per priorità.

<details>
<summary>Griglia di valutazione</summary>

**Immediato (nelle prossime ore):**

1. **Ruota la password RDS subito** — crea nuova password, aggiornala su RDS, aggiorna il codice in produzione per usarla. Non aspettare: la password è già compromessa, va trattata come tale.
2. **Verifica i log di accesso** — controlla CloudTrail e i log di connessione RDS per accessi anomali nelle ultime ore/giorni. Se ci sono connessioni da IP inattesi, assume breach.
3. **Verifica l'accesso al repo** — chi aveva accesso al repo nelle ultime 8 mesi? Era davvero privato? Bitbucket/GitHub mostrano chi ha clonato il repo.

**A breve (nelle prossime 48 ore):**

4. **Migra la password su Secrets Manager** — non su un `.env` o su una variabile d'ambiente del container. Lambda/EC2 la leggono a runtime via IAM Role.
5. **Rimuovi la password dalla git history** — `git filter-branch` o `git-filter-repo`. Nota: questo riscrive la history, richiede force push, e ogni membro del team deve aggiornare il clone locale. Complesso ma necessario.
6. **Configura `git-secrets` o simili** — pre-commit hook che blocca il commit di pattern che sembrano credenziali (chiavi AWS, password). Installa su tutte le macchine del team.

**Strutturale:**

7. **Rivedi tutti gli altri segreti** — se c'era una password hardcoded, probabilmente non è l'unica. Scansiona l'intero repo con `truffleHog` o `detect-secrets`.
8. **IAM Role per ogni servizio** — niente access key nel codice, mai.
9. **Secrets Manager come standard** — policy del team: nessun segreto fuori da Secrets Manager o SSM Parameter Store.

**Trappole comuni:**
- Cambiare la password senza verificare i log (potresti stare chiudendo la porta dopo che il danno è fatto)
- Dimenticare che "privato" in un repo aziendale non significa "sicuro": ex dipendenti, leak del token di accesso, terze parti con accesso al codice
- Pensare che rimuovere il branch rimuova il commit dalla history
</details>

---

## Scenario D — Architettura multi-team con least privilege

**Contesto**: startup con 3 team (Frontend, Backend, Data). Un singolo account AWS. Il team Frontend ha bisogno di deployare su S3 + CloudFront. Il team Backend ha bisogno di gestire Lambda, API Gateway, DynamoDB, e Secrets Manager per le proprie Lambda. Il team Data ha bisogno di scrivere su S3 (data lake) e leggere da DynamoDB. Nessun team deve poter toccare le risorse degli altri.

**Domanda**: come strutturi IAM per i tre team? Usa il principio del least privilege. Cosa metti nei ruoli, cosa metti nelle policy, e come eviti che un team acceda alle risorse degli altri?

<details>
<summary>Griglia di valutazione</summary>

**Approccio consigliato: ABAC (*Attribute-Based Access Control*) via tag + policy per team**

La chiave è usare i **tag** sulle risorse per distinguerle. Ogni risorsa di un team ha un tag `Team: Frontend` (o `Backend`, `Data`). Le policy usano condition sui tag.

**IAM Role per il team Frontend:**
```json
{
  "Effect": "Allow",
  "Action": ["s3:PutObject", "s3:GetObject", "s3:DeleteObject"],
  "Resource": "arn:aws:s3:::*",
  "Condition": {
    "StringEquals": { "s3:ResourceTag/Team": "Frontend" }
  }
}
```
Più permessi CloudFront per le distribuzioni taggate `Team: Frontend`.

**IAM Role per il team Backend:**
- `lambda:*` su Lambda taggate `Team: Backend`
- `apigateway:*` sulle API taggate `Team: Backend`
- `dynamodb:*` sulle tabelle taggate `Team: Backend`
- `secretsmanager:GetSecretValue` sui segreti con ARN in un path dedicato (es. `/backend/*`)

**IAM Role per il team Data:**
- `s3:PutObject` sul bucket data lake (identificato da ARN, non da tag)
- `dynamodb:Query`, `dynamodb:Scan` su tabelle taggate `Team: Backend` (read-only cross-team)
- No write su DynamoDB Backend

**Cosa un senior aggiungerebbe:**
- **Permission boundary** sui ruoli di ogni team: cap massimo sui permessi, così anche se un developer crea un ruolo "admin" dentro il suo ambiente, non supera il confine.
- Con 3 team e un account, si inizia a sentire il bisogno di account separati (**AWS Organizations + multi-account**). Non è eccessivo: a 10 persone è la norma.
- **Service Control Policy (SCP)** a livello di organizzazione: impedisce che chiunque crei risorse fuori da certe regioni (es. blocca tutto tranne eu-west-1 per compliance GDPR).

**Trappole comuni:**
- Usare nomi delle risorse per distinguerle (invece dei tag): i nomi cambiano, i tag sono policy
- Non separare le risorse di produzione e sviluppo (stesso account, stesse policy — il developer che testa in prod)
- Dare a ogni team il ruolo `AdministratorAccess` "per comodità": è la configurazione che garantisce il massimo caos
</details>

---

## Per continuare

Se hai risposto a tutti e quattro senza guardare le griglie, sei pronto per la Parte 2. Se hai guardato prima di rispondere su qualcuno, rivedi la lezione relativa domani (spacing).

I concetti che più spesso mancano nei senior di background full-stack: Security Group con reference a SG (non a CIDR), il costo del NAT Gateway, la distinzione User vs Role per i processi automatici, e il fatto che "privato" nel repo non equivale a "sicuro".
