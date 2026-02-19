# Team of Skeptics

**A reusable quality architecture for AI agent systems.**

> *The best outputs do not need protection from scrutiny. They earn trust through it.*

## What This Is

A pattern and service layer that adds structured adversarial review to any AI agent pipeline. Five specialised skeptic agents — each with a single, non-overlapping mandate — challenge agent output before it becomes a decision.

| Skeptic | Mandate | Question |
|---------|---------|----------|
| **Improver** | Output optimisation | "Could this be better?" |
| **Falsifier** | Adversarial review | "Can I prove this is false?" |
| **Auditor** | Evidence assessment | "Do I believe this? Show me the source." |
| **Cross-Checker** | Consistency verification | "Does this align with what other agents produced?" |
| **Closer** | Completeness assurance | "Did we deliver everything we said we would?" |

## Why It Matters

Most agent pipelines trust unchallenged AI output as source of truth. This pattern inserts deterministic checkpoints and adversarial evaluation at every handoff — the difference between a system that demos well and one that can be trusted in production.



## Source Material

- [Foundational reference doc](docs/team-of-skeptics.md) — full context, all five skeptic definitions, structured prompts, anti-patterns, extension guide
- [LinkedIn article](https://www.linkedin.com/pulse/why-agents-need-team-skeptics-michael-mcclary-y8bac/) — published summary
