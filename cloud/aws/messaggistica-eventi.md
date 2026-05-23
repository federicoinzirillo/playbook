---
title: "Messaggistica ed eventi su AWS: SQS, SNS, EventBridge, Step Functions"
sidebar_position: 3
---

# Messaggistica ed eventi su AWS: SQS, SNS, EventBridge, Step Functions

<div class="lesson-meta">
  <span class="badge-stato evoluzione">Bozza</span>
  <span>Lezione 4.3</span>
</div>

> Lezione in arrivo. Vedi il [SYLLABUS](/cloud/SYLLABUS), punto 4.3.
>
> Scope previsto: è il "secondo piano" della lezione universale 2.4 (async ed event-driven), in versione AWS-pratica. **SQS** (code FIFO/standard, visibility timeout, DLQ — la rete di sicurezza per i messaggi che falliscono), **SNS** (pub/sub a fan-out: un evento, N sottoscrittori), **EventBridge** (event bus con regole di routing, schema registry, integrazione con SaaS), **Step Functions** (orchestrazione di workflow con stato). Pattern ricorrenti: fan-out SNS→SQS, idempotenza nei consumer, retry con backoff esponenziale, gestione del veleno (poison pill) con DLQ. Quando ognuno è la scelta giusta — e quando combinarli.
