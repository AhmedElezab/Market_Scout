# PLAN.md: Where Claude Code starts

Read `CLAUDE.md` first, including the "Subagents, models, and effort" section. Work through the phases in order. Finish each phase's acceptance check before starting the next. After each phase, run the `reviewer` subagent on that phase's output, then stop and give me a short summary.

## Models and effort at a glance
| Phase | Orchestrator (main) | Subagents used (model, effort) |
|---|---|---|
| 0 Setup | Fable 5.2, low | builder (Sonnet 5.5, low) |
| 1 Define problem | Fable 5.2, high | architect (Fable 5.2, high); reviewer (Opus 5.5, high) |
| 2 Manual research pass | Fable 5.2, medium | researcher x N in parallel (Sonnet 5.5, medium); labeler (Haiku 5.5, low); reviewer (Opus 5.5, high) |
| 3 Build tool core | Fable 5.2, medium | builder (Sonnet 5.5, medium); architect for classifier and scoring design (Fable 5.2, high); labeler (Haiku 5.5, low) for the fallback; reviewer (Opus 5.5, high) |
| 4 Make it a loop | Fable 5.2, medium | builder (Sonnet 5.5, medium); reviewer (Opus 5.5, high) |
| 5 Specialization decision | Fable 5.2, high | strategist (Fable 5.2, xhigh); reviewer (Opus 5.5, high) |

## Phase 0: Project setup (about 30 min)
Models: orchestrator Fable 5.2 at low effort; builder Sonnet 5.5 at low effort.

Tasks
1. Create the folder structure from `CLAUDE.md`, `README.md`, `.gitignore` (include `.env`, `data/raw/`, `__pycache__/`), `pyproject.toml`, and `config.yaml` (include a `models:` section for the runtime labeler and escalation models).
2. Create the subagent files in `.claude/agents/` for architect, researcher, builder, labeler, reviewer, and strategist, exactly as specified in the `CLAUDE.md` table (model, tools, a clear `description`). Give `reviewer` read-only tools.
3. Verify each agent: invoke each one with a trivial task and confirm it loads and runs on the intended model. If effort cannot be set in an agent file, record that and pass the effort in each delegation prompt.
4. Initialize git. Explain each git command to me in one line, since I am new to it.
5. Set up `ruff` and `pytest` with one passing placeholder test.

Acceptance: `pytest` and `ruff check` both pass, all six agents load, and the first commit is made.

## Phase 1: Define the problem precisely (no code yet)
Models: orchestrator Fable 5.2 at high; architect Fable 5.2 at high; reviewer Opus 5.5 at high.

Tasks
1. Write `docs/taxonomy.md` from the taxonomy in `CLAUDE.md`, adding keyword lists for each label (for example: "LLC", "resonant", "half-bridge", "magnetics", "transformer design", "loop compensation", "PFC", "SiC", "GaN").
2. Write `docs/schema.md`. Proposed fields: `id`, `source`, `url`, `collected_at`, `posted_at`, `title`, `excerpt`, `budget_min`, `budget_max`, `budget_type` (fixed/hourly), `currency`, `proposals_count`, `design_scope_label`, `topology`, `power_class`, `application`, `deliverables[]`, `device_tech`, `label_method`, `confidence`.
3. Write `docs/sources.md`: a table of candidate sources with access method and ToS status. Candidates to evaluate: Upwork, Freelancer, Fiverr (as demand signal only), Toptal, Contra, Guru, PeoplePerHour, Malt, LinkedIn services and posts, Catalant, specialist consulting networks (for example GLG-style expert networks), engineering Q&A and forums (EEVblog, Power Electronics News, r/PowerElectronics, Stack Exchange EE), university and research-lab collaboration calls, and public RFQ or tender portals. For each, record: public access? API or RSS? robots/ToS allow automated reading? Mark each `automated`, `manual only`, or `skip`.

Acceptance: the reviewer has checked the three docs for gaps and ToS mistakes. I review and approve them. Claude asks me to confirm the design-only rule and the source list before continuing.

## Phase 2: First research cycle by hand (find out what the web actually gives us)
Models: orchestrator Fable 5.2 at medium; researcher Sonnet 5.5 at medium (one instance per source, in parallel); labeler Haiku 5.5 at low; reviewer Opus 5.5 at high.

Tasks
1. Using web search and fetch tools (and Firecrawl if connected), run a manual pass over the `automated`-approved sources, with one `researcher` subagent per source in parallel. Target 100 to 200 relevant items for a first dataset.
2. Save items in `data/raw/` with URL and date. Do not guess missing fields.
3. Classify them by hand-guided rules, noting where rules fail.
4. Write the first report `docs/findings/YYYY-MM-DD.md`: what share of postings is truly design-only, which topologies and deliverables appear most, typical budgets, and how much data each source gave.

Acceptance: report exists with sample sizes. Claude states clearly whether the data is enough to conclude anything. If the design-only share is tiny, say so and suggest how to reframe (for example, searching for "design review", "consulting", "technical advisory", or "feasibility study" wording).

## Phase 3: Build the tool core
Models: orchestrator Fable 5.2 at medium; builder Sonnet 5.5 at medium; architect Fable 5.2 at high for classifier and scoring design and for any bug that survives two attempts; labeler Haiku 5.5 at low for the fallback; reviewer Opus 5.5 at high after each module.

Tasks
0. Before coding the classifier and scorer, have `architect` write a short design note for each (inputs, rules, thresholds, escalation path). Then `builder` implements. Then `reviewer` checks the code against the note.
1. `src/collect/`: a base collector class (rate limiting, caching, dedupe) and one module per approved source. Start with the one or two easiest, most permissive sources.
2. `src/classify/`: rules-based classifier driven by the keyword lists, with an optional LLM fallback behind a config flag. Log method and confidence.
3. `src/score/`: demand and fit scoring with weights in `config.yaml`.
4. `src/report/`: generate the dated markdown report and a CSV of ranked clusters.
5. A single command: `python -m src.run --cycle` that performs collect, classify, score, report.
6. Tests for the classifier (use at least 20 hand-labeled examples from Phase 2 as a test set) and for deduplication.

Acceptance: `python -m src.run --cycle` runs end to end twice with no duplicates on the second run. Classifier accuracy on the hand-labeled set is reported (target 80% or better on design-only versus not). Report accuracy separately for rules-only, Haiku 5.5, and Sonnet 5.5 escalation so I can decide which runtime model to keep.

## Phase 4: Make it a loop
Models: orchestrator Fable 5.2 at medium; builder Sonnet 5.5 at medium; reviewer Opus 5.5 at high.

Tasks
1. Add a `--since` option so each cycle only processes new items and the report shows changes since the last cycle.
2. Add a trend view: counts per topology and deliverable over time, using the dated findings.
3. Add the "Reflect" step output: three suggested improvements appended to a `## Backlog` section in this file.
4. Offer scheduling options and let me choose: a weekly GitHub Actions workflow, a local scheduled task, or manual runs. Do not set up anything that runs unattended without my approval.

Acceptance: two consecutive cycles produce a report that highlights what is new. I choose the schedule.

## Phase 5: Turn findings into a specialization decision
Models: orchestrator Fable 5.2 at high; strategist Fable 5.2 at xhigh; reviewer Opus 5.5 at high.

Tasks
1. Cluster the data into 5 to 10 opportunity clusters (for example "LLC design for EV charger, 3 to 11 kW, SiC").
2. For each: demand evidence, budget range, competition level, and fit with my expertise. Flag gaps where demand is high and my fit is moderate (skills to build) and where fit is high and demand is low (niche or content-marketing play).
3. Draft a one-page positioning summary for each of the top 3 clusters: who buys, what the deliverable looks like, and a suggested service package.
4. Note which clusters would suit the later licensed design tool (repeatable, spec-driven design work).

Acceptance: one-page summary with a recommended top 3 and the evidence behind each. The reviewer has checked every claim against the data and flagged any conclusion drawn from fewer than 30 relevant records.

## Known risks to watch
- Freelance platforms skew toward hardware and firmware work, so design-only postings may be rare there. The real demand may sit in direct consulting inquiries, LinkedIn, and expert networks. If the platform data is thin, report that rather than forcing a conclusion.
- Postings often hide scope. Treat `unclear` as a valid label and track its share.
- Budgets shown online are biased low for expert design work. Treat them as a floor.

## Backlog
(Claude appends improvement ideas here after each cycle.)

## First prompt to give Claude Code
> Read CLAUDE.md and PLAN.md. Start with Phase 0 and Phase 1 only, using the subagents and models in the "Subagents, models, and effort" table. Restate the goal, list the files you will create, and stop after Phase 1 for my review.

Before sending it, start Claude Code on Fable 5.2 (use `/model` to select it and set the effort for the main session). Change the main-session effort between phases as listed in "Models and effort at a glance".
