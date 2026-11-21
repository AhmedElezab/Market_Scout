# Record schema (single source of truth)

One JSON object per line (JSONL) in `data/processed/`. A pydantic model in `src/` will mirror this
file exactly. **Changing this file requires the owner's approval** (`CLAUDE.md` rule 7).

Rules: every record has `url` and `collected_at`. A missing value is `null`, never a guess. Excerpts
are under 25 words. No contact details. Client names only if publicly shown, and only the company name.

## Fields

| Field | Type | Required | Notes |
|---|---|---|---|
| `id` | string | yes | sha256 of the normalized `url`, first 16 hex chars. Dedupe key #1. |
| `source` | string | yes | Source key from `docs/sources.md` (e.g. `upwork`). |
| `url` | string | yes | Public link to the item. |
| `collected_at` | string (ISO 8601 date-time, UTC) | yes | When we collected it. |
| `posted_at` | string (ISO 8601) or null | no | When the item was posted, if shown. |
| `title` | string | yes | Posting title. |
| `excerpt` | string or null | no | Under 25 words, verbatim. |
| `budget_min` | number or null | no | In `currency`. |
| `budget_max` | number or null | no | In `currency`. |
| `budget_type` | `fixed` \| `hourly` \| null | no | |
| `currency` | string (ISO 4217) or null | no | |
| `proposals_count` | integer or null | no | Competition signal. |
| `design_scope_label` | `design_only` \| `design_plus_impl` \| `impl_only` \| `unclear` | yes | See `docs/taxonomy.md`. |
| `design_scope_note` | string or null | no | Required when label is `design_plus_impl`: which part is design. |
| `topology` | enum or null | no | Values from taxonomy section 2 (now includes `CLLC`, `PFC`, `motor-drive inverter`, `grid-side inverter`). |
| `topology_confidence` | number 0-1 or null | no | Confidence in `topology`. |
| `topology_notes` | string or null | no | Secondary topologies, power-range notes. Never put these in `excerpt`. |
| `content_hash` | string | yes | sha256 of title + excerpt. Dedupe key #2 with `id`. |
| `power_class` | enum or null | no | `<100W`, `100W-1kW`, `1-10kW`, `10-100kW`, `>100kW`. |
| `application` | enum or null | no | Taxonomy section 4. |
| `deliverables` | list of enum | yes (may be empty) | Taxonomy section 5. |
| `device_tech` | list of enum | yes (may be empty) | `Si`, `SiC`, `GaN`. |
| `label_method` | `rules` \| `llm_labeler` \| `llm_escalation` \| `manual` | yes | Which method produced the labels. |
| `confidence` | number 0-1 | yes | Confidence in `design_scope_label`. Below `config.yaml` threshold -> escalate. |

## Remaining proposed additions (not yet approved)

| Field | Type | Why |
|---|---|---|
| `client_public_name` | string or null | Public company name only, for the repeat-clients signal. |
| `remote_ok` | boolean or null | Fit score input. |
| `labeler_model` | string or null | Model ID that labeled the record. |
| `schema_version` | string | Migration support. |

## Raw items

`data/raw/` holds unmodified collected items (gitignored). Each raw item keeps `url`,
`collected_at`, and `source`; the classifier reads raw items and writes processed records.

## Example (illustrative only; not real data)

```json
{"id":"0000000000000000","source":"example","url":"https://example.com/item/1",
 "collected_at":"2026-11-21T00:00:00Z","posted_at":null,"title":"Example title",
 "excerpt":null,"budget_min":null,"budget_max":null,"budget_type":null,"currency":null,
 "proposals_count":null,"design_scope_label":"unclear","design_scope_note":null,
 "topology":null,"power_class":null,"application":null,"deliverables":[],"device_tech":[],
 "label_method":"rules","confidence":0.4}
```

## URL normalization (for `id`)
Lowercase scheme and host, drop `www.`, strip fragments and tracking params (`utm_*`, `ref`, `fbclid`,
`gclid`), remove trailing slash, keep other query params in sorted order.

## Approved 2026-11-21
`content_hash` required; `topology_confidence` and `topology_notes` added; `PFC` topology added.
