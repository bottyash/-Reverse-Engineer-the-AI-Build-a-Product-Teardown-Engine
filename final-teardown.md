# Final Teardown — Sarvatra (RAG Search Engine)
> Produced by: V2 System Prompt × ChatGPT 5.2 (Standard mode)
> Best-parts merged from multi-LLM comparison. This is the highest-quality single output.

---

## Product: Sarvatra
**Stack summary:** Full RAG pipeline — live web search → HTML extraction → chunking → hybrid retrieval → cross-encoder reranking → local qwen3:4b generation via Ollama → cited answer

---

### Layer 1 — Data Foundation

**What's happening:** Web search agents call live search APIs (Bing/Google/CustomSearch/Tavily) and Playwright-driven headless crawlers fetch raw HTML from JS-heavy pages. The raw payload is passed through boilerplate removal (Readability.js or Boilerpipe) → language detection → overlapping chunk splitting (512–1024 tokens, 10–20% overlap) with metadata attached (source URL, title, timestamp, xpath, BM25 pre-score). Pipelines write to both an Elasticsearch BM25 sparse index and a FAISS HNSW dense vector index simultaneously, with raw archives stored in S3 and chunk metadata in PostgreSQL.

**Key technologies likely used:** Playwright (JS-heavy crawling), Readability/Boilerpipe (boilerplate removal), LangChain RecursiveCharacterTextSplitter or custom tokenizer-aware chunker, HuggingFace sentence-transformers for embedding generation, FAISS HNSW for ANN vector index, Elasticsearch for BM25 sparse index, PostgreSQL for chunk metadata + provenance, Airflow or Prefect for ETL orchestration.

**The engineering challenge:** Guaranteeing consistent tokenization between the embedding stage (where chunks are indexed) and the generation stage (where they're assembled into the prompt). If the embedding model tokenizes "Anthropic's Claude" differently from the generation model's tokenizer, chunk boundaries silently misalign — leading to truncated context and broken citations without any visible error.

**Skill required:** Web crawling at scale (handling paywalls, CAPTCHAs, dynamic JS rendering), tokenizer-aware chunking strategy, embedding pipeline design, data provenance for traceable citations, ETL orchestration, GDPR/copyright awareness for scraped content.

**Honesty check:** HIGHLY USED. This is the most foundational layer — without robust, provenance-aware ingestion, every downstream layer produces hallucinated or untraceable answers. The citation quality that defines Sarvatra's product value lives or dies at Layer 1.

---

### Layer 2 — Statistics & Analysis

**What's happening:** System telemetry tracks retrieval quality (nDCG@10, recall@k, MRR), generation quality (citation accuracy, hallucination rate via human evals), and system health (p95 latency per stage, GPU utilization, throughput). A/B tests compare chunking strategies, embedding models, reranker architectures, and prompt templates; statistical significance is validated with bootstrap confidence intervals and paired t-tests before any production change ships. User feedback signals (thumbs up/down) feed an offline evaluation pipeline that re-scores historical queries against the new configuration before rollout.

**Key technologies likely used:** Prometheus + Grafana for real-time telemetry, Weights & Biases or MLflow for experiment tracking, LangSmith or Phoenix for RAG trace evaluation, Python scipy/numpy/pandas for offline statistical analysis, PostHog for product analytics, custom evaluation scripts for citation accuracy.

**The engineering challenge:** Designing offline metrics that actually predict online user satisfaction. nDCG can increase when the reranker surfaces more keyword-relevant passages — but if those passages are factually outdated or contradict each other, generation quality drops. Sarvatra must invest in human evaluation pipelines alongside automated metrics, or risk optimizing the wrong thing.

**Skill required:** Experimental design for ML systems, applied statistics (bootstrap CI, power analysis, significance testing), RAG evaluation methodology (not just retrieval metrics — generation faithfulness and citation grounding), A/B test instrumentation.

**Honesty check:** MODERATELY USED initially, HIGHLY USED in production. Early-stage Sarvatra may rely on manual vibe-checks. But without rigorous measurement, iteration stalls — you can't tell if V2 retrieval is actually better than V1.

---

### Layer 3 — Machine Learning Models

**What's happening:** Two-tier retrieval: (1) fast hybrid retrieval combining dense bi-encoder ANN search (FAISS HNSW, top-200 candidates) with sparse BM25 recall (Elasticsearch, top-200 candidates), merged and de-duplicated; (2) MMR (Maximal Marginal Relevance) applied to the union to enforce topical diversity before expensive re-ranking; (3) a cross-encoder re-ranker (fine-tuned on relevance labels) re-scores the top-M (M=20) candidates using full query–passage interaction, producing the final ordered list from which the top-8 passages are assembled for generation. Online feedback (clicks, thumbs up/down) feeds a learning-to-rank pipeline that fine-tunes the cross-encoder and recalibrates BM25/dense fusion weights.

**Key technologies likely used:** Sentence-transformers (bi-encoder, e.g., BAAI/bge-large-en-v1.5 via HuggingFace), Elasticsearch (BM25 sparse), FAISS HNSW (dense ANN), cross-encoder reranker (fine-tuned qwen3 variant or ms-marco-MiniLM), LightGBM or RankLib for learning-to-rank, MMR implementation (LangChain or custom), feature store for ranking signals (recency, domain trust score, embedding similarity, BM25 score).

**The engineering challenge (context collapse via pure cosine similarity):** If the system retrieves candidates by dense similarity alone, the top-8 passages often come from the same source — the same Wikipedia article phrased five ways — which gives the generator a false sense of corroboration for a single viewpoint. The cross-encoder alone doesn't fix this; MMR must be applied *before* re-ranking to ensure the generator sees diverse perspectives. Most teams skip MMR and discover this failure mode only after users complain about confident but one-sided answers.

**Skill required:** Two-tower retrieval architecture, approximate nearest-neighbor search (ScaNN vs. FAISS trade-offs at scale), cross-encoder design and fine-tuning, learning-to-rank (LTR) with LightGBM, MMR diversity implementation, hybrid fusion strategies (RRF vs. weighted score merging).

**Honesty check:** MOST CRITICAL LAYER. Sarvatra's answer quality is a direct function of what the generator receives. The LLM cannot rescue bad retrieval — it can only faithfully synthesize whatever context it's given. Ranking is the answer quality multiplier.

---

### Layer 4 — LLM / Generative AI

**What's happening:** The RAG orchestrator assembles a structured prompt: system instructions + condensed session memory (retrieved from the user's memory namespace by semantic + recency similarity) + top-8 re-ranked passages with provenance tags (source URL, timestamp, chunk ID) + current query. The assembled prompt is sent to qwen3:4b running locally via Ollama, which generates a streamed answer with inline citations forced by the prompt template. A lightweight verifier step cross-checks each factual claim against the retrieved chunks — claims without a grounding chunk are flagged for removal or hedging before the answer is shown to the user.

**Key technologies likely used:** qwen3:4b via Ollama (local inference), sentence-transformers (memory retrieval embeddings), FAISS or Chroma (user memory vector namespace), LangChain or custom RAG orchestration, ConversationBufferWindowMemory or custom condensation pipeline, a small cross-encoder or LLM-as-judge verifier for citation fidelity, Redis for caching recent prompt/response pairs.

**Memory — application-layer illusion:** Sarvatra has no persistent model memory. Instead, after each session, a condensation model (a smaller LLM) ingests the session transcript and writes a short summary + embedding into the user's memory namespace in the vector DB, tagged with recency and an importance score. At query time, memory vectors are retrieved by semantic similarity + recency weighting and fused into the prompt as "user context / past answers." TTLs and importance decay keep the memory namespace compact and relevant — not an ever-growing pile of old sessions.

**The engineering challenge:** Prompt budget management for qwen3:4b's limited context window. Assembling 8 passages + condensed memory + system instructions + conversation history can exceed the context limit — requiring intelligent context selection (prioritize by recency of source, relevance score, and citation coverage) with graceful truncation that preserves provenance metadata even when trimming passage text.

**Skill required:** Prompt engineering for instruction-following with local models, RAG orchestration (retrieval-generation interplay), memory condensation architecture, LLM ops (quantization, Ollama serving config, context window management), citation grounding and hallucination mitigation.

**Honesty check:** HIGHLY USED — this is the synthesis brain of Sarvatra. But it is entirely dependent on Layers 1–3. A well-tuned qwen3:4b with excellent context will outperform a poorly-retrieved GPT-4 prompt every time.

---

### Layer 5 — Deployment & Infrastructure

**What's happening:** The qwen3:4b model runs as a persistent Ollama daemon on a GPU-equipped node (quantized to 4-bit or 8-bit with GGUF format to fit within VRAM budget), with Celery workers handling asynchronous web scraping and embedding jobs in parallel. Synchronous FastAPI endpoints serve the query → retrieval → generation pipeline with Redis caching for recent queries and embedding vectors. A separate embedding service (sentence-transformers served via FastAPI or Triton) runs independently so embedding updates don't require restarting the generation service — decoupled deployments prevent the rolling update consistency problem.

**Key technologies likely used:** Ollama (local qwen3:4b serving with GGUF quantization), Docker + Docker Compose (service isolation), FastAPI (query + embedding API endpoints), Celery + Redis (async crawling/embedding job queue), Triton Inference Server (optional, for batched embedding serving), Prometheus + Grafana (inference latency + GPU utilization monitoring), CUDA drivers + nvidia-container-toolkit (GPU access in Docker).

**The engineering challenge:** Rolling update consistency — when a new embedding model version is deployed, all existing FAISS index vectors become misaligned with new query embeddings until the index is fully re-embedded. During the transition window, retrieval silently degrades. Sarvatra must implement a blue/green embedding deployment strategy: build the new index in parallel, validate recall@k parity, then atomically switch traffic.

**Skill required:** MLOps for local LLM inference (quantization strategy, VRAM budgeting, Ollama config), container orchestration (Docker Compose → Kubernetes upgrade path), GPU resource management, async job queue design (Celery), inference latency optimization (batching, caching).

**Honesty check:** HIGHLY USED — because the model is local rather than API-hosted, every infra decision is the team's responsibility. VRAM management, thermal throttling, and update consistency are real daily operational concerns, not theoretical.

---

### Layer 6 — System Design & Scale

**What's happening:** For Sarvatra's current local-first deployment, a single-node setup (Ollama + FAISS + Elasticsearch + Redis on one beefy machine or small cluster) is sufficient. If Sarvatra moves to a public-facing B2C product, the architecture shards: stateless FastAPI query frontends behind a load balancer (nginx/Kong), multi-tenant FAISS shards partitioned by user namespace, replicated qwen3:4b inference pools with quantized model weights cached in VRAM, and Kafka for eventing (crawl jobs, embedding jobs, feedback ingestion). Progressive degradation serves cached answers or routes to a smaller/faster model under heavy load to maintain availability SLAs.

**Key technologies likely used:** Kong or nginx (API gateway + rate limiting), Kubernetes + autoscaling (inference pool scaling), Milvus or sharded FAISS (distributed vector search), Kafka (event streaming for async pipelines), Redis Cluster (distributed session/memory cache), CDN (cached popular query results), RBAC + tenant isolation (multi-tenant user memory namespaces).

**The engineering challenge (multi-tenant GPU starvation):** A single noisy tenant running 100 concurrent searches can saturate the GPU inference pool, starving other users. At scale, Sarvatra needs per-tenant request quotas enforced at the API gateway, a queue-based inference scheduler that batches requests across tenants, and a fallback CPU-inference path for burst traffic — without these, SLA violations cascade.

**Skill required:** Distributed systems design (sharding strategies, CAP trade-offs for vector DBs), capacity planning for GPU inference pools, API gateway configuration (rate limiting, quota enforcement), message queue architecture (Kafka consumer groups), multi-tenant isolation (namespace partitioning, RBAC).

**Honesty check:** THIS LAYER IS THIN for current Sarvatra. If deployed as a local productivity tool or small team product, full sharding and Kubernetes are over-engineering. A single Docker Compose stack with Redis + FAISS is sufficient and operationally simpler. Layer 6 only becomes load-bearing when monthly active users exceed ~10,000 concurrent sessions.

---

## Overall Analysis

**Most critical layer:** Layer 3 — Ranking. Sarvatra runs on qwen3:4b, a 4-billion parameter model that cannot independently synthesize conflicting evidence, infer unstated context, or correct bad retrieval. Every answer quality metric (citation accuracy, factual correctness, answer diversity) is a direct multiplier of retrieval quality. A broken ranking layer — producing top-8 passages from one source, or missing the most recent web evidence — results in confidently wrong answers with valid-looking citations. No prompt engineering can fix bad context.

**Complexity rating:** Advanced. Full RAG pipeline with hybrid retrieval, cross-encoder re-ranking, application-layer memory, local inference, and citation verification is non-trivial to build and maintain — but uses established tooling rather than novel research.

**If rebuilding from scratch:** "The first thing you'd need to get right is the live search → chunking → hybrid retrieval pipeline, because without provenance-tagged, diversity-enforced chunks flowing into the generator, everything downstream produces confidently wrong, untraceable answers — and no amount of prompt engineering on qwen3:4b will compensate for a broken retrieval foundation."

---

## Implementation Checklist

1. **Ingestion:** Playwright crawler + Readability.js → HTML → language detect (langdetect) → chunk (512–1024 tokens, 20% overlap) → attach metadata (URL, title, timestamp, chunk ID) → store raw archive in S3 + chunk records in PostgreSQL.
2. **Embeddings & Sparse Index:** Embed chunks with `BAAI/bge-large-en-v1.5` (HuggingFace sentence-transformers) → store vectors in FAISS HNSW index with PostgreSQL chunk ID pointers. Build parallel Elasticsearch BM25 index on same chunks.
3. **Retrieval pipeline:** Query → BM25 top-200 (Elasticsearch) + dense ANN top-200 (FAISS) → union → de-dup → MMR to select diverse candidate set (top-50) → cross-encoder re-rank top-20 → select top-8 for generation.
4. **Cross-encoder reranker:** Fine-tune a cross-encoder (`ms-marco-MiniLM-L-6-v2` or qwen3 variant) on human-labeled query–passage relevance pairs. Use features: embedding similarity, BM25 score, recency (days since publication), domain trust score.
5. **Generation:** Assemble: `[system instructions] + [condensed memory] + [top-8 passages with provenance tags] + [user query]` → stream to qwen3:4b via Ollama. Use instruction-following system prompt that forces inline citation format: `[Claim] [Source: URL]`.
6. **Memory condensation:** After session end, run condensation model (smaller LLM) on session transcript → store summary embedding in user memory namespace (FAISS/Chroma) with TTL=30 days + importance score. Retrieve top-3 memory vectors at query time by cosine similarity + recency weighting.
7. **Verifier:** After generation, run a lightweight LLM-as-judge pass that checks each cited claim against the source chunk. Flag claims with no grounding chunk → replace with hedged language ("Based on available sources...") before rendering.
8. **Infra:** Docker Compose (Ollama + FastAPI + Redis + Elasticsearch + FAISS service), Celery + Redis for async crawling/embedding jobs, Prometheus + Grafana for latency + GPU utilization dashboards.

---

## Final Notes (Honest, Blunt)

- **Don't let the generator be the source of truth.** Force every factual claim to trace back to a retrieved chunk with a source URL and timestamp. If the claim can't be grounded, hedge it or remove it.
- **Ranking-first mindset.** Retrieval quality is the answer quality multiplier. Every hour spent improving the cross-encoder reranker returns more than an hour spent prompt-engineering qwen3:4b.
- **Memory is an illusion at the application layer.** Condensed vectors + retrieval beat context window expansion every time — more reliable, cheaper, and auditable.
