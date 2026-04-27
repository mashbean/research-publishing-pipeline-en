# Publish Policy

## Principle

A push is not completion, and a successful deploy is not final completion. The live URL is the source of truth.

## Publishing Flow

1. Write the final article file.
2. Check frontmatter.
3. Commit.
4. Push.
5. Watch deploy.
6. Verify the live URL.

## Minimum Verification Requirements

### Deploy

- Workflow completed.
- Status is success.

### Live URL

- HTTP 200.
- Title is correct.
- First paragraph or lead is correct.
- Canonical short URL works.

### Path Consistency

- Canonical path works.
- Legacy path, if present, redirects to the canonical path or displays the correct article.
- Verbose long URL behavior is explicit and consistent.

## Failure States

### publish-failed

- Git push failed.
- Workflow failed.
- Build stalled.

### verification-failed

- Live URL returns 404.
- Live title is wrong.
- Old path points to the wrong article.
- Canonical and legacy behavior are inconsistent.

## Definition of Done

An article is complete only when all conditions hold:

- Repository updated.
- Deploy succeeded.
- Canonical live URL content is correct.
- Long or legacy URLs, when present, behave as expected.
