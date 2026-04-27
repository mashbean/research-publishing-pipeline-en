# Launch Guide: Chat / Cowork / Code

The pipeline can be used across three environments, each with a different role.

## Short-Term Setup

### Chat: claude.ai Conversation Mode

**Best for**: exploring topics, initial research, and interacting with Deep Research.

**How to start**:

1. Paste your topic and questions into Chat.
2. Ask Claude to produce structured research notes using `deep-research-packet.yaml` as the reference format.
3. Save the research output as a `.md` file.

**Example prompt**:

```text
I want to write an English blog post about the sustainability of nonprofit platforms.
Core question: Can nonprofit social platforms survive over the long term?
Please research:
1. Five relevant cases, including at least two failures.
2. Revenue structure and governance model for each case.
3. Claims that need stronger sources.
Output format: evidence_map + source_notes + high_risk_claims
```

**Bring the output back into the pipeline**:

```bash
# Save the Chat research output
pbpaste > raw/deep-research-output.md  # macOS
python3 scripts/run_deep_research.py <job-id> save-raw raw/deep-research-output.md
python3 scripts/run_deep_research.py <job-id> integrate
```

### Cowork: claude.ai Artifacts

**Best for**: co-writing, live editing, and discussing rewrite direction.

**How to start**:

1. Open an Artifact in Cowork.
2. Paste `drafts/research-draft.md` or `drafts/blog-rewrite.md`.
3. Work with Claude on the rewrite.

**Example prompt**:

```text
This is my research draft in the Artifact.
Please turn it into blog prose and follow these rules:
- Avoid repetitive not-X-but-Y framing.
- Avoid report-like scaffolding.
- Avoid colon overuse in body prose.
- Make the opening tell readers what question the article answers.
```

**Bring the output back into the pipeline**:

```bash
# Save the revised Cowork article
pbpaste > drafts/blog-rewrite.md
python3 scripts/update_job_state.py <job-id> --status rewritten \
  --last-deliverable drafts/blog-rewrite.md \
  --add-version-note "Cowork rewrite completed"
```

### Code: Claude Code CLI

**Best for**: full workflow automation, subagent dispatch, publishing, and verification.

**How to start**:

```bash
cd tools/research-publishing-pipeline

# Create a job and advance it automatically
python3 scripts/start_article_job.py 2026-04-01-my-topic \
  --title "Article title" \
  --audience "Readers" \
  --core-question "Core question" \
  --thesis "Working thesis"

python3 scripts/run_pipeline.py 2026-04-01-my-topic auto
```

Claude Code then runs the research, draft, fact-check, rewrite, and editorial-pass stages with subagents.

When the pipeline stops for human input, review the relevant output and continue:

```bash
python3 scripts/run_pipeline.py 2026-04-01-my-topic auto
```

## Medium-Term Setup

### Full Subagent Flow in Code

In Claude Code, say:

```text
Help me write a blog post about whether nonprofit platforms can survive.
The core question is whether nonprofit social platforms can operate sustainably.
The audience is people interested in digital governance.
Use research-publishing-pipeline to run the full workflow.
```

Claude Code will:

1. Read `CLAUDE.md` for the workflow.
2. Create a job.
3. Dispatch research, writer, critic, and editor subagents.
4. Pause after fact-checking for confirmation.
5. Publish and verify after confirmation.

### Cross-Environment Workflow

```text
Chat           Cowork          Code
 |               |               |
 +-- explore     |               |
 +-- research    |               |
 |               |               |
 |    save-raw ----------------->+-- integrate
 |               |               +-- auto (draft -> fact-check)
 |               |               |
 |               +-- rewrite <---+  (export blog-rewrite.md)
 |               +-- revise      |
 |               |               |
 |    pbpaste ------------------>+-- editorial pass
 |               |               +-- publish
 |               |               +-- verify
 |               |               +-- done
```

**Principles**:

- Chat handles exploration and Deep Research.
- Cowork handles co-writing and live editing.
- Code handles state management, quality checks, publishing, and verification.

## Quick Comparison

| Action | Chat | Cowork | Code |
|------|------|--------|------|
| Explore topic | Best | Usable | Too heavy |
| Deep Research | Best | Poor fit | Possible with subagent |
| Co-write article | Usable | Best | Possible with subagent |
| Style-rule checks | Manual | Manual | Automated |
| Fact-check | Usable | Usable | Subagent |
| Publish | Not available | Not available | Primary path |
| Verify | Not available | Not available | Automated |
| State management | Not available | Not available | Automated |
