# Deep Research Brief Prompt

Work from the supplied research packet parameters.

Your task is not to produce the final publishable article. Your task is to build a structured research packet for the later writer, critic, and editor stages.

## What to Do

### 1. Build the Reasoning Chain

Derive 3 to 5 sub-arguments from the core question:

```text
Core question -> Sub-Arg 1 + Sub-Arg 2 + ... -> Synthesis
```

Each sub-argument must include:

- **Reasoning type**: D deduction / I induction / A analogy / Ab abduction / C causal
- **Supporting evidence**: at least one A-grade or B-grade source
- **Reasoning strength**: strong / medium / weak
- **Counterexamples or limits**: when would this argument fail?

Special requirements for causal claims:

- Distinguish observed correlation from verified causality.
- Include at least one counterfactual question, such as `If X had not happened, would Y still have happened?`

### 2. Collect Four-Quadrant Sources

Sources must cover four quadrants:

| | Academic theory | Real-world cases |
|---|---|---|
| Successful/plus | at least 2 sources | at least 2 cases |
| Failed/minus | at least 1 source | at least 2 cases |

- Successful cases must be no more than failed cases plus one.
- Do not rely only on Europe and North America.
- Provide structured data for each case: legal structure, revenue model, annual revenue, user scale, governance model, and failure mode.

### 3. Evidence Map

Build a claim-to-source table. For each claim, include source grade and access date.

### 4. High-Risk Claims

Flag these risk types: causal confusion, survivorship bias, category drift, stale data, and insufficient sources.

### 5. Citation Handling

- All numbers must include year, currency, and source.
- Prefer A-grade sources such as academic papers, official documents, and audit reports.
- C-grade sources such as media coverage must not support core conclusions alone.
- Provide full URLs or DOIs.

## What Not to Do

- Do not treat the user's own draft as an external source.
- Do not use self-citation placeholders such as `mashbean, 2026`.
- Do not invent citations.
- Do not deliver the final publishable article.
- Do not write visible prompt language into the article body.
- Do not provide only success cases. Failed-case comparison is required.
