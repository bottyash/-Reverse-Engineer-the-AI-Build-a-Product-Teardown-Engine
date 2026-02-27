# Best-Parts Map — Sarvatra Teardown

| Layer | Best LLM for This Layer | What to Extract |
|-------|------------------------|-----------------|
| Layer 1: Data Foundation | ChatGPT 5.2 | Full tech stack detail: Playwright + Readability/Boilerpipe, ETL orchestration (Airflow/Prefect), Kafka for streaming, S3 + PostgreSQL for storage, GDPR/copyright awareness, and the insight about guaranteeing consistent tokenization across embedding and generation stages |
| Layer 2: Statistics & Analysis | ChatGPT 5.2 | Named specific statistical tests (bootstrap CI, paired t-test, Mann-Whitney), called out the proxy metric trap — "increasing nDCG but worsening user satisfaction" — and tied Prometheus + Grafana + W&B into a coherent observability story |
| Layer 3: ML Models | ChatGPT 5.2 (content depth) + Gemini 3.1 Pro (key insight) | From ChatGPT 5.2: the K=50–200 candidate sizing, MMR diversity step, feature store for ranking signals, LightGBM/RankLib for LTR. From Gemini: the "context collapse" framing for pure cosine similarity — a sharp, interview-ready insight |
| Layer 4: LLM / Generative AI | ChatGPT 5.2 | The condensation pipeline architecture for memory (condensation model → summary + embeddings → TTL + importance scoring → retrieval at query time), the MMR → cross-encoder → top-8 assembly order, and the verifier model step for citation fidelity |
| Layer 5: Deployment & Infrastructure | ChatGPT 5.2 | Triton as inference server alternative to Ollama, batching vs. latency trade-off framing, rolling update consistency risk across model components (embedding + reranker + generator), GPU utilization via quantized shards |
| Layer 6: System Design & Scale | ChatGPT 5.2 (depth) + Gemini 3.1 Pro (calibration) | From ChatGPT 5.2: hot/cold data separation, progressive degradation under load, multi-tenant GPU starvation problem, Kong + Kafka + CDN stack. From Gemini: the honest calibration that Layer 6 is "marginally used" for a local-first product — use this as the honesty-check model |
| Overall Analysis / Hardest Problem | ChatGPT 5.2 | "Ranking-first" framing, the concrete implementation checklist (8 actionable steps), and the blunt final notes: "Don't let the generator be the source of truth" |
| Writing Style / Structure | ChatGPT 5.2 | Went beyond the prompt template by adding an implementation checklist and final notes section — structured output that is both scannable AND actionable |

## Key takeaway
ChatGPT 5.2 dominated content depth across all 8 dimensions. Gemini 3.1 Pro Thinking contributed two genuinely valuable additions: (1) the "context collapse" retrieval insight in Layer 3, and (2) the calibrated honesty in Layer 6 about scope. V2 prompt should force all LLMs to adopt both the depth of ChatGPT 5.2 and the scope-calibration instinct of Gemini.
