# CLAUDE.md — nem-battery-sim

Context and working rules for any agent (Claude Code or similar) working in this repo.
Read this first, every session.

## What this project is

A **degradation-aware BESS (battery energy storage system) dispatch simulator** run
against **real NEM (Australian National Electricity Market) price data**. It optimises
battery dispatch — energy arbitrage + FCAS — and, unlike naive dispatch models, prices
the **cost of battery degradation** directly into the optimisation using explicit
degradation models. The headline result is the difference between naive revenue and
degradation-aware revenue: *what a battery actually earns once it pays for its own wear.*

See [README.md](README.md) for the owner's thesis (authoritative — match its voice).

## Why it exists (read before "helping" too much)

This is a **portfolio + learning artifact**, not a product. The owner is a battery
simulation engineer building public NEM/optimisation credibility. The value of this repo
is **evidence that the owner understands the work** — especially the degradation model,
which is the owner's professional expertise and the whole point of the project (the "moat").

Consequence: an agent that writes the interesting parts *destroys the artifact's value*.
Your job is to remove friction from the plumbing so the owner spends their time on the
parts that are theirs to own. See the working agreement below — it is not optional.

## Working agreement — what to delegate vs. what is the owner's

The curriculum tags every task 🤖 / 🤝 / 🧠. Respect it.

- 🤖 **You may do (and are expected to do well):** repo scaffolding, data-loader and
  ingestion boilerplate, plot formatting, test harnesses, refactors, docstrings, doc
  polish, environment/tooling setup, translating an owner-specified spec into library
  syntax. Produce it, then explain it clearly so the owner can understand it before committing.

- 🤝 **Mixed — you do the mechanical half, owner owns the logic:** API syntax, pandas
  wrangling, rolling-horizon loop mechanics. You write *how*; the owner decides *what to
  pull, what it means, and whether it's right.* Surface the judgement calls; don't bury them.

- 🧠 **DO NOT do these — they are the learning and the moat:**
  - **The degradation cost model / cost curves.** Never design or author the degradation
    economics. You may *translate a spec the owner has already decided* into linopy syntax,
    and you may act as a tutor (explain an API, explain why a constraint is infeasible), but
    you must not decide the model. If you write the cost curve, the project stops being evidence.
  - **The LP/MILP objective and constraint design, and the economic reasoning** behind them.
  - **NEM domain interpretation** — what a price event means, whether two data sources
    agree and why, what a revenue split implies. The owner writes these in their own words.
  - **Result interpretation and all write-ups / narrative.** "It ran successfully" is not
    "the data is right" and not "the number is meaningful." Report facts; let the owner judge.

When unsure whether something is delegable, ask: *"If I did this whole thing unsupervised,
would the resulting artifact still be evidence of the owner's understanding?"* If no, stop
and hand it back with your reasoning, don't just do it.

## Tech stack & conventions

- **Python 3.12**, managed with **`uv`** (see `.python-version`, `pyproject.toml`, `uv.lock`).
- **Package (src) layout:** importable code lives in `src/nem_battery_sim/`. Import as
  `from nem_battery_sim import ...`. Keep the loader/simulator logic in the package;
  keep exploratory pulls in scripts/notebooks.
- Add dependencies with **`uv add <pkg>`** (updates `pyproject.toml` + `uv.lock`) — never
  hand-edit the dependency list or use bare `pip`.
- `uv.lock` **is committed** (reproducible env). `.venv/`, `__pycache__/`, `*.parquet`,
  and `data/` are gitignored.

### Dependencies and their role
| Package | Role |
|---|---|
| `nemosis` | Primary source: historical AEMO NEMweb tables (dispatch & FCAS prices, demand). |
| `openelectricity` | Second, independent source (OpenElectricity/OpenNEM API) for cross-checking + recent data. Needs API key. |
| `linopy` | Builds the dispatch optimisation model (arbitrage/FCAS LP-MILP). |
| `highspy` | HiGHS solver — solves the linopy model. |
| `pandas` | Time-series and results wrangling. |
| `matplotlib` | Plots for the README and write-ups. |
| `python-dotenv` | Loads the API key from `.env`. |

## Secrets — hard rules

- The OpenElectricity API key lives in **`.env`** (gitignored). `.env.example` is the
  committed template. **Never** commit `.env`, and **never** print, echo, or paste the key
  value into chat, logs, code, or commits — mask it if you must reference it.
- Load it with `from dotenv import load_dotenv; load_dotenv()` at the top of scripts;
  `OEClient()` then reads `OPENELECTRICITY_API_KEY` from the environment automatically.
- Status: key is set and has been validated against the live API (COMMUNITY / free tier —
  pull modestly, mind rate limits).

## Commands

```bash
uv sync                                   # create/refresh env from uv.lock
uv add <pkg>                              # add a dependency
uv run python path/to/script.py           # run in the project env
uv run python -c "import nemosis, openelectricity, linopy, highspy"   # import smoke test
```
Note: `load_dotenv()` auto-discovers `.env` when the calling script lives inside the repo.
It fails to auto-discover when code is piped via stdin (`python - <<'PY'`) — pass an explicit
path there, or just use a real file.

## Data layer — known caveats

- ⚠ **AEMO migrated/decommissioned the NEMweb endpoints in April 2026.** If a NEMOSIS pull
  404s, check NEMOSIS release notes for the updated base URL / pin a working version; fall
  back to the OpenElectricity API or [`nem-data`](https://github.com/ADGEfficiency/nem-data).
- Always **cross-check** NEMOSIS against OpenElectricity for the same region/interval and
  flag disagreements — this is a deliberate exercise, not an inconvenience. The owner decides
  whether the two agree and why they might not.
- Watch for the usual NEM data traps: DST transitions, region codes (VIC1, NSW1, …),
  5-minute vs 30-minute settlement resolution, and silent nulls/gaps. Report them; don't
  paper over them.

## Where the project is heading (roadmap)

Milestone-gated build (each milestone is a reviewable PR). Don't jump ahead of the current phase.

1. **Phase 0 — scaffolding** (current): repo, env, key, first smoke-test pull.
2. **M1 Data layer:** NEMOSIS → parquet/DuckDB store, 2+ yrs energy + FCAS prices, 2–3 regions,
   clean loader API. *You scaffold ingestion; owner validates data quality.*
3. **M2 Arbitrage LP:** perfect-foresight energy arbitrage (linopy + HiGHS): power/energy
   limits, efficiency, SoC dynamics. *Owner designs constraints & objective.*
4. **M3 FCAS co-optimisation:** contingency + regulation enablement; validate revenue mix.
5. **M4 Degradation module — the moat (🧠 owner-only):** rainflow benchmark vs the owner's own
   physics-informed cost curve; compare. *You do not write the cost model.*
6. **M5 Imperfect foresight:** rolling-horizon dispatch with a naive/persistence forecast.
7. **M6 Write-up & launch:** results notebook + post, in the owner's voice.

## Definition-of-done rule

Every phase exits on a concrete **evidence artifact** (committed code + a short write-up /
plot / notes in the owner's own words). "The agent made it" is never evidence — the owner
must be able to explain any generated code and every result. Don't mark work done until that
holds. Commit only when the owner asks; never commit secrets or raw data files.
