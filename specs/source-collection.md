# Source Collection Strategy

## Goal

Systematically collect academic literature and real-world cases while avoiding survivorship bias, source homogeneity, and C-grade sources carrying core arguments alone.

## Source Categories

### Academic Sources: A Grade

| Type | Search channels | Signals |
|---|---|---|
| Journal articles | Google Scholar, SSRN, arXiv, JSTOR | peer review, DOI |
| Dissertations | ProQuest, national libraries | advisor, committee |
| Academic books | Google Books, WorldCat | university press or academic press |
| Conference papers | ACM DL, IEEE Xplore, SSRN | acceptance rate, peer review |

Search strategy:

1. Extract 2 to 3 academic keywords from the core question.
2. Search Google Scholar and sort by citation count.
3. Find 1 to 2 highly cited survey or review papers.
4. Snowball through the survey's references.
5. Check whether there is newer research from the last 2 years.

### Policy and Official Documents: A Grade

| Type | Search channels | Signals |
|---|---|---|
| Primary legal text | official legal databases, EUR-Lex | official gazette |
| Government reports | OECD iLibrary, audit reports | government authorship |
| International organizations | UN, World Bank, ITU | institutionally published |
| Audit and oversight | GAO, NAO, ANAO, audit offices | independent audit |

### Real-World Cases: B/C Grade but Necessary

| Type | Search channels | Notes |
|---|---|---|
| Company filings / 990s | SEC EDGAR, ProPublica Nonprofit Explorer | A-grade data when directly cited |
| Annual reports | Organization websites | B-grade, promotional risk |
| Founder or team blogs | Medium, personal sites | C-grade, first-person but biased |
| Media coverage | Major media outlets | C-grade, facts usable but analysis cannot stand alone |
| Community discussions | HN, Reddit, Fediverse | C-grade, useful for atmosphere but not factual grounding |

## Four-Quadrant Collection Method

Every article's sources must cover four quadrants:

```text
                         Academic theory       Real-world cases
Successful/plus          academic framework    successful cases
Failed/minus             critical theory       failed cases
```

Rules:

- Each quadrant should have at least 2 sources when possible.
- Successful cases must be no more than failed cases plus one.
- Each successful case should have at least one failed comparison from the same domain.

## Case Collection SOP

### Step 1: Identify Case Categories

Derive required case types from the core question:

```text
Core question: Can nonprofit social platforms be sustainable?
-> Need: nonprofit platforms, successful and failed
-> Need: cooperative platforms, successful and failed
-> Need: hybrid models, for-profit subsidiary plus nonprofit parent
-> Need: different scales, global vs regional vs community
-> Need: different regions, not only Europe and North America
```

### Step 2: Include At Least One Plus and One Minus Case per Category

| Category | Success | Failure | Comparison value |
|---|---|---|---|
| Nonprofit platform | Wikipedia | Diaspora* | Brand and scale vs no scale |
| Federated platform | Mastodon, surviving | App.net | Open protocol vs closed network |
| Cooperative | Stocksy | Ello | Transactional vs social setting |
| Hybrid model | Mozilla | none yet | Dependency on commercial revenue |

### Step 3: Collect Structured Data for Each Case

```yaml
name: "Mastodon"
type: "nonprofit federated social platform"
founded: 2016
status: "active"
legal_structure: "gGmbH, German nonprofit limited liability company"
revenue_model: ["Patreon donations", "corporate sponsorship"]
annual_revenue: "EUR..."
user_scale: "..."
governance: "BDFL -> limited institutionalization"
key_risk: "founder dependency, Threads competition"
source_grade: "B, official blog, not independently audited"
failure_mode: null  # or "funding depletion" / "governance collapse" / "user churn"
```

### Step 4: Cross-Validate

- Use at least two independent sources for different dimensions of the same case.
- Financial data must come from A-grade sources such as 990s, audit reports, or annual reports when possible.
- User data must include source and access date.

## Search Prompt Templates

### Academic Literature Search

```text
Search: Google Scholar
Query: "[core concept]" governance sustainability nonprofit platform survey
Goal: find review papers and high-citation theoretical anchors.
```

### Real-World Case Search

```text
Search: Web
Query: nonprofit social platform failed funding governance
Goal: identify failed cases for comparison.
```

### Regional Case Search

```text
Search: Web with language or region filters
Query: nonprofit social platform Taiwan OR Japan OR Korea
Goal: find cases outside Europe and North America.
```

## Place in the Pipeline

Source collection happens at two points:

1. `intake -> scoped`: the user provides known sources, and the pipeline identifies needed case quadrants.
2. `scoped -> researching`: the Deep Research prompt includes four-quadrant collection instructions.

Deliverables:

- `notes/source-registry.yaml`, structured source registry
- `notes/case-comparison.md`, plus/minus case comparison table
