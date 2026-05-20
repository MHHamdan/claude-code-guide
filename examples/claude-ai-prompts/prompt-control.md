# Prompt Control Modifiers

Fine-grained control over tone, format, and length. These work best combined with other commands.

| Command | Effect | Example |
|---|---|---|
| `/toneformal` | Formal, professional tone | `/toneformal /expand explain containerization` |
| `/tonecasual` | Casual, conversational tone | `/tonecasual explain what a p-value is` |
| `/persuasive` | Convincing, argument-led tone | `/persuasive the case for TypeScript over JavaScript` |
| `/data` | Include stats and numbers | `/data /analyst state of LLMs in production 2025` |
| `/examplesonly` | Skip theory — examples only | `/examplesonly show me list comprehensions in Python` |
| `/noexamples` | Skip examples — concepts only | `/noexamples /expand explain the CAP theorem` |
| `/limit` | Limit word count | `/limit 100 words what is retrieval augmented generation` |
| `/expandpoints` | Expand each bullet point fully | `/expandpoints risks of LLMs in production` |
| `/bullet` | Bullet format output | `/bullet best practices for pandas performance` |
| `/nobullet` | Paragraph format only | `/nobullet /expand explain transformer architecture` |

## Combining commands

The real power is combining modifiers with thinking styles and output modes:

```
/analyst /data /bullet compare PostgreSQL vs MongoDB for a high-write workload
→ Deep analysis + statistics + bullet format

/brief /ghost what's the default learning rate in Adam optimizer?
→ Minimal response, answer only

/deepdive /noexamples /toneformal the mathematics of attention mechanisms
→ Detailed, formal, theory-only

/critic /bullet /expandpoints this system design: [paste]
→ Flaws only, each expanded with explanation

/learn /tonecasual /examples what is a confusion matrix
→ Casual explanation, example-driven
```

## Practical modifier cheat sheet for DS/ML work

| Goal | Command combo |
|---|---|
| Understand a paper quickly | `/summary /bullet /limit 200 words` |
| Deep understanding of a concept | `/deepdive /explainwhy /noexamples` |
| Interview prep | `/brief /bullet /practice` |
| Write a technical blog post | `/authority /nobullet /expand` |
| Quick sanity check | `/ghost /critic` |
| Learn with examples | `/learn /examplesonly /tonecasual` |
| Architecture review | `/analyst /data /expandpoints /bullet` |
