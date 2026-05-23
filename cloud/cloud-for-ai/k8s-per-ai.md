---
title: Kubernetes per workload AI
sidebar_position: 5
---

# Kubernetes per workload AI

<div class="lesson-meta">
  <span class="badge-stato evoluzione">In evoluzione</span>
  <span>Lezione 5.5</span>
  <span>~12 min di lettura</span>
</div>

<p class="lesson-lead">Kubernetes non è nato per l'AI, ma è diventato lo standard de facto per orchestrare workload GPU in produzione. Il motivo è uno solo: nessun altro sistema gestisce bene la complessità di schedulare GPU eterogenee, batch job e inferenza real-time sullo stesso cluster.</p>

Probabilmente hai già sentito parlare di K8s nel contesto delle applicazioni web. Per l'AI le sfide sono diverse: le GPU costano 10-100× le CPU, il cold start è di minuti non secondi, e un job di training può monopolizzare risorse per ore. Questo cambiaanche il modo in cui si progetta lo scheduling.

## Perché K8s invece di ECS/Fargate per l'AI

ECS e Fargate funzionano bene per container CPU-based. Per i workload AI mancano pezzi fondamentali:

| Funzionalità | ECS/Fargate | Kubernetes |
|---|---|---|
| GPU scheduling con risorse frazionabili | Limitato | Sì (MIG, time-slicing) |
| Batch job con dipendenze tra step | No nativo | Sì (Volcano, Argo Workflows) |
| Node autoscaling GPU-aware | No | Sì (Karpenter) |
| Priority scheduling (inferenza > training) | No | Sì (PriorityClass) |
| Multi-GPU su pod singolo (NVLink) | No | Sì |
| Workflow ML complessi (pipeline) | No | Sì (KubeFlow Pipelines) |

Non significa che devi sempre usare K8s. Se il tuo workload AI è solo inferenza su Bedrock (API managed), ECS è più semplice e sufficiente. K8s diventa necessario quando hai GPU self-hosted, job di training, o pipeline ML complesse.

## Il NVIDIA GPU Operator

Il prerequisito per usare GPU su K8s è il **NVIDIA GPU Operator** — un Helm chart che installa automaticamente i driver NVIDIA, il device plugin (che espone le GPU come risorsa K8s), e il runtime container per abilitare `nvidia.com/gpu` nei pod.

Su EKS (Elastic Kubernetes Service — il Kubernetes managed di AWS) con nodi GPU:

```yaml
# Richiesta di una GPU intera per un pod di inferenza
resources:
  requests:
    nvidia.com/gpu: 1
  limits:
    nvidia.com/gpu: 1
```

Il scheduler di K8s vede `nvidia.com/gpu: 1` come una risorsa e piazza il pod solo su nodi con almeno una GPU libera. Se nessun nodo ha GPU disponibili, il pod rimane in `Pending` finché non si libera spazio o Karpenter scala un nuovo nodo.

**GPU taint**: i nodi GPU di solito hanno un `taint` che impedisce ai pod normali di atterrarci (altrimenti un pod CPU occuperebbe la GPU senza usarla). Solo i pod con la `toleration` corrispondente vengono schedulati su nodi GPU:

```yaml
tolerations:
  - key: "nvidia.com/gpu"
    operator: "Exists"
    effect: "NoSchedule"
```

## Karpenter — autoscaling GPU-aware

**Karpenter** è il node autoscaler di AWS per EKS (ma funziona anche altrove). A differenza del vecchio Cluster Autoscaler, Karpenter crea il nodo *esatto* di cui il pod ha bisogno, non solo il prossimo nodo del node group.

Esempio: hai un pod che richiede `nvidia.com/gpu: 8` (8 GPU, tipico per un job di training con parallelismo). Karpenter può lanciare direttamente un `p4d.24xlarge` (8 A100) invece di scalare 8 nodi `g4dn.xlarge` — perché sa quale istanza corrisponde alla richiesta.

La configurazione chiave è il `NodePool` (o il vecchio `Provisioner`):

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: gpu-nodepool
spec:
  template:
    spec:
      requirements:
        - key: "karpenter.k8s.aws/instance-gpu-name"
          operator: In
          values: ["t4", "a10g", "a100"]
        - key: "karpenter.sh/capacity-type"
          operator: In
          values: ["spot", "on-demand"]  # spot per training, on-demand per inferenza
  disruption:
    consolidationPolicy: WhenEmpty  # non spostare pod GPU attivi
```

Il campo `consolidationPolicy: WhenEmpty` è critico per i nodi GPU: K8s non deve spostare un pod di inferenza in corso su un altro nodo (causerebbe downtime e sprecherebbe il warm-up del modello in VRAM).

<details>
<summary>GPU time-slicing e MIG — condividere una GPU tra più pod</summary>

Una GPU intera per ogni pod di inferenza è spesso uno spreco. Un modello leggero (Llama 3 8B in INT4 = ~4GB VRAM) lascia liberi 70GB su un A100 80GB.

Due tecniche per condividere una GPU:

**Time-slicing**: la GPU viene divisa temporalmente tra più pod. Ogni pod vede tutta la VRAM, ma l'esecuzione è interleaved. Semplice da configurare, ma nessun isolamento di memoria — un pod può consumare tutta la VRAM e fare OOM agli altri.

**MIG (Multi-Instance GPU)**: disponibile su A100 e H100. La GPU viene partizionata in istanze hardware isolate (es. 7× `1g.10gb` su A100). Ogni istanza ha VRAM e motori di calcolo dedicati — isolamento garantito. Ideale per inferenza di modelli piccoli su GPU grandi.

Configurazione MIG su K8s (con NVIDIA GPU Operator abilitato per MIG):
```yaml
resources:
  requests:
    nvidia.com/mig-1g.10gb: 1
```

Su EKS: le istanze `p4d.24xlarge` (A100) supportano MIG. Configura il profilo MIG prima di usarlo nel cluster.

</details>

## Volcano — batch scheduling per training

Il default scheduler di K8s è progettato per applicazioni web: piazza un pod alla volta, in ordine di arrivo. Per il training distribuito questo è un disastro: un job con 8 pod (8 GPU) potrebbe avere 6 pod schedulati e 2 in attesa — occupi 6 GPU per niente mentre aspetti le altre 2.

**Volcano** è uno scheduler batch per K8s. Il concetto chiave è il **gang scheduling** — tutti i pod di un job o vengono schedulati insieme, o nessuno viene schedulato. Zero spreco di GPU parziali.

Volcano aggiunge anche la coda con priorità: i job di inferenza real-time hanno priorità più alta dei job di training — se arriva una richiesta urgente, il training viene preempted (sospeso) per liberare GPU.

```yaml
apiVersion: batch.volcano.sh/v1alpha1
kind: Job
metadata:
  name: training-llama
spec:
  minAvailable: 8      # gang scheduling: o 8 GPU o nessuna
  schedulerName: volcano
  priorityClassName: training-low  # priorità bassa rispetto all'inferenza
  tasks:
    - replicas: 8
      name: worker
      template:
        spec:
          containers:
            - name: training
              image: nvcr.io/nvidia/pytorch:24.01-py3
              resources:
                limits:
                  nvidia.com/gpu: 1
```

## PriorityClass — chi va prima

K8s supporta **PriorityClass** per definire la gerarchia di importanza tra i pod. Quando le risorse scarseggiano, i pod a bassa priorità vengono evicted per fare spazio a quelli ad alta priorità.

```yaml
# Alta priorità — inferenza real-time
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: inference-critical
value: 1000000
globalDefault: false

# Bassa priorità — training asincrono
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: training-batch
value: 100000
```

Con questo schema, se arriva un nuovo pod di inferenza e tutte le GPU sono occupate da training, K8s può preemptare un pod di training per fare spazio. Il training ripartirà da checkpoint.

## KubeFlow — MLOps su K8s

**KubeFlow** è la piattaforma MLOps per Kubernetes. Aggiunge tre componenti rilevanti per un AI Engineer:

- **KubeFlow Pipelines**: DAG — Directed Acyclic Graph — di step ML (preprocessing → training → evaluation → deploy). Ogni step è un container. Le pipeline sono versionabili e riproducibili.
- **KubeFlow Training Operator**: gestisce i job distribuiti per i principali framework (PyTorch, TensorFlow, XGBoost) con risorse GPU.
- **KubeFlow Serving (KServe)**: deployment di modelli con autoscaling, A/B testing e canary release.

KubeFlow ha una curva di installazione ripida. Per iniziare, considera **SageMaker Pipelines** (più gestito, meno flessibile) o **Argo Workflows** (più leggero di KubeFlow Pipelines) prima di investire in KubeFlow completo.

## Quando K8s vale il costo

K8s aggiunge complessità operativa reale — non è la scelta predefinita per ogni progetto AI.

**Vale il costo quando:**
- Hai GPU self-hosted (Bedrock/SageMaker non bastano per costo o privacy)
- Hai job di training periodici che competono con l'inferenza real-time
- Il team ha già K8s in produzione per le applicazioni web
- Vuoi pipeline ML riproducibili e versionabili

**Non vale il costo quando:**
- Usi solo API managed (Bedrock, OpenAI) — niente GPU da gestire
- Workload stateless senza GPU — ECS/Fargate è più semplice
- Team piccolo senza esperienza K8s — il costo operativo supera il beneficio

## Cosa non è

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "K8s gestisce le GPU come le CPU" | Le GPU hanno caratteristiche diverse: non sono frazionabili per default (a meno di MIG/time-slicing), richiedono driver e runtime speciali, e hanno cold start di minuti. Il scheduler di base di K8s non è sufficiente — servono GPU Operator, Karpenter e Volcano. |
| "Karpenter risolve il cold start delle GPU" | Karpenter scala i nodi in 1-2 minuti. Ma dopo che il nodo è pronto, il container deve ancora avviarsi e il modello deve essere caricato in VRAM (altri 2-5 minuti per modelli grandi). Il cold start totale rimane di 3-7 minuti. |
| "Volcano è necessario per qualsiasi job GPU" | Volcano serve per il training distribuito multi-GPU. Per l'inferenza con un pod singolo per GPU, il default scheduler funziona benissimo. Non aggiungere complessità dove non serve. |

## Verifica di comprensione

1. Cos'è un GPU taint e perché si usa?
2. Qual è la differenza tra Cluster Autoscaler e Karpenter per i nodi GPU?
3. Cos'è il gang scheduling di Volcano e quale problema risolve?
4. Quando useresti MIG invece di time-slicing per condividere una GPU?
5. Hai inferenza real-time e training batch sullo stesso cluster. Come configuri le priorità per evitare che il training blocchi l'inferenza?
6. Un job di training con 8 pod rimane in `Pending` da 30 minuti anche se ci sono 7 nodi GPU liberi. Qual è la causa probabile?
7. *(anticipazione)* Hai GPU, caching, K8s. Ora vuoi mettere insieme tutti i pezzi — retrieval, modello, guardrail, osservabilità — in un'architettura coerente. Da dove inizieresti a disegnare il sistema?

## Glossario della lezione

- **GPU Operator**: Helm chart NVIDIA che installa driver e device plugin su K8s per esporre le GPU come risorsa schedulabile.
- **Taint/Toleration**: meccanismo K8s per impedire ai pod generici di atterrare su nodi specializzati (GPU) — solo i pod con la toleration corretta vengono schedulati.
- **Karpenter**: node autoscaler AWS-native per EKS — lancia il tipo di istanza esatto richiesto dal pod, incluse le istanze GPU.
- **Gang scheduling**: tecnica Volcano che schedula tutti i pod di un job insieme o nessuno — previene l'occupazione parziale delle GPU.
- **MIG (Multi-Instance GPU)**: partizionamento hardware di una GPU (A100/H100) in istanze isolate con VRAM e calcolo dedicati.
- **PriorityClass**: risorsa K8s che definisce la priorità di un pod — in caso di scarsità, i pod a bassa priorità vengono preemptati.
- **KubeFlow**: piattaforma MLOps per K8s con pipeline, training operator e serving.
- **KServe**: componente KubeFlow per il serving di modelli con autoscaling e canary.

## Per approfondire

- **NVIDIA GPU Operator**: cerca "NVIDIA GPU Operator" su `docs.nvidia.com` — guida di installazione su EKS.
- **Karpenter**: cerca "Karpenter getting started" su `karpenter.sh` — documentazione ufficiale con esempi NodePool.
- **Volcano**: cerca "Volcano batch system" su `volcano.sh` — guide per gang scheduling e job prioritization.

## Prossima lezione

Hai i mattoni: GPU, caching, orchestrazione. Ora li metti insieme. La prossima lezione disegna l'architettura completa di un sistema AI in produzione — dall'API Gateway al modello, passando per il LLM gateway, il retrieval, i guardrail e l'osservabilità.
