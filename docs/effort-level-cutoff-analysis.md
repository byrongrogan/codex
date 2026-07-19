# Effort-Level Cutoff Analysis: Sonnet 5, Opus 4.8, Fable 5

**Purpose:** Data supporting a fleet-wide policy on which reasoning-effort levels to disable
for Claude models (Sonnet 5, Opus 4.8, Fable 5) across ~4,000 managed seats, based on public
benchmark and pricing data.

**Prepared:** 2026-07-19

---

## TL;DR — Recommended cutoffs

| Model | Allow | Disable | One-line rationale |
|---|---|---|---|
| **Sonnet 5** | `low` / `medium` / `high` | `xhigh`, `max` | At `xhigh`+ Sonnet 5 costs **more per task than Opus 4.8** while scoring lower on aggregate intelligence — a dominated configuration. Users who need more than Sonnet-5-`high` should switch model, not raise effort. |
| **Opus 4.8** | `low` → `xhigh` | `max` | `xhigh` is Anthropic's recommended setting for coding/agentic work and the **Claude Code default** — disabling it degrades the primary workflow. `max` adds "significant cost for relatively small quality gains" (Anthropic's own wording). |
| **Fable 5** | `low` → `high` (default) | `xhigh`, `max` | Fable 5's differentiation shows up **even at medium effort** (FrontierCode, spreadsheet suites); its `low`/`medium` "often exceed `xhigh` performance on prior models" per Anthropic. At $10/$50 per MTok, the effort dial is a much smaller cost lever than model selection itself. |

The single best-supported change is **disabling `max` on all three models**. Every source —
Anthropic's own documentation and independent benchmarkers — agrees `max` is a
diminishing-returns setting for all but frontier research problems.

---

## 1. Background: what the effort levels are

All three models accept `output_config.effort` with five levels: `low`, `medium`, `high`,
`xhigh`, `max`. Effort controls **all** token spend — thinking depth, number/verbosity of tool
calls, and response length — not just a thinking budget.

Two defaults matter for policy design:

- **API default: `high`** (omitting the parameter = `high`) on all three models.
- **Claude Code default: `xhigh`** — so an org-wide cutoff below `xhigh` silently changes the
  product default for every Claude Code user.

Anthropic's per-model guidance (from the [effort documentation](https://platform.claude.com/docs/en/build-with-claude/effort)):

| Level | Anthropic's characterization (Opus 4.7/4.8 guidance) |
|---|---|
| `low` | "Efficient, but best for short, scoped tasks" (subagents, classification) |
| `medium` | "The drop-in for the average workflow where you want good results while reducing costs" |
| `high` | "Often the best balance of quality and token efficiency" — the API default |
| `xhigh` | "Recommended starting point for coding and agentic work… expect meaningfully higher token usage than high" |
| `max` | "Reserve for genuinely frontier problems. **On most workloads max adds significant cost for relatively small quality gains**, and on some structured-output or less intelligence-sensitive tasks it can lead to overthinking." |

For Fable 5 specifically, Anthropic recommends starting at `high` (not `xhigh`), and notes:
"Lower effort settings on Claude Fable 5 still perform well and **often exceed `xhigh`
performance on prior models**."

For Sonnet 5, Anthropic publishes an explicit cross-model equivalence:
**Sonnet 5 @ `medium` ≈ Sonnet 4.6 @ `high`**, and **Sonnet 5 @ `high` ≈ Sonnet 4.6 @ `max`**.

---

## 2. Pricing baseline

| Model | Input $/MTok | Output $/MTok | Notes |
|---|---|---|---|
| Sonnet 5 | $3 ($2 intro) | $15 ($10 intro) | Intro pricing ends 2026-08-31 |
| Opus 4.8 | $5 | $25 | No long-context premium at 1M ctx |
| Fable 5 | $10 | $50 | Requires 30-day data retention |

Per-token price is misleading on its own: Sonnet 5 uses a new tokenizer (~1.0–1.35× tokens for
the same text vs. Sonnet 4.6) and is substantially more verbose per task (see §3), so **cost
per completed task** is the metric that matters.

---

## 3. Independent data: Artificial Analysis (max effort)

Artificial Analysis benchmarks the max-effort variants of each model on its Intelligence Index
(a composite of ~10 evals). This is the most trustworthy independent cost-per-task data
available:

| Model (max effort) | Intelligence Index | Cost per task | Cost to run full index | Output tokens used | Output speed |
|---|---|---|---|---|---|
| Fable 5 | **60** | $2.75 | ~$5.6–6.2K (most expensive model AA has ever benchmarked) | — | — |
| Opus 4.8 | 56 | ~$2.00 | $3,753 | 120M | 56.7 tok/s |
| Sonnet 5 | 53 | **$2.29** | $4,010 | **300M** | 82.2 tok/s |
| Sonnet 4.6 (reference) | 47 | ~$1.15 | — | — | — |

Key findings from AA:

1. **Sonnet 5 at max effort costs ~15% more per task than Opus 4.8 at max effort** ($2.29 vs
   ~$2.00) while scoring 3 points lower — despite per-token prices that are 40% lower. The
   driver is token volume: Sonnet 5 emitted **2.5× the output tokens** of Opus 4.8 on the same
   benchmark suite, used ~40% more output tokens per task than Sonnet 4.6, and ~3× the agentic
   turns on knowledge-work evals.
2. **Effort is a large token multiplier:** on GDPval-AA, max effort used **~6× more turns than
   low effort**.
3. Fable 5's 4-point Intelligence Index lead over Opus 4.8 costs ~2× per token and ~37% more
   per task at max effort.

Sources: [AA on Sonnet 5 cost-per-task](https://artificialanalysis.ai/articles/claude-sonnet-5-agentic-cost),
[AA Sonnet 5 model page](https://artificialanalysis.ai/models/claude-sonnet-5),
[AA Opus 4.8 model page](https://artificialanalysis.ai/models/claude-opus-4-8),
[AA Fable 5 launch analysis](https://artificialanalysis.ai/articles/claude-fable-5-mythos-intelligence-index),
[AA threads on X](https://x.com/ArtificialAnlys/status/2072062595482456431).

---

## 4. Vendor benchmark data (headline scores)

Anthropic's system-card standard configuration is **adaptive thinking at max effort, average
of 5 trials** — i.e., published headline numbers are ceiling numbers, not what users get at
the default `high`:

| Benchmark | Sonnet 5 | Opus 4.8 | Fable 5 |
|---|---|---|---|
| SWE-bench Verified | 85.2% | 88.6% | — |
| SWE-bench Pro | 63.2% | 69.2% | **80.3%** |
| Terminal-Bench 2.1 | **80.4%** | 74.6% | — |
| OSWorld-Verified | 81.2% | 83.4% | — |
| HLE (with tools) | 57.4% | 57.9% | — |
| GDPval-AA v2 (Elo) | **1618** | 1615 | best overall per AA |
| FrontierCode Diamond | — | 13.4% | **29.3% (at medium effort)** |

Effort-relevant statements from the launch materials:

- **Sonnet 5 @ `xhigh` ≈ Opus 4.8 @ medium-to-high** on OSWorld and BrowseComp — but
  "running Sonnet 5 at x-high can **cost more than running Opus 4.8 at a comparable accuracy
  point**" ([VentureBeat](https://venturebeat.com/technology/anthropic-launches-claude-sonnet-5-at-a-steep-discount-to-its-top-model-as-the-company-races-toward-a-blockbuster-ipo),
  corroborated by AA's per-task data in §3).
- **Fable 5 beats Opus 4.8 "at every effort level"** on Anthropic's spreadsheet suite, with
  25–30% fewer turns — i.e., Fable's advantage is the model, not the effort dial.
- **Fable 5 leads FrontierCode "even at medium effort."**
- Opus 4.8's headline coding scores are quoted at max effort; Anthropic separately notes
  Opus 4.8 at `xhigh` "spends a similar number of tokens as Opus 4.7's default, but with
  better performance."

---

## 5. Analysis by cutoff decision

### 5.1 `max` — disable on all three models (strongest case)

- Anthropic's own guidance: "on most workloads max adds significant cost for relatively small
  quality gains" and it "can lead to overthinking."
- AA: max effort ≈ 6× the agentic turns of low effort on knowledge work; a few Intelligence
  Index points separate max from lower settings at multiples of the cost.
- The published headline benchmark deltas between successive model generations (e.g., Opus 4.7
  → 4.8 = +1.0 on SWE-bench Verified) are of similar magnitude to what max buys over
  high/xhigh — but model upgrades are free, while max is paid on every request.
- No mainstream workflow depends on it: neither the API default (`high`) nor the Claude Code
  default (`xhigh`) is affected.

**Expected pushback:** research-style users with genuinely frontier problems. Mitigation:
allow exception-listing rather than keeping it on for 4,000 seats.

### 5.2 Sonnet 5 `xhigh` — disable (dominated configuration)

The dominance argument, which is the defensible core of this policy:

1. Sonnet 5 @ `xhigh`/`max` reaches roughly Opus 4.8 @ `medium`–`high` capability (OSWorld,
   BrowseComp).
2. But it costs **more per task** at that point than Opus 4.8 does (AA: $2.29 vs ~$2.00;
   vendor materials concede the same).
3. And Opus 4.8 retains headroom above that point (SWE-bench Pro 69.2 vs 63.2; Intelligence
   Index 56 vs 53).

So for any task hard enough to justify Sonnet-5-`xhigh`, **Opus 4.8 at high/xhigh is both
cheaper per task and better**. The rational escalation path is *model*, not *effort*:
`Sonnet 5 high → Opus 4.8 high/xhigh`.

**Caveat to disclose:** the dominance is not universal. Sonnet 5 beats Opus 4.8 on
Terminal-Bench 2.1 (80.4 vs 74.6) and edges it on GDPval-AA (1618 vs 1615), so a minority of
terminal-heavy/knowledge-work tasks lose a genuinely better option. Also note the Claude Code
default is `xhigh` — Sonnet users in Claude Code will silently drop to `high`, which per
Anthropic's own equivalence still equals Sonnet 4.6 @ `max`, the previous generation's
ceiling.

### 5.3 Opus 4.8 `xhigh` — keep

- It is Anthropic's explicit recommendation for coding and agentic work and the Claude Code
  default; disabling it changes the out-of-box behavior of your primary coding tool.
- Cost at `xhigh` is bounded: Anthropic states Opus 4.8 `xhigh` spends tokens comparable to
  Opus 4.7's *default*.
- **Effort→cost is not monotonic on agentic work.** Anthropic's migration guidance notes
  higher effort up front "often *reduces* turn count and total cost" because the model plans
  better and thrashes less. Cutting Opus to `high` may not save what a per-token analysis
  predicts, and can cost quality on exactly the long-horizon tasks Opus is chosen for.

### 5.4 Fable 5 `xhigh` — disable (cap at `high`, its default)

- Anthropic's Fable guidance already starts at `high`, not `xhigh` (unlike Opus).
- Fable's measured advantages materialize at *medium* effort (FrontierCode Diamond 29.3% vs
  Opus 13.4%; spreadsheet suite wins "at every effort level").
- Anthropic: Fable's `low`/`medium` "often exceed `xhigh` performance on prior models" —
  users capped at `high` still get above previous-generation-ceiling results.
- At $10/$50 per MTok and ~$2.75/task at max (AA), Fable is the costliest surface in the
  fleet; the effort cap is a secondary control to whatever seat/model gating you apply, but
  `xhigh`+`max` are where its long-horizon runs (30-minute-plus, million-token budgets) live.

---

## 6. Caveats — say these up front when the policy is questioned

1. **Public per-effort benchmark data is thin.** Anthropic publishes headline scores at max
   effort and cost-performance *curves* (BrowseComp, OSWorld) without per-point numbers;
   Artificial Analysis benchmarks only the max variants. The quantitative anchors here are
   (a) AA's cost-per-task at max, (b) Anthropic's stated cross-model equivalences, and (c) the
   6× turns multiplier between max and low. Nobody publicly publishes, e.g., "Sonnet 5
   SWE-bench at high vs xhigh."
2. **Validate on your own telemetry before enforcing.** You have the best dataset available:
   per-request token usage by model × effort across 4,000 users. Measure (i) what fraction of
   spend is at `xhigh`/`max`, (ii) task-completion or retry-rate differences by effort on
   comparable task types. A 2-week A/B (cap a cohort at the proposed levels, watch
   retries/escalations and spend) turns this from "benchmark-based" into "measured on our
   traffic."
3. **Non-monotonic cost on agentic work** (see §5.3) — the cheapest configuration is not
   always the lowest effort. Expect some capped workloads to consume *more* turns.
4. **Effort is a behavioral signal, not a hard budget.** Models still think on hard problems
   at low effort; disabling levels changes the distribution of spend, it doesn't cap it. Task
   Budgets (`output_config.task_budget`) are the complementary hard-ish control if the goal is
   cumulative spend limits on agentic loops.
5. **Intro pricing distortion:** Sonnet 5's $2/$10 intro pricing ends 2026-08-31; any
   cost-per-task comparison done today at intro rates understates Sonnet's post-September
   cost by ~33%. The AA figures above use standard ($3/$15) pricing.

---

## 7. Suggested rollout

1. **Phase 1 (now):** disable `max` on all three models fleet-wide; exception list for named
   power users. Lowest risk, uncontroversial, backed by vendor's own docs.
2. **Phase 2 (2–4 weeks):** cap Sonnet 5 at `high` and Fable 5 at `high`; publish the
   escalation path ("need more than Sonnet high? use Opus") alongside the dominance data in
   §5.2. Keep Opus 4.8 `xhigh`.
3. **Continuously:** report realized $/task by model×effort from your own logs; revisit the
   Sonnet cap if Terminal-Bench-shaped workloads (heavy CLI agents) show regressions.

---

## Sources

- [Anthropic — Effort parameter documentation](https://platform.claude.com/docs/en/build-with-claude/effort)
- [Anthropic — Introducing Claude Opus 4.8](https://www.anthropic.com/news/claude-opus-4-8)
- [Anthropic — Introducing Claude Sonnet 5](https://www.anthropic.com/news/claude-sonnet-5)
- [Anthropic — Claude Fable 5 and Claude Mythos 5](https://www.anthropic.com/news/claude-fable-5-mythos-5)
- [Artificial Analysis — Claude Sonnet 5: strong agentic performance at a higher cost per task](https://artificialanalysis.ai/articles/claude-sonnet-5-agentic-cost)
- [Artificial Analysis — Claude Fable 5 launches at #1 on the Intelligence Index](https://artificialanalysis.ai/articles/claude-fable-5-mythos-intelligence-index)
- [Artificial Analysis — Claude Opus 4.8 model page](https://artificialanalysis.ai/models/claude-opus-4-8)
- [Artificial Analysis — Claude Sonnet 5 model page](https://artificialanalysis.ai/models/claude-sonnet-5)
- [Artificial Analysis — Claude Fable 5 model page](https://artificialanalysis.ai/models/claude-fable-5)
- [Artificial Analysis threads: Sonnet 5 Intelligence Index & cost per task](https://x.com/ArtificialAnlys/status/2072062592923930666), [cost breakdown](https://x.com/ArtificialAnlys/status/2072062595482456431), [Fable 5 cost](https://x.com/ArtificialAnlys/status/2067384319942029379)
- [Vellum — Claude Sonnet 5 benchmarks explained](https://www.vellum.ai/blog/claude-sonnet-5-benchmarks-explained)
- [Vellum — Claude Fable 5 & Mythos 5 benchmarks explained](https://www.vellum.ai/blog/claude-fable-5-and-mythos-5-benchmarks-explained)
- [Vellum — Claude Opus 4.8 benchmarks explained](https://www.vellum.ai/blog/claude-opus-4-8-benchmarks-explained)
- [VentureBeat — Anthropic launches Claude Sonnet 5](https://venturebeat.com/technology/anthropic-launches-claude-sonnet-5-at-a-steep-discount-to-its-top-model-as-the-company-races-toward-a-blockbuster-ipo)
- [emergent.sh — Claude Sonnet 5 benchmarks](https://emergent.sh/learn/claude-sonnet-5-benchmark)
