# Sovico Group — Master Data Architecture

A single-page, self-contained map of how data sits and moves across Sovico Group entities.

**[View the map →](https://chuongh.github.io/master-data-architecture/)**

The page is one large pannable canvas. Drag to pan, <kbd>⌘</kbd>/<kbd>Ctrl</kbd> + scroll to zoom,
<kbd>0</kbd> to fit. Everything — logos, the eight AWS service icons, the brand motif — is inlined,
so `index.html` renders identically offline with no network dependency.

## State: AS-IS

This is the **observed baseline**, not the target. The target architecture will be designed from it
in a later pass.

Each entity card reads top to bottom — **source systems → data lake → lakehouse** (or *data
warehouse*, where the entity runs one) — and the three-segment meter beside its name shows the state
of each tier in that order, so you can see where data stops moving.

| Entity | Region | How far data gets | Evidence |
|---|---|---|---|
| Victoria School | Education | Products on a POC lake | Own inventory |
| VietJet Cargo | Aviation | Sources only — no lake, nothing modelled | Own inventory |
| HDSaison | Finance & Banking | Lakehouse ready | Source deck — **unverified** |
| VietJet Air | Aviation | Lakehouse in progress | Source deck — **unverified** |
| Galaxy | — | Ladder gap — catalog over a relational database | Source deck — **unverified** |
| HDBank | Finance & Banking | Not surveyed | Content to follow |
| Vikki Bank | Finance & Banking | Not surveyed | Content to follow |
| Airport NEO | Aviation | Not surveyed | Content to follow |
| VietJet MRO | Aviation | Not surveyed | Content to follow |
| Victoria Aviation Academy | Education | Not surveyed | Content to follow |

Three coloured region boxes group the business lines — **Aviation** (4) down the left column,
**Finance & Banking** (3) and **Education** (2) on the right. Galaxy sits outside any region.
Region members are drawn adjacent, so membership decides where an entity lands.

A tier can be live while the tier below it is empty. That inversion is the point of the map, not a
rendering fault, so the meter shows each tier's own state rather than one averaged score.

## What is fact, what is not

Two entities are drawn from their own inventories. The other three come from
`master-achitecture.pptx`, slide 2, **and should not be trusted**:

> Slide 2 credits **VietJet Cargo** with Athena, Redshift, SageMaker, Lake Formation and Glue Data
> Catalog. Its own inventory shows **none of them** — one cargo system and a partial SQL replica.
> For the first entity where the deck could be checked against reality, it overstated by five
> services.

- **Confirmed** — Victoria School and VietJet Cargo: tiers, per-system owners and admins, and
  products, all from the entities themselves.
- **Seeded, unconfirmed** — source systems on the three deck-derived entities. The slide names *no*
  source systems at all; these are prompts for the survey and render dashed until confirmed.
- **Unknown** — every hub connection reads *not surveyed*. The slide's arrows describe target state,
  so none is drawn as live.

## Editing

All content lives in one `MODEL` object at the top of the `<script>` block in `index.html`.
Adding an entity, adding a source system, or changing a connection status is a JSON edit — the
layout engine repositions cards, redraws the spokes and recomputes the survey counters on its own.
No HTML or CSS needs to change.

| Field | Values |
|---|---|
| `tiers.<tier>.status` | `live` · `poc` · `partial` · `planned` · `none` · `unknown` |
| `tiers.<tier>.empty` | the one sentence shown when the tier does not exist |
| `tiers.lakehouse.title` | optional name for tier 3, e.g. `Data Warehouse`. Defaults to `Lakehouse` |
| `tiers.sources.items[].confirmed` | `false` → dashed "to confirm"; `true` → solid |
| `tiers.sources.items[].owner` / `.admin` | per system, because ownership differs system by system |
| `link.status` | `unknown` · `manual` · `planned` · `live` |
| `region` | groups consecutive same-column entities in one labelled box, e.g. `Aviation` |
| `placeholder: true` | renders an awaiting-content card — delete the flag to make it a real entity |

Two things are derived rather than declared, so they cannot drift: the **readiness label** is
computed from the tier statuses, and the **AWS badge** appears only if some tier actually references
an AWS service.

## Design rationale

[`docs/superpowers/specs/2026-08-14-sovico-data-platform-as-is-design.md`](docs/superpowers/specs/2026-08-14-sovico-data-platform-as-is-design.md)
— the full extraction from the source deck, the layout decisions and the known risks.
