# Author-Voice (`mashbean-accent`): Opt-in Spec (since 2026-05-03)

> **Note for English-speaking users**: `mashbean-accent` is the original author's
> personal blog voice (idiosyncratic Chinese expressions, self-deprecation,
> parenthetical asides, cross-domain analogies). It is **not relevant for English
> articles** unless you have built an equivalent author-voice skill. This spec
> mainly documents how the pipeline decides whether to apply *any* author-voice
> stylization vs. produce neutral academic prose. The opt-in flag is named after
> the original skill but the mechanism is general.

Starting 2026-05-03, **the research-publishing-pipeline default voice is
academic / no author-accent**. Author-accent is opt-in; it is applied only when
`intake.yaml` explicitly enables it.

Historical context: article 01 ([2026-05-02-accountability-without-identification]
in the upstream monorepo) used `formal_academic: true` to opt out of accent.
After running it, the practical pattern was clear — academic prose is the
dominant case across pipeline output (research articles, policy briefs, doctoral
chapters); accent is the minority. So the default flipped.

## Default mode (no accent)

No flag needed. Default pipeline behavior:

- **Writer**: produces academic structure (context → core question → roadmap →
  body → conditional conclusion)
- **Editor**: skips the accent calibration step entirely
- **Does not introduce**: author lexicon, self-deprecation, parenthetical meta
  remarks, cross-domain analogies
- **Conclusion**: written as a conditional academic close, not a "looking-back
  emotional wrap"
- **Opening**: written as a paper introduction, not a personal scene anchor
- After `run_editorial_pass.py` passes: state advances directly to
  `ready-to-publish`, skipping `accent-pending`

Use cases:

- Doctoral dissertation chapters
- Research reports for `pro.<your-domain>` blogs
- Long-form articles with high citation density (≥30)
- Policy analysis, industry observation, technical evaluation

## Opt-in mode (`apply_mashbean_accent`)

Set *either* of these flags in `intake.yaml`:

```yaml
apply_mashbean_accent: true
# or
content_goal: personal_blog
```

Behavior when enabled:

- **Writer**: applies the author-voice skill (default location:
  `~/.claude/skills/mashbean-accent/`), targeting the skill's nine-item
  self-check list (≥1 hit minimum; the rest is left to the accent-pass)
- **Editor**: runs the full mashbean-accent calibration section
- Opening uses scene anchoring; ending uses an emotional wrap; lexicon items
  distributed naturally
- After `run_editorial_pass.py` passes: state advances to `accent-pending`,
  awaiting the accent subagent to finish polishing

Use cases:

- Personal blog articles (lived experience, reflections, narrative essays)
- Audiences who expect the author's recognizable voice
- Hybrid posts that mix technical content with personal voice

## Flag precedence

| `intake.yaml` setting | Actual behavior | Notes |
|---|---|---|
| Nothing set / default | No accent | New default since 2026-05-03 |
| `apply_mashbean_accent: true` | Accent | Explicit opt-in |
| `content_goal: personal_blog` | Accent | Alias |
| `formal_academic: true` | No accent | Legacy alias (matches default; harmless) |
| `content_goal: academic_paper` | No accent | Legacy alias (matches default) |
| Both `apply_mashbean_accent: true` and `formal_academic: true` | Accent | Explicit opt-in beats legacy alias |

## Implementation

| File | Where the change lives |
|---|---|
| `scripts/run_editorial_pass.py` | ~line 390-420: reads `apply_mashbean_accent`, advances state accordingly |
| `prompts/agent-editor.md` | Mode-judgment block at the top |
| `prompts/agent-writer.md` | Mode-judgment block at the top + branched task lists for default vs accent |
| `prompts/agent-accent.md` | Accent subagent's own rules (only triggered in opt-in mode; content unchanged) |
| `scripts/lib.py` | State machine still includes `accent-pending`, used only in opt-in mode |
| `scripts/run_pipeline.py` | `step_accent_pass` still exists, only fires in opt-in mode |

## Testing

Verify default behavior (should not route through accent):

```bash
# Create a test job; intake.yaml does NOT include apply_mashbean_accent
python3 scripts/start_article_job.py 2026-05-03-test-default \
  --title "..." --core-question "..." --thesis "..."
# ... run through to editorial-pass ...
python3 scripts/run_editorial_pass.py jobs/2026-05-03-test-default --auto-advance
# Expected: state advanced to ready-to-publish (no-accent default)
```

Verify opt-in (should route to `accent-pending`):

```yaml
# intake.yaml
apply_mashbean_accent: true
```

```bash
python3 scripts/run_editorial_pass.py jobs/2026-05-03-test-accent --auto-advance
# Expected: state advanced to accent-pending (apply_mashbean_accent=true)
```

## Suggested mapping for English forks

If you adapt this pipeline for English-language work and a different author voice:

1. Replace the `mashbean-accent` skill with your own at `~/.claude/skills/<your-skill>/`
2. Either rename the flag (`apply_<your-skill>: true`) by editing
   `scripts/run_editorial_pass.py` and the writer/editor prompts, **or** keep
   the existing `apply_mashbean_accent` flag name as a generic "apply
   author-voice" toggle
3. Update `prompts/agent-writer.md` and `prompts/agent-editor.md` to reference
   your skill path instead of `mashbean-accent`
4. Most users won't need this — the default no-accent academic mode handles
   the typical research-article use case without any per-author customization

## Recommendation for downstream users

Most pipeline jobs do not need to set `apply_mashbean_accent`. The default
no-accent mode is what you want for research articles, policy reports, and
academic-style writing.

Only opt in when:

- The article is a personal blog post (lived-experience reflections, event
  recap, free-form essay)
- You deliberately want the author's recognizable voice rather than neutral
  analysis
- The article targets a personal-blog destination rather than a research site

The author-voice skill itself (e.g., `mashbean-accent`) remains usable outside
the pipeline — for instance, when you write a quick personal blog post in the
main conversation without going through the pipeline at all.

## Changelog

- **2026-05-03**: Inverted the default. Previously `formal_academic: true` was
  opt-in to skip accent. Now `apply_mashbean_accent: true` is opt-in to apply
  accent. Legacy `formal_academic` / `content_goal: academic_paper` flags are
  still recognized (they match the new default; harmless).
- **2026-05-02**: Initial version. At that time `formal_academic: true` was
  opt-in to skip accent; default still went through mashbean-accent.
