# Sovico Group — Data Platform AS-IS Map (single-page HTML)

Date: 2026-08-14
Status: approved, ready to implement
Source of truth: `~/Desktop/master-achitecture.pptx` (slide 2)

## Problem

Sovico Group has no single picture of how data actually moves today. The only artefact is one
PowerPoint slide showing a target hub-and-spoke topology: four business units each running an AWS
stack, all pointing into a "Central Governance — Galaxy Global TMO" hub. The slide states the
*intent*; it does not state the *current* state — it names no source systems, no databases, and
says nothing about whether any of those arrows are live.

The goal of this page is the AS-IS view: for every entity in the group, what data sources exist,
whether that entity already has a data lake or is still database-only, and what its actual
connection to the central platform is today. That map becomes the evidence base for designing the
TO-BE architecture in a later pass.

## Measured current state (extracted from the pptx, not guessed)

Slide 2 is 13.333in × 7.5in and contains exactly one diagram. Extracted via `lxml` over
`ppt/slides/slide2.xml`:

| Block | Position (in) | Border | AWS services named |
|---|---|---|---|
| Central Governance / Galaxy Global TMO | 4.44, 2.36 · 3.61×3.47 | `#0000F5` | SageMaker Unified Studio |
| VietJet Air | 0.69, 1.83 · 3.06×2.67 | `#E4803A` | Glue, Lake Formation, Open Table Format, Glue Data Catalog |
| VietJet Cargo | 0.70, 4.63 · 3.06×2.67 | `#55B497` | Athena, Redshift, Lake Formation, Glue Data Catalog, SageMaker |
| HDSaison | 8.75, 1.83 · 2.99×2.79 | `#0000F5` | Lake Formation, Redshift, Athena, Open Table Format, Glue Data Catalog |
| Galaxy | 8.75, 4.80 · 2.99×2.49 | `#BE3C8E` | Lake Formation, Amazon RDS instance, Glue Data Catalog |

Four `rightArrow`/`leftArrow` shapes (`#9AA6B8`) point from the four unit cards into the centre.

Design tokens read off the same XML:

- Font: Montserrat throughout (`Montserrat ExtraBold` for headings)
- Title 20pt `#0000FF` · subtitle 14pt `#5B6478` · service labels 10pt `#5B6478`
- Accent rule `#CF2C91` · card = `roundRect`, white fill, 2pt (`w="25400"`) accent border
- Media: Galaxy Global logo (`image1.svg`), brand diagonal-stripe motif (`image2.png`),
  AWS wordmark badge (`image3.png`), and 8 official AWS service icons as SVG
  (`image4`–`image11`: SageMaker Unified Studio, Glue, Lake Formation, Glue Data Catalog, S3,
  Athena, Redshift, RDS instance)

What the extraction proves, and what the page should say out loud: **AWS Lake Formation and AWS
Glue Data Catalog are the only two services present in all four units.** The governance plane is
the sole thing already standard across the group; storage format and compute are not. VietJet Air
has no consumption service at all; Galaxy has no lake, only an RDS instance; VietJet Cargo and
HDSaison have no Glue ETL; only VietJet Air and HDSaison use an open table format.

## Scope

In scope:

- One self-contained HTML page, one large pannable/zoomable canvas, AS-IS state only
- Hub-and-spoke topology preserved from the slide: two entities left, two right, hub centre
- Four entities from the slide, plus empty dashed placeholder slots for entities not yet surveyed
- Per entity: source systems, platform state (**data lake** if one exists, otherwise **database**),
  consumption services, and the connection status of its spoke into the hub
- All content driven by a single `MODEL` object at the top of the file
- Every fact not present in the pptx is rendered in "to confirm" styling

Out of scope (deliberately):

- The TO-BE architecture. Decided by the user: AS-IS first, design TO-BE from it afterwards.
- Responsive layout and fixed widths. The user plans a very large diagram; the canvas is sized to
  its content and navigated by pan/zoom instead.
- Unit tests. Waived by the user for this single presentational page; verification is visual
  (see below).
- Layered/swimlane presentation. Proposed and rejected by the user in favour of hub-and-spoke.
- Entities beyond the four in the slide (HDBank, Sovico Aviation, Phú Long, Furama). Empty
  placeholder slots stand in for them until surveyed.

## Approach

### Layout

A fixed-size canvas (~1900 × 1650 px, grown by the layout engine as entities are added), holding a
three-column arrangement that mirrors the slide:

```
   left column            centre            right column
   [ VietJet Air ]                          [ HDSaison ]
                     [ CENTRAL HUB ]
   [ VietJet Cargo ]  Galaxy Global TMO     [ Galaxy ]
   [ + to survey ]                          [ + to survey ]
```

The hub is vertically centred against the full stack of entity cards. New entities alternate
left/right and extend the canvas downward.

### Regions

An entity may declare a `region`. Consecutive entities in the same column sharing one region are
wrapped in a single labelled box — a solid neutral border with a light tint and a label reading
`AVIATION · 2 entities`. The border is **solid on purpose**: dashed already means "unconfirmed"
everywhere else on this page, so a dashed container would read as a status rather than a grouping.

The run has to be contiguous within its column; that constraint is what lets the box stay one
rectangle rather than needing a path around scattered members. Regions sit at `z-index: 1`, behind
the spokes and cards, so a spoke leaving a card inside a region crosses the region border cleanly.

One region exists: **Aviation**, holding VietJet Air, VietJet Cargo, Airport NEO and VietJet MRO —
the whole aviation line, not just the two airline entities. Because members must be contiguous in
one column, all four sit in the left column and Victoria School moved to the right.

That leaves the columns very uneven — an Aviation block of roughly 1,800px against 2,700px of
individual entities. **The shorter column is therefore centred vertically against the taller one.**
Top-aligning it left a large gap at the bottom that read as a mistake; centring reads as deliberate.
The shift happens after both columns are placed and moves each card's recorded box as well as its
DOM position, so spokes stay anchored — verified by checking every spoke still starts at its card's
hub-facing edge midpoint after the shift.

### Entity card — the three-tier readiness ladder

Revised 2026-08-14 after the first review. The card no longer lists services by function
(ingestion / storage / governance / consumption); it stacks the three tiers data actually passes
through, so the card answers *where is the data and how far has it got*:

1. **Header** — entity name in the accent colour, the owner line, and a **three-segment readiness
   meter**, one segment per tier in order, each coloured by that tier's own status.
2. **① Source systems** — one panel per system, typed by icon (OLTP, ERP, CRM, file, API, stream).
   Dashed until confirmed.
3. **② Data lake** — the raw landing tier: object storage plus the ingestion mechanism.
4. **③ Lakehouse** — open table format, catalog, governance, query engines, business domains, and
   the **data products** hanging off the end. Tier 3 takes an optional title so an entity can name
   what it actually runs; Victoria School's reads **Data Warehouse**.

Each tier band carries its own status — `Live` · `POC` · `Partial` · `Planned` · `Not built` ·
`Not surveyed` — rendered as a tinted band (teal / blue / amber) or a dashed outline, and its own
owner line.

**A tier that does not exist gets its header and one sentence. Nothing else** — no owner, no list
of what is absent, and on tier 3 no data-products block. An earlier revision filled empty tiers
with a stand-in box, a `Missing` chip row and a callout where the source deck disagreed; it made
the emptiest cards the busiest ones. Anything worth saying about an absent tier fits in the
sentence, and anything that does not fit belongs in the provenance band instead.

**The AWS badge is derived, not declared.** An entity carries it only if some tier actually
references an AWS service key. VietJet Cargo runs SmartKargo and a Microsoft SQL replica, so it has
no badge — and the badge cannot drift as the model changes.

`POC` is distinct from `Partial`: POC means the tier is built and running but not production;
Partial means it is production but incomplete. Victoria School's lake is POC.

Everything inside a tier is a **panel** — a box with an icon, the role it plays, and the service
behind it. Three kinds:

- **Source system panels** carry their own **owner and admin**, because ownership differs system by
  system; a layer-level owner would hide that.
- **Capability panels** in the lake tier: Ingestion, Storage, Governance, Orchestration, Security.
- **Product panels** at the end of the lakehouse tier.

Four services in Victoria School's platform have no icon in the source pptx — ECS, Airflow, KMS and
IAM. They are drawn as tiles in the official AWS category colours (containers orange `ED7100`,
application integration pink `E7157B`, security red `DD344C`) so they sit beside the real icons
without looking foreign.

**The meter is deliberately not one cumulative score.** A tier can be live while the tier below it
is empty, and that inversion is the most valuable thing on the page. The header label names the
pattern instead of averaging it away:

Each status has a rank: `live` = 2, `poc` and `partial` = 1, everything else = 0.

| Condition | Label |
|---|---|
| lake rank 0, lakehouse rank > 0 | `Ladder gap` |
| lake 2 + lakehouse 2 + products | `Products live` |
| lake ≥ 1 + lakehouse ≥ 1 + products | `Products on POC lake` |
| lake 2 + lakehouse 2 | `Lakehouse ready` |
| lake ≥ 1 + lakehouse ≥ 1 | `Lakehouse in progress` |
| lake ≥ 1 | `Lake only` |
| otherwise | `Sources only` |

The gap test comes first deliberately: an upper tier standing on nothing outranks any other reading.

A `Ladder gap` card also renders an explicit magenta flag naming the inversion. One entity hits it:
Galaxy has Lake Formation and Glue Data Catalog registered over a relational database.

Tier assignment for the three remaining deck-derived entities is **our reading** of a flat service
list — the slide does not label tiers. The mapping used: object storage and ingestion → lake; open
table format, catalog, governance and engines → lakehouse. This is stated on the page itself.

### Victoria School — the worked example

The design sample, sourced from the entity's own inventory rather than the deck:

| Tier | Asset | Owner |
|---|---|---|
| Sources | HubSpot (CRM), PowerSchool (SIS), Victoria Portal, Dashboard Postgres | per system: App Owners · System Admins |
| Lake — **POC** | AWS S3 Landing & RAW, Hanoi Local Zone. Five capability panels: Ingestion (ECS), Storage (S3), Governance (Glue/Athena), Orchestration (Airflow), Security (KMS/IAM) | IT / Data Team · Data Engineering · AWS IAM / Data Ops |
| **Data Warehouse** | Victoria CORE & Business Marts · Admissions, Enrollment, Tuition, Feedback | IT / Data Team · Lead Data Architect |
| Products | BOD Executive Dashboard, Admissions Funnel, Enrollment Overview, Tuition Collection | Executive / Operations · BOD / Domain Leads · Dashboard Admin |

The `Data Platform` row of the source table is not a tier of its own — its five capabilities live in
the lake tier as panels, and the platform name renders as a caption on the card header.

Per-system owners currently repeat (`App Owners` / `System Admins` on all four) because the source
table records ownership per layer, not per system. The structure is in place for real values.

Only `BOD Executive Dashboard` is explicitly a dashboard in the source; the other three are typed
`KPI report` from the table's own phrase "executive dashboards, KPI reports, and secure
drill-through APIs". Those drill-through APIs are named nowhere, so none are drawn.

### VietJet Cargo — the worked example for an entity with nothing built

Replaced the deck-derived version on 2026-08-14 with the entity's own inventory:

| Tier | State |
|---|---|
| Sources | **SmartKargo** (cargo management) and a **Microsoft SQL** replica of some of its tables. Both owned by Nelson, administered by Lê Minh Trí, under ICT. |
| Lake | *No lake of any kind.* |
| Lakehouse | *Nothing modelled on top.* |

The replica is a copy rather than a system of record, but it is one of the two places Cargo's data
sits, so it belongs on the source tier — not described inside a lake tier that does not exist.

**This entity is why the rest of the map should be distrusted.** Slide 2 credits VietJet Cargo with
Athena, Redshift, SageMaker, Lake Formation and Glue Data Catalog. Its own inventory has none of
them. For the first entity where a real inventory could be compared against the deck, the deck
overstated by five services. VietJet Air, HDSaison and Galaxy are still drawn from that deck and
must be treated as unverified until their owners confirm what runs. The provenance band says so.

### Spokes

One edge per entity from the card's hub-facing edge to the hub, drawn as a cubic bezier in an SVG
layer whose coordinates are computed from live element positions after layout, so edges stay
correct at any canvas size. Four line styles carry the connection status:

| Style | Meaning |
|---|---|
| Solid, accent colour | Automated and live |
| Dashed | Exists but manual / batch / one-way |
| Dotted magenta `#CF2C91` | Does not exist yet — belongs to TO-BE |
| Grey + `?` | Not yet surveyed |

All four spokes ship as **not surveyed**. The slide's arrows describe target state, not observed
state; claiming them as live would be inventing a fact. The page says so in a footnote.

### Seeded source systems

Seeded by industry so the user edits rather than types from scratch; every one carries the
*to confirm* treatment (dashed border, muted text):

- VietJet Air — PSS / Reservation, Departure Control, Loyalty, Web & Mobile clickstream, Revenue accounting
- VietJet Cargo — Cargo booking, AWB / Manifest, Ground handling, Interline partner feeds
- HDSaison — Core lending, Loan origination, Collection & recovery, CRM, Credit bureau feed
- Galaxy — ERP, CRM, Ticketing / POS, Content & media metadata

### Data model

Everything renders from one `const MODEL = { meta, hub, entities: [...] }` block at the top of the
file. Adding an entity, a source system, or changing a connection status is a JSON edit; no HTML or
CSS is touched. This is the property that makes the page survive many rounds of survey input.

### Assets and self-containment

The eight AWS service icons are inlined as SVG `<symbol>`s taken verbatim from the pptx; the Galaxy
logo, AWS wordmark and brand stripe motif are inlined as SVG / base64 data URIs. The page opens
offline with no network fetch. Montserrat is loaded from Google Fonts with a full local fallback
stack, so the page degrades to Helvetica/Arial rather than breaking when offline.

### Interaction

Drag to pan, `+` / `−` / `0` and on-screen buttons to zoom, a **Fit** button to frame the whole
canvas. Hovering an entity dims the others and highlights its spoke. A survey-progress readout
reports how many connections are still unconfirmed.

## How to verify

No unit tests — waived by the user, and the deliverable is a single presentational page with no
extractable logic beyond layout. Verification is therefore visual and structural:

1. Open the file in the in-app browser at full canvas and at Fit zoom.
2. Confirm against this document: 4 entity cards present with the correct accent colours; every
   AWS service from the extraction table appears on the right card and no service appears that the
   extraction did not name; hub shows SageMaker Unified Studio.
3. Confirm all four spokes render in the *not surveyed* style and the survey readout says 0 of 4
   confirmed.
4. Check for the defects that actually occur: text overflowing a card, spokes crossing through
   cards, edges detached from their anchors after zoom, placeholder slots rendering as if they
   were real entities.
5. Read the browser console — zero errors.

## Publication

Published 2026-08-14 to `chuongh/master-data-architecture`, served by GitHub Pages at
<https://chuongh.github.io/master-data-architecture/>. The map is the repository root
`index.html`.

The repository was made **public**, which is a deliberate trade rather than an oversight: the
account is on the free plan, where GitHub Pages is only available for public repositories.
Upgrading would not have changed the exposure — a Pages site is publicly reachable regardless of
repository visibility, and access-controlled Pages is an Enterprise Cloud feature.

To limit reach, `index.html` carries `<meta name="robots" content="noindex, nofollow, noarchive,
nosnippet">`. That tag is the mechanism that works. A `robots.txt` is committed as well, but for a
project page it resolves under `/master-data-architecture/` and crawlers only read the domain root
(`chuongh.github.io/robots.txt`, owned by a separate repository), so it has no effect today.

**The residual exposure is real and should be understood by anyone extending this page:** the
content is reachable by anyone holding the URL. It names four Sovico entities, states each one's
data platform maturity, and asserts that only the governance plane is standardised group-wide.
Before adding genuinely confidential material — real source system names, account IDs, volumes,
vendor contracts — either move the repository back to private and share `index.html` as a file, or
split the sensitive detail into a separate unpublished model.

## Known risks

- **Seeded source systems could be mistaken for surveyed fact.** Mitigated by the dashed *to
  confirm* treatment on every seeded chip plus a legend entry, but the risk is real if someone
  screenshots one card out of context. If the user prefers, the seed can be emptied in one edit.
- **The pptx is the only source.** Where the slide is silent — VietJet Cargo's storage format, every
  entity's ingestion mechanism — the page shows an explicit unknown rather than a guess. Those
  unknowns are the survey backlog, not a rendering defect.
- **Bezier spokes can cross entity cards** once many entities are added down the canvas. Acceptable
  at six slots; if the canvas grows past that, spoke routing needs revisiting.
- **Google Fonts is a network dependency** for exact brand typography. The fallback stack keeps the
  page usable offline but metrics will shift slightly.
