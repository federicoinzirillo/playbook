---
title: Decision drill — I costi sono raddoppiati
sidebar_position: 7
---

# Decision drill — I costi sono raddoppiati

<div class="lesson-meta">
  <span class="badge-stato stabile">Stabile</span>
  <span>Lezione 6.7</span>
  <span>~10 min di lettura</span>
</div>

<p class="lesson-lead">Lo scenario è reale. I numeri cambiano, la logica no: questo è il ragionamento che distingue chi capisce il cloud da chi lo subisce.</p>

Questo drill è diverso dalle lezioni precedenti. Non ci sono definizioni da imparare: c'è uno scenario con vincoli reali, una decisione da prendere, e una griglia che mostra cosa avrebbe ragionato un cloud engineer senior.

Leggi lo scenario, **chiudi la pagina**, scrivi la tua analisi su carta o in un file separato. Poi torna e confronta.

---

## Scenario

Sei cloud engineer in un'azienda che gestisce una piattaforma SaaS B2B per la gestione documentale. L'infrastruttura gira su AWS in `eu-west-1`.

Tre settimane fa il CEO ha ricevuto la bolletta AWS: **€18.400 contro €9.200 del mese precedente**. Raddoppio esatto. Non c'è stato nessun lancio di prodotto, nessun accordo con nuovi clienti, nessun evento straordinario evidente.

**L'architettura attuale** (semplificata):
- Frontend: S3 + CloudFront
- Backend API: 12 istanze EC2 `c5.xlarge` dietro un ALB, in due AZ
- Database: RDS PostgreSQL Multi-AZ (`db.r6g.2xlarge`)
- Storage documenti: S3 (circa 8 TB totali)
- Pipeline di elaborazione documenti: SQS + 6 Lambda (per OCR e indicizzazione)
- Cache: ElastiCache Redis (2 nodi `cache.r7g.large`)
- Rete: VPC con 2 subnet pubbliche e 4 private, un NAT Gateway per AZ (2 totali)

Il team di sviluppo ha fatto due deploy nelle ultime 4 settimane:
- Deploy A (3 settimane fa): nuova funzionalità di ricerca semantica — le query di ricerca ora chiamano un embedding model via API esterna.
- Deploy B (2 settimane fa): refactoring del pipeline di elaborazione documenti — ora ogni documento fa più chiamate S3 (lettura + scrittura di versioni intermedie).

Hai accesso a AWS Cost Explorer, CloudWatch, e CloudTrail.

**Vincoli:**
- Non puoi spegnere niente in produzione senza approvazione del CTO
- La piattaforma ha SLA del 99,5% mensile con i clienti enterprise
- Il budget mensile approvato è €10.000; serve portare i costi entro quella cifra entro 30 giorni

---

## Decidi e giustifica

Prima di leggere la griglia, rispondi a queste domande:

1. Quali sono le prime tre voci che controlli in Cost Explorer? In che ordine e perché?
2. Qual è l'ipotesi più probabile per il raddoppio? Su cosa punteresti la prima analisi?
3. Come distingui se il problema viene dal Deploy A, dal Deploy B, o da qualcos'altro?
4. Nomina almeno due ottimizzazioni immediate (senza cambio architetturale significativo) che potresti proporre al CTO.
5. C'è una voce nella lista dell'architettura che, guardandola, ti sembra il candidato più ovvio per un costo nascosto rilevante?

---

## Griglia di valutazione

### Dove guardare prima

**1. Cost Explorer — breakdown per servizio, confronto settimana per settimana.**

Il primo gesto non è cercare la causa: è localizzare la voce. Qual servizio AWS è cresciuto? Di quanto? Da quando? Cost Explorer con granularità giornaliera e breakdown per servizio ti dà la risposta in 5 minuti. Se la crescita è concentrata su EC2, Data Transfer, S3, Lambda, o "Other", hai già ristretto il campo di metà.

**2. Data Transfer — la voce nascosta.**

In questa architettura, il candidato principale è il **traffico di rete**. Perché? Tre segnali:
- Deploy B ha aumentato le chiamate S3 (lettura + scrittura di versioni intermedie per ogni documento). Ogni chiamata S3 da Lambda che è dentro la VPC passa attraverso il NAT Gateway → costo per GB processato.
- Le query di ricerca semantica del Deploy A chiamano un'API esterna → egress Internet per ogni query.
- 2 NAT Gateway attivi (costo fisso: ~$0.045/h ciascuno, ~$65/mese fisso + variabile per GB).

La cifra da verificare: in Cost Explorer, guarda "Data Transfer" e "NAT Gateway" come voci separate. Se uno dei due è esploso nella settimana del Deploy B, hai la causa.

**3. S3 — costo per richiesta, non solo storage.**

S3 si paga non solo per i GB: si paga per ogni operazione (PUT, GET, LIST). Se il Deploy B fa 10 operazioni S3 per documento invece di 2, su un volume di documenti significativo il costo per richiesta scala. In Cost Explorer, cerca il dettaglio "S3 Requests" separato da "S3 Storage".

---

### L'ipotesi più probabile

Il candidato principale è il **Deploy B** — il refactoring del pipeline documenti con più operazioni S3 intermedie.

Ragionamento: le Lambda del pipeline girano dentro la VPC (presumibilmente, per accedere a RDS o ElastiCache). Ogni chiamata S3 da Lambda in VPC, se S3 non ha un VPC Endpoint configurato, passa attraverso il NAT Gateway e genera costo di processamento dati. Moltiplicato per il volume documenti e per il numero di operazioni S3 aggiuntive, il raddoppio è plausibile.

Il Deploy A è il secondo candidato: le chiamate all'embedding API esterna generano egress. Ma l'egress puro è $0.09/GB — serve un volume elevato per raddoppiare la bolletta. Più probabile che sia rumore rispetto al NAT Gateway.

---

### Come distinguere tra Deploy A e Deploy B

Vai su Cost Explorer e imposta la granularità giornaliera. Il giorno in cui la spesa ha iniziato a crescere è la data del deploy causante. Se la crescita inizia 3 settimane fa → Deploy A. Se inizia 2 settimane fa → Deploy B.

Poi: CloudWatch Metrics per il NAT Gateway — "BytesOutToSource" e "BytesOutToDestination" — con breakdown per timestamp. Se c'è uno spike coincidente con il Deploy B, il caso è chiuso.

---

### Ottimizzazioni immediate (senza cambio architetturale)

**S3 VPC Endpoint (Gateway Endpoint).** Se le Lambda chiamano S3 dalla VPC senza un Gateway Endpoint, il traffico passa dal NAT Gateway. Un **S3 Gateway Endpoint** è gratuito: crea un percorso privato diretto da VPC a S3 senza passare per NAT. Una riga di configurazione Terraform (o 5 click in console), nessun impatto sul traffico, risparmio potenzialmente significativo. Questa è la prima cosa da fare.

**Ridurre le operazioni S3 intermedie nel pipeline.** Se il Deploy B fa 10 operazioni S3 dove prima ne bastava 1 (es. lettura → processing → scrittura versione 1 → scrittura versione 2 → scrittura finale), valutare se alcune versioni intermedie possono rimanere in memoria invece che essere materializzate su S3. Richiede una piccola modifica al codice, basso rischio.

**Revisione dell'embedding cache.** Il Deploy A chiama un'API esterna per ogni query di ricerca. Se le query si ripetono (ricerche simili sugli stessi documenti), una cache Redis davanti alle chiamate all'embedding API riduce sia il costo dell'API esterna sia l'egress. ElastiCache Redis è già nell'architettura — costa solo il codice.

---

### Il candidato ovvio per il costo nascosto

**I 2 NAT Gateway.** Costo fisso ~$130/mese (2 × $0.045 × 24h × 30gg), più il costo variabile per ogni GB processato. Se il pipeline documenti — con le sue chiamate S3 intensive — passa per il NAT Gateway, e il volume è alto, questa singola voce può spiegare il raddoppio.

Nota: avere 2 NAT Gateway (uno per AZ) è architetturalmente corretto per la resilienza. Il problema non è la presenza dei NAT Gateway: è che il traffico S3 non dovrebbe passarci sopra. L'S3 Gateway Endpoint risolve esattamente questo.

---

### Il trade-off che un senior nominerebbe

Prima di ottimizzare aggressivamente: verifica che il comportamento sia un bug o una scelta. Se il Deploy B è by design (versioni intermedie S3 sono parte del design del pipeline per audit trail o rollback), la soluzione non è toglierle — è spostare il traffico fuori dal NAT Gateway con l'Endpoint. Se invece sono un'inefficienza involontaria del refactoring, si eliminano dal codice.

La distinzione è importante: ottimizzare senza capire può rimuovere funzionalità, non solo costi.

---

## Cosa avrebbe detto un senior che non sai ancora

Un cloud engineer senior, davanti a questo scenario, avrebbe fatto una cosa prima ancora di aprire Cost Explorer: **guardare i budget alert**.

Se i budget alert fossero stati configurati correttamente — con notifica al 80% della soglia mensile — il problema sarebbe emerso nella prima settimana, non dopo tre settimane quando la bolletta era già al doppio. La retroazione è: imposta budget alert sul day-one di ogni account, non a fine mese.

Il secondo commento: il fatto che il costo sia raddoppiato esattamente in coincidenza con due deploy — senza nessun cambiamento del traffico esterno — è quasi sempre un segnale di **costo generato dal proprio codice**, non dal traffico utente. Il primo posto dove guardare non è "i clienti hanno fatto più cose" ma "abbiamo cambiato qualcosa nell'infrastruttura o nel codice che ha cambiato il profilo di chiamate AI interne".

---

## Per approfondire

- **AWS Cost Explorer documentation** — `docs.aws.amazon.com/cost-management`. Come usare il breakdown per servizio, tag, e account.
- **S3 VPC Endpoints** — `docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html`. Gateway vs Interface endpoint, configurazione, nessun costo aggiuntivo per Gateway.
- **NAT Gateway pricing** — `aws.amazon.com/vpc/pricing`. Dettaglio costi fissi e variabili, confronto con PrivateLink.

## Prossima lezione

La Parte 6 è completa. La Parte 7 è la sintesi: una reference architecture cloud, il capstone end-to-end su AWS con IaC, e il portfolio che fa assumere. Il lavoro fatto fin qui — foundations, networking, compute, IaC, AWS, AI, operations — converge in un unico artefatto da mostrare.
