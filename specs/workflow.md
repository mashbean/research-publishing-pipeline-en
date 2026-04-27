# Workflow

## Goal

Split research article production into reviewable stages so research, writing, fact-checking, and publishing do not collapse into one opaque task.

## State Machine

```text
intake -> scoped -> researching -> evidence-mapped -> drafted ->
fact-checking -> rewritten -> editorial-pass -> ready-to-publish ->
published -> verified
```

Exception states:

- `blocked`
- `needs-decision`
- `publish-failed`
- `verification-failed`

### Valid Rollback Paths

These rollbacks are expected in production:

- `fact-checking -> researching`: more research is needed after an evidence gap is found.
- `rewritten -> fact-checking`: the rewrite introduced new unverified claims.
- `editorial-pass -> rewritten`: major structure or tone issues remain.

Every rollback must record the reason in `state.json` under `versions[]`.

## Minimum Deliverable by Stage

### intake

- Topic or topic direction
- Core question
- Target format
- Known sources or leads
- Constraints and prohibited patterns

### scoped

- What the article answers
- What the article does not answer
- Working thesis
- High-risk claim list

### researching

- `prompts/deep-research-prompt.md`, generated automatically
- Initial source collection
- Source credibility grading

### evidence-mapped

- `verification/evidence-map.md`, mapping claims to sources
- Core arguments have matching evidence

### drafted

- `drafts/research-draft.md`, a complete research draft
- Report-like prose is acceptable, but citations must not be invented

### fact-checking

- `verification/fact-check-report.md`, a claim-by-claim review
- Citation sanitation
- High-risk claim flags

### rewritten

- `drafts/blog-rewrite.md`, converted into blog prose or the requested format
- Self-citations and prohibited patterns removed

### editorial-pass

- `verification/editorial-pass-report.md`, automated check report
- Text order, headings, opening, and rhythm completed
- No prompt leakage or PDF-summary voice

### ready-to-publish

- `final/*.md`, with complete frontmatter
- Publish checklist passes

### published

- `publish/publish-record.json`, including commit SHA and deploy run
- Commit and push complete
- Deploy started

### verified

- `publish/live-check.json`, HTTP verification record
- Deploy succeeded
- Live URL returns 200
- Live title, lead, and canonical behavior are correct

## Work Layers

### Writing Pipeline

- intake
- scope
- research, including generated Deep Research prompt
- evidence map, including automatic integration of Deep Research output
- draft
- fact-check
- rewrite
- editorial pass, including automated style-rule checks

### Repo Pipeline

- render blog markdown
- write file
- commit and push, then write state back automatically
- watch deploy
- live verify, then write state back automatically

Rule: do not enter the repo pipeline until the writing pipeline is complete.

## Automation Script Map

| State transition | Script | Automation level |
|----------|------|-----------|
| intake -> scoped | `run_pipeline.py auto` | Automatic intake validation |
| scoped -> researching | `run_deep_research.py generate-prompt` | Automatic prompt generation |
| researching -> evidence-mapped | `run_deep_research.py integrate` | Automatic result integration |
| drafted -> fact-checking | `run_pipeline.py next` | Semi-automatic, requires human review |
| rewritten -> editorial-pass | `run_editorial_pass.py --auto-advance` | Automatic check and advance |
| ready-to-publish -> published | `publish_blog_entry.py` | Automatic git and state update |
| published -> verified | `verify_publish.py` | Automatic HTTP and state update |

## Multi-Agent Guidance

- `research`: source collection, credibility grading, evidence map
- `writer`: research draft and blog rewrite
- `critic`: fact-checking, overclaim detection, and citation gaps
- `editor`: title, opening, flow, compression, and tone
- `main`: state advancement, repo operations, publishing, and verification

## Automatic Progress Principle

- Once a research article job starts, it should switch to `activeWork=true`.
- Active tasks should be checked every 10 minutes until the job enters `verified`, `publish-failed`, `verification-failed`, `blocked`, or is explicitly closed.
- If there is no new progress, perform a safe self-push before reporting.
- If there is no meaningful progress for 30 minutes, escalate to a stalled watchdog.
- When stalled, report the blocked reason and recovery action.
- Acceptance target: once the user starts a job, the workflow keeps moving until completion without requiring repeated `please continue` prompts.

## Version Tracking

Record every major revision in `state.json` under `versions[]`, including Deep Research integration, post-fact-check rewrites, and editorial-pass fixes:

```json
{
  "versions": [
    {"note": "Deep Research output integrated", "at": "2026-03-30T14:00:00+08:00"},
    {"note": "post-fact-check rewrite v2", "at": "2026-03-30T16:00:00+08:00"}
  ]
}
```
