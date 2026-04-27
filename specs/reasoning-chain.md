# Reasoning Chain Specification

## Goal

Every article should have a traceable reasoning structure from core question to final conclusion. Each step should identify whether it is fact, inference, or rhetoric, and every inference should name its reasoning type and strength.

## Reasoning Chain Structure

A research article reasoning chain has four layers:

```text
Layer 1: Core Question
    |
Layer 2: Sub-Arguments, each answering one part of the core question
    |
Layer 3: Evidence, with at least one A-grade or B-grade source for each sub-argument
    |
Layer 4: Synthesis, deriving a defensible conclusion from the sub-arguments
```

## Reasoning Type Labels

Every reasoning step from Layer 3 to Layer 2 or from Layer 2 to Layer 4 must be labeled:

| Type | Symbol | Description | Risk level |
|------|------|------|---------|
| Deduction | `D` | If the premises are true, the conclusion necessarily follows | Low |
| Induction | `I` | Generalizes a pattern from multiple cases | Medium |
| Analogy | `A` | Infers from a similar context | Medium-high |
| Abduction | `Ab` | Infers the best available explanation | High |
| Causal | `C` | Claims that X causes Y | Must distinguish observed correlation from verified causality |

## Reasoning Chain Template

```markdown
## Core Question

[One sentence]

## Reasoning Chain

### Sub-Arg 1: [sub-argument name]

- **Claim**: [one sentence]
- **Reasoning type**: I, induction from three cases
- **Supporting evidence**:
  1. [A-grade source] Wikimedia FY2024-25 audit -> $208.6M revenue, mostly donations
  2. [A-grade source] Signal 990 -> $8.6M annual deficit
  3. [B-grade source] Mastodon blog -> roughly EUR700K/year
- **Reasoning strength**: medium, because the three cases differ substantially in scale
- **Counterexamples or limits**: The cases operate at very different scales, limiting generalization.
- **Conclusion**: [safe conclusion derived from the evidence]

### Sub-Arg 2: ...

## Synthesis

- **Derived from Sub-Args 1-N**: [final conclusion]
- **Reasoning path**: Sub-Arg 1 (I) + Sub-Arg 2 (C) + Sub-Arg 3 (A) -> Synthesis (Ab)
- **Conclusion strength**: [strong/medium/weak]
- **Known blind spots**: [unaddressed counterevidence or alternative explanations]
```

## Required Reasoning Pitfalls

### 1. Causation vs Correlation

Bad: `After Python's BDFL transition, its TIOBE ranking rose`, when written to imply causation.

Good: `After Python completed its governance transition in 2018, its TIOBE ranking continued to rise. Whether the two are causally related remains unsettled.`

Rule: unless there is a controlled study or quasi-experimental design, observed covariation must be written as correlation rather than causation.

### 2. Survivorship Bias

Bad: `Wikipedia has survived for 25 years, proving the nonprofit model works.`

Good: `Wikipedia is one of the few nonprofit platforms to survive for more than 20 years. Failed peer cases such as Diaspora*, Ello, and App.net show that survival rates are low.`

Rule: every successful case needs at least one comparable failed case.

### 3. Category Drift

Bad: `All nonprofit platforms can use the cooperative model.`

Good: `Platform cooperatives such as Stocksy fit transactional settings. Pure social platforms have fewer successful cooperative precedents.`

Rule: mark each case's applicable category in the evidence map. Do not generalize directly across categories without justification.

### 4. Missing Counterfactuals

Bad: `Mastodon's nonprofit structure made it unable to scale quickly.`

Good: `Mastodon's scaling speed is constrained by its federated architecture and limited gGmbH resources. Whether nonprofit structure itself is the constraint, or whether factors such as UX and onboarding matter more, needs comparison with Bluesky, which is nonprofit but centralized.`

Rule: every causal claim must include at least one counterfactual question.

## Place in the Pipeline

The reasoning chain is built between `evidence-mapped` and `drafted`.

Deliverable: `notes/reasoning-chain.md`

Timing: after the evidence map is complete and before the research draft begins.

Owner: the research subagent or the writer subagent's first step.

Reviewer: the critic subagent must compare against the reasoning chain during fact-checking and evaluate whether each inference type and strength is reasonable.
