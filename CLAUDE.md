# CLAUDE.md: PE Market Scout

## Mission
Find out which **design-only power electronics freelance projects** are most in demand online, then build a tool that keeps that answer current and ranks opportunities by fit with my expertise. The end goal is to decide what to specialize in for an international freelance and consulting practice.

## Owner context
- I am Ahmed, a power electronics engineer (PhD), based in Ontario, Canada.
- Strengths: resonant DC/DC converters (LLC, SRC, DAB), magnetics and planar transformers, PFC stages, control loop compensation (Type II/III), WBG devices (SiC, GaN), ML for converter classification.
- Tools I use: MATLAB, Python, PLECS, Simulink, GitHub. I am still new to Git/GitHub, so keep git steps simple and explain them briefly.
- I am forming a consulting company with two co-founders. Findings may later feed a licensed design tool, so keep code clean and modular.

## What "design-only" means (the classification rule)
Count a project as **design-only** if the deliverable is analysis and documents, not physical hardware or production firmware:
- topology selection and trade study
- component ratings and loss/thermal estimates
- magnetics design (transformer or inductor spec, winding, core selection)
- control design and loop compensation
- simulation models (PLECS, LTspice, Simulink) and design reports
- schematic or design review, BOM review, EMI pre-compliance design guidance

Label every posting with exactly one of:
`design_only` | `design_plus_impl` | `impl_only` | `unclear`

Not design-only: PCB layout for fabrication, firmware coding, board bring-up, prototype build, lab testing, certification testing. If a posting mixes both, label it `design_plus_impl` and note which part is design.

## Hard rules
1. **Confidentiality.** Never put employer (Eaton) material, schematics, or data in this repo, in prompts, or in outputs. Only public data and my own personal-project material.
2. **Respect site terms.** Check robots.txt and terms before collecting from any site. No login-wall scraping, no bypassing anti-bot measures, no fake accounts. Prefer official APIs, search tools, RSS, and pages that are publicly readable. If a source disallows automated access, record it in `docs/sources.md` as "manual only" and use pasted or exported data that I provide.
3. **No automated bidding or messaging.** The tool reads, classifies, and ranks. It never applies to jobs, sends messages, or posts anything.
4. **No invented data.** Every record must carry its source URL and the date collected. If a field (budget, date, client) is missing, store `null`, never a guess.
5. **Personal data.** Do not store client names or personal details beyond what is needed (platform, public company name if shown). No contact details.
6. **Quote sparingly.** Store short excerpts (under 25 words) and a link. Do not copy full posting text into reports.
7. **Ask before** installing heavy dependencies, adding paid APIs, or changing the schema in `docs/schema.md`.

## Subagents, models, and effort
Work is split across specialist subagents so each task runs on the model and effort that fit it. The main session is the **orchestrator**: it plans, delegates, and integrates. It does not do bulk collection or bulk coding itself.

Model IDs: Fable 5.2 = `claude-fable-5-2`, Opus 5.5 = `claude-opus-5-5`, Sonnet 5.5 = `claude-sonnet-5-5`, Haiku 5.5 = `claude-haiku-5-5`.

| Role (file in `.claude/agents/`) | Model | Effort | Used for | Tools |
|---|---|---|---|---|
| orchestrator (main session) | Fable 5.2 | medium (high in Phases 1 and 5) | Planning, delegation, integration, summaries to me | all |
| architect | Fable 5.2 | high | Taxonomy, schema, scoring design, source ToS judgment, hard bugs | read, write docs |
| researcher | Sonnet 5.5 | medium | Web search and fetch, collecting items, source evaluation | read, web, write `data/raw/` |
| builder | Sonnet 5.5 | medium | Collectors, classifier, scorer, reporter, tests | read, edit, bash |
| labeler | Haiku 5.5 | low | High-volume labeling of postings and the LLM classification fallback | read, write `data/processed/` |
| reviewer | Opus 5.5 | high | Independent review of code, data quality, and findings; never edits | read only |
| strategist | Fable 5.2 | xhigh | Phase 5 synthesis and the specialization recommendation | read, write `docs/findings/` |

Rules for using subagents:
1. **Delegate by default.** Anything that reads many pages or files goes to `researcher`. Anything that writes code goes to `builder`. Subagents return short summaries, so the main context stays small.
2. **Independence.** The `reviewer` must not be the model or session that wrote the thing under review. After every phase, run the reviewer on that phase's output and list its findings before moving on.
3. **Parallel work.** Run `researcher` instances in parallel, one per source, when sources are independent. Run only one `builder` at a time on the same files.
4. **Escalate, don't guess.** If `labeler` confidence is below the threshold in `config.yaml`, escalate the item to `builder`/Sonnet 5.5, and the hardest to `architect`. Never force a label.
5. **Effort is a dial, not a default.** Use low or medium for mechanical work, high where a mistake is costly to undo, and xhigh only for the final strategy synthesis or a bug that has survived two attempts. If an agent file cannot set effort, state the effort in the delegation prompt.
6. **Agent files.** Create the role files in `.claude/agents/` during Phase 0, each with a clear `description`, the `model` from the table, and the minimum tools it needs. Confirm in the first session that each agent file loaded and used the intended model.
7. **Runtime models are configuration.** The finished tool's models (for the labeler and fallback) live in `config.yaml` under `models:`, never in code. Defaults: labeler = `claude-haiku-5-5`, escalation = `claude-sonnet-5-5`. Re-measure accuracy on the hand-labeled set before changing them.
8. **Cost awareness.** Log which model handled each task in `docs/findings/` run notes so I can see where usage goes.

## Repo layout
```
pe-market-scout/
  CLAUDE.md
  PLAN.md
  README.md
  .claude/
    agents/           # architect, researcher, builder, labeler, reviewer, strategist
    commands/         # reusable slash commands (e.g. /cycle, /review-phase)
  docs/
    sources.md        # source list: access method, ToS status, notes
    taxonomy.md       # topology / power level / deliverable labels
    schema.md         # record schema (single source of truth)
    findings/         # one dated report per research cycle (YYYY-MM-DD.md)
  data/
    raw/              # unmodified collected items (gitignored if large)
    processed/        # classified records (JSONL)
  src/
    collect/          # one module per source
    classify/         # rules first, LLM second
    score/            # demand and fit scoring
    report/           # markdown and CSV report generation
  tests/
  config.yaml         # sources, keywords, weights, thresholds
```

## Technical standards
- Python 3.11+, type hints, small functions, `pytest` for tests, `ruff` for linting.
- Store records as JSONL, validated with `pydantic`. Schema lives in `docs/schema.md` and mirrors the model in code.
- Configuration in `config.yaml`, never hardcoded. Secrets only in `.env` (gitignored), never committed.
- Classification: deterministic keyword rules first, then an LLM pass only for items the rules cannot settle. Log which method labeled each record.
- Every collector must be rate-limited, cache results, and be safe to re-run (deduplicate by URL and content hash).
- Keep functions pure where possible so the future licensed design tool can reuse them.

## Taxonomy (extend in `docs/taxonomy.md`)
- Topology: LLC, SRC, DAB, PSFB, flyback, forward/ACF, buck/boost, totem-pole PFC, interleaved PFC, 3-phase inverter, grid-tied PV inverter, MPPT, battery charger, other.
- Power class: <100 W, 100 W to 1 kW, 1 to 10 kW, 10 to 100 kW, >100 kW.
- Application: EV charging, solar and storage, server and telecom, aerospace and defense, industrial, medical, consumer.
- Deliverable: topology study, component ratings, magnetics design, loop compensation, simulation model, design review, EMI guidance, other.
- Device tech: Si, SiC, GaN.

## Demand and fit scoring (weights live in `config.yaml`)
- Demand signals: posting frequency per topology and deliverable, repeat clients, budget range, number of proposals (competition), recency.
- Fit score: overlap with my expertise list above, design-only purity, remote-friendly, typical budget versus effort.
- Output the top opportunity clusters with evidence counts, not just a single number.

## The research loop (each cycle)
1. **Collect** (`researcher`, Sonnet 5.5, medium, parallel per source): run collectors for enabled sources and add new items to `data/raw/`.
2. **Classify** (rules first, then `labeler`, Haiku 5.5, low; low-confidence items escalate): label design-only status, topology, power class, application, deliverable.
3. **Score** (`builder` code, `architect` for weight changes): compute demand and fit per cluster.
4. **Report** (orchestrator): write `docs/findings/YYYY-MM-DD.md` with: top 10 clusters, what changed since the last cycle, sample sizes, data gaps, source-quality warnings, and which model handled each step.
5. **Review** (`reviewer`, Opus 5.5, high): check the report for unsupported claims, small samples, and mislabeled items. Fix or flag before it reaches me.
6. **Reflect:** list at most 3 concrete improvements to the tool (new source, better rule, fixed bug), and add them to `PLAN.md`.
7. **Stop and summarize** for me. Wait for my decision before the next cycle unless a scheduled run is configured.

## Working style
- Start every task by restating the goal in one sentence and listing the files you will touch.
- Prefer small commits with clear messages. Commit after each working step.
- Show uncertainty. Say "only N samples" when the sample is small. Never present a trend from fewer than 30 relevant records as a conclusion.
- Run tests and the linter before declaring a step done.
- If a source blocks you or the data is thin, say so and propose an alternative. Do not work around access controls.

## Definition of done for any feature
Works on a real run, has at least one test, is documented in `README.md` or `docs/`, and its output includes source links.
