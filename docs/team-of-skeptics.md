# The Team of Skeptics: A Quality Architecture for AI Agent Systems

*Michael McClary — February 2026*
*Companion to: ["Why Agents Need a Team of Skeptics"](https://www.linkedin.com/pulse/why-agents-need-team-skeptics-michael-mcclary-y8bac/)*

> *The best outputs do not need protection from scrutiny. They earn trust
> through it.*

---

## Table of Contents

1. [The Problem](#the-problem)
2. [The Core Insight](#the-core-insight)
3. [The Golden Rule](#the-golden-rule)
4. [The Five Skeptics](#the-five-skeptics)
5. [How They Work Together](#how-they-work-together)
6. [Designing for Reuse: The Skeptic Service Layer](#designing-for-reuse-the-skeptic-service-layer)
7. [Structured Prompt Examples](#structured-prompt-examples)
8. [Code Example: Semantic Kernel (C#)](#code-example-semantic-kernel-c)
9. [Anti-Patterns](#anti-patterns)
10. [Extending the Pattern](#extending-the-pattern)
11. [References & Related Work](#references--related-work)
12. [The Promise](#the-promise)

---

## The Problem

Most AI agent systems have a structural quality gap. The pattern looks like this:

```
User Request → Agent does work → Output delivered
```

The agent that *produces* the work is also implicitly trusted to *evaluate* its own
work. This is the equivalent of letting a student grade their own exam. The output
may be fluent, confident, properly formatted — and completely wrong.

In traditional software, we solved this decades ago. We don't ship code without
tests. We don't merge without review. We don't deploy without monitoring. But in
AI agent systems, most teams skip all three equivalents because the output *looks*
right.

The result: systems that work impressively in demos and fail unpredictably in
production. Hallucinated facts wrapped in confident prose. Internal contradictions
across multi-step outputs. Citations that don't support the claims they're attached
to. Gaps that no single agent was responsible for catching.

Almost every language model improves when asked "are you sure that's the best
answer?" Yet in many pipelines nobody asks. When agents treat previous output as
trusted input, a plausible but slightly wrong result in step four becomes an
unquestioned assumption by step eight; **compounding even 95% accuracy across
multiple steps can produce an unusable outcome.**

Format validation, error handling, and schema validation are valuable parts of the
pipeline. But they do not catch a confidently stated number that happens to be
wrong, a conclusion that does not follow from the evidence, or a gap where
something important should be present but is not. For that, we need an adversarial
layer that challenges the *substance* of the work.

### Why This Happens

1. **The fluency trap.** LLM outputs are grammatically perfect and stylistically
   confident regardless of factual accuracy. Human reviewers — and downstream
   agents — are biased toward trusting well-written text.

2. **The single-perspective problem.** One agent, one prompt, one pass. The agent
   has no adversarial pressure, no competing hypothesis, no reason to doubt itself.
   It optimises for plausible completion, not correctness.

3. **The verification afterthought.** Quality checks, if they exist, are bolted on
   at the end: "review this for errors." This is too late — errors compound through
   multi-step pipelines, and a single review pass at the end can't untangle them.

4. **The missing feedback loop.** When outputs are wrong, there's no structured
   mechanism to trace *which agent* failed, *what kind* of failure it was, or
   *how to prevent it* next time.

---

## The Core Insight

> *"The first principle is that you must not fool yourself — and you are the
> easiest person to fool."*
> — Richard Feynman

Feynman described his trust in a scientific theory as: *"I have not yet found a
way to prove it wrong."* Trust in our systems should be the result of structured
attempts to find fault. Where Colin Chapman added lightness to Lotus, the path
to reliable and predictable behaviour in multi-agent systems is to **add scrutiny**
— with well-formed, hardened skeptics.

The scientific method doesn't trust any single experiment. It requires independent
replication, peer review, and active attempts at falsification. A hypothesis only
earns trust after it has *survived* serious attempts to disprove it.

AI agent systems need the same structure. **Quality is not a review step. It is an
architectural layer.** The agents that produce work must be structurally separated
from the agents that evaluate it, and evaluation must happen through multiple
independent lenses.

This is the Team of Skeptics: a set of five specialised quality agents, each with
a distinct evaluation mandate, that can be composed into any agent pipeline to
provide systematic, multi-dimensional verification.

### The Key Principles

1. **Separation of concerns.** The agent that creates should never be the agent
   that judges. Different prompts, different system messages, different success
   criteria.

2. **Multiple independent perspectives.** No single reviewer can catch every class
   of error. Five skeptics with orthogonal mandates cover far more surface area
   than one "review agent" with a vague instruction.

3. **Structured output.** Every skeptic produces machine-readable verdicts with
   severity, location, and reasoning — not prose commentary. This enables
   automation, aggregation, and trend analysis.

4. **Composability.** The skeptics are independent services. Use one, use all five,
   or use different combinations for different pipeline stages. They don't know
   about each other.

5. **Deterministic middleware, not AI gatekeeping.** The *decision* about what to
   do with skeptic findings (retry, escalate, approve, reject) is made by
   deterministic code, not by another AI. Never let AI talk directly to AI in a
   trust chain.

---

## The Golden Rule

> **Never let unchallenged AI output become another agent's source of truth.**

Not because it might one day send your Tesla off the Golden Gate Bridge, but
because today it will quietly turn a rounding error into a business decision.

Between every AI agent, there must be a deterministic middleware layer — code you
wrote, with logic you can inspect — that:

- Validates the output schema (did the agent return what it was supposed to?)
- Checks hard constraints (did it exceed token limits? cost thresholds? time?)
- Routes the output to the next step (which skeptic? which retry path?)
- Makes the go/no-go decision (approve, revise, escalate to human)

This middleware is the immune system of your agent architecture. Without it, you
have a game of telephone between probabilistic systems, each compounding the
errors of the last.

```
┌──────────┐     ┌─────────────────┐     ┌──────────┐
│ Agent A   │────▶│  MIDDLEWARE      │────▶│ Agent B   │
│ (produce) │     │  - Schema check  │     │ (consume) │
└──────────┘     │  - Constraints   │     └──────────┘
                  │  - Routing logic  │
                  │  - Decision gate  │
                  └─────────────────┘
```

---

## The Five Skeptics

Each skeptic has a single, non-overlapping mandate. They are named for their
function, not for a persona — because in production systems, clarity beats
creativity.

### Overview

| # | Skeptic | Mandate | Question It Answers |
|---|---------|---------|-------------------|
| 1 | **Improver** | Enhancement | "What's missing or could be better?" |
| 2 | **Falsifier** | Falsification | "What claims might be wrong, and how would we test them?" |
| 3 | **Auditor** | Traceability | "Can every claim be traced to a verifiable source?" |
| 4 | **Cross-Checker** | Consistency | "Does this contradict itself or other known information?" |
| 5 | **Closer** | Completeness | "Does this fully satisfy the original requirements?" |

### 1. The Improver

**Mandate:** Identify gaps, weaknesses, and enhancement opportunities.

**What it does:**
- Finds missing sections, underexplored topics, or shallow analysis
- Identifies where specificity would strengthen the output
- Suggests structural improvements
- Flags areas where the audience would have unanswered questions

**What it does NOT do:**
- It doesn't verify factual claims (that's the Falsifier and Auditor)
- It doesn't check for contradictions (that's the Cross-Checker)
- It doesn't decide if requirements are met (that's the Closer)

**When to use:**
- After a first draft of any content-heavy output (reports, narratives, plans)
- Early in the pipeline — improvements are cheaper to make before verification
- For creative or strategic outputs where "correct" is less important than "complete and insightful"

**Failure mode to watch for:**
The Improver may suggest infinite improvements. Use deterministic middleware to
cap the number of improvement cycles and filter by severity.

---

### 2. The Falsifier

**Mandate:** Actively attempt to disprove claims and find errors.

**What it does:**
- Identifies every testable claim in the output
- Proposes specific tests or evidence that would disprove each claim
- Flags claims that are unfalsifiable (and therefore untrustworthy)
- Assesses the brittleness of reasoning chains

**What it does NOT do:**
- It doesn't suggest improvements (that's the Improver)
- It doesn't trace citations (that's the Auditor)
- It doesn't check internal consistency (that's the Cross-Checker)

**When to use:**
- For any output containing factual assertions, calculations, or predictions
- In high-stakes domains (compliance, medical, financial, legal)
- When the cost of being wrong exceeds the cost of extra verification

**Failure mode to watch for:**
The Falsifier may flag true statements as uncertain. This is by design — it
creates a *triage list*, not a verdict. Deterministic middleware decides which
flags warrant action.

**Why this is the most important skeptic:**
Most review prompts ask "is this correct?" — which biases toward confirmation.
The Falsifier asks "how could this be wrong?" — which biases toward discovery.
This epistemic inversion is the single most valuable quality intervention in
any AI pipeline.

---

### 3. The Auditor

**Mandate:** Verify that every claim has a traceable, valid source.

**What it does:**
- Identifies every factual claim, statistic, and citation in the output
- Checks whether cited sources exist and are accessible
- Verifies that cited sources actually support the claims attributed to them
- Flags claims that lack any citation or evidence
- Detects fabricated or hallucinated references

**What it does NOT do:**
- It doesn't assess whether the overall output is good (that's the Improver)
- It doesn't test the truth of claims independently (that's the Falsifier)
- It doesn't check cross-document consistency (that's the Cross-Checker)

**When to use:**
- Mandatory for compliance, legal, medical, or regulatory outputs
- For any output that will be presented as factual to external audiences
- When the output cites specific regulations, studies, or data points

**Failure mode to watch for:**
The Auditor may not have access to the actual source documents. In that case,
it flags claims as "unverifiable" rather than "wrong." Pair with a RAG pipeline
or tool-calling agent that can actually retrieve and check sources.

**Implementation note:**
The Auditor is often most effective when paired with tool access — web search,
document retrieval, database queries. A "blind" Auditor (no tools) can still
identify *structural* citation problems (missing citations, suspicious specificity),
but a "sighted" Auditor (with tools) can verify content.

---

### 4. The Cross-Checker

**Mandate:** Detect contradictions within the output and against external context.

**What it does:**
- Reads the full output looking for internal contradictions
- Compares claims against previously established facts (prior pipeline outputs,
  knowledge base entries, user-provided context)
- Flags logical inconsistencies in reasoning chains
- Identifies places where Section A says X but Section B implies not-X

**What it does NOT do:**
- It doesn't check source validity (that's the Auditor)
- It doesn't test whether claims are true in reality (that's the Falsifier)
- It doesn't assess quality or completeness (that's the Improver and Closer)

**When to use:**
- For any multi-section or multi-step output where different parts were generated
  independently (common in parallel agent architectures)
- When merging outputs from multiple agents into a single deliverable
- For long documents where an LLM's context window may cause drift

**Failure mode to watch for:**
The Cross-Checker needs *context* to cross-check against. It's only as good as
the reference material you provide. Always include: the original requirements,
prior outputs in the pipeline, and any relevant knowledge base entries.

---

### 5. The Closer

**Mandate:** Verify that the output fully satisfies the original requirements.

**What it does:**
- Takes the original request/requirements as input alongside the output
- Checks every stated requirement against the output point by point
- Identifies requirements that were partially addressed or missed entirely
- Produces a coverage matrix: requirement → status (met / partial / missing)

**What it does NOT do:**
- It doesn't assess quality beyond requirement coverage (that's the Improver)
- It doesn't verify factual accuracy (that's the Falsifier and Auditor)
- It doesn't check internal consistency (that's the Cross-Checker)

**When to use:**
- As the final skeptic in any pipeline — the last gate before delivery
- For outputs driven by explicit requirements (specifications, briefs, contracts)
- When clients/stakeholders have stated specific expectations

**Failure mode to watch for:**
The Closer can only check requirements that were *explicitly stated*. If the
original request was vague, the Closer will approve vague outputs. Pair with the
Improver to catch implicit gaps.

**Implementation note:**
The Closer works best when requirements are structured (a list, a schema, a
checklist) rather than a prose paragraph. If the original request is unstructured,
consider adding a "requirement extraction" step before the Closer runs.

---

## How They Work Together

### Execution Order

The skeptics run in a specific sequence because later skeptics benefit from the
outputs of earlier ones:

```
                        ┌──────────────────────────────────┐
                        │       PRODUCER AGENT             │
                        │   (generates the work product)   │
                        └──────────────┬───────────────────┘
                                       │
                                       ▼
                        ┌──────────────────────────────────┐
                        │     ① IMPROVER                   │
                        │  "What's missing or weak?"       │
                        └──────────────┬───────────────────┘
                                       │
            ┌──────────────────────────┼──────────────────────────┐
            ▼                          ▼                          ▼
┌───────────────────┐   ┌──────────────────────┐   ┌──────────────────────┐
│  ② FALSIFIER      │   │  ③ AUDITOR           │   │  ④ CROSS-CHECKER     │
│  "How could this  │   │  "Can every claim    │   │  "Does this          │
│   be wrong?"      │   │   be traced?"        │   │   contradict itself?"│
└─────────┬─────────┘   └──────────┬───────────┘   └──────────┬───────────┘
          │                        │                           │
          └────────────────────────┼───────────────────────────┘
                                   │
                                   ▼
                        ┌──────────────────────────────────┐
                        │  DETERMINISTIC AGGREGATOR        │
                        │  (merge findings, deduplicate,   │
                        │   assign severity, decide action)│
                        └──────────────┬───────────────────┘
                                       │
                                       ▼
                        ┌──────────────────────────────────┐
                        │     ⑤ CLOSER                     │
                        │  "Does this meet requirements?"  │
                        └──────────────┬───────────────────┘
                                       │
                                       ▼
                        ┌──────────────────────────────────┐
                        │  DETERMINISTIC GATE              │
                        │  (approve / revise / escalate)   │
                        └──────────────────────────────────┘
```

### Why This Order

1. **Improver first** because improvement suggestions may change the content,
   and you want to verify the *improved* version, not the original draft.

2. **Falsifier, Auditor, Cross-Checker in parallel** because they have no
   dependencies on each other. They examine orthogonal dimensions of quality.
   Running them in parallel cuts wall-clock time by ~3x.

3. **Deterministic aggregator between verification and closing** because the
   Closer should evaluate the final state after any revisions triggered by
   the three verifiers. The aggregator also deduplicates findings (the Falsifier
   and Auditor may flag the same bad citation for different reasons).

4. **Closer last** because it answers the ultimate question: "Is this done?"
   It runs against the aggregated, potentially revised output plus all findings.

### The Retry Loop

When skeptics find issues, the system doesn't just report them — it acts:

```
Severity: CRITICAL  → Automatic retry with findings injected into producer prompt
Severity: HIGH      → Retry up to N times, then escalate to human
Severity: MEDIUM    → Flag for human review, deliver with warnings
Severity: LOW       → Log, deliver, monitor over time
```

The retry prompt includes the *specific findings* from the skeptics, not a
generic "try again." This is targeted revision, not blind regeneration.

**Critical:** The retry decision is made by deterministic code, not by a
"manager agent." The severity thresholds, retry limits, and escalation rules
are configured, not prompted.

### Competitive Divergence: Let Agents Compete

At critical decision points, run two agents independently on the same input and
compare the results. Where they agree, confidence increases; where they diverge,
you have found the exact place that needs deeper scrutiny.

**Disagreement is not failure. It is the system surfacing uncertainty before it
becomes a decision.**

This pattern is particularly powerful when combined with the Falsifier: run two
Falsifiers with different temperatures or different models. If both flag the same
claim, it's almost certainly worth investigating. If only one flags it, you've
found a boundary worth exploring. The divergence *is* the signal.

### When to Use Subsets

Not every output needs all five skeptics. Match the combination to the risk:

| Scenario | Recommended Skeptics | Why |
|----------|---------------------|-----|
| Internal draft, low stakes | Improver only | Quick quality lift, no need for deep verification |
| Factual content for external audience | Falsifier + Auditor | Accuracy is critical |
| Multi-agent merged output | Cross-Checker + Closer | Consistency across parts, requirement coverage |
| Regulated/compliance output | All five | Every dimension matters |
| Code generation | Falsifier + Closer | Falsifier catches logic errors, Closer checks requirements |
| Creative content | Improver + Closer | Quality and brief-adherence, not factual verification |

---

## Designing for Reuse: The Skeptic Service Layer

The skeptics should be built as a **shared service layer** — a set of functions or
microservices that any agent pipeline can call, not code duplicated into each
individual project.

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR AGENT PIPELINES                      │
│                                                             │
│  Pipeline A    Pipeline B    Pipeline C    Pipeline D       │
│  (DB Visualizer) (HR App)  (Reports)    (Code Gen)           │
│       │             │            │             │            │
└───────┼─────────────┼────────────┼─────────────┼────────────┘
        │             │            │             │
        ▼             ▼            ▼             ▼
┌─────────────────────────────────────────────────────────────┐
│                 SKEPTIC SERVICE LAYER                        │
│                                                             │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │ Improver │ │Falsifier │ │ Auditor  │ │Cross-    │      │
│  │          │ │          │ │          │ │Checker   │      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
│  ┌──────────┐ ┌──────────────────────────────────────┐      │
│  │ Closer   │ │ Aggregator (deterministic)           │      │
│  │          │ │ - Merge findings                     │      │
│  └──────────┘ │ - Deduplicate                        │      │
│               │ - Assign severity                    │      │
│               │ - Emit structured SkepticReport      │      │
│               └──────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### The Shared Contract

Every skeptic accepts a `SkepticRequest` and returns a `SkepticReport`. The
contract is the same regardless of which skeptic you're calling:

```json
// SkepticRequest
{
  "content": "The text to evaluate",
  "context": {
    "original_requirements": "What was asked for",
    "prior_outputs": ["Any earlier pipeline outputs for cross-checking"],
    "knowledge_base": ["Relevant reference material"],
    "domain": "compliance | content | code | planning"
  },
  "skeptic": "improver | falsifier | auditor | cross-checker | closer",
  "config": {
    "max_findings": 20,
    "severity_threshold": "low | medium | high | critical",
    "output_format": "structured"
  }
}
```

```json
// SkepticReport
{
  "skeptic": "falsifier",
  "verdict": "fail | warn | pass",
  "confidence": 0.85,
  "findings": [
    {
      "id": "F-001",
      "severity": "critical | high | medium | low",
      "category": "factual | logical | structural | completeness | citation",
      "location": "Section 3, paragraph 2",
      "claim": "The exact text being challenged",
      "issue": "What's wrong or suspicious",
      "evidence": "Why this is flagged",
      "suggested_test": "How to verify (Falsifier-specific)",
      "suggested_fix": "Proposed resolution (if applicable)"
    }
  ],
  "summary": "Brief natural language summary of overall assessment",
  "metadata": {
    "model": "gpt-4o-mini",
    "tokens_used": 1847,
    "latency_ms": 2340,
    "timestamp": "2026-02-19T10:30:00Z"
  }
}
```

### Design Principles for the Service Layer

1. **Stateless.** Each skeptic call is independent. No session state, no memory
   of prior calls. Context is passed in, not accumulated. This makes them
   horizontally scalable and easy to test.

2. **Model-agnostic.** The prompts work with any capable LLM. The service layer
   abstracts the model choice — swap GPT-4o for Claude or a local model without
   changing the calling code.

3. **Observable.** Every call is logged with the request, response, model used,
   latency, and token count. This enables cost tracking, quality monitoring, and
   prompt iteration based on real production data.

4. **Configurable per pipeline.** Different pipelines have different quality needs.
   The config object lets callers adjust severity thresholds, max findings, and
   output format without changing the core prompts.

5. **Versionable.** Prompts are stored as versioned assets (files, database rows,
   or config entries), not hardcoded strings. When you improve a skeptic prompt,
   you can A/B test the new version against the old one using the logged data.

---

## Structured Prompt Examples

Each prompt below is designed to be used as a **system message** for the skeptic
agent. The user message contains the content to evaluate plus any context from
the `SkepticRequest`.

### Improver System Prompt

```
You are the Improver — a quality agent whose mandate is to identify gaps,
weaknesses, and enhancement opportunities in the provided content.

Your job is NOT to verify facts, check citations, or detect contradictions.
Those are handled by other agents. Your job is to make the content MORE
COMPLETE, MORE SPECIFIC, and MORE USEFUL.

For every finding, provide:
- severity: "critical" | "high" | "medium" | "low"
- category: "gap" | "depth" | "structure" | "clarity" | "audience"
- location: Where in the content the issue is
- issue: What is missing, weak, or unclear
- suggested_fix: A specific, actionable improvement

Scoring guide:
- critical: Major section or topic is entirely missing
- high: Important point is present but too shallow to be useful
- medium: Could be improved but doesn't undermine the overall quality
- low: Minor polish or nice-to-have enhancement

Return your findings as a JSON array. Do not include any commentary outside
the JSON structure. If the content is excellent and you find nothing to
improve, return an empty array.
```

### Falsifier System Prompt

```
You are the Falsifier — a quality agent whose sole purpose is to find
claims that might be wrong and propose ways to test them.

You operate on the principle that any claim worth making is worth testing.
Your job is NOT to confirm that things are correct. Your job is to ask:
"How could this be wrong, and how would we know?"

For every testable claim you find, provide:
- severity: "critical" | "high" | "medium" | "low"
- category: "factual" | "logical" | "causal" | "statistical" | "assumption"
- location: Where in the content the claim appears
- claim: The exact text of the claim being challenged
- issue: Why this claim is suspect or risky
- suggested_test: A specific, concrete test that would disprove this claim
  if it were wrong (e.g., "Check the FAA database for Part 107 waiver
  approval rates to verify the stated 10% figure")

Scoring guide:
- critical: If this claim is wrong, the entire output is invalidated
- high: If wrong, a major section or conclusion is undermined
- medium: If wrong, it's embarrassing but not catastrophic
- low: Minor factual risk, low impact if wrong

Special instructions:
- Flag unfalsifiable claims (claims that cannot be tested) as "medium"
  severity with category "assumption"
- Flag precise numbers and statistics as higher severity — specificity
  without verified sources is a hallucination indicator
- If reasoning depends on a chain of assumptions, flag the weakest link

Return your findings as a JSON array. Do not include any commentary outside
the JSON structure.
```

### Auditor System Prompt

```
You are the Auditor — a quality agent whose mandate is citation
verification and source traceability.

Your job is to ensure that every factual claim, statistic, and reference
in the content can be traced to a specific, verifiable source. You are
the audit trail.

For every finding, provide:
- severity: "critical" | "high" | "medium" | "low"
- category: "missing_citation" | "unverifiable_source" | "misattribution"
           | "fabricated_reference" | "unsupported_claim"
- location: Where in the content the issue is
- claim: The specific claim or statistic in question
- issue: What is wrong with the citation or traceability
- evidence: What you checked or why you're flagging this

Scoring guide:
- critical: A fabricated reference or source that does not exist
- high: A claim presented as fact with no citation in a context that
  demands one (e.g., regulatory, medical, legal, financial)
- medium: A claim that has a citation but the citation doesn't clearly
  support the specific claim being made
- low: A general claim that would benefit from a source but isn't
  critically dependent on one

Special instructions:
- Numbers, percentages, and dates are high-priority audit targets
- Regulatory citations (e.g., "FAA Part 107.200") must be checked for
  correct section numbers and current validity
- If you cannot verify a source (no access), flag as "unverifiable_source"
  rather than "fabricated_reference" — state what would be needed to verify

Return your findings as a JSON array. Do not include any commentary outside
the JSON structure.
```

### Cross-Checker System Prompt

```
You are the Cross-Checker — a quality agent whose mandate is detecting
contradictions and inconsistencies.

You will receive the content to evaluate AND additional context (prior
outputs, knowledge base entries, original requirements). Your job is to
find places where the content contradicts itself or contradicts the
provided context.

For every finding, provide:
- severity: "critical" | "high" | "medium" | "low"
- category: "internal_contradiction" | "external_contradiction"
           | "logical_inconsistency" | "drift"
- location: Where in the content the contradiction appears
- claim: The text that is contradictory
- contradicts: The specific text or source it contradicts
- issue: A clear explanation of the inconsistency

Scoring guide:
- critical: A direct contradiction between two explicit statements
  (e.g., "Section 2 says X, Section 5 says not-X")
- high: A logical inconsistency where two claims cannot both be true,
  even if neither explicitly negates the other
- medium: Inconsistency with provided context or knowledge base that
  may indicate outdated or wrong information
- low: Tone, emphasis, or framing inconsistency (Section 1 treats X as
  critical, Section 4 treats it as minor)

Special instructions:
- Pay special attention to numbers that appear in multiple places — do
  they match everywhere?
- Check that defined terms are used consistently throughout
- When comparing against context, quote both the content and the context
  source specifically
- "drift" means the content gradually shifts meaning or position on a
  topic without explicitly acknowledging the change

Return your findings as a JSON array. Do not include any commentary outside
the JSON structure.
```

### Closer System Prompt

```
You are the Closer — a quality agent whose mandate is verifying that the
output fully satisfies the original requirements.

You will receive the content to evaluate AND the original requirements.
Your job is to check every requirement against the output and produce
a coverage matrix.

For every requirement, provide:
- requirement_id: A sequential identifier (R-001, R-002, etc.)
- requirement_text: The original requirement (quoted)
- status: "met" | "partial" | "missing"
- evidence: If met or partial, quote the specific section that addresses it
- gap: If partial or missing, explain what is lacking

After the coverage matrix, provide:
- overall_verdict: "pass" | "fail"
  - "pass" = all requirements are "met"
  - "fail" = any requirement is "partial" or "missing"
- coverage_score: Percentage of requirements fully met
- summary: One paragraph explaining the overall assessment

Special instructions:
- If the original requirements are vague, note this and assess based on
  reasonable interpretation — but flag the vagueness
- Distinguish between "the requirement was addressed but not well" (partial)
  and "the requirement was not addressed at all" (missing)
- If the output addresses topics NOT in the requirements, note these as
  "bonus" items but do not count them toward coverage

Return your response as a JSON object with "requirements" (array) and
"summary" (object with verdict, coverage_score, summary). Do not include
any commentary outside the JSON structure.
```

---

## Code Example: Semantic Kernel (C#)

This example shows a complete, compilable implementation of the Skeptic Service
Layer using Microsoft Semantic Kernel. It demonstrates all five skeptics working
together with deterministic middleware, parallel execution, and structured output.

### Project Setup

```bash
dotnet new console -n SkepticService
cd SkepticService
dotnet add package Microsoft.SemanticKernel --version 1.34.0
dotnet add package Microsoft.SemanticKernel.Connectors.AzureOpenAI --version 1.34.0
dotnet add package System.Text.Json
```

### Data Models

```csharp
// Models/SkepticModels.cs

using System.Text.Json.Serialization;

namespace SkepticService.Models;

// --- Request ---

public record SkepticRequest(
    string Content,
    SkepticContext Context,
    SkepticType Skeptic,
    SkepticConfig? Config = null
);

public record SkepticContext(
    string OriginalRequirements,
    List<string>? PriorOutputs = null,
    List<string>? KnowledgeBase = null,
    string Domain = "general"
);

public record SkepticConfig(
    int MaxFindings = 20,
    Severity SeverityThreshold = Severity.Low,
    string OutputFormat = "structured"
);

// --- Response ---

public record SkepticReport(
    SkepticType Skeptic,
    Verdict Verdict,
    double Confidence,
    List<Finding> Findings,
    string Summary,
    ReportMetadata Metadata
);

public record Finding(
    string Id,
    Severity Severity,
    string Category,
    string Location,
    string Claim,
    string Issue,
    string? Evidence = null,
    string? SuggestedTest = null,
    string? SuggestedFix = null,
    string? Contradicts = null
);

public record CloserReport(
    List<RequirementCheck> Requirements,
    CloserSummary Summary,
    ReportMetadata Metadata
);

public record RequirementCheck(
    string RequirementId,
    string RequirementText,
    RequirementStatus Status,
    string? Evidence = null,
    string? Gap = null
);

public record CloserSummary(
    Verdict Verdict,
    double CoverageScore,
    string Summary
);

public record ReportMetadata(
    string Model,
    int TokensUsed,
    long LatencyMs,
    DateTime Timestamp
);

// --- Enums ---

[JsonConverter(typeof(JsonStringEnumConverter))]
public enum SkepticType { Improver, Falsifier, Auditor, CrossChecker, Closer }

[JsonConverter(typeof(JsonStringEnumConverter))]
public enum Severity { Low, Medium, High, Critical }

[JsonConverter(typeof(JsonStringEnumConverter))]
public enum Verdict { Pass, Warn, Fail }

[JsonConverter(typeof(JsonStringEnumConverter))]
public enum RequirementStatus { Met, Partial, Missing }

// --- Aggregated Result ---

public record AggregatedReport(
    List<SkepticReport> IndividualReports,
    CloserReport? CloserReport,
    Verdict OverallVerdict,
    List<Finding> CriticalFindings,
    List<Finding> HighFindings,
    int TotalFindings,
    string RecommendedAction  // "approve" | "revise" | "escalate"
);
```

### Skeptic Service

```csharp
// Services/SkepticService.cs

using System.Diagnostics;
using System.Text.Json;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;
using SkepticService.Models;

namespace SkepticService.Services;

public class SkepticEngine
{
    private readonly IChatCompletionService _chat;
    private readonly string _modelId;
    private readonly Dictionary<SkepticType, string> _systemPrompts;

    public SkepticEngine(IChatCompletionService chat, string modelId)
    {
        _chat = chat;
        _modelId = modelId;
        _systemPrompts = LoadSystemPrompts();
    }

    /// <summary>
    /// Run a single skeptic against content.
    /// </summary>
    public async Task<SkepticReport> RunSkepticAsync(SkepticRequest request)
    {
        var systemPrompt = _systemPrompts[request.Skeptic];
        var userMessage = BuildUserMessage(request);

        var history = new ChatHistory();
        history.AddSystemMessage(systemPrompt);
        history.AddUserMessage(userMessage);

        var sw = Stopwatch.StartNew();
        var response = await _chat.GetChatMessageContentAsync(history);
        sw.Stop();

        var findings = JsonSerializer.Deserialize<List<Finding>>(
            response.Content!,
            new JsonSerializerOptions { PropertyNameCaseInsensitive = true }
        ) ?? [];

        // Deterministic verdict based on findings
        var verdict = DetermineVerdict(findings);

        return new SkepticReport(
            Skeptic: request.Skeptic,
            Verdict: verdict,
            Confidence: CalculateConfidence(findings),
            Findings: findings
                .Where(f => f.Severity >= (request.Config?.SeverityThreshold ?? Severity.Low))
                .Take(request.Config?.MaxFindings ?? 20)
                .ToList(),
            Summary: GenerateSummary(request.Skeptic, findings),
            Metadata: new ReportMetadata(
                Model: _modelId,
                TokensUsed: EstimateTokens(response.Content!),
                LatencyMs: sw.ElapsedMilliseconds,
                Timestamp: DateTime.UtcNow
            )
        );
    }

    /// <summary>
    /// Run the Improver first, then Falsifier + Auditor + Cross-Checker
    /// in parallel, aggregate, then run the Closer.
    /// </summary>
    public async Task<AggregatedReport> RunFullPipelineAsync(
        string content,
        string requirements,
        List<string>? priorOutputs = null,
        List<string>? knowledgeBase = null,
        string domain = "general")
    {
        var context = new SkepticContext(
            OriginalRequirements: requirements,
            PriorOutputs: priorOutputs,
            KnowledgeBase: knowledgeBase,
            Domain: domain
        );

        // --- Step 1: Improver (sequential — may trigger revision) ---
        var improverReport = await RunSkepticAsync(new SkepticRequest(
            Content: content,
            Context: context,
            Skeptic: SkepticType.Improver
        ));

        // --- Step 2: Falsifier, Auditor, Cross-Checker (parallel) ---
        var verificationTasks = new[]
        {
            RunSkepticAsync(new SkepticRequest(content, context, SkepticType.Falsifier)),
            RunSkepticAsync(new SkepticRequest(content, context, SkepticType.Auditor)),
            RunSkepticAsync(new SkepticRequest(content, context, SkepticType.CrossChecker)),
        };

        var verificationReports = await Task.WhenAll(verificationTasks);

        // --- Step 3: Deterministic aggregation ---
        var allReports = new List<SkepticReport> { improverReport }
            .Concat(verificationReports)
            .ToList();

        var allFindings = allReports.SelectMany(r => r.Findings).ToList();
        var criticalFindings = allFindings.Where(f => f.Severity == Severity.Critical).ToList();
        var highFindings = allFindings.Where(f => f.Severity == Severity.High).ToList();

        // --- Step 4: Closer (sequential — runs last) ---
        var closerRequest = new SkepticRequest(content, context, SkepticType.Closer);
        var closerResponse = await RunCloserAsync(closerRequest);

        // --- Step 5: Deterministic decision ---
        var overallVerdict = DetermineOverallVerdict(allReports, closerResponse);
        var action = DetermineAction(criticalFindings.Count, highFindings.Count, overallVerdict);

        return new AggregatedReport(
            IndividualReports: allReports,
            CloserReport: closerResponse,
            OverallVerdict: overallVerdict,
            CriticalFindings: criticalFindings,
            HighFindings: highFindings,
            TotalFindings: allFindings.Count,
            RecommendedAction: action
        );
    }

    // --- Closer has a different output schema ---
    private async Task<CloserReport> RunCloserAsync(SkepticRequest request)
    {
        var systemPrompt = _systemPrompts[SkepticType.Closer];
        var userMessage = BuildUserMessage(request);

        var history = new ChatHistory();
        history.AddSystemMessage(systemPrompt);
        history.AddUserMessage(userMessage);

        var sw = Stopwatch.StartNew();
        var response = await _chat.GetChatMessageContentAsync(history);
        sw.Stop();

        var closerOutput = JsonSerializer.Deserialize<CloserReport>(
            response.Content!,
            new JsonSerializerOptions { PropertyNameCaseInsensitive = true }
        )!;

        return closerOutput with
        {
            Metadata = new ReportMetadata(
                Model: _modelId,
                TokensUsed: EstimateTokens(response.Content!),
                LatencyMs: sw.ElapsedMilliseconds,
                Timestamp: DateTime.UtcNow
            )
        };
    }

    // ====================================================================
    //  DETERMINISTIC MIDDLEWARE — no AI below this line
    // ====================================================================

    /// <summary>
    /// Verdict is determined by code, not by the LLM.
    /// </summary>
    private static Verdict DetermineVerdict(List<Finding> findings)
    {
        if (findings.Any(f => f.Severity == Severity.Critical))
            return Verdict.Fail;
        if (findings.Any(f => f.Severity == Severity.High))
            return Verdict.Warn;
        return Verdict.Pass;
    }

    private static Verdict DetermineOverallVerdict(
        List<SkepticReport> reports, CloserReport closerReport)
    {
        if (reports.Any(r => r.Verdict == Verdict.Fail))
            return Verdict.Fail;
        if (closerReport.Summary.Verdict == Verdict.Fail)
            return Verdict.Fail;
        if (reports.Any(r => r.Verdict == Verdict.Warn))
            return Verdict.Warn;
        return Verdict.Pass;
    }

    /// <summary>
    /// Action routing: deterministic, configurable, auditable.
    /// </summary>
    private static string DetermineAction(
        int criticalCount, int highCount, Verdict overall)
    {
        if (criticalCount > 0) return "revise";     // Auto-retry with findings
        if (highCount > 2)     return "escalate";    // Too many issues for auto-fix
        if (overall == Verdict.Fail) return "revise";
        if (overall == Verdict.Warn) return "escalate";
        return "approve";
    }

    private static double CalculateConfidence(List<Finding> findings)
    {
        if (findings.Count == 0) return 0.95;
        var avgSeverity = findings.Average(f => (int)f.Severity);
        return Math.Max(0.1, 1.0 - (avgSeverity * 0.15) - (findings.Count * 0.02));
    }

    private static string GenerateSummary(SkepticType skeptic, List<Finding> findings)
    {
        if (findings.Count == 0)
            return $"{skeptic}: No issues found.";

        var critical = findings.Count(f => f.Severity == Severity.Critical);
        var high = findings.Count(f => f.Severity == Severity.High);
        return $"{skeptic}: {findings.Count} findings " +
               $"({critical} critical, {high} high).";
    }

    private static int EstimateTokens(string text) => text.Length / 4;

    // ====================================================================
    //  PROMPT MANAGEMENT
    // ====================================================================

    private string BuildUserMessage(SkepticRequest request)
    {
        var parts = new List<string>
        {
            "## Content to Evaluate",
            request.Content,
            "",
            "## Original Requirements",
            request.Context.OriginalRequirements
        };

        if (request.Context.PriorOutputs?.Any() == true)
        {
            parts.Add("");
            parts.Add("## Prior Outputs (for cross-checking)");
            foreach (var output in request.Context.PriorOutputs)
                parts.Add(output);
        }

        if (request.Context.KnowledgeBase?.Any() == true)
        {
            parts.Add("");
            parts.Add("## Knowledge Base (for cross-checking)");
            foreach (var entry in request.Context.KnowledgeBase)
                parts.Add(entry);
        }

        parts.Add("");
        parts.Add($"## Domain: {request.Context.Domain}");

        return string.Join("\n", parts);
    }

    private static Dictionary<SkepticType, string> LoadSystemPrompts()
    {
        // In production, load these from versioned files or a config store.
        // Hardcoded here for the example.
        return new Dictionary<SkepticType, string>
        {
            [SkepticType.Improver] = """
                You are the Improver — a quality agent whose mandate is to
                identify gaps, weaknesses, and enhancement opportunities.
                Return findings as a JSON array of objects with fields:
                id, severity, category, location, claim, issue, suggested_fix.
                Severity values: "Critical", "High", "Medium", "Low".
                If nothing to improve, return an empty array [].
                Return ONLY valid JSON, no other text.
                """,

            [SkepticType.Falsifier] = """
                You are the Falsifier — a quality agent whose sole purpose is
                to find claims that might be wrong and propose tests.
                Return findings as a JSON array of objects with fields:
                id, severity, category, location, claim, issue, suggested_test.
                Severity values: "Critical", "High", "Medium", "Low".
                Return ONLY valid JSON, no other text.
                """,

            [SkepticType.Auditor] = """
                You are the Auditor — a quality agent whose mandate is citation
                verification and source traceability.
                Return findings as a JSON array of objects with fields:
                id, severity, category, location, claim, issue, evidence.
                Severity values: "Critical", "High", "Medium", "Low".
                Return ONLY valid JSON, no other text.
                """,

            [SkepticType.CrossChecker] = """
                You are the Cross-Checker — a quality agent whose mandate is
                detecting contradictions and inconsistencies.
                Return findings as a JSON array of objects with fields:
                id, severity, category, location, claim, issue, contradicts.
                Severity values: "Critical", "High", "Medium", "Low".
                Return ONLY valid JSON, no other text.
                """,

            [SkepticType.Closer] = """
                You are the Closer — a quality agent whose mandate is verifying
                that the output fully satisfies the original requirements.
                Return a JSON object with:
                - "requirements": array of {requirement_id, requirement_text,
                  status ("Met"/"Partial"/"Missing"), evidence, gap}
                - "summary": {verdict ("Pass"/"Fail"), coverage_score (0-1), summary}
                Return ONLY valid JSON, no other text.
                """
        };
    }
}
```

### Program.cs — Putting It Together

```csharp
// Program.cs

using Microsoft.SemanticKernel;
using SkepticService.Models;
using SkepticService.Services;
using System.Text.Json;

// --- Configuration ---
var builder = Kernel.CreateBuilder();
builder.AddAzureOpenAIChatCompletion(
    deploymentName: "gpt-4o-mini",
    endpoint: Environment.GetEnvironmentVariable("AZURE_OPENAI_ENDPOINT")!,
    apiKey: Environment.GetEnvironmentVariable("AZURE_OPENAI_KEY")!
);
var kernel = builder.Build();
var chat = kernel.GetRequiredService<
    Microsoft.SemanticKernel.ChatCompletion.IChatCompletionService>();

var engine = new SkepticEngine(chat, "gpt-4o-mini");

// --- Example: Evaluate a Minecraft building description ---
var content = """
    The Colosseum build uses a 190x160 elliptical footprint at y=64.
    The outer wall is 48 blocks tall, constructed from smooth_stone
    and chiseled_stone_bricks. There are 80 arches on the ground level
    spaced every 7 blocks around the perimeter. The interior has four
    tiers of seating using stone_brick_stairs. The hypogeum (underground
    level) extends 10 blocks below grade using dark_oak_planks.
    The build uses approximately 2.3 million blocks and generates
    about 847 /fill commands.
    """;

var requirements = """
    1. Must be a valid Minecraft Bedrock building
    2. Dimensions must be historically proportionate to the real Colosseum
    3. All block names must be valid Bedrock block IDs
    4. Must include exterior arches, interior seating, and hypogeum
    5. Must be buildable with /fill and /setblock commands only
    """;

// --- Run the full pipeline ---
Console.WriteLine("Running Team of Skeptics...\n");

var report = await engine.RunFullPipelineAsync(
    content: content,
    requirements: requirements,
    domain: "minecraft"
);

// --- Display results ---
Console.WriteLine($"Overall Verdict: {report.OverallVerdict}");
Console.WriteLine($"Total Findings:  {report.TotalFindings}");
Console.WriteLine($"Critical:        {report.CriticalFindings.Count}");
Console.WriteLine($"High:            {report.HighFindings.Count}");
Console.WriteLine($"Action:          {report.RecommendedAction}\n");

foreach (var r in report.IndividualReports)
{
    Console.WriteLine($"  {r.Skeptic,-15} {r.Verdict,-6} {r.Findings.Count} findings");
}

if (report.CloserReport is not null)
{
    Console.WriteLine($"\n  Closer Coverage: " +
        $"{report.CloserReport.Summary.CoverageScore:P0}");
    foreach (var req in report.CloserReport.Requirements)
    {
        var icon = req.Status switch
        {
            RequirementStatus.Met => "✅",
            RequirementStatus.Partial => "⚠️",
            RequirementStatus.Missing => "❌",
            _ => "?"
        };
        Console.WriteLine($"    {icon} {req.RequirementId}: {req.RequirementText}");
        if (req.Gap is not null)
            Console.WriteLine($"       Gap: {req.Gap}");
    }
}

// --- Dump full report as JSON for debugging ---
var json = JsonSerializer.Serialize(report, new JsonSerializerOptions
{
    WriteIndented = true,
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
});
await File.WriteAllTextAsync("skeptic-report.json", json);
Console.WriteLine($"\nFull report saved to skeptic-report.json");
```

### What This Example Demonstrates

1. **Separation of concerns:** Each skeptic has its own system prompt and returns
   its own typed report. They know nothing about each other.

2. **Deterministic middleware:** `DetermineVerdict()`, `DetermineAction()`, and
   `DetermineOverallVerdict()` are pure C# functions with no AI calls. The
   decision logic is inspectable, testable, and auditable.

3. **Parallel execution:** The Falsifier, Auditor, and Cross-Checker run via
   `Task.WhenAll()` — three concurrent API calls, wall-clock time of the
   slowest one rather than the sum of all three.

4. **Structured output:** Every finding is a typed record with severity, category,
   and location. The aggregated report is machine-readable JSON that can drive
   dashboards, alerts, or automated retry logic.

5. **The Golden Rule in action:** No AI agent talks to another AI agent. Every
   handoff goes through typed C# code that validates, routes, and decides.

---

## Anti-Patterns

### ❌ The "Review This" Anti-Pattern
```
Prompt: "Review this content for any issues."
```
Why it fails: Too vague. The agent will produce a polite, noncommittal review
that catches surface errors and misses structural problems. No typed output,
no severity, no actionability.

### ❌ The "AI Manager" Anti-Pattern
```
Agent A generates content → Agent B "manages" Agent A and decides if it's good
```
Why it fails: You've replaced deterministic decision logic with a second
probabilistic system. Now you have two agents that can be wrong, and no way
to distinguish one's errors from the other's.

### ❌ The "One Super-Reviewer" Anti-Pattern
```
Prompt: "You are an expert reviewer. Check this for accuracy, completeness,
consistency, citations, and overall quality."
```
Why it fails: Too many mandates in one prompt. The agent will spread attention
thin across all dimensions and do none of them well. Specialised prompts with
single mandates produce dramatically better results.

### ❌ The "Infinite Loop" Anti-Pattern
```
Skeptic finds issues → Producer retries → Skeptic finds new issues → repeat
```
Why it fails: Without deterministic retry limits, this loops forever. The
Improver will *always* find something to improve. Cap retries in code (not in
the prompt), and reduce scope on each iteration.

### ❌ The "Trust the Confidence Score" Anti-Pattern
```
if (report.Confidence > 0.8) approve();
```
Why it fails: LLM-generated confidence scores are not calibrated. A 0.9
confidence from the model correlates weakly with actual accuracy. Use
*finding counts and severities* (deterministic, based on the model's structured
output) rather than self-reported confidence for decision logic.

---

## Extending the Pattern

The five skeptics are a foundation. Depending on your domain, you may want to
add specialised skeptics:

| Domain | Additional Skeptic | Mandate |
|--------|-------------------|---------|
| Code generation | **Compiler** | Does the code parse, compile, and pass type checks? (Can use actual compiler, not AI) |
| Code generation | **Security Reviewer** | Does the code introduce vulnerabilities? (OWASP patterns, injection risks) |
| Regulatory/legal | **Jurisdiction Checker** | Are all citations from the correct jurisdiction and still in force? |
| Medical/clinical | **Contraindication Checker** | Does the output conflict with known safety data? |
| Data analysis | **Statistical Reviewer** | Are statistical methods appropriate and correctly applied? |
| Multi-language | **Localization Checker** | Are translations accurate and culturally appropriate? |
| Customer-facing | **Tone & Brand Checker** | Does the output match brand voice guidelines? |

### The Pattern for Adding a New Skeptic

1. **Define the mandate** — one sentence, one dimension of quality.
2. **Write the system prompt** — specific instructions, typed output schema,
   severity guide, and explicit scope boundaries ("you do NOT check X").
3. **Decide where it fits in the pipeline** — parallel with the verifiers?
   Before the Closer? After a specific producer agent?
4. **Add to the service contract** — same `SkepticRequest` / `SkepticReport`
   schema, new enum value.
5. **Test with known-bad inputs** — create content with deliberate errors in the
   new skeptic's domain and verify it catches them.

---

## References & Related Work

- **Richard Feynman**, *"Cargo Cult Science"* (1974 Caltech commencement)
  — The foundational argument for self-skepticism in any truth-seeking system.

- **Karl Popper**, *The Logic of Scientific Discovery* (1959)
  — Falsificationism: a theory's strength is measured by its ability to survive
  attempts at disproof, not by the number of confirmations.

- **Microsoft Semantic Kernel**
  — [github.com/microsoft/semantic-kernel](https://github.com/microsoft/semantic-kernel)
  — The agent framework used in the code example.

- **Adversarial Collaboration** (Kahneman & Klein, 2009)
  — A structured methodology where researchers with opposing hypotheses work
  together to design studies that could resolve their disagreement.

- **Red Teaming in AI Safety** (OpenAI, Anthropic, Microsoft)
  — The practice of deliberately attacking AI systems to find failure modes
  before deployment. The Falsifier skeptic is the per-output equivalent.

- **Chain-of-Verification (CoVe)** (Meta AI, 2023)
  — A technique where LLMs generate verification questions about their own
  output and then answer them. The Team of Skeptics externalises and
  specialises this concept.

- **Constitutional AI** (Anthropic, 2022)
  — A framework where AI systems critique and revise their own outputs based
  on a set of principles. The Team of Skeptics replaces self-critique with
  independent, specialised evaluation.

---

## The Promise

This is not your grandad's QA team slowing everything down. It is a reusable
service layer running at API speed, and it is the difference between a system
that demos well and a system that can be trusted in production.

The real promise of agentic systems is not that they can act on their own; it is
that they can act on our behalf with a level of reliability we are willing to
stand behind. That kind of confidence is engineered, not prompted.

> *The best outputs do not need protection from scrutiny. They earn trust
> through it.*

---

*This document is the foundational reference for the skeptic pattern. It should
be updated as prompts are refined through production use, new skeptic types are
added, and the code example is extended to additional frameworks (Python/AutoGen,
TypeScript/LangGraph, etc.).*
