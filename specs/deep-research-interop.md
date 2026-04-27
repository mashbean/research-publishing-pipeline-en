# Deep Research Interoperability Specification

## Goal

Deep Research may be run by ChatGPT, Claude chat, a Claude Code subagent, or Codex. Regardless of tool, it should produce structured output that the pipeline can integrate automatically.

## Problem

1. Deep Research output is often free-form Markdown with inconsistent headings.
2. `run_deep_research.py integrate` is fragile when it relies on fuzzy heading matching.
3. Output structures vary across tools.
4. Research prompts often do not say exactly what structure downstream tools need.
5. After research returns, humans must decide which parts belong in which files.

## Solution: Structured Output Protocol

Deep Research must output the following fixed structure. Each section begins with a `<!-- SECTION: name -->` marker:

```markdown
<!-- SECTION: research_summary -->

# Research Summary

[2 to 3 paragraphs summarizing findings]

<!-- SECTION: reasoning_chain -->

# Reasoning Chain

## Core Question

[One sentence]

## Reasoning Path

### Sub-Arg 1: [name]

- **Claim**: ...
- **Reasoning type**: I/D/A/Ab/C
- **Supporting evidence**:
  1. [source grade] source description -> key finding
- **Reasoning strength**: strong/medium/weak
- **Counterexamples or limits**: ...

## Synthesis

- **Conclusion**: ...
- **Reasoning path**: Sub-Arg 1 (I) + Sub-Arg 2 (C) -> Synthesis
- **Conclusion strength**: strong/medium/weak
- **Known blind spots**: ...

<!-- SECTION: evidence_map -->

# Evidence Map

| # | Claim | Source | Source grade | Confidence | Access date | Notes |
|---|---|---|---|---|---|---|
| 1 | ... | ... | A/B/C | high/medium/low | 2026-03-30 | ... |

<!-- SECTION: source_registry -->

# Source Registry

## Academic Literature

- Author. "Title." URL or DOI.

## Official Documents

- Organization. "Title." URL. Accessed YYYY-MM-DD.

## Cases and Reporting

- Source. "Title." URL. Accessed YYYY-MM-DD.

<!-- SECTION: case_comparison -->

# Plus/Minus Case Comparison

| Category | Success case | Failure case | Comparison value |
|---|---|---|---|
| ... | ... | ... | ... |

## Structured Case Data

### [case name]

- Type: ...
- Founded: ...
- Status: active / struggling / failed / acquired
- Legal structure: ...
- Revenue model: ...
- Annual revenue: ... (source, year)
- User scale: ... (source, access date)
- Governance model: ...
- Key risks: ...
- Failure mode: ... (if applicable)

<!-- SECTION: high_risk_claims -->

# High-Risk Claims

| # | Claim | Risk type | Explanation | Suggested handling |
|---|---|---|---|---|
| 1 | ... | causal confusion / survivorship bias / category drift / stale data | ... | Rewrite as ... |

<!-- SECTION: open_questions -->

# Open Questions

- ...

<!-- SECTION: rewrite_warnings -->

# Rewrite Warnings

- ...
```

## Why HTML Comments

1. ChatGPT and Claude chat can both produce them.
2. They are easy to parse, for example with `re.split(r'<!-- SECTION: (\w+) -->')`.
3. They remain readable in Markdown.
4. If one section is missing, other sections can still be parsed.

## Use Across Tools

### ChatGPT Deep Research

Append this to the prompt:

```markdown
## Output Format Requirement

Use the following structure. Each section must begin with `<!-- SECTION: name -->`.

Required sections, in order:

1. `<!-- SECTION: research_summary -->` Research Summary
2. `<!-- SECTION: reasoning_chain -->` Reasoning Chain, including core question, sub-arguments, and synthesis
3. `<!-- SECTION: evidence_map -->` Evidence Map, as a Markdown table
4. `<!-- SECTION: source_registry -->` Source Registry, grouped by academic, official, and case/reporting sources
5. `<!-- SECTION: case_comparison -->` Plus/minus case comparison and structured case data
6. `<!-- SECTION: high_risk_claims -->` High-Risk Claims
7. `<!-- SECTION: open_questions -->` Open Questions
8. `<!-- SECTION: rewrite_warnings -->` Rewrite Warnings

These markers allow downstream tools to split your research output automatically.
```

### Claude Chat

Append the same format requirement to the prompt. Claude usually follows explicit structure well.

### Claude Code Subagent

Reference this spec directly in the subagent prompt:

```python
Agent(
    description="Deep Research",
    subagent_type="general-purpose",
    prompt=f"Read {job_dir}/prompts/agent-research.md and specs/deep-research-interop.md. Produce research output using the structured protocol and save it to {job_dir}/raw/deep-research-output.md."
)
```

### Codex

When running a research task in Codex, include the format requirement in the task description. Save Codex output to `raw/deep-research-output.md`, then integrate it:

```bash
python3 scripts/run_deep_research.py <job-id> integrate
```

## Automatic Integration

`run_deep_research.py integrate` should:

1. Read `raw/deep-research-output.md`.
2. Split by `<!-- SECTION: (\w+) -->`.
3. Write each section to the matching file:

| Section | Output file |
|---|---|
| `research_summary` | `notes/research-summary.md` |
| `reasoning_chain` | `notes/reasoning-chain.md` |
| `evidence_map` | `verification/evidence-map.md` |
| `source_registry` | `notes/source-notes.md` |
| `case_comparison` | `notes/case-comparison.md` |
| `high_risk_claims` | `verification/high-risk-claims.md` |
| `open_questions` | `notes/open-questions.md` |
| `rewrite_warnings` | `notes/rewrite-warnings.md` |

4. Update `state.json` to `evidence-mapped`.
5. If `reasoning_chain` exists, advance directly to `evidence-mapped`. If only `evidence_map` exists, still advance to `evidence-mapped`. If no sections parse, save the full output to `drafts/research-draft.md` as a fallback.

## Quality Gate

After integration, check:

1. `evidence_map` exists and is not empty.
2. `reasoning_chain` includes at least 2 sub-arguments.
3. `case_comparison` includes at least 1 failed case.
4. `source_registry` includes at least 1 A-grade source.

If any check fails, set status to `needs-decision` and record what is missing.
