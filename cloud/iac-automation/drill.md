---
title: Decision drill — Cosa automatizzare
sidebar_label: "4.4 Decision drill"
sidebar_position: 4
---

# Decision drill — Cosa automatizzare

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 4.4</span>
  <span>~10 min di lettura</span>
</div>

<p class="lesson-lead">Tre scenari con vincoli reali: team size, budget, rischio di compliance. L'obiettivo non è automatizzare tutto — è sapere cosa vale la complessità dell'automazione e cosa no.</p>

Questa lezione applica IaC, policy-as-code e CI/CD a situazioni concrete. Prova a rispondere prima di aprire la griglia di valutazione.

---

## Scenario A — Startup con un dev, infrastruttura crescente

**Situazione**: sei l'unico sviluppatore di una startup con 3 ambienti (dev, staging, prod) su AWS. Fino a ieri gestivi tutto dalla console. L'infrastruttura è diventata troppo complessa per ricordare a memoria. Non hai un team DevOps.

**Vincoli**: budget limitato, non puoi permetterti Terraform Cloud. Hai bisogno di qualcosa che funzioni in una giornata.

**Domande**:
1. Da dove inizi con l'IaC? Migri tutto subito o solo le parti nuove?
2. Come gestisci il tfstate senza Terraform Cloud?
3. Checkov sì o no? Ha senso con una persona sola?

<details>
<summary>Griglia di valutazione</summary>

**Risposta difendibile**: Terraform (o OpenTofu) per le nuove risorse, backend S3 per lo stato.

- **Migrazione**: non migrare l'infrastruttura esistente subito — importare in Terraform risorse già create (`terraform import`) è tedioso e rischioso. Inizia con le risorse nuove. Le vecchie le migri gradualmente quando devi modificarle comunque.
- **Backend remoto gratis**: S3 bucket + DynamoDB table per il locking. Cost: centesimi al mese. Il bucket va creato manualmente (bootstrapping chicken-and-egg), poi tutto il resto via Terraform.
- **Checkov sì**: si installa con `pip install checkov`, si esegue con `checkov -d .` in 30 secondi. Anche da soli, un tool che ti dice "questo security group è aperto al mondo" è meglio di non averlo. Puoi integrarlo come pre-commit hook locale senza CI.

**Trappola comune**: voler fare tutto bene da subito — Terraform Cloud, Sentinel, GitOps completo. Con un dev solo, il costo cognitivo supera il beneficio. Meglio Terraform semplice + S3 backend + Checkov locale che un sistema perfetto che non si mantiene.
</details>

---

## Scenario B — Team di 8 dev, compliance SOC 2 in arrivo

**Situazione**: startup serie B, 8 sviluppatori, AWS. Tra 6 mesi dovete passare l'audit SOC 2 (*System and Organization Controls 2* — standard di conformità per sicurezza e disponibilità dei dati). Il security team ha prodotto un documento di 50 pagine con le policy ("tutti i bucket cifrati", "tutti i log abilitati", "no accesso pubblico"). Attualmente l'infrastruttura è mista: 60% Terraform, 40% clickops.

**Vincoli**: 6 mesi di tempo, team già sovraccarico di feature development. Non puoi dedicare un ingegnere full-time.

**Domande**:
1. Come trasformi le 50 pagine di policy in controlli automatici?
2. Come gestisci il 40% di infrastruttura non-Terraform senza un refactor completo?
3. Quale approccio CI/CD per le PR di IaC?

<details>
<summary>Griglia di valutazione</summary>

**Risposta difendibile**: Checkov per le policy IaC + AWS Config per il 40% existente + Atlantis per le PR.

- **Policy come codice**: non serve OPA custom per iniziare — Checkov copre la maggior parte delle policy SOC 2 standard (bucket cifrati, log abilitati, security group chiusi) out of the box. Esegui `checkov -d . --framework terraform` e ottieni un report con mapping a CIS Benchmark. Per le policy specifiche non coperte, scrivi check custom in Python.
- **40% clickops**: AWS Config può rilevare risorse non conformi anche quelle non gestite da Terraform. Configura AWS Config Rules per le policy critiche (es. `S3_BUCKET_PUBLIC_READ_PROHIBITED`, `ENCRYPTED_VOLUMES`). Questo copre il 40% finché non lo migri.
- **Atlantis per le PR**: invece di `terraform apply` manuale, Atlantis (self-hosted su un'EC2 t3.micro, ~$8/mese) esegue `terraform plan` automaticamente sui PR e richiede approvazione prima di `apply`. Questo dà il trail di audit che SOC 2 richiede senza Terraform Cloud.

**Trappola comune**: cercare di migrare il 40% clickops in Terraform in 6 mesi mentre si va anche in produzione. Usa AWS Config per coprire il gap di conformità ora; pianifica la migrazione Terraform dopo l'audit.
</details>

---

## Scenario C — Enterprise con ambienti multipli e team distribuiti

**Situazione**: azienda enterprise, 5 team di prodotto su AWS, ~200 risorse gestite da Terraform. Il problema: team diversi hanno stili Terraform diversi, moduli duplicati, e occasionalmente si calpestano lo state file l'uno con l'altro. Ogni deploy in produzione richiede un ticket approvato dal team platform.

**Vincoli**: budget disponibile, ma il team platform non può diventare un bottleneck. L'obiettivo è abilitare i team a deployare in autonomia con guardrail automatici.

**Domande**:
1. Come strutturi il codice Terraform per 5 team senza conflitti di state?
2. Come sostituisci il ticket di approvazione manuale con un gate automatico?
3. Quale strumento di policy-as-code per policy differenziate per team?

<details>
<summary>Griglia di valutazione</summary>

**Risposta difendibile**: state separati per team + Terraform Cloud con Sentinel + workspace per ambienti.

- **State separati**: ogni team ha il proprio root module Terraform con il proprio state file nel proprio S3 bucket con locking DynamoDB. Non uno state file globale. I moduli condivisi vivono in un repository separato (registry interno) — ogni team li referenzia per versione, non per path locale. Questo elimina i conflitti.
- **Gate automatico**: Terraform Cloud (o Enterprise) con Sentinel policies. Il team platform scrive le Sentinel policies ("nessuna istanza EC2 fuori dai tipi approvati", "nessun database senza backup"); i team di prodotto deployan liberamente purché il piano non violi le policy. Il bottleneck del ticket manuale sparisce — è sostituito da un gate automatico.
- **Policy differenziate**: Sentinel in Terraform Cloud supporta policy sets per workspace. Il team che gestisce dati PII ha policy più restrittive (cifratura obbligatoria, log di accesso, nessuna IP pubblica) rispetto al team che gestisce il microsite marketing.

**Trappola comune**: un singolo state file condiviso "per avere visibilità totale". Un singolo state file è un collo di bottiglia (una sola operazione alla volta) e un blast radius enorme (un `terraform destroy` errato abbatte tutto). La separazione degli state per team è la scelta corretta.
</details>

---

## Riepilogo dei pattern

| Scenario | Pattern vincente | Trappola principale |
|---|---|---|
| A — 1 dev, startup | Terraform + S3 backend + Checkov locale | Over-engineering con strumenti enterprise |
| B — SOC 2 in 6 mesi | Checkov + AWS Config + Atlantis | Migrare il 40% clickops prima dell'audit |
| C — Enterprise multi-team | State separati + Sentinel + policy set | State file globale condiviso |

## Prossima parte

Hai completato la Parte 4 — IaC, policy-as-code, CI/CD. Ora il quadro concettuale è completo. La Parte 5 traduce tutto questo in AWS concreto: i servizi specifici con i loro nomi, le loro console, i loro prezzi. Terraform con provider AWS, SQS/SNS/EventBridge, RDS/DynamoDB/S3, ECS/Lambda con API Gateway — i concetti delle Parti 0-4 con un'etichetta AWS sopra.
