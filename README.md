# Sovico Group — Master Data Architecture

A single-page, self-contained map of how data sits and moves across Sovico Group entities.

**[View the map →](https://chuongh.github.io/master-data-architecture/)**

The page is one large pannable canvas. Drag to pan, <kbd>⌘</kbd>/<kbd>Ctrl</kbd> + scroll to zoom,
<kbd>0</kbd> to fit. Everything — logos, the eight AWS service icons, the brand motif — is inlined,
so `index.html` renders identically offline with no network dependency.

## State: AS-IS

This is the **observed baseline**, not the target. The target architecture will be designed from it
in a later pass.

| Entity | Platform today | Services named in the source |
|---|---|---|
| VietJet Air | Data lake | Glue, Lake Formation, Open table format, Glue Data Catalog |
| VietJet Cargo | Warehouse-led | Lake Formation, Glue Data Catalog, Athena, Redshift, SageMaker |
| HDSaison | Data lake | Lake Formation, Glue Data Catalog, Athena, Redshift, Open table format |
| Galaxy | Database only | Lake Formation, Glue Data Catalog, Amazon RDS instance |

**Lake Formation and Glue Data Catalog are the only two services present in all four entities.**
The governance plane is already standard across the group; storage format, ingestion and consumption
are not. That gap is the argument for the central governance hub.

## What is fact, what is not

Everything is labelled on the page itself, but in short:

- **Fact** — entities, AWS services and the hub-and-spoke topology, extracted from
  `master-achitecture.pptx`, slide 2.
- **Seeded, unconfirmed** — every source system chip. The source slide names *no* source systems at
  all; these are seeded by industry as prompts for the survey. They render dashed until confirmed.
- **Unknown** — all four hub connections read *not surveyed*. The slide's arrows describe target
  state, so none is drawn as live. Ingestion is unidentified for three of the four entities.

## Editing

All content lives in one `MODEL` object at the top of the `<script>` block in `index.html`.
Adding an entity, adding a source system, or changing a connection status is a JSON edit — the
layout engine repositions cards, redraws the spokes and recomputes the survey counters on its own.
No HTML or CSS needs to change.

| Field | Values |
|---|---|
| `sources[].confirmed` | `false` → dashed "to confirm"; `true` → solid |
| `link.status` | `unknown` · `manual` · `planned` · `live` |
| `platform.kind` | `lake` · `db` · `unknown` |
| `placeholder: true` | renders an empty "to be surveyed" slot — delete the flag to make it a real entity |

## Design rationale

[`docs/superpowers/specs/2026-08-14-sovico-data-platform-as-is-design.md`](docs/superpowers/specs/2026-08-14-sovico-data-platform-as-is-design.md)
— the full extraction from the source deck, the layout decisions and the known risks.
