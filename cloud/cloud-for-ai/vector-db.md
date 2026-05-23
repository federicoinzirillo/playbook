---
title: Vector database managed vs self-hosted
sidebar_position: 7
---

# Vector database managed vs self-hosted

<div class="lesson-meta">
  <span class="badge-stato evoluzione">In evoluzione</span>
  <span>Lezione 5.7</span>
  <span>~11 min di lettura</span>
</div>

<p class="lesson-lead">Non esiste il "miglior vector database". Esiste quello giusto per il tuo caso — corpus size, query al secondo, budget, infrastruttura già presente. Questa lezione ti dà la mappa per scegliere senza dover provare tutti.</p>

Nella lezione 5.6 hai visto il vector store come componente del layer di retrieval. Ora entriamo nel dettaglio: cosa sono queste opzioni, cosa le differenzia davvero, e come scegliere.

## Cosa fa un vector database

Un vector database conserva **embedding** — vettori numerici ad alta dimensionalità (tipicamente 768-3072 dimensioni) che rappresentano documenti, chunk di testo, immagini — e permette di fare ricerche per similarità: "dammi i K vettori più vicini a questo".

La ricerca per similarità è alla base del RAG: embeddisci la query dell'utente, cerchi i chunk più simili nel corpus, li passi al modello come contesto.

Internamente, ogni vector database usa un **indice ANN** — Approximate Nearest Neighbor — per rendere la ricerca veloce su corpus grandi. Gli algoritmi principali sono **HNSW** (Hierarchical Navigable Small World — un grafo a livelli che permette di saltare rapidamente a zone dello spazio vettoriale) e **IVF** (Inverted File Index — divide lo spazio in cluster, cerca solo nel cluster più vicino). HNSW è più veloce in query, IVF è più efficiente in memoria.

## Le opzioni concrete

### pgvector — il più semplice se hai già PostgreSQL

**pgvector** è un'estensione di PostgreSQL che aggiunge il tipo di dato `vector` e gli operatori di similarità (cosine distance, L2, inner product).

```sql
-- Crea tabella con embedding
CREATE TABLE documents (
  id     BIGSERIAL PRIMARY KEY,
  content TEXT,
  embedding vector(1536)  -- dimensioni del modello di embedding
);

-- Crea indice HNSW
CREATE INDEX ON documents
  USING hnsw (embedding vector_cosine_ops);

-- Ricerca top-5 più simili
SELECT id, content,
       1 - (embedding <=> $1) AS similarity
FROM documents
ORDER BY embedding <=> $1
LIMIT 5;
```

**Quando sceglierlo**: corpus &lt; 5M vettori, team già su PostgreSQL (RDS o Aurora), vuoi zero infrastruttura aggiuntiva, budget limitato. Per un RAG aziendale tipico (100K-500K documenti), pgvector su RDS `db.r7g.large` (~$200/mese) basta abbondantemente.

**Limiti**: non scala orizzontalmente (singolo nodo), non ha full-text search nativo integrato con la ricerca vettoriale (ci vuole una query ibrida manuale), indice HNSW deve stare in RAM.

### OpenSearch con k-NN — il managed AWS per casi medi

**OpenSearch** è il fork open-source di Elasticsearch gestito da AWS. Con il plugin k-NN aggiunge ricerca vettoriale nativa — e la combina con la ricerca full-text di OpenSearch in una sola query (ricerca ibrida).

La ricerca ibrida è il punto di forza: puoi cercare "documenti semanticamente simili alla query" E "documenti che contengono queste parole chiave" e combinare i punteggi. Per un RAG su testo tecnico (documentazione, knowledge base) la ricerca ibrida batte la sola ricerca vettoriale di 10-20% in precision.

```json
{
  "query": {
    "hybrid": {
      "queries": [
        { "match": { "content": "come resetto la password" } },
        {
          "knn": {
            "embedding": {
              "vector": [0.12, -0.34, ...],
              "k": 20
            }
          }
        }
      ]
    }
  }
}
```

**Quando sceglierlo**: corpus medio-grande (500K-50M documenti), vuoi ricerca ibrida (keyword + semantica), team già su OpenSearch o Elasticsearch, budget per un cluster managed (~$100-400/mese per un cluster `r6g.large.search`).

**Limiti**: più pesante di pgvector da gestire (cluster, shard, replica), costo non trascurabile per cluster piccoli.

### Pinecone — zero ops, massimo costo

**Pinecone** è un vector database SaaS — niente server da gestire, API REST semplice, autoscaling automatico. Carica i tuoi vettori, fai query, fine.

Il problema è il costo. Il piano Starter è gratuito fino a 100K vettori, poi il piano Standard parte da $70/mese per storage + $0.10 per 1M query. Per corpus grandi o volume elevato, i costi crescono rapidamente e diventano difficili da prevedere.

**Quando sceglierlo**: prototipo o startup in fase early (zero ops overhead conta), team senza expertise di infrastruttura, corpus &lt; 1M vettori con volume di query moderato.

**Quando non**: volume alto (cost unpredictable), dati sensibili che non possono uscire dalla tua VPC, budget ristretto.

### Weaviate — ricco di feature, self-hosted

**Weaviate** è un vector database open-source con molte feature: multi-tenancy nativa, hybrid search, moduli di embedding integrati (chiama automaticamente il modello di embedding per te), GraphQL API.

Può girare self-hosted su Kubernetes (Helm chart ufficiale) o come cloud managed (Weaviate Cloud). Per il self-hosted su EKS, un deployment production-ready richiede ~3 pod + storage persistente.

**Quando sceglierlo**: vuoi le feature avanzate (multi-tenancy, moduli integrati), hai un team con expertise K8s, corpus grandi con query complesse. Meno comune in ambito AWS rispetto a pgvector + OpenSearch.

## Decision table

| Scenario | Scelta consigliata | Motivo |
|---|---|---|
| Hai già PostgreSQL, corpus &lt;1M documenti | **pgvector** | Zero infrastruttura aggiuntiva, costo minimo |
| Corpus medio, ricerca ibrida keyword+semantic | **OpenSearch k-NN** | Hybrid search nativo, managed AWS |
| Prototipo rapido, niente ops | **Pinecone Starter** | Zero setup, API semplice |
| Corpus &gt;50M vettori, high QPS, SLA strict | **Pinecone / Weaviate Cloud** o **OpenSearch large** | Scala orizzontalmente, managed |
| Dati in VPC, self-hosted K8s | **pgvector** o **Weaviate** | No uscita dati, controllo pieno |
| Multi-tenant con isolamento per cliente | **Weaviate** | Multi-tenancy nativa |

<details>
<summary>Embedding model e dimensionalità — impatto sulla scelta del vector store</summary>

La dimensionalità degli embedding influenza i requisiti del vector store:

- **text-embedding-3-small** (OpenAI): 1536 dimensioni — il punto di equilibrio comune.
- **text-embedding-3-large** (OpenAI): 3072 dimensioni — qualità superiore, indici 2× più grandi.
- **Titan Embeddings v2** (Amazon Bedrock): 1024 o 256 dimensioni — più leggero, buona qualità per inglese.
- **Cohere embed-multilingual-v3**: 1024 dimensioni — ottimo per testi in più lingue.

Con pgvector: ogni vettore da 1536 float32 occupa 6KB. 1M documenti = ~6GB di soli vettori in RAM (l'indice HNSW deve stare in memoria per performance ottimali). Pianifica l'istanza RDS di conseguenza.

Con OpenSearch: usa il tipo `knn_vector` con `dimension` corrispondente al modello. Il plugin k-NN supporta HNSW e Faiss (IVF).

</details>

## Cosa non è

| Il pensiero sbagliato | Come stanno le cose |
|---|---|
| "Pinecone è il migliore perché è dedicato ai vettori" | Pinecone è il più semplice da avviare. Per la maggior parte dei team AWS, pgvector o OpenSearch sono più economici, integrati nell'infrastruttura esistente, e sufficientemente veloci. "Dedicato" non significa "superiore" per ogni caso d'uso. |
| "pgvector non scala per corpus grandi" | pgvector su un'istanza RDS `r7g.4xlarge` (128GB RAM) gestisce ~20M vettori da 1536 dimensioni con latenza p95 &lt;10ms. Non scala orizzontalmente, ma verticalmente copre la maggior parte dei casi reali. |
| "La ricerca vettoriale restituisce i risultati migliori" | La ricerca vettoriale cattura la similarità semantica ma manca i match esatti di keyword (es. numeri di versione, nomi propri, codici prodotto). La ricerca ibrida (vettoriale + full-text) batte la sola ricerca vettoriale nella maggior parte dei casi RAG su testo misto. |

## Verifica di comprensione

1. Cos'è HNSW e perché viene preferito a una ricerca esatta per corpus grandi?
2. Quando la ricerca ibrida supera la sola ricerca vettoriale?
3. Hai un RAG su 200K documenti aziendali, team già su RDS PostgreSQL, nessun requisito di ricerca full-text avanzata. Quale vector store sceglieresti e perché?
4. Qual è il limite principale di pgvector rispetto a una soluzione managed dedicata come Pinecone?
5. Perché i vettori di dimensione alta (3072 vs 1536) aumentano il costo dell'infrastruttura?
6. Un team vuole multi-tenancy — ogni cliente deve vedere solo i propri documenti. Quale vector database supporta questo nativamente?
7. *(anticipazione)* Hai il vector store pronto. Devi ora ingestire e aggiornare periodicamente 1M documenti — chunking, embedding, ingestion. Come organizzeresti questa pipeline su AWS?

## Glossario della lezione

- **Embedding**: vettore numerico ad alta dimensionalità che rappresenta il significato semantico di un testo.
- **ANN (Approximate Nearest Neighbor)**: algoritmo di ricerca vettoriale che trova i vicini più probabili in modo efficiente, accettando una piccola imprecisione per guadagnare velocità.
- **HNSW (Hierarchical Navigable Small World)**: algoritmo ANN a grafo — il più usato per la sua combinazione di velocità in query e qualità di recall.
- **IVF (Inverted File Index)**: algoritmo ANN che divide lo spazio vettoriale in cluster e cerca solo nel cluster più vicino — più efficiente in memoria di HNSW.
- **Ricerca ibrida**: combinazione di ricerca vettoriale (semantica) e full-text (keyword) nella stessa query — migliora la precision rispetto all'uso di uno solo.
- **pgvector**: estensione PostgreSQL per la ricerca vettoriale — aggiunge tipo `vector`, operatori di distanza e indice HNSW/IVF.
- **Multi-tenancy**: capacità di isolare i dati di clienti diversi all'interno della stessa istanza del database.

## Per approfondire

- **pgvector**: cerca "pgvector" su `github.com/pgvector/pgvector` — README completo con esempi di index HNSW e IVF.
- **OpenSearch k-NN**: cerca "k-NN search OpenSearch" su `opensearch.org/docs` — guida agli algoritmi e alla ricerca ibrida.
- **Weaviate**: cerca "Weaviate documentation" su `weaviate.io/developers/weaviate` — guida a multi-tenancy e hybrid search.

## Prossima lezione

Il vector store è pronto. Mancano i dati — il corpus di documenti deve essere ingestito, suddiviso in chunk, embeddato e caricato nel vettore store. La prossima lezione coprirà le training pipeline e le pipeline MLOps managed su SageMaker.
