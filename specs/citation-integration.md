# Citation Integration Policy

## Goal

A citation is not just an attached source. It should advance the argument and answer an implied reader question.

## Three Citation Functions

### 1. Factual Anchor

Use this to establish a hard factual baseline.

```text
Good: The Wikimedia Foundation FY2024-25 audit report shows annual revenue of $208.6 million, with more than 85% from individual donations.
```

Rules:

- Use A-grade sources.
- Numbers must include year and source.
- Currency must be explicit, such as USD, EUR, or TWD.

### 2. Argument Support

Use this to support an inference.

```text
Good: Elinor Ostrom's 1990 work shows that durable commons governance depends on clear boundary rules, collective decision-making, and conflict-resolution procedures. Those conditions also matter for digital public goods governance.
```

Rules:

- State the argument first, then bring in the source.
- Label the reasoning type. The example above is analogy, because it moves from natural commons to digital public goods.
- If the analogy crosses domains, state the limits.

### 3. Tension Display

Use this to show a live disagreement and make the article deeper.

```text
Good: Doctorow treats enshittification as a predictable outcome of platform business models, while Benkler's commons-based peer production theory argues that non-market collaboration can route around that trap. The disagreement is whether nonprofit structure can really insulate a platform from market pressure.
```

Rules:

- Present at least two views.
- Identify the point of disagreement.
- You do not need to resolve the disagreement inside the citation sentence.

## Citation Format

### Citations in Body Prose

Use a source description plus key evidence. Avoid academic footnote numbering in the body:

```text
Good: ProPublica's public Form 990 record shows that Signal Foundation's net assets fell by $4.7 million in FY2024, with annual expenses of about $8.6 million.

Bad: Signal Foundation's net assets fell by $4.7 million[1].
Bad: According to research (Signal Foundation, 2024), net assets fell.
```

### Source Notes Section

Use this format at the end of the article:

```markdown
## References

### Academic Literature

- Ostrom, Elinor (1990). *Governing the Commons*. Cambridge University Press.
- Scholz, Trebor (2016). "Platform Cooperativism." *Rosa Luxemburg Stiftung*.
  https://scholarworks.umb.edu/...

### Official Documents and Financial Records

- Wikimedia Foundation. "FY2024-25 Audit Report." wikimediafoundation.org
- Signal Foundation. "Form 990 (FY2024)." via ProPublica Nonprofit Explorer.

### Cases and Reporting

- Mastodon gGmbH. "2024 Annual Report." blog.joinmastodon.org. Accessed 2026-03-30.
- Doctorow, Cory (2023). "Enshittification." pluralistic.net.
```

### URL Handling

- A-grade and B-grade sources: provide a full URL or DOI.
- 990 filings: mark as via ProPublica Nonprofit Explorer.
- Blogs and media: include access date because pages can disappear.
- Academic papers: prefer DOI, then arXiv ID.

## Citation Density

| Paragraph type | Citation density | Notes |
|---------|---------|------|
| Factual statement | High, one source every 1 to 2 sentences | Numbers, years, and amounts need sources |
| Analytical argument | Medium, 1 to 2 sources per paragraph | Sources support inference rather than every sentence |
| Synthetic commentary | Low, one source near the opening or close | This is the author's reasoning |
| Opening or ending | Very low | May be uncited when it is clearly the author's framing |

### Anti-Patterns

```text
Bad: citing every sentence, which makes the article read like a paper.
Bad: making many factual claims in a paragraph with no citations.
Bad: source piling, where readers cannot tell who said what.
```

## Handling Citation Conflicts

When two sources disagree about the same fact:

1. For number conflicts, prefer A-grade sources. If both are A-grade, use the newer source.
2. For viewpoint conflicts, present both sides and explain the disagreement.
3. For methodology conflicts, explain how each source measured the thing.

```text
Good: FediDB estimated Mastodon monthly active users at about 1 million in mid-2024, while Mastodon's official blog cited about 1.5 million. The gap likely comes from different definitions of active use, since FediDB counts accounts with activity in the previous 30 days.
```

## Place in the Pipeline

Citation integration happens in three stages:

1. `evidence-mapped`: every claim is paired with a source in `evidence-map.md`.
2. `drafted`: the writer embeds citations into the research draft.
3. `fact-checking`: the critic checks whether citations are used accurately and not overextended.

Deliverables:

- The `References` section at the end of `final/*.md`
- Access dates in the source field for every claim in the evidence map
