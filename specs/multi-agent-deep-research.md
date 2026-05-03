# Multi-Agent Deep Research Spec

When the core thesis requires deep retrieval across multiple disciplines or
reasoning types, **a single research subagent cannot complete the work within
the watchdog window**. This spec defines the *5+1+1 multi-agent pattern*: split
the deep-research stage into N parallel Opus subagents, plus a lightweight
hand-assembled integration step, plus an optional external cross-validation
pass.

First successful application: 2026-05-02-accountability-without-identification
(5 sub-args × 6 sources × 250+ citations × 100+ A-grade) in the upstream
monorepo.

## When to use

Trigger conditions (any one):

1. The core thesis's reasoning chain has **≥4 sub-arguments**, and each
   sub-arg lives in a distinct retrieval domain (political philosophy /
   cryptography / law / political economy / public health / standards / etc.)
2. The original thesis includes a "strong counter-argument" or "boundary
   condition" that needs independent rigorous testing
3. Estimated deep research output ≥100KB

Multi-agent mode is *independent* of the no-accent vs accent axis. Default is
no-accent (since 2026-05-03); see [mashbean-accent-opt-in.md](mashbean-accent-opt-in.md).
Multi-agent typically pairs with no-accent (academic articles), but opt-in
accent is allowed (e.g., a personal-voice blog post that happens to need 5
sub-args).

Do not use multi-agent mode for:

- Single-domain blog posts (use the standard single-agent flow)
- Theses with ≤3 sub-arguments
- Citation needs ≤30

## Flow overview

```
Step 1  Reasoning-chain validation (main agent)
        ↓ decomposes into N sub-args (typically 5)
Step 2  Dispatch N parallel Opus sub-arg agents
        ↓ each writes raw/sub-args/sub-arg-N-<topic>.md
Step 3  Integration (main agent does this directly, NOT via subagent)
        ↓ writes raw/deep-research-output.md
Step 4  [Optional] External cross-validation
        ↓ python3 scripts/run_external_deep_research.py <job-id>
        ↓ writes raw/external-deep-research-claude.md
Step 5  Pipeline integrate
        ↓ python3 scripts/run_deep_research.py <job-id> integrate
        ↓ advances to evidence-mapped state
Step 6  Enter the existing pipeline (writer → critic → rewriter → editor)
```

## Step 1 — Reasoning-chain validation

The main agent reads `intake.yaml`, produces a one-page reasoning chain, and
**pauses for user confirmation**. The chain must include:

- Core question (one sentence)
- N sub-args (each with claim, reasoning type D/I/A/Ab/C, reasoning strength,
  the most critical counter-example)
- Explicit cross-level inference flags
- The 1-2 most fragile assumptions

Once confirmed, advance. This step happens entirely in the main conversation;
no subagent.

## Step 2 — Parallel sub-arg agents

Dispatch one independent Opus subagent per sub-arg. **Critical: all N agents
must be launched in the same main message via parallel `Agent` tool calls**;
otherwise they degrade into a serial flow.

Each sub-arg agent's prompt must include:

1. **Task scope**: "You are responsible for Sub-Arg N: <claim>"
2. **Scale requirements**: ≥3000 words (Chinese) or ≥4000 words (English),
   ≥30 sources, ≥10 A-grade
3. **Required A-grade source list**: 5-10 authoritative sources (the main
   agent fills this in advance)
4. **Multilingual requirements**: domain-dependent (political philosophy
   often needs German/French; cryptography is mostly English; law needs
   primary case texts; etc.)
5. **Counter-examples and counter-arguments**: agent must surface
   ≥3 references that could refute the claim
6. **Cross-level inference warnings**: flag any leaps that may not survive
7. **Counterfactuals**: ≥2 well-developed counterfactual scenarios
8. **Output location**: `raw/sub-args/sub-arg-N-<topic>.md`
9. **Output structure**: fixed 8-13 sections (claim restatement / search
   path / source list / reasoning chain / cross-level warnings /
   counterfactuals / open questions / handoff to other sub-args)

Each sub-arg agent takes ~30-60 minutes; run them in the background
(`run_in_background=true`).

### Example sub-arg decomposition

From the article 01 experience:

| Sub-Arg | Domain | Reasoning type | Role |
|---|---|---|---|
| 1 | Political philosophy | D (deduction) | Normative foundation |
| 2 | Cryptographic engineering | I (induction) | Engineering feasibility |
| 3 | Legal precedent | A (analogy) | Institutional template |
| 4 | Causal mechanism / case studies | C (causal) | Reverse validation |
| 5 | Boundary conditions | Ab (abduction) | Self-critique |

You don't have to use 5 sub-args; minimum is 4, maximum is 6 (more makes
integration unmanageable).

## Step 3 — Integration

**Integration is done directly by the main agent, NOT a subagent**. Hard-won
lesson: subagents trying to integrate 5 × 40KB inputs into a 200KB+ output
consistently failed within the 600-second watchdog window.

Main-agent integration workflow:

1. Use `Bash` to extract each sub-arg's "§1 Claim restatement" (~30-50 lines)
2. Use `Bash` to extract each sub-arg's "§ Open questions / interview needs"
3. `Write` a single `raw/deep-research-output.md` containing 8 SECTION markers:
   - `<!-- SECTION: research_summary -->` — 3 paragraphs, including the
     argument upgrades discovered during research
   - `<!-- SECTION: reasoning_chain -->` — uses each sub-arg's *revised*
     claim version
   - `<!-- SECTION: evidence_map -->` — 15+ entries that map cross-sub-arg
     supporting evidence + pointers to the 5 source files
   - `<!-- SECTION: source_registry -->` — 30+ A-grade sources in a
     consolidated list + pointers to original files
   - `<!-- SECTION: case_comparison -->` — cross-sub-arg case comparison
   - `<!-- SECTION: high_risk_claims -->` — consolidated [TODO-VERIFY] list
   - `<!-- SECTION: open_questions -->` — consolidated open-question list
   - `<!-- SECTION: rewrite_warnings -->` — guidance for the downstream writer

Target file size: 50-80KB; do not exceed 100KB (it would crowd out the writer's
context budget). The full 250+ citations live in the 5 sub-arg source files;
the integration file only does *cross-sub-arg indexing*, no full reproduction.

`rewrite_warnings` **must** include the *argument upgrades* discovered during
research (e.g., "pseudonymous vs. real-name identification distinction",
"AML/KYC counter-evidence reverses to support thesis"); without these
explicit notes, the downstream writer tends to revert to the original
`intake.yaml` thesis.

## Step 4 — [Optional] External cross-validation

Independent-path verification. Run:

```bash
python3 scripts/run_external_deep_research.py <job-id>
```

This script uses the Anthropic API directly with Opus 4.7 + adaptive thinking +
`web_search_20260209` + Task Budgets to run an independent deep-research pass.
See `scripts/run_external_deep_research.py` docstring for details. Output:
`raw/external-deep-research-claude.md`.

**Known limitation**: `web_search_20260209` includes built-in dynamic filtering
that uses a server-side Python sandbox. If Claude writes long output to a file
inside that sandbox, the file cannot be exported back to the host filesystem;
only the streamed summary survives. When this happens, treat cross-validation
as a "soft pass" (no contradictions found is enough); do not require a full
11K-word external pack.

## Step 5 — Pipeline integrate

```bash
python3 scripts/run_deep_research.py <job-id> integrate
```

This splits `raw/deep-research-output.md`'s 8 SECTIONs into per-file outputs:

- `notes/research-summary.md`
- `notes/reasoning-chain.md`
- `verification/evidence-map.md` (required)
- `notes/source-registry.md`
- `notes/case-comparison.md`
- `verification/high-risk-claims.md`
- `notes/open-questions.md`
- `notes/rewrite-warnings.md`

State auto-advances to `evidence-mapped` on success.

## Step 6 — Enter the existing pipeline

From here, the flow is identical to the standard single-agent flow:

1. Writer subagent → `drafts/research-draft.md`
2. Critic subagent → `verification/fact-check-report.md`
3. Writer rewrite subagent → `drafts/blog-rewrite.md` (if critic requires)
4. Editor subagent → `final/article-final.md`
5. `run_editorial_pass.py --auto-advance`

By default no accent is applied; `editorial-pass` advances directly to
`ready-to-publish`. If `intake.yaml` has `apply_mashbean_accent: true`, the
flow routes through `accent-pending` → accent subagent. See
[mashbean-accent-opt-in.md](mashbean-accent-opt-in.md).

## Failure modes & recovery

### All sub-arg agents complete but integration stalls

**Symptom**: sub-arg agents succeed; integrator subagent gets killed by the
600-second watchdog with no progress.

**Cause**: integration output volume too large (200KB+); the subagent's stream
breaks during generation.

**Recovery**: main agent does the integration directly via `Bash` extract +
`Write`. Historically takes 5-10 minutes.

### Sub-arg agent's revised thesis conflicts with intake.yaml

**Symptom**: writer drafts using the original `intake.yaml` thesis instead of
the sub-arg agent's revised version (e.g., misses the "pseudonymous vs.
real-name" distinction).

**Cause**: `rewrite_warnings` did not explicitly list the argument upgrades,
or the writer ignored them.

**Recovery**: critic catches it. If critic also misses, editor catches it at
the final structural review.

### Citation errors (publication form / author order / journal vs. book)

**Symptom**: critic catches a `cite` mismatch with `source-registry.md`
(e.g., a paper cited as a book that's actually a journal article).

**Cause**: the source registry was hand-assembled by the main agent during
integration based on partial information; sub-arg source files usually have
the correct citation.

**Recovery**: rewrite stage — the writer cross-checks against the sub-arg
source files and corrects both the citation and `source-registry.md`.

### External cross-validation output lost in sandbox

**Symptom**: `run_external_deep_research.py` reports high `output_tokens`
(e.g., 46K) but the saved `.md` is only a few KB.

**Cause**: Claude wrote a long file inside the `web_search` Python sandbox;
the host filesystem only received the streamed summary.

**Recovery**: extract the framing-level findings from the streamed summary
and add them to the integration file's `research_summary` section (label
them as "external cross-validation"); do not require a full external pack.

## Launch example

```python
# Step 0: standard article-job creation (same as single-agent flow)
python3 scripts/start_article_job.py 2026-MM-DD-<slug> \
  --title "..." \
  --audience "..." \
  --core-question "..." \
  --thesis "..." \
  --notes "multi-agent mode (no-accent default)"

# (intake.yaml defaults to no-accent; add apply_mashbean_accent: true if you
#  want the author voice)

# Step 1: main agent produces reasoning chain, pauses for user confirmation

# Step 2: main agent dispatches 5 parallel sub-arg agents in a single message
# (each prompt fills in the 9 requirements above)

# Step 3: after all 5 complete (background notifications), main agent does
# Bash extract + Write to assemble deep-research-output.md

# Step 4 (optional): external cross-validation
python3 scripts/run_external_deep_research.py 2026-MM-DD-<slug>

# Step 5: advance pipeline
python3 scripts/run_deep_research.py 2026-MM-DD-<slug> integrate

# Step 6+: standard writer → critic → rewrite → editor
# (no-accent by default; editorial-pass auto-advances to ready-to-publish)
```

## Relationship to the existing pipeline

Multi-agent is an *opt-in enhancement*, not a replacement for the standard
single-agent flow. Decision tree:

```
intake.yaml core_question has ≥4 distinct sub-args?
  ├─ yes → multi-agent flow (this spec)
  └─ no  → single-agent flow (CLAUDE.md Steps 3-7)
```

Author-voice is an independent axis (unrelated to multi-agent):

```
intake.yaml apply_mashbean_accent: true?
  ├─ yes → editorial-pass routes to accent-pending → accent subagent
  └─ no  (default) → editorial-pass advances directly to ready-to-publish
```

Both flows merge after Step 5 (integrate) and use the same writer / critic /
editor stages.

## Future extensions

- **`start_multi_agent_research.py` orchestrator script**: scaffolds N
  sub-arg prompt templates so the main agent only needs to fill in
  domain-specific content
- **Dynamic N**: choose 4-6 sub-args based on reasoning-chain complexity
- **Cross-article citation dedup**: when the same author writes a
  multi-article series (e.g., a 19-article dissertation series), share
  the source registry across articles
- **Interview-list aggregation**: aggregate per-article interview needs
  into a single dissertation-level interview roadmap

## Changelog

- 2026-05-02: Initial release. Based on the article 01 successful experience.
- 2026-05-03: Pipeline default flipped to no-accent. Multi-agent flow is now
  independent of the accent axis.
