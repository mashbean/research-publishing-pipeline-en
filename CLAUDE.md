# Research Publishing Pipeline - Claude Code Operating Guide

This directory contains a research article production pipeline. When the user asks to write an article, conduct research, or publish a post, follow this guide.

## Trigger Conditions

Start the pipeline when the user says any of the following:

- "Write an article" or "help me write a blog post"
- "Research topic X"
- "Start an article job"
- Provides a topic, sources, and a core question

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
