---
name: reading-synthetic-markets
description: Use whenever working with TRKR synthetic market data or its MCP tools — simulated futures, scenario analysis, portfolio stress testing, "what could happen" questions about markets. Establishes how to read an ensemble of simulated futures without drawing conclusions the data does not support.
---

# Reading synthetic markets

TRKR does not give you one projection. It gives you an **ensemble**: hundreds of
complete, internally consistent futures for the same set of real assets, each ten
years of daily prices, all anchored to today's actual market conditions.

Everything below follows from that one fact. The ensemble is the claim; no member
of it is.

## Start here, every time

Call `get_dataset_overview` before anything else. **The number of futures and the
asset list change between runs.** A run may hold 100 futures or 1,000; it may or
may not carry a given ticker. Never state a count from memory, and never assume
an asset is present — the overview is the only authority.

The ticker axis also carries series the engine derived rather than simulated as
tradeable assets: `CASH` is a compounded fed-funds proxy, `MSR_SIM` and
`MSR_HIST` are the optimizer's own max-Sharpe portfolios. The overview reports
`investableAssets` separately for this reason. "How many assets does this cover?"
means the investable count.

## The spread is the product

**Never quote a median without the band around it.** The p50 of a simulated
distribution is not the expected outcome in any useful sense — it is one point in
a distribution whose whole purpose is to show you the range.

- Bad: "The portfolio grows to $24,000."
- Bad: "Expect about 9% a year."
- Good: "Median $24,000, but the range across futures runs $11,000 to $52,000 —
  the 5th-percentile case loses money over ten years."

When a user asks "what will happen", the honest answer names the spread first and
the centre second. The 5th percentile is usually the number that matters, because
it is the one a client has to be able to live through without abandoning the plan.

## One path is not a forecast

`get_path` returns a single simulated future in detail — prices, the macro
environment, the regime per day. This is the right tool for *"why did this
future do that?"*, and the regimes and macro levels are the mechanism to point at.

It is the wrong tool for *"what happens to SPY?"*. Anything you conclude from one
path is a statement about that path. Before generalising, check
`get_asset_quantiles`, which gives the distribution across all futures.

Also note: quantile bands are computed **across futures at each date**. The p95
line is not a trajectory any single future follows — no path rides the top of the
band the whole way.

## Units differ by series, and the difference is silent

- **Asset** statistics are log-return based and expressed as fractions (`%`).
- **Macro** statistics are differences in **percentage points** (`pp`).

A macro "annual return" of `-0.04` means *four hundredths of a percentage point*,
not −4%. Reading one convention as the other misstates every number without
producing anything that looks wrong. When you report a macro figure, say "pp".

## Scenario conditioning is legitimate. Outcome conditioning is not.

This is the one rule that matters more than the others, because breaking it
produces a claim that is both impressive and meaningless.

- **Scenario conditioning** — filtering futures by their macro *inputs*: "in the
  futures where the fed funds rate went above 6%…". This is a legitimate
  conditional claim. The condition is a cause, and it was set before outcomes
  were known.

- **Outcome conditioning** — filtering futures by their realized *returns*: "in
  the futures where equities fell 30%…". This is selecting on the dependent
  variable.

The failure is specific: if you select the futures where stocks did badly and
then report that a defensive allocation beat a stock benchmark in them, you have
discovered nothing. It beat the benchmark **by construction** — you chose those
futures *because* stocks did badly. The comparison is a tautology wearing the
clothes of a finding.

**So: never present a benchmark or outperformance comparison for a slice selected
on realized returns.** If a user asks for one, say plainly why the number would be
circular, and offer the honest alternatives:

- Condition on the macro cause instead ("futures where credit spreads blew out"),
  then compare — the selection no longer knows the answer.
- Or report the defensive allocation's own outcomes in that slice, without a
  comparison, and say the slice was chosen on outcomes.

TRKR's server enforces this structurally where it can: no tool returns per-path
outcomes, so the numbers needed to build the tautology are never sent. That is a
guardrail, not a substitute for the judgment — the same error is possible by hand
from an exported CSV.

## Credibility: show the scorecard, don't assert

When asked whether the data is trustworthy, do not vouch for it. Call
`get_quality` and report what it says: how many real risk measurements fall
inside the synthetic 5th–95th percentile band, and which assets miss.

"Inside the band" means the real value is a plausible draw from the simulated
distribution — the simulation is not contradicted by history. It does not mean
the simulation is correct about the future. Some rows are inapplicable (a Sharpe
ratio for the cash leg) and score null; exclude them from counts rather than
treating them as failures.

A model that misses on a few assets is normal and worth naming specifically. That
is more convincing than a clean sweep, and it is what a professional will check.

## Portfolio work

`simulate_portfolio` runs weights through **every** future and returns the
distribution. `compare_portfolios` runs two through the same futures and
differences them — prefer it for "which is better", because the futures are then
paired and the differences are real rather than noise.

Reading the output:
- A better median with a worse 5th percentile is a **trade**, not an improvement.
  Say so.
- Max drawdown is the number most clients react to emotionally. Lead with it when
  the question is about risk tolerance.
- `get_engine_portfolios` returns the optimizer's max-Sharpe weights. Treat them
  as a reference point, not a recommendation: max-Sharpe weights are notoriously
  sensitive to estimated moments, and the historical variant is fitted to the one
  history that happened.

## What this is not

TRKR is simulated data for analysis and stress testing. It is **not investment
advice**, and no output is a prediction about real markets. If the user is an
advisor planning to put any of this in front of a client, the methodology is
documented at https://www.trkr.ai/methodology for their compliance review — and
the obligations for client-facing material are theirs.
