# Thinking Styles

Control how Claude approaches a problem — the reasoning mode, not just the output format.

| Command | Effect | Best used for |
|---|---|---|
| `/analyst` | Deep analysis — examines assumptions, dependencies, tradeoffs | Architecture reviews, complex decisions |
| `/critic` | Find flaws only — no praise, no balance | Stress-testing ideas before committing |
| `/optimizer` | Improve what's given — keep structure, enhance quality | Refining drafts, tightening code |
| `/simplify` | Strip to essentials, explain for a non-expert | Writing docs for non-technical stakeholders |
| `/eli5` | Explain Like I'm 5 — pure analogy, no jargon | Quick intuition-building on unfamiliar topics |
| `/deepdive` | Go very detailed — cover edge cases, nuance, depth | Research, learning a complex topic thoroughly |
| `/compare` | Side-by-side comparison of options | Choosing between tools, approaches, architectures |
| `/proscons` | Structured pros and cons list | Making a case, evaluating risk |
| `/firstprinciples` | Break down to fundamentals — why does this work? | Understanding root causes, avoiding cargo culting |
| `/contrarian` | Challenge the idea — argue the opposite | Pressure-testing assumptions, avoiding groupthink |

## Examples

```
/critic my plan to use a single Postgres DB for all microservices
→ Coupling risk: schema changes in one service affect all others...
   Scaling bottleneck: can't scale read/write independently per service...

/firstprinciples why does gradient descent work?
→ At its core: we want to minimize a loss function L(θ)...
   The gradient ∇L tells us the direction of steepest increase...

/contrarian using LLMs for structured data extraction
→ LLMs are probabilistic — structured extraction needs determinism...
   Regex + schema validation is faster, cheaper, and auditable...

/compare RAG vs fine-tuning for a domain-specific Q&A system
→ RAG: better for dynamic knowledge, no retraining cost, hallucination risk...
   Fine-tuning: better for style/format consistency, higher upfront cost...
```

## DS/ML-specific usage

```
/analyst my model has 94% accuracy but performs poorly in production
/critic this feature engineering pipeline [paste code]
/firstprinciples why does batch normalization help training stability?
/contrarian using neural networks for tabular data
/compare XGBoost vs LightGBM vs CatBoost for this use case
/deepdive how does RLHF actually work under the hood?
```
