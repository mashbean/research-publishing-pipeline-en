# Research Publishing Pipeline - Claude Code Operating Guide

This directory contains a research article production pipeline. When the user asks to write an article, conduct research, or publish a post, follow this guide.

## Trigger Conditions

Start the pipeline when the user says any of the following:

- "Write an article" or "help me write a blog post"
- "Research topic X"
- "Start an article job"
- Provides a topic, sources, and a core question

## Mode Judgment (decide before Step 0)

**Default behavior since 2026-05-03 = academic / no author-accent.** The
mashbean-accent voice (or any equivalent author-voice skill you've installed)
is now opt-in; it is applied only when `intake.yaml` explicitly enables it.

Two `intake.yaml` flags decide which path the pipeline takes:

| `apply_mashbean_accent` | `core_question` shape | Mode | Spec |
|---|---|---|---|
| `false` or unset (**default**) | ≥4 distinct sub-args | **Academic + multi-agent** (no accent) | [specs/mashbean-accent-opt-in.md](specs/mashbean-accent-opt-in.md) + [specs/multi-agent-deep-research.md](specs/multi-agent-deep-research.md) |
| `false` or unset (**default**) | ≤3 sub-args | **Academic + single-agent** (no accent) | [specs/mashbean-accent-opt-in.md](specs/mashbean-accent-opt-in.md) |
| `true` or `content_goal: personal_blog` | any | **Personal blog (with author voice)** | [specs/mashbean-accent-opt-in.md](specs/mashbean-accent-opt-in.md) §"Opt-in" |

**No-accent default**: the editor agent does not introduce author-voice
lexicon, self-deprecation, parenthetical meta, or cross-domain analogies;
`run_editorial_pass.py` advances directly to `ready-to-publish`.

**Opt-in accent mode**: the editor and writer apply the author-voice skill;
after editorial-pass passes, the flow routes through `accent-pending`
awaiting an accent subagent.

**Multi-agent mode** decomposes Step 2 into 5+1 parallel subagents; the main
agent assembles results directly. Independent of the accent axis.

**Legacy aliases (backward-compat)**: `formal_academic: true` and
`content_goal: academic_paper` are still recognized as no-accent signals
(redundant with the new default; harmless).

## Pipeline Workflow

### Step 0: Create a Job

```bash
python3 scripts/start_article_job.py <job-id> \
  --title "..." --audience "..." --core-question "..." --thesis "..."
```

### Step 1: Advance to Deep Research

```bash
python3 scripts/run_pipeline.py <job-id> auto
```

The pipeline automatically advances from `intake` to `scoped` to `researching`, then stops until Deep Research output is available.

### Step 2: Research with a Subagent

#### 2a. Single-agent flow (standard)

Start a research subagent:

```text
Agent(
  description="Research for article <job-id>",
  subagent_type="general-purpose",
  prompt="You are the research agent. Read the full instructions in <job-dir>/prompts/agent-research.md, then read <job-dir>/intake.yaml and <job-dir>/deep-research-packet.yaml. Run the research task and write results into <job-dir>/verification/ and <job-dir>/notes/. When finished, run: python3 scripts/run_deep_research.py <job-id> integrate"
)
```

If the user already has Deep Research output from ChatGPT or Claude:

```bash
python3 scripts/run_deep_research.py <job-id> save-raw <file>
python3 scripts/run_deep_research.py <job-id> integrate
```

#### 2b. Multi-agent flow (academic + ≥4 sub-args)

Full flow: see [specs/multi-agent-deep-research.md](specs/multi-agent-deep-research.md). Summary:

1. **Dispatch 5 Opus sub-arg agents in parallel** (single message, parallel
   `Agent` calls). Each prompt names a domain, the claim, 5-10 must-have
   A-grade sources, the ≥30 sources / ≥10 A-grade scale target, and the
   output location `raw/sub-args/sub-arg-N-<topic>.md`.
2. **Integration is done by the main agent, NOT a subagent**. Use `Bash` to
   extract each sub-arg's "§1 claim" and "§ open questions", then `Write` a
   single `raw/deep-research-output.md` with 8 SECTION markers. Hard-won
   lesson: subagent integration stalls within the 600s watchdog window.
3. **Optional: external cross-validation**:
   ```bash
   python3 scripts/run_external_deep_research.py <job-id>
   ```
   An independent Anthropic API path (Opus 4.7 + adaptive thinking +
   web_search) for a second opinion.
4. Run `python3 scripts/run_deep_research.py <job-id> integrate` to split
   the 8 SECTIONs into `notes/` + `verification/`.

The integration file's `<!-- SECTION: rewrite_warnings -->` **must** list
the argument upgrades discovered during research (e.g., key conceptual
distinctions, counter-evidence reversals); without them, the downstream
writer tends to revert to the original `intake.yaml` thesis.

### Step 3: Writer with a Subagent

Start a writer subagent to produce the research draft:

```text
Agent(
  description="Write research draft for <job-id>",
  subagent_type="general-purpose",
  prompt="You are the writer agent. Read the full instructions in <job-dir>/prompts/agent-writer.md, then read intake.yaml, evidence-map.md, and source-notes.md inside <job-dir>. Produce drafts/research-draft.md. Language: English."
)
```

Advance the state after completion:

```bash
python3 scripts/update_job_state.py <job-id> --status drafted --last-deliverable drafts/research-draft.md
```

### Step 4: Critic with a Subagent

```text
Agent(
  description="Fact-check draft for <job-id>",
  subagent_type="general-purpose",
  prompt="You are the critic agent. Read the full instructions in <job-dir>/prompts/agent-critic.md, then read drafts/research-draft.md and verification/evidence-map.md inside <job-dir>. Produce verification/fact-check-report.md."
)
```

Follow the critic's decision:

- `Ready for blog rewrite`: run `update_job_state.py --status fact-checking`
- `More research needed`: return to Step 2

### Step 5: Writer Rewrite with a Subagent

```text
Agent(
  description="Blog rewrite for <job-id>",
  subagent_type="general-purpose",
  prompt="You are the writer agent in rewrite mode. Read <job-dir>/prompts/agent-writer.md, then read research-draft.md and fact-check-report.md. Produce drafts/blog-rewrite.md. Focus on converting the research report into readable blog prose and applying all style rules."
)
```

```bash
python3 scripts/update_job_state.py <job-id> --status rewritten --last-deliverable drafts/blog-rewrite.md
```

### Step 6: Editor with a Subagent

```text
Agent(
  description="Editorial pass for <job-id>",
  subagent_type="general-purpose",
  prompt="You are the editor agent. Read the full instructions in <job-dir>/prompts/agent-editor.md, read blog-rewrite.md, run structural review, style-rule checks, and tone calibration, then produce final/article-final.md. When finished, run: python3 scripts/run_editorial_pass.py <job-dir> --auto-advance"
)
```

### Step 6.5: Accent Pass (subagent, **opt-in only**)

> **The default mode does not reach this step**. When `apply_mashbean_accent`
> is unset (the new default), `run_editorial_pass.py` advances directly from
> editorial-pass to `ready-to-publish`. Step 6.5 fires only when
> `intake.yaml` has `apply_mashbean_accent: true` or
> `content_goal: personal_blog`. See [specs/mashbean-accent-opt-in.md](specs/mashbean-accent-opt-in.md).

In opt-in accent mode, after editorial-pass passes, the state advances to
`accent-pending` to await an accent subagent that applies the author-voice
skill before reaching `ready-to-publish`.

### Step 7: Publish and Verify

```bash
python3 scripts/run_pipeline.py <job-id> publish
python3 scripts/run_pipeline.py <job-id> verify
python3 scripts/sync_automation_state.py end "Published: <title>"
```

## Status Checks

Check job status at any time:

```bash
python3 scripts/run_pipeline.py <job-id> status
```

## Subagent Rules

1. Use subagents for research, writer, critic, and editor tasks because they consume substantial context.
2. Keep state management in the main agent. Run `update_job_state.py` and `run_pipeline.py` from the main conversation.
3. Keep publishing operations in the main agent. Git operations require user awareness and should not run inside a subagent.
4. Do not rely on shared memory between subagents. Each subagent must reread the relevant files.
5. Research and writer tasks may run in parallel for different articles, but writer and critic must run sequentially for the same article.
6. **Integration of large subagent outputs is done by the main agent directly, not via a subagent**. When you need to merge several subagent outputs (totalling ≥200KB) into a single file, do not dispatch an integrator subagent — historical lesson: subagents cannot produce 100KB+ single-file output within the 600s watchdog window. Use `Bash` to extract relevant sections from each input, then `Write` directly.

## No-accent default + Multi-agent mode (added 2026-05-03)

Two opt-in pipeline enhancements driven by `intake.yaml` flags:

- **`apply_mashbean_accent: true`** (opt-in, default false): apply the
  author-voice skill; route through `accent-pending`. Output usually targets
  a personal blog. See [specs/mashbean-accent-opt-in.md](specs/mashbean-accent-opt-in.md).
- **Multi-agent mode** (thesis with ≥4 sub-args): Step 2 deep research is
  decomposed into 5 parallel Opus subagents + main-agent integration +
  optional external cross-validation. Independent of the accent axis. See
  [specs/multi-agent-deep-research.md](specs/multi-agent-deep-research.md).

The author-voice skill is no longer the pipeline default. It can still be
used outside the pipeline (e.g., when writing a personal blog post directly
in the main conversation without going through the pipeline).

## Launch example (academic + multi-agent)

```bash
# Step 0
python3 scripts/start_article_job.py 2026-MM-DD-<slug> \
  --title "..." \
  --core-question "..." \
  --thesis "..."

# (intake.yaml defaults to no-accent. Add apply_mashbean_accent: true if you
#  deliberately want the author voice.)

# Step 1: main agent produces a reasoning chain, pauses for user confirmation

# Step 2: main agent dispatches 5 parallel sub-arg agents in a single message
# (each prompt fills in domain + must-have A-grade sources)

# Step 3: after all 5 complete, main agent does Bash extract + Write to
# assemble deep-research-output.md

# Step 4 (optional): external cross-validation
python3 scripts/run_external_deep_research.py 2026-MM-DD-<slug>

# Step 5: advance pipeline
python3 scripts/run_deep_research.py 2026-MM-DD-<slug> integrate

# Step 6+: standard writer → critic → rewrite → editor
# (no-accent default; editorial-pass auto-advances to ready-to-publish)
```
