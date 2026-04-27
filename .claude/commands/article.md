---
description: Start or continue an article job using the research publishing pipeline
allowed-tools: Agent, Bash, Read, Write, Edit, Glob, Grep, TodoWrite
---

# Article Pipeline Command

The user wants to start or continue a research article.

## Instructions

Find the pipeline root directory, which is the `research-publishing-pipeline/` folder containing `CLAUDE.md`, then read `CLAUDE.md` for the full workflow.

The pipeline may be in one of these locations:

- `tools/research-publishing-pipeline/`
- The project root, if this repository is the pipeline

## If the User Provides a Topic for a New Article

1. Extract `title`, `audience`, `core_question`, and `thesis` from the user's message.
2. Generate a job ID in the format `YYYY-MM-DD-topic-slug`.
3. Enter the pipeline directory and run:

```bash
python3 scripts/start_article_job.py <job-id> \
  --title "<title>" \
  --audience "<audience>" \
  --core-question "<question>" \
  --thesis "<thesis>"
python3 scripts/run_pipeline.py <job-id> auto
```

4. Follow `CLAUDE.md` to dispatch research, writer, critic, and editor subagents.
5. Pause at steps that require human confirmation.

## If the User Wants to Continue an Existing Article

1. Find the newest job:

```bash
ls -t <pipeline-dir>/jobs/ | head -5
```

2. Check its status:

```bash
python3 scripts/run_pipeline.py <job-id> status
```

3. Continue from the current state.

## User Input

$ARGUMENTS
