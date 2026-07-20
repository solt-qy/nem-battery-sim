# Progress Log — nem-battery-sim

Reverse-chronological record of progress, tied to the learning curriculum's phase/task
IDs (see the `career/learning/curriculum.md` in the owner's notes). Newest entries first.

## 2026-07-20 — Phase 0 kickoff (setup week)

**Done**
- **P0-1 — repo scaffolded.** `uv init` with a package/src layout (`src/nem_battery_sim/`),
  Python 3.12. Dependencies added via `uv add`: `nemosis`, `openelectricity`, `linopy`,
  `highspy`, `pandas`, `matplotlib`. README written with the project thesis + a one-line
  rationale per dependency; `.gitignore`, `.env.example`, and `uv.lock` in place. Thesis
  rewritten in the owner's own voice.
- **P0-2 — OpenElectricity API key.** Obtained (COMMUNITY / free tier). Stored in a
  gitignored `.env`; `python-dotenv` wired so `OEClient()` auto-reads the key. Validated
  against the live API (`get_current_user()` → auth OK).
- **P0-4 — CLAUDE.md.** Starter agent-context file: project context, the 🤖/🤝/🧠
  delegation boundary (agents scaffold plumbing; the owner owns the degradation model,
  economic reasoning, and write-ups), tooling conventions, secrets rules, data caveats,
  and the milestone roadmap.

**Open**
- **P0-3 — smoke-test data pull** *(the hands-on learning task)*: one week of VIC1
  `DISPATCHPRICE` via NEMOSIS + OpenElectricity, plot both, decide whether the two sources
  agree and why they might not. Watch the April-2026 NEMweb endpoint migration.
- **P0-5 — AEMO NEM Foundations eLearning**, module 1.
