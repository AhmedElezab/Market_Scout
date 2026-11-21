# Taxonomy

Single reference for labels and keyword lists. Used by the rules classifier first, then the LLM
fallback (see `CLAUDE.md`). Keyword lists are **starting points**; Phase 2 will show where they fail.
Matching is case-insensitive, on title + excerpt. **Every keyword of 4 characters or fewer** (SiC, GaN,
LLC, DAB, SRC, PSFB, PV, UPS, OBC, BOM, NPC, POL, OCP, DAB, MPPT) must match on word boundaries, so
"SiC" never hits "basic" and "GaN" never hits "began". LLC, SRC and DAB also need a power-electronics
context word nearby ("LLC" is also a company type). Negations ("no PCB layout", "not required",
"without firmware") cancel the signal that follows them within the same clause. Signals found under
headings like "Requirements" or "Nice to have" are weaker than those in the title or scope text.

## 1. Design scope label (exactly one per posting)

| Label | Meaning |
|---|---|
| `design_only` | Deliverable is analysis, models, specs, or documents. No physical hardware, no production firmware. |
| `design_plus_impl` | Mixes design with build/firmware/test. Record which part is design in `design_scope_note`. |
| `impl_only` | Pure implementation: PCB layout, firmware, bring-up, prototype build, lab or certification testing. |
| `unclear` | Scope cannot be determined from the text. A valid label; track its share. Never force a label. |

### Decision procedure (rules classifier)
1. Find **design signals** (list D) and **implementation signals** (list I) in title + excerpt.
2. D present, I absent -> `design_only`.
3. I present, D absent -> `impl_only`.
4. Both present (after negation handling) -> `design_plus_impl`; store the design part in `design_scope_note`.
5. Neither present, or only generic words ("power supply", "engineer needed") -> `unclear`.
6. Ambiguous words (see below) never decide on their own. Bare "design" plus a topology (for example
   "LLC design needed") has no scope signal, so it stays `unclear` with low confidence until D/I
   signals appear.
7. Rules `confidence` (proposal for the architect to set in Phase 3): 0.9 for D-only or I-only with
   2+ distinct signals, 0.75 with one, 0.6 for both-present mixes, 0.4 or lower for `unclear`.

### List D: design-only signals
topology selection, topology study, trade study, trade-off analysis, feasibility study, power-stage architecture,
component selection, component ratings, stress analysis, derating, loss estimate, loss analysis,
efficiency estimate, thermal estimate, thermal simulation, magnetics design, transformer design,
inductor design, planar transformer, winding design, core selection, litz, loop compensation,
loop design, Type II, Type III, small-signal model, Bode, stability analysis, PLECS, LTspice,
Simulink, PSIM, simulation model, average model, design report, design calculations, design review,
schematic review, BOM review, EMI pre-compliance, EMI filter design, CM choke design, reference
design review, spec development, technical advisory, white paper, mathcad, MATLAB script.

### List I: implementation signals (not design-only)
PCB layout, PCB design, Altium, KiCad, schematic capture, Gerber, layout for fabrication, PCB routing, firmware, embedded C, DSP code, STM32, TI C2000
code, FPGA coding, bring-up, prototype build, board assembly, soldering, hardware testing, lab testing,
bench testing, oscilloscope, debug on bench, certification testing, UL testing, CE testing, FCC testing, EMC compliance testing, production build,
manufacturing support, DFM, test fixture, wire harness, enclosure, mechanical design.

### Ambiguous words (never decide alone)
"design" (could be PCB design), "prototype" (could be simulated), "testing" (could be simulation),
"layout" (PCB vs magnetics winding layout), "architecture" and "control" (firmware vs power stage), "Simulink" (model vs code generation), "EMI" (design guidance vs lab measurement),
"develop" and "build". Resolve by context or leave `unclear`.

## 2. Topology (one primary per posting; `other` if none)

| Value | Keywords |
|---|---|
| `LLC` | LLC, LLC resonant, half-bridge resonant |
| `SRC` | series resonant converter, SRC, LCC, series-parallel resonant |
| `CLLC` | CLLC, bidirectional resonant (now its own value, approved) |
| generic resonant | bare "resonant converter" with no type: `topology = LLC` is NOT assumed; use `other` with low confidence |
| `DAB` | dual active bridge, DAB, phase-shift bidirectional, isolated bidirectional DC/DC |
| `PSFB` | phase-shifted full bridge, PSFB, ZVS full bridge |
| `flyback` | flyback, QR flyback, active clamp flyback |
| `forward/ACF` | forward converter, active clamp forward, two-switch forward |
| `buck/boost` | buck, buck-boost, non-isolated DC/DC, multiphase buck, point-of-load (bare "boost" counts only without "PFC") |
| `totem-pole PFC` | totem pole, totem-pole PFC, bridgeless PFC |
| `interleaved PFC` | interleaved PFC, interleaved boost |
| `PFC` (generic) | PFC, boost PFC, CCM PFC, CrM PFC, power factor correction (approved) |
| `motor-drive inverter` | motor drive inverter, traction inverter, servo drive, VFD (approved split) |
| `grid-side inverter` | three-phase grid inverter, T-type, NPC, active front end |
| `grid-tied PV inverter` | grid-tied inverter, PV inverter, string inverter, microinverter, grid-forming |
| `MPPT` | MPPT, maximum power point tracking, solar charge controller |
| `battery charger` | battery charger, OBC, onboard charger, fast charger (EV charging is an application, not a topology) |
| `other` | none of the above |

A posting may name several topologies; store the primary one in `topology` and note the rest in the
excerpt. Generic "PFC" maps to `PFC`.

## 3. Power class (from stated power; `null` if unstated; never infer from topology)

| Value | Range |
|---|---|
| `<100W` | below 100 W |
| `100W-1kW` | 100 W up to 1 kW |
| `1-10kW` | 1 kW up to 10 kW |
| `10-100kW` | 10 kW up to 100 kW |
| `>100kW` | above 100 kW |

Parse "W", "kW", "MW", "kVA". Boundaries: 100 W belongs to `100W-1kW`; 1 kW to `1-10kW`; 10 kW to
`10-100kW`; 100 kW to `10-100kW`; above 100 kW to `>100kW`. Ranges spanning classes: use the upper
bound's class. kVA is recorded as kW only if a power factor is stated, otherwise `null`.

## 4. Application

| Value | Keywords |
|---|---|
| `EV charging` | EV charger, onboard charger, OBC, DC fast charging, V2G, charging station |
| `solar and storage` | solar, PV, photovoltaic, energy storage, BESS, microgrid, hybrid inverter |
| `server and telecom` | server PSU, data center, 48 V, telecom rectifier, AI server, OCP rack (Open Compute) |
| `aerospace and defense` | aerospace, avionics, aircraft, defense, military, MIL-STD, satellite, spacecraft |
| `industrial` | industrial, motor drive, welding, UPS, industrial power supply, railway |
| `medical` | medical, IEC 60601, patient, imaging |
| `consumer` | consumer, USB-C PD, adapter, charger for phone or laptop, appliance, LED driver |

Default `null` when no application keyword appears.

## 5. Deliverable (multi-valued list)

| Value | Keywords |
|---|---|
| `topology study` | topology selection, trade study, feasibility study, architecture comparison |
| `component ratings` | component selection, ratings, stress analysis, derating, loss estimate, thermal estimate |
| `magnetics design` | magnetics, transformer design, inductor design, planar transformer, winding, core selection, litz |
| `loop compensation` | loop compensation, control loop, Type II, Type III, Bode, stability, small-signal, voltage-mode, current-mode |
| `simulation model` | PLECS, LTspice, Simulink, PSIM, SPICE model, average model, simulation |
| `design review` | design review, schematic review, BOM review, reference design review, second opinion |
| `EMI guidance` | EMI pre-compliance, EMI filter design, CM choke, conducted emissions guidance |
| `other` | design-related but none of the above |

## 6. Device technology (multi-valued; empty list if none stated)

| Value | Keywords |
|---|---|
| `Si` | silicon MOSFET, IGBT, Si MOSFET, superjunction |
| `SiC` | SiC, silicon carbide, SiC MOSFET |
| `GaN` | GaN, gallium nitride, GaN HEMT, eGaN |

## 7. Fit-relevant tags (used by scoring, not classification)

Owner expertise list (from `CLAUDE.md`): resonant DC/DC (LLC, SRC, DAB), magnetics and planar
transformers, PFC stages, loop compensation (Type II/III), WBG devices (SiC, GaN), ML for converter
classification. Scoring (Phase 3) will map topology and deliverable values onto this list; weights
live in `config.yaml`.

## Decisions (2026-11-21)
CLLC is its own topology. Inverters split into motor-drive and grid-side. BMS is **out of scope**:
postings about BMS design are labeled `other` and excluded from fit scoring.
