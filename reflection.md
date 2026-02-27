# Reflection — AI Product Teardown Engine (Sarvatra)

---

## Question 1: Which of the 6 layers surprised you the most in terms of complexity for the product you chose? Why?

Layer 3 — Machine Learning Models (Ranking) was the most surprising in terms of hidden complexity. On the surface, "retrieve relevant passages and pass them to the LLM" sounds like a solved problem — but the teardown revealed that naive dense-only retrieval produces a specific, insidious failure mode: context collapse, where the top-8 passages are all paraphrases of the same source, giving the generator false corroboration for a single viewpoint. The fix requires a three-step pipeline (dense + sparse hybrid retrieval → MMR diversity enforcement → cross-encoder re-ranking) that most teams don't implement fully, and each step involves non-trivial engineering trade-offs (K sizing for cross-encoder cost, MMR lambda tuning, BM25/dense fusion weights). What looked like a "fetch and generate" system turned out to have a ranking pipeline more complex than many standalone search engines.

---

## Question 2: What was the single biggest difference you noticed between the LLMs you tested?

The most striking difference was not accuracy or writing quality — it was *operational density*: the number of specific, actionable engineering decisions packed into each layer. ChatGPT 5.2 consistently named concrete parameters (K=50–200 candidates, top-8 final context, 20% chunk overlap), identified specific failure modes (rolling update consistency, click-data overfitting, proxy metric trap), and went beyond the prompt template to add an implementation checklist. Gemini 3.1 Pro Thinking, despite being in reasoning mode, produced thinner layers — it correctly identified the right frameworks but rarely explained *why* a specific design choice was made over alternatives or what breaks at scale if you skip a step. The gap wasn't that one model knew more facts — both were accurate. The gap was that ChatGPT 5.2 reasoned like an engineer who has *shipped* the system, while Gemini reasoned like an engineer who has *studied* the system. That distinction is exactly what separates a 4-rating from a 5-rating on the specificity scale, and it suggests that for production-facing technical tools, model selection matters more than prompt engineering alone.
