---
title: Serving e inference
sidebar_label: "6.1 Serving e inference"
sidebar_position: 1
---

# Serving e inference

<div class="lesson-meta">
  <span class="badge-stato evoluzione">In evoluzione</span>
  <span>Lezione 6.1</span>
  <span>~12 min di lettura</span>
</div>

<p class="lesson-lead">In Parte 5 hai scelto i pattern. Qui parliamo degli strumenti reali con cui un endpoint LLM si espone, si scala e non muore al primo picco: gateway, motori di inferenza, batching, autoscaling. È la zona "giorno-2" — dove un sistema che funziona in demo diventa un servizio che funziona alle 3 di notte.</p>

C'è una differenza enorme tra "il modello risponde" e "il modello risponde a 500 utenti contemporaneamente con latenza prevedibile". La prima è una chiamata HTTP; la seconda richiede pensare a throughput, batching, coda, isolamento dei tenant. Se usi un provider esterno (OpenAI, Anthropic), buona parte di questo problema è sua. Se hai un modello open-weight servito in proprio, è tuo.

## L'asse della scelta: API esterna o serving in proprio

La prima domanda di serving non è "quale GPU" ma **"chi serve il modello?"**.

**API esterna** — OpenAI, Anthropic, Google, Mistral via API. Paghi a token, scali "infinito" senza pensieri, niente GPU da gestire. Trade-off: dati che escono dall'infrastruttura, latenza non controllabile, dipendenza dal provider, costo lineare con il volume.

**Serving in proprio (open-weight)** — Llama, Mistral, Qwen, DeepSeek serviti su GPU che gestisci tu (cloud o on-premise). Trade-off: dati restano dentro, latenza e capacità sotto controllo, costo fisso del hardware indipendente dal volume. Richiede competenze di ML serving e ops GPU.

**Cloud-managed open-weight** — AWS Bedrock, Azure AI Foundry, Together AI, Fireworks. Servono modelli open-weight su loro infrastruttura. Via di mezzo: niente GPU da gestire, prezzo a token, ma più scelta di modello rispetto alle API proprietarie e spesso garanzie di data isolation migliori.

La scelta non è religiosa. La maggior parte dei sistemi produttivi oggi è ibrida: API esterna per la maggior parte del traffico, serving in proprio (o cloud-managed open-weight) per casi specifici con vincoli di privacy, costo elevato a volume alto, o latenza critica.

## Il gateway: il punto unico di ingresso

Quando il sistema cresce, una chiamata diretta dal codice applicativo alle API del modello diventa ingestibile. Servono autenticazione, rate limiting, routing tra modelli, logging unificato, retry, fallback. Tutta questa logica vive nel **gateway LLM** — *AI gateway*.

Un gateway LLM è un proxy applicativo che sta davanti a uno o più provider e gestisce:

- **Routing** — instradare la richiesta al modello giusto in base a regole (header, tenant, complessità della query).
- **Fallback** — se il provider primario fallisce o è lento, dirotta su un secondario.
- **Rate limiting per tenant** — un singolo cliente non può esaurire la capacità (o il budget).
- **Caching** — cache esatta (stessa query → stessa risposta dalla cache) e semantica (query simile → risposta cached con soglia di similarità).
- **Logging e tracing unificati** — tutte le chiamate passano da qui, tutto il logging è centralizzato.
- **Trasformazione del payload** — normalizzare formati diversi (OpenAI vs Anthropic vs locale) in un'API unica per il codice applicativo.

<details>
<summary>Implementazioni di gateway LLM</summary>

Esistono soluzioni open-source e commerciali. **LiteLLM** è il riferimento open-source — un proxy che unifica decine di provider sotto un'API stile OpenAI. **Portkey** e **Helicone** sono offerte commerciali con UI di monitoring integrata. **Cloudflare AI Gateway** è una versione managed se sei già nel loro ecosistema. Per casi semplici puoi scriverne uno tu in poche centinaia di righe (FastAPI o simili); per casi complessi conviene partire da LiteLLM.

Lo schema è sempre lo stesso: il codice applicativo parla col gateway via un'API stabile; il gateway gestisce la complessità del mondo reale (provider che cambiano, modelli che vengono deprecati, prezzi che variano).
</details>

## I motori di inferenza: cosa fa girare il modello

Se servi un modello open-weight, non lo carichi con `transformers.pipeline()` e lo esponi via Flask. Quella è la versione "funziona sul mio portatile". In produzione servono motori di inferenza ottimizzati.

**vLLM** <span class="badge-stato stabile">Stabile</span> — il riferimento de facto per il serving di LLM open-weight. Implementa **continuous batching** — invece di aspettare di avere un batch pieno prima di processarlo, processa token in arrivo da richieste diverse nello stesso forward pass del modello. Risultato: throughput drasticamente più alto a parità di GPU. Implementa anche **PagedAttention**, una gestione della KV cache (memoria delle attenzioni) che evita la frammentazione e permette di servire più richieste in parallelo sulla stessa GPU.

**TGI — Text Generation Inference** <span class="badge-stato stabile">Stabile</span> — il motore di Hugging Face. Stesso concetto, alternativa pratica a vLLM. Spesso si sceglie l'uno o l'altro per integrazione con l'ecosistema (HuggingFace Inference Endpoints usa TGI).

**TensorRT-LLM** <span class="badge-stato stabile">Stabile</span> — di NVIDIA, ottimizzato per le loro GPU. Massimo throughput, ma più complesso da configurare. Si usa quando si vuole spremere ogni ultimo token al secondo dall'hardware.

**SGLang** <span class="badge-stato evoluzione">In evoluzione</span> — motore più recente con focus su prompt complessi e prefix caching aggressivo. Promettente, in adozione crescente.

<details>
<summary>Perché il continuous batching cambia tutto</summary>

Un LLM generativo produce un token alla volta in modo iterativo. Senza batching, ogni richiesta usa la GPU al ~10% del suo throughput teorico — le moltiplicazioni matriciali sono molto più larghe della singola riga di output. Con il batching tradizionale aspetti di avere N richieste pronte, poi le processi insieme: ottimo per training, pessimo per serving (le richieste arrivano scaglionate). Il continuous batching aggiunge richieste al batch *durante* la generazione: appena una richiesta finisce, ne entra un'altra. Il batch è quasi sempre pieno, la GPU lavora vicino al massimo. Su workload reali, il throughput cresce di 5-20x rispetto al naive serving.
</details>

## Il KV cache: il vincolo nascosto sulla capacità

Ogni richiesta attiva in generazione occupa memoria GPU non solo per i pesi del modello (condivisi tra tutte le richieste) ma anche per la **KV cache** — la memoria delle attenzioni accumulate token per token. La KV cache cresce linearmente con la lunghezza del contesto e moltiplicata per il numero di richieste concorrenti.

Tradotto: la capacità in richieste concorrenti di una GPU non dipende solo dai pesi del modello, ma da **contesto medio × richieste concorrenti × dimensione del modello**. Un Llama 70B su una A100 80GB può servire poche richieste con contesti lunghi, o molte richieste con contesti corti.

È il motivo per cui PagedAttention (gestione paginata della KV cache) e prefix caching (riuso della KV cache su prefissi condivisi del prompt) fanno una differenza enorme nel costo per richiesta.

## Autoscaling: il pattern non è quello del web tradizionale

Le applicazioni web tradizionali scalano in pochi secondi: parte un nuovo container, è in load balancer. Gli endpoint LLM no.

**Cold start lunghi.** Caricare un modello da 70B parametri in GPU richiede da decine di secondi a minuti. L'autoscaling reattivo (parto quando arriva traffico) non funziona — il primo utente del nuovo replica aspetta troppo.

**Strategie reali:**
- **Warm pool** — tenere sempre N replica caldi, anche idle. Costa, ma garantisce latenza.
- **Pre-scaling su pattern noti** — se sai che il traffico cresce alle 9, scali alle 8:45.
- **Spillover su API esterna** — quando i replica locali sono saturi, dirotti su un provider esterno (più costoso, ma evita il "no capacity"). Pattern reso pulito dal gateway.
- **Modelli più piccoli per assorbire i picchi** — sotto carico, si fa routing su un modello più piccolo invece che provare a scalare il grande.

Cosa NON funziona: l'HPA (Horizontal Pod Autoscaler) di Kubernetes basato su CPU. Le GPU non si misurano in CPU%. Servono metriche custom: token in coda, utilizzo della KV cache, latenza al P95.

## Streaming: è una scelta di serving, non solo di UX

Lo streaming dei token (Server-Sent Events tipicamente) non è solo "fa più bello". Ha implicazioni reali:

- **Latenza percepita** — il primo token in 500ms con generazione di altri 3 secondi è meglio di una risposta completa in 3 secondi senza nulla prima.
- **Time-to-first-token (TTFT)** — diventa la metrica primaria, non la latenza totale. Va monitorata separatamente.
- **Connessioni HTTP lunghe** — le SSE tengono aperta la connessione per tutta la durata della generazione. Cambia la configurazione dei timeout su tutti i layer (load balancer, gateway, server).
- **Cancellation** — l'utente che chiude la pagina deve interrompere la generazione lato server, altrimenti continui a spendere token su una richiesta che nessuno vedrà.

## Cosa NON è il serving LLM

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Lo metto dietro Flask e ho fatto" | Throughput 10x inferiore al motore ottimizzato. Su carico reale crolli. |
| "Scalo orizzontalmente come un'API normale" | Cold start lunghi e GPU costose rendono lo scaling reattivo impraticabile. |
| "Più GPU = più capacità" | La capacità è strozzata dalla KV cache, non solo dai pesi. Più contesto = meno richieste concorrenti. |
| "Il gateway è un dettaglio implementativo" | Senza gateway, ogni cambio di provider/modello richiede modifiche al codice applicativo. |

## Cosa dura, cosa evitare

Dura: il **gateway come pattern**, il **continuous batching**, la **separazione TTFT/latenza totale**.

Evita di affezionarti a motori di inferenza specifici come fossero permanenti. Lo spazio si muove velocemente — vLLM è oggi il riferimento, ma SGLang sta guadagnando terreno e TensorRT-LLM continua a evolversi. Il pattern (motore ottimizzato + gateway + autoscaling con warm pool) è stabile; l'implementazione specifica no.

---

## Verifica di comprensione

> Rispondi a memoria. Le incerte rivedile domani.

1. Cosa fa un gateway LLM e quali tre problemi risolve?
2. Cos'è il continuous batching e perché cambia il throughput rispetto al batching tradizionale?
3. Perché l'autoscaling reattivo standard non funziona bene per endpoint LLM?
4. Cos'è la KV cache e perché impatta la capacità in richieste concorrenti?
5. Quando ha senso servire un modello in proprio invece di usare un'API esterna?

---

## Glossario

- **Gateway LLM** — proxy applicativo che gestisce routing, fallback, rate limiting, caching e logging unificato per le chiamate a modelli.
- **Continuous batching** — tecnica di serving che aggiunge richieste a un batch in corso di generazione, mantenendo la GPU saturata.
- **KV cache** — memoria delle key e value dell'attention accumulate durante la generazione; cresce con la lunghezza del contesto.
- **PagedAttention** — gestione paginata della KV cache (vLLM) che riduce la frammentazione e aumenta la concorrenza.
- **vLLM / TGI / TensorRT-LLM / SGLang** — motori di inferenza ottimizzati per servire LLM open-weight in produzione.
- **TTFT — Time to First Token** — latenza dal momento della richiesta al primo token generato; metrica primaria nello streaming.
- **Warm pool** — replica del modello sempre caldi, anche idle, per evitare cold start su picchi di traffico.
- **Spillover** — quando i replica locali sono saturi, dirottamento del traffico su un provider esterno.

---

## Per approfondire

- **Documentazione di vLLM** (`docs.vllm.ai`) — leggi le sezioni su continuous batching e PagedAttention. Capire le metriche è metà del lavoro.
- **LiteLLM** su GitHub — il gateway open-source più adottato, esempi pratici di routing e fallback.
- **"Inference, fast and slow"** — cerca articoli che spiegano TTFT vs throughput vs latenza totale.

---

## Prossima lezione

**6.2 Costi e FinOps a runtime.** Hai il serving, scala, regge i picchi. Ora arriva la fattura. La 5.3 ti ha fatto ragionare sul triangolo in fase di design; la 6.2 è la gestione reale: monitoraggio della spesa, alert, cosa fai quando i costi raddoppiano in una settimana.
