# CLAUDE.md — Portfolio123 Claude Skill

A Claude Skill that makes Claude reliable on Portfolio123 (P123): screen rules and formulas,
ranking-system XML, and the REST API through the official `p123api` wrapper. The repository
root **is** the skill — `SKILL.md` plus `references/` plus `scripts/` are the shipped
payload, and `portfolio123.skill` is that payload packaged.

Independent community project, maintained by Quant Solvings. Not affiliated with or endorsed
by Portfolio123, Inc.

## The one rule everything else serves

**Every factor and function name in this repo was extracted from the official P123 Factor
Reference on 2026-06-09 and verified programmatically.** 4,463 factors, 465 functions,
473 constants/series-IDs/operators, 33 REST operations, 38 `p123api` methods.

So: **never invent a name, and never soften a name you cannot source.** If a factor or
function is not in `references/`, it does not exist — look up the correct one. This is the
skill's entire value proposition; a single plausible-sounding invention undoes it, because
a user cannot tell a hallucinated factor from a real one until P123 rejects the formula.

The same rule governs *edits*. Adding a name means having the source. `SKILL.md` carries a
`<!-- name-whitelist: ... -->` comment listing identifiers that legitimately appear without
being P123 factors (env var names, API fields, wrapper methods) — extend it deliberately,
never to silence a check.

## Layout

```
SKILL.md              frontmatter (trigger description), routing table, formula essentials
references/           13 category files — the extracted dictionary
scripts/              9 numbered runnable examples + p123_helpers.py + README
portfolio123.skill    the packaged bundle
BUILD-STATE.md        phase/gate tracker for the v3 build and its two verification rounds
CHANGELOG.md          release history
docs/                 banner image
```

`references/` splits by P123's own 13 official categories, and `SKILL.md`'s routing table is
how a session finds the right one. **Read only what the task needs** — the point of the split
is that a formula question about `FRank` loads `advanced-functions.md` and nothing else.

| Task | File |
|---|---|
| REST API, `p123api` calls, auth, quota | `references/api.md` |
| Valuation ratios, margins, growth, quality scores | `references/ratios-statistics.md` |
| Statement line items, Compustat/FactSet vendor mapping | `references/financials.md` |
| Company metadata, prices as fundamentals, ownership, short interest | `references/fundamentals.md` |
| Consensus estimates, revisions, surprises, recommendations | `references/estimates.md` |
| Price/volume indicators, returns, volatility, beta | `references/technical.md` |
| `FRank`, `FHist`, `Aggregate`, Loops, regression, conditionals | `references/advanced-functions.md` |
| Buy/sell rule factors, rebalance idioms | `references/strategy.md` |
| `UnivAvg`/`UnivCnt`/`UnivMedian`/`UnivSum` | `references/universe-operations.md` |
| `UnivExclude`, `UnivSubset`, `UnivRBICS` | `references/universe-filters.md` |
| `BenchClose` | `references/benchmark-functions.md` |
| RBICS sector/industry classification | `references/industry-sector.md` |
| ETF taxonomy vocabularies | `references/taxonomy.md` |
| Math/set/date utilities, `InList`, `GetSeries`, macro series IDs, operators | `references/misc.md` |
| **Generating or editing ranking-system XML** | `references/ranking-system-xml.md` — mandatory, not optional |

## Formula-language traps this skill exists to prevent

- **P123's formula language is not Python.** `SetVar(@x, expr)`, `Eval(cond, a, b)`.
- **`IsNA(a, b)` is a replacement function, not a boolean test.** It returns `b` when `a` is
  NA. Test for NA with `= NA` (`LastSellPrice = NA`). Getting this backwards produces a
  formula that runs and is silently wrong, which is the worst failure mode here.
- **`FRank` defaults are `#All` scope, `#DESC` sort, `#InclNA`** — state them when they
  matter rather than relying on a reader knowing them.
- Canonical casing is documented; a claim that names are case-insensitive was removed in
  verification as unsourced. Don't reintroduce it.

## The example scripts

Nine numbered scripts in `scripts/`, sharing `p123_helpers.py`. The conventions are
load-bearing and apply to any script added:

- **Credentials come from `P123_API_ID` / `P123_API_KEY`.** Never hardcoded, never printed.
  `.env` is gitignored and must stay that way.
- **`p123api` is used as a context manager** — the documented usage pattern — and pinned
  `>= 2.2.0`.
- **Every call costs quota.** Responses carry `cost` and `quotaRemaining` (the OpenAPI
  `SharedResult` schema); `print_quota` surfaces them. Don't burn quota silently.
- **`09_strategy_rebalance_dryrun.py` is a dry run and stays one.** Nothing in `scripts/`
  may change account state without an explicit, obvious opt-in — a user running an example
  should never be able to trade by accident.
- `p123_helpers.py` is a library module: it makes no API calls and changes no state.

## Packaging

`build/` is gitignored; packaging stages the skill under `build/stage/portfolio123/` and runs
the official skill-creator tool. Two things that have already gone wrong once each:

1. **The packager takes the skill name from the staging *directory* name, not the SKILL.md
   frontmatter.** Stage into a directory named `portfolio123` or the package name is wrong.
2. **`.env` must never end up inside the staged tree.** The packager has no `.env` exclusion,
   so a credentials file sitting next to the skill ships inside `portfolio123.skill`. This
   was caught as a release blocker in verification, not by the tooling — keep credentials out
   of the staged directory rather than trusting a filter that does not exist.

Re-run `check_names.py` over any changed reference file and `SKILL.md` before packaging, and
keep `evals.json` consistent with the docs it asserts against — a verification round found an
eval expecting a `RankingSystem2` root that contradicted `ranking-system-xml.md`.

## Working conventions

- **`BUILD-STATE.md` is the build ledger.** Update it when a phase or gate changes state;
  it records what passed, when, and what each verification round found and fixed. Don't
  rewrite its history — append.
- Bump `CHANGELOG.md` for any change to shipped skill content. v3.0.0 was a ground-up rebuild;
  version the dictionary honestly, since users rely on the extraction date.
- This is a fork of `cmoralesm/P123-Claude-Skill`. Check upstream before diverging on the
  shipped skill, and keep the attribution and the not-affiliated disclaimer intact.
