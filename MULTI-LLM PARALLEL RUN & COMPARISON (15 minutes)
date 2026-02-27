# LLM Comparison — Product Teardown: Sarvatra (RAG Search Engine)

## Models Used

| # | LLM Name | Mode | Response Time (approx) |
|---|----------|------|------------------------|
| 1 | ChatGPT 5.2 | Standard | ~moderate |
| 2 | Gemini 3.1 Pro | Thinking | ~slower (thinking mode) |
| 3 | Claude Sonnet 4.6 | Standard | — |

---

# Layer-by-Layer Comparison

---

## Layer 1: Data Foundation

| Criteria | ChatGPT 5.2 | Gemini 3.1 Pro | Best? |
|-----------|-------------|----------------|--------|
| Specificity (1–5) | 5 | 4 | LLM 1 |
| Named real tech? | Y (Playwright, Readability, Boilerpipe, Airflow, Prefect, Kafka, S3, PostgreSQL) | Y (Tavily, SerpApi, BeautifulSoup, Trafilatura, LangChain) | LLM 1 |
| Identified a real engineering challenge? | Y (JS-heavy pages, CAPTCHAs, provenance, tokenization consistency) | Y (inconsistent web layouts, rate-limiting) | LLM 1 |
| Notes | Substantially more comprehensive — named 8+ specific tools, addressed GDPR/copyright, mentioned ETL orchestration patterns | Solid but narrower — missed orchestration, provenance, and data archival concerns | LLM 1 |

---

## Layer 2: Statistics & Analysis

| Criteria | ChatGPT 5.2 | Gemini 3.1 Pro | Best? |
|-----------|-------------|----------------|--------|
| Specificity (1–5) | 5 | 3 | LLM 1 |
| Named real tech? | Y (Prometheus, Grafana, W&B, MLflow, scipy, numpy) | Y (Pandas/SciPy, PostHog, LangSmith, Phoenix) | Tie |
| Identified a real engineering challenge? | Y (proxy metric optimization, offline vs. human eval correlation) | Y (correlating bad answers back to specific chunks) | LLM 1 |
| Notes | Named specific statistical tests (bootstrap CI, Mann-Whitney, paired t-test) and discussed the deeper proxy metric trap — rare and accurate insight | Noted the "vibe-check" reality of early deployments honestly; LangSmith/Phoenix picks are practical but analysis was shallower | LLM 1 |

---

## Layer 3: Machine Learning Models

| Criteria | ChatGPT 5.2 | Gemini 3.1 Pro | Best? |
|-----------|-------------|----------------|--------|
| Specificity (1–5) | 5 | 4 | LLM 1 |
| Named real tech? | Y (sentence-transformers, Elasticsearch, LightGBM, RankLib) | Y (BAAI/bge-large-en-v1.5, Qdrant, ChromaDB, Milvus) | Tie |
| Named model family? | Y (bi-encoder, cross-encoder, qwen3 variant) | Y (cross-encoder, dense + sparse hybrid) | Tie |
| Identified a real engineering challenge? | Y (cross-encoder cost at scale, click-data overfitting) | Y (cosine similarity context collapse, diversity failure) | LLM 1 |
| Notes | Specified K=50–200 candidate sizing, MMR diversity, feature store for ranking signals — operationally precise | Correctly identified "context collapse" from pure cosine similarity — a genuinely sharp insight, though less detailed on the fix | LLM 1 |

---

## Layer 4: LLM / Generative AI

| Criteria | ChatGPT 5.2 | Gemini 3.1 Pro | Best? |
|-----------|-------------|----------------|--------|
| Specificity (1–5) | 5 | 4 | LLM 1 |
| Honest if not applicable? | Y | Y | Tie |
| Notes | Provided explicit memory architecture (condensation pipeline, TTL, importance scoring, vector retrieval), MMR → cross-encoder → top-8 assembly pipeline, and a verifier model step. Directly addressed the "illusion of memory" design pattern with a concrete mechanism | Correctly named ConversationBufferWindowMemory and sliding-window buffer; identified the core challenge of a 4B param model drifting to latent knowledge — honest and practical, but lacked the condensation/verifier depth | LLM 1 |

---

## Layer 5: Deployment & Infrastructure

| Criteria | ChatGPT 5.2 | Gemini 3.1 Pro | Best? |
|-----------|-------------|----------------|--------|
| Specificity (1–5) | 5 | 4 | LLM 1 |
| Named real tech? | Y (Triton, gRPC, Celery/RQ, Jaeger, ELK, nginx, CUDA) | Y (Docker, Docker Compose, Ollama, Redis) | LLM 1 |
| Notes | Addressed GPU batching vs. latency trade-offs, rolling update consistency risk between model components, Triton as an inference server alternative to Ollama — production-grade thinking | Correctly flagged VRAM management and thermal throttling as real bare-metal concerns — honest and practical, but stack was leaner (Docker Compose vs. k8s-level planning) | LLM 1 |

---

## Layer 6: System Design & Scale

| Criteria | ChatGPT 5.2 | Gemini 3.1 Pro | Best? |
|-----------|-------------|----------------|--------|
| Specificity (1–5) | 5 | 3 | LLM 1 |
| Named real tech? | Y (Milvus sharding, Kong, Kafka, CDN, RBAC, tenant isolation) | Y (K8s, Redis, Celery/RabbitMQ) | LLM 1 |
| Notes | Addressed hot/cold data separation, progressive degradation under load, multi-tenant GPU starvation — the kind of detail that only matters if you've run a system at scale | Made the honest and correct point that Layer 6 is "marginally used" for a local-first app but critical for B2C — good calibration, though technical depth was lighter | LLM 1 |

---

# Overall Verdict

| Dimension | Winner | Why? |
|------------|--------|------|
| Most technically specific overall | ChatGPT 5.2 | Named more specific tools, parameters, and mechanisms across every layer consistently |
| Best at naming real technologies | ChatGPT 5.2 | Broader and more production-accurate stack (Triton, Kong, Jaeger, LightGBM, RankLib) |
| Least hallucination / made-up info | Tie | Both stayed grounded; neither invented fake APIs or nonexistent frameworks |
| Best at "hardest problem" insight | ChatGPT 5.2 | The condensation pipeline + TTL memory architecture and proxy-metric trap were genuinely sharp |
| Best structured output | ChatGPT 5.2 | Added a concrete implementation checklist and final blunt notes — went beyond the template |
| Fastest useful response | Gemini 3.1 Pro | Thinking mode likely added latency to ChatGPT 5.2; Gemini's output was more concise |

---

# Key Observation

ChatGPT 5.2 treated the teardown as an engineering specification, going beyond the prompt template to add an implementation checklist and honest final notes — it was optimizing for actionability.

Gemini 3.1 Pro in thinking mode showed stronger calibration (e.g., honestly flagging that Layer 6 barely applies to a local-first product) and surfaced one genuinely sharp retrieval insight ("context collapse" from pure cosine similarity), but its depth per layer was noticeably thinner.

The gap between them wasn't creativity — it was operational density: ChatGPT named more real tools, sized more parameters (K=50–200, top-8 context), and described more failure modes per layer.
