# LLM Selection Decision — Sarvatra Teardown Engine

## Decision Framework

| Decision Factor | My Choice | Reason |
|----------------|-----------|--------|
| Which LLM followed my prompt structure most faithfully? | ChatGPT 5.2 | Produced output in the exact section format specified, added an implementation checklist unprompted, and respected the "honesty check" instruction on every layer |
| Which LLM was most technically accurate (least hallucination)? | Tie — ChatGPT 5.2 / Gemini 3.1 Pro | Neither invented nonexistent frameworks; ChatGPT 5.2 named more real tools (Triton, RankLib, Jaeger, Kong) while Gemini stayed conservative but accurate |
| Which LLM's output was most readable and well-organized? | ChatGPT 5.2 | Clear headers, consistent section structure, and a final "blunt notes" section that made the output feel authored — not just generated |
| Which LLM handled the "honesty check" best? | Gemini 3.1 Pro Thinking | Correctly flagged Layer 6 as "marginally used" for a local-first product — the only model to apply scope calibration without being asked explicitly |

---

## Selected LLM for Final Tool

**Selected LLM:** ChatGPT 5.2 (Standard mode)

**Why:** ChatGPT 5.2 produced the highest operational density across all 6 layers — it named more specific tools, sized actual parameters (K=50–200, top-8 context window), described more failure modes, and went beyond the prompt template with a concrete implementation checklist. While Gemini 3.1 Pro Thinking showed stronger calibration on scope, ChatGPT 5.2's depth-per-layer advantage was decisive for a tool that will be used in client-facing and interview contexts where technical specificity is the primary signal of competence.

---

## Trade-off Acknowledged

Choosing ChatGPT 5.2 means accepting slightly weaker honesty calibration (it was less likely to say a layer was thin). The V2 prompt compensates for this with Rule 3 ("THIS LAYER IS THIN HERE") explicitly forcing that behavior — so the model's natural depth is retained while the calibration gap is closed at the prompt level.
