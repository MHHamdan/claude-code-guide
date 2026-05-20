# Coding & Tech

Commands for software development tasks in Claude.ai chat.

| Command | Effect | Best used for |
|---|---|---|
| `/debug` | Find the bug — explain root cause, not just symptom | Paste an error or broken code |
| `/refactor` | Clean the code — improve structure without changing behavior | Legacy code, messy functions |
| `/optimizecode` | Improve performance — time/space complexity, I/O, caching | Slow queries, hot paths |
| `/systemdesign` | Architecture design — components, interfaces, tradeoffs | Designing new systems |
| `/api` | API structure — endpoints, request/response schemas, versioning | REST/GraphQL design |
| `/database` | DB design — schema, indexes, normalization, query patterns | Data modeling |
| `/scalability` | Scaling approach — horizontal/vertical, caching, queuing | Preparing for load |
| `/security` | Security checks — vulnerabilities, attack vectors, mitigations | Pre-launch review |
| `/testcases` | Generate tests — unit, integration, edge cases | Untested code |
| `/pseudocode` | Logic only — language-agnostic algorithm sketch | Planning before implementation |

## Examples

```
/debug
AttributeError: 'NoneType' object has no attribute 'transform'
[paste stack trace]
→ Root cause: your pipeline fitted on training data returns None when...

/optimizecode [paste slow pandas merge]
→ Current: O(n²) merge on string column
   Fix: convert to categorical first, then merge — 40x faster on large frames

/testcases write tests for this function: [paste]
→ Happy path: valid input returns expected output
   Edge case: empty list input
   Edge case: all-null values
   Error case: wrong type raises TypeError

/systemdesign a real-time leaderboard for a mobile game (10M DAU)
→ Write path: game server → Kafka → Redis sorted set (ZADD)
   Read path: client → CDN cache → Redis ZREVRANK
   Persistence: Redis → Postgres async via consumer
```

## DS/ML-specific usage

```
/debug my PyTorch training loop loss is NaN after epoch 2 [paste code]
/optimizecode this feature engineering pipeline runs in 4 hours [paste]
/testcases for this data preprocessing function [paste]
/systemdesign an ML feature store for 500 features across 50M users
/database design a schema to store model experiments, runs, and artifacts
/security review this API that serves model predictions [paste]
/pseudocode an online learning system that updates model weights in real-time
```
