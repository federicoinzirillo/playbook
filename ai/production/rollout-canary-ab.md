---
title: "Rollout sicuro: canary, A/B e shadow traffic per LLM"
sidebar_position: 4
---

# Rollout sicuro: canary, A/B e shadow traffic per LLM

<div class="lesson-meta">
  <span class="badge-stato evoluzione">Bozza</span>
  <span>Lezione 6.4</span>
</div>

> Lezione in arrivo. Vedi il [SYLLABUS](/ai/SYLLABUS), punto 6.4.
>
> Scope previsto: cambio di modello, prompt, o pipeline RAG in produzione senza far scoppiare tutto. Canary release (5% → 25% → 100%) con criteri di rollback automatico legati alle metriche di 3.2/6.3. A/B testing per scelte qualitative (quale prompt converte meglio). Shadow traffic: il nuovo sistema riceve le richieste ma non risponde — confronto a costo controllato. Differenza vs deploy classico: la "regressione" è qualitativa, non binaria, e si misura su una distribuzione di output, non su un assert.
