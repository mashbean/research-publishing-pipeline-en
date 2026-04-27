# research-publishing-pipeline

A semi-automated research article pipeline for moving from topic intake to publication and live verification with Claude Code.

## What It Does

Given a research topic, the pipeline moves through:

```text
intake -> research -> fact-check -> writing -> editing -> publishing -> live verification
```

Each stage is handled by a focused AI subagent, while a state machine manages progress, handoffs, and valid rollback paths.

## Installation

### Option 1: Automatic Installation

```bash
git clone <this-repo-url> /tmp/research-pipeline
cd /tmp/research-pipeline
./setup.sh ~/your-project
```

### Option 2: Manual Installation

```bash
# 1. Copy the pipeline into your project
cp -R research-publishing-pipeline/ ~/your-project/tools/research-publishing-pipeline/

# 2. Install the /article command
mkdir -p ~/your-project/.claude/commands
cp .claude/commands/article.md ~/your-project/.claude/commands/article.md

# 3. Install the Python dependency
pip3 install pyyaml
```

## Usage

After installation, type this in Claude Code:

```text
/article zero-knowledge proof applications for digital identity
```

Or provide more structured parameters:

```text
/article
Title: Governance challenges in public digital infrastructure
Audience: Technology policy researchers
Core question: How can DPI balance efficiency and privacy?
```

### Continue an Unfinished Article

```text
/article continue
```

### Check a Job Status

```bash
cd tools/research-publishing-pipeline
python3 scripts/run_pipeline.py <job-id> status
```

## Pipeline Structure

```text
scripts/        # Automation scripts for state, research, editing, publishing
prompts/        # Subagent instruction templates
specs/          # Workflow, citation, style, and publishing policies
templates/      # Job templates
references/     # Case studies
jobs/           # Article job directories
docs/           # User guides
CLAUDE.md       # Claude Code operating guide
.claude/commands/article.md  # /article command definition
```

## Customization

### Writing Style Rules

Edit `specs/style-policy-en.md` and the patterns in `scripts/run_editorial_pass.py`.

### Agent Behavior

Edit `prompts/agent-*.md`:

- `agent-research.md`: research strategy and source collection
- `agent-writer.md`: writing style, tone, and prohibited patterns
- `agent-critic.md`: fact-checking standards and evidence requirements
- `agent-editor.md`: editing standards and structural review

### Self-Citation Detection

Add your own names or recurring placeholders to `SELF_CITATION_PATTERNS` in `scripts/run_editorial_pass.py`:

```python
(r"yourname,?\s*20\d{2}", "self-citation placeholder"),
```

### Publishing Target

Specify a publishing repository when you create a job:

```bash
python3 scripts/start_article_job.py my-article \
  --publish-repo path/to/blog-repo
```

## Requirements

- Python 3.10+
- PyYAML (`pip install pyyaml`)
- Claude Code CLI or IDE extension
- gh CLI, optional for deploy detection

## State Machine

```text
intake -> scoped -> researching -> evidence-mapped -> drafted ->
fact-checking -> rewritten -> editorial-pass -> ready-to-publish ->
published -> verified
```

Valid rollback paths:

- `fact-checking -> researching`, when more research is needed
- `rewritten -> fact-checking`, when rewrite adds unverified claims
- `editorial-pass -> rewritten`, when major structure or tone issues remain

## License

MIT
