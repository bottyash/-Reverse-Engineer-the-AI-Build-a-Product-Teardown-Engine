# Stress Test — V2 Prompt on: Basic Calculator App

> Purpose: Verify the tool discriminates correctly — it should produce a thin teardown for a product with minimal AI, not artificially inflate depth to match the Sarvatra output.

---

## Product: Basic Calculator App
*(e.g., the default calculator on Android/iOS, or a web-based calculator like calculator.net)*

---

### Layer 1 — Data Foundation

**What's happening:** A basic calculator app has no meaningful data pipeline. User inputs (button taps or keystrokes) are ephemeral — processed in memory and discarded after the result is displayed. There is no data collection, storage, cleaning, or indexing layer.

**Key technologies likely used:** None relevant to AI/ML. Standard UI frameworks (Swift/UIKit for iOS, Kotlin/Jetpack Compose for Android, React or plain JS for web). Local state management (React useState, ViewModel on Android). No database, no storage, no ETL.

**The engineering challenge:** There is no AI engineering challenge at this layer. The engineering challenge is UI precision — floating point representation errors (e.g., `0.1 + 0.2 = 0.30000000000004`) require deliberate rounding or BigDecimal arithmetic to display results correctly.

**Skill required:** None specific to AI/ML. Mobile or web UI development, basic arithmetic precision in IEEE 754 floating point.

**Honesty check:** THIS LAYER IS THIN HERE — essentially absent. A calculator app collects no training data, runs no pipelines, and stores nothing. Any attempt to describe a "data foundation" for a basic calculator would be padding.

---

### Layer 2 — Statistics & Analysis

**What's happening:** No statistical analysis layer exists in a basic calculator. There are no ML metrics to track, no retrieval quality to measure, no A/B tests on model variants.

**Key technologies likely used:** Possibly basic product analytics (Firebase Analytics or Mixpanel) to track button tap frequency or crash rates — but this is standard mobile telemetry, not AI/ML analysis.

**The engineering challenge:** Not applicable at an AI/ML level. Standard crash monitoring (Sentry, Crashlytics) covers operational health.

**Skill required:** None specific to AI/ML data analysis.

**Honesty check:** THIS LAYER IS ABSENT. A basic calculator has no statistical or ML analysis layer. The only "analysis" is the arithmetic itself.

---

### Layer 3 — Machine Learning Models

**What's happening:** No machine learning models are used in a basic calculator. All computation is deterministic arithmetic executed at runtime. There is no prediction, classification, ranking, or pattern recognition.

**Key technologies likely used:** None. Arithmetic operations are handled by the language runtime (Java, Swift, JavaScript). BigDecimal or arbitrary-precision libraries for scientific calculators.

**The engineering challenge:** Not applicable. The closest thing to a "model decision" is choosing between floating-point and arbitrary-precision arithmetic — a deterministic engineering choice, not an ML design decision.

**Skill required:** None specific to ML. Basic numerical methods knowledge for scientific calculator variants.

**Honesty check:** THIS LAYER IS ABSENT. A basic calculator is one of the clearest examples of a software product with zero ML content. Adding ML here (e.g., "it could predict which function you'll use next") would describe a *hypothetical smarter product*, not the product being analyzed.

---

### Layer 4 — LLM / Generative AI

**What's happening:** No LLM or generative AI component exists in a basic calculator. Computation is rule-based and deterministic.

**Key technologies likely used:** None. A scientific calculator might use a parser (e.g., shunting-yard algorithm) for expression evaluation — this is classical computer science, not generative AI.

**The engineering challenge:** Not applicable.

**Skill required:** None specific to LLM/GenAI.

**Honesty check:** THIS LAYER IS ABSENT. If the product were upgraded to include a natural language input (e.g., "what's 15% of 340?"), Layer 4 would become relevant. For a standard calculator, it does not apply.

---

### Layer 5 — Deployment & Infrastructure

**What's happening:** A basic calculator app is deployed as a native mobile app (App Store / Play Store) or as a static web page. There is no server-side inference, no model serving, no GPU infrastructure. The "deployment" is the standard app store release pipeline.

**Key technologies likely used:** Xcode + TestFlight (iOS), Android Studio + Play Console (Android), GitHub Actions or Bitrise for CI/CD, static web hosting (GitHub Pages, Netlify, Vercel) for web calculators.

**The engineering challenge:** None specific to AI/ML. Standard mobile release engineering (version management, backward compatibility, app store review latency).

**Skill required:** Mobile CI/CD, app store deployment. No MLOps, no inference infrastructure.

**Honesty check:** THIS LAYER IS THIN from an AI perspective. Deployment exists (every app has it), but there is no AI/ML-specific infrastructure challenge here.

---

### Layer 6 — System Design & Scale

**What's happening:** A basic calculator app runs entirely client-side. There is no backend to scale, no vector DB to shard, no inference pool to autoscale. "Millions of users" is handled by the App Store CDN distributing the binary — not by any server architecture the team owns.

**Key technologies likely used:** CDN (App Store / Play Store infrastructure) for binary distribution. That's it. No Kubernetes, no Redis, no Kafka, no load balancers owned by the product team.

**The engineering challenge:** Not applicable at an AI/ML systems level. At massive scale, the only operational challenge is App Store review times and binary size optimization.

**Skill required:** None specific to distributed systems or AI infrastructure.

**Honesty check:** THIS LAYER IS ABSENT from an AI/ML perspective. The product scales trivially because it is stateless and runs on the user's device.

---

## Overall Analysis

**Most critical layer:** None — in the AI/ML sense. A basic calculator has no AI stack. The most critical *software engineering* layer is the arithmetic evaluation engine (expression parsing, floating-point precision), but this is classical CS, not ML.

**Complexity rating:** Simple (for AI/ML purposes) or Trivial. A basic calculator is the baseline case — zero ML layers, fully deterministic, client-side only.

**If rebuilding from scratch:** "The first thing you'd need to get right is the arithmetic evaluation engine — specifically handling operator precedence, parentheses, and floating-point precision — because incorrect arithmetic in a calculator is an immediate trust-destroying product failure. This has nothing to do with AI."

---

## Stress Test Verdict

**Did the tool discriminate correctly?** ✅ Yes.

The V2 prompt's Rule 3 ("THIS LAYER IS THIN HERE") forced honest calibration across all 6 layers. The output correctly identified that a basic calculator has:
- 0 layers with meaningful AI/ML content
- 1 layer (Deployment) with thin but real relevance
- 5 layers that are fully absent from an AI perspective

This is the expected behavior. A tool that produced an equally dense teardown for a calculator as for Sarvatra would be over-fitting to the template rather than actually analyzing the product — exactly the failure mode this stress test was designed to catch.

**Conclusion:** The V2 prompt generalizes correctly. It produces depth when the product has depth and thin output when the product warrants it.
