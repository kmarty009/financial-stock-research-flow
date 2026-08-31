# financial-stock-research-flow

Scheduled stock and ETF research, alerting, and reporting — built on
[CrewAI Flows](https://docs.crewai.com/concepts/flows).

Four jobs on four cadences, sharing one persisted SQLite state store:

| Job | Cadence | What it does |
| --- | --- | --- |
| Dip watcher | every 15 min, market hours | Texts me when a watchlist name drops ≥2% from prior close |
| Daily research | 4:30pm CT, trading days | New quarterly filings + analyst grade changes → email digest |
| Dividend research | Sundays 7:00am CT | Screens dividend ETFs/funds for retirement → email |
| Monthly report | 1st trading day, 6:00am CT | Per-ticker month performance from stored history |

**The state is the product.** Every job's correctness depends on remembering
what it already reported. Read [`docs/SPEC.md`](docs/SPEC.md) before writing
code — particularly the "Memory — the part that matters" section.

## Repo layout

```
docs/SPEC.md            The behavioral spec. Source of truth.
docs/DECISIONS.md       Why the design is the way it is.
config/watchlist.yaml   Tickers and per-job thresholds. No code edit to add one.
studio_exports/         Drop CrewAI Studio downloads here first. See its README.
src/stock_research_flow/
  flows/                Flow classes — deterministic orchestration.
  crews/                Agents and tasks — judgment work only.
  tools/                Schwab, EDGAR, ratings, Twilio, email adapters.
  state/                Pydantic state models + persistence.
  dashboard/            FastAPI read-only dashboard.
tests/acceptance/       One test per acceptance criterion in the spec.
```

## Setup

```bash
git clone git@github.com:kmarty009/financial-stock-research-flow.git
cd financial-stock-research-flow
uv venv && source .venv/bin/activate
uv pip install -e ".[dev]"
cp .env.example .env    # then fill in your keys
```

## Working with CrewAI Studio

Build in Studio, download the artifacts, drop the archive in
`studio_exports/`, then promote the pieces into `src/`. The
[`studio_exports/README.md`](studio_exports/README.md) has the rules for what
gets promoted and what stays generated — worth reading once before the first
import, because Studio will happily overwrite hand-written code.

## The rule that matters most

Set arithmetic decides what is new. The model only writes about what set
arithmetic already selected. No LLM ever touches the identifier sets in state —
if it does, the automation will eventually re-report something already seen,
which is the one failure mode this whole design exists to prevent.
