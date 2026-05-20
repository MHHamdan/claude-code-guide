# Output Modes

Control the length and structure of Claude's response.

| Command | Effect | Best used for |
|---|---|---|
| `/ghost` | Final answer only — no explanation, no reasoning | Quick lookups, yes/no decisions |
| `/minimal` | Shortest possible response | Mobile, quick reference |
| `/brief` | 3–5 lines max | Sanity checks, fast answers |
| `/expand` | Detailed explanation with context | Learning something new |
| `/stepbystep` | Numbered steps, sequential | Debugging, setup guides, walkthroughs |
| `/checklist` | Actionable checkbox list | Pre-launch reviews, process verification |
| `/framework` | Structured framework with named components | Decision-making, mental models |
| `/blueprint` | Implementation plan with phases | Project planning, architecture |
| `/playbook` | Repeatable system with triggers and actions | SOPs, runbooks, team processes |
| `/roadmap` | Timeline-based steps with milestones | Quarterly plans, learning paths |

## Examples

```
/ghost what's the time complexity of quicksort?
→ O(n log n) average, O(n²) worst case.

/stepbystep how do I set up a Python virtual environment with uv?
→ 1. Install uv: curl -LsSf https://astral.sh/uv/install.sh | sh
   2. Create env: uv venv
   3. Activate: source .venv/bin/activate
   4. Add packages: uv add pandas numpy

/blueprint build a feature flag system for a Python microservice
→ Phase 1: Storage layer (Redis key-value for flags)...
   Phase 2: Evaluation engine...
   Phase 3: Admin API...

/playbook handle an on-call incident for a down API
→ Trigger: PagerDuty alert fires
   Step 1: Check status page and recent deploys...
   Step 2: Pull logs from CloudWatch...
```

## DS/ML-specific usage

```
/stepbystep walk me through implementing SMOTE for class imbalance
/blueprint design an offline feature store for a recommendation system
/checklist things to verify before deploying an ML model to production
/framework evaluate whether to fine-tune vs RAG for this use case
/roadmap go from zero to production MLOps in 3 months
```
