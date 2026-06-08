# Creative Performance Audit — Diagnose Every Ad in Your Meta Account

**Audit a Meta/Facebook ad account's creative performance and generate a diagnostic report. Every ad gets a verdict; every problem routes to the downstream skill that fixes it.**

Give Claude your ad account. It pulls last 30 days at ad level (sales-objective only), derives personal-best benchmarks from your own ads (segmented by format × audience stage), reads every ad through the UCJ Metric Stack, applies the Metric-to-Diagnosis Mapping, classifies each ad into Winner / Workhorse / Loser / Untested, runs a Stage-Balance Check to catch funnel-shape problems, and routes each diagnosed ad to the downstream skill that fixes it. The audit is diagnosis only — solutions live downstream.

## Install

**Claude Desktop (Cowork):** download [`creative-performance-audit.skill`](https://github.com/the-baweja/creative-performance-audit/releases/latest) → Settings → Skills → drop it in.

**Claude Code:**

```
git clone https://github.com/the-baweja/creative-performance-audit.git ~/.claude/skills/creative-performance-audit
```

**Manual:** clone this repo into any skills directory your Claude setup reads from.

## What it does

You give it your ad account. It runs the full audit:

1. **Preflight Check + Setup Runbook** — detects what's connected (Meta Ads MCP, audience metadata, trend tool, /watch skill), gives exact install instructions for anything missing, offers fallback paths so you're never blocked
2. **Data import (3-tier ladder)** — Meta Ads MCP → CSV upload from Ads Manager → manual paste
3. **Format + Stage classification** — every ad labeled by format (Video / Static / Carousel / Collection) and funnel stage (Cold / Warm / Hot / Existing) using ad-set name heuristics + custom audience metadata
4. **Personal-Best Benchmark derivation** — top-quartile median per `(format × stage × placement)` cell, from your own account's last 30 days, sales-objective only. Sample-size guards fall back gracefully when cells are thin
5. **UCJ Metric Stack scoring** — reads every ad through Spend → Results → CPM → CTR → CPC → CPLV → CR, sorted by spend, left-to-right. The first metric that fails is the structural mistake; everything downstream is symptom
6. **CPC Decomposition Rule** — CPC is a byproduct of CPM × CTR. Decompose before diagnosing. High CPM means hook problem; healthy CPM but low CTR means offer problem
7. **Metric-to-Diagnosis Mapping** — every ad gets one explicit verdict: hook problem, hold problem, offer/CTA problem, LP congruency problem, fatigue, real winner, budget-starved winner, or untested
8. **Tier classification** — Winner / Workhorse / Loser / Untested. Winners require spend past the 4× CPA gate
9. **Stage-Balance Check** — funnel-shape diagnostic. Catches structural problems no per-ad diagnosis would surface (bottom-heavy funnel, missing TOFU pipeline, etc.)
10. **Pattern analysis** — what's common across your winners (DNA to replicate), what's common across your losers (avoid list), what's common across fatigued ads
11. **Diagnostic action plan + series handoffs** — every ad routes to the downstream skill that fixes its specific problem

## The audit is diagnosis only — solutions live downstream

Per the Audit Positioning Doctrine: most ad-account problems are simple structural mistakes, not algorithm or budget issues. The audit names the mistake. The skill that fixes it runs next.

| Verdict | Routes to |
|---|---|
| Real Winner | [Creative Iteration Engine](https://github.com/the-baweja/creative-iteration-engine) — 10 iterations to scale without burning out |
| Hook problem | [Ad Hook Generator](https://github.com/the-baweja/ad-hook-generator) — 30+ scroll-stopping hooks |
| Offer / CTA problem | [Ad Copy Writer](https://github.com/the-baweja/ad-copy-writer) — full ad copy packages |
| LP congruency problem | Landing Page Audit (Ep 10) — post-click conversion audit |
| Fatigue | Ad Fatigue Detector (Ep 9) — trend-confirmed refresh plan |

## Output

A branded DOCX report with:

- Executive summary with top 3 actions + stage-balance verdict
- "How to Read This Audit" onboarding + vocabulary cheat-sheet
- Personal-Best Benchmarks grid per cell
- Stage-Balance Check (funnel-shape diagnostic)
- Performance Tier Breakdown
- Top 10 Diagnostic Cards — full UCJ Stack readout per ad with ✓/✗ indicators, plain-English diagnosis, numbered action steps, expected outcome, downstream skill handoff
- Winner / Loser / Untested pattern analysis
- Fatigue Watchlist with refresh playbook
- Step-by-step Action Plan for the top 3 actions, with "If you do nothing" consequence framing
- Series Handoffs page — every problem routed to the right downstream skill
- Contrarian Insight (one finding that goes against conventional "highest-spend = best ad" thinking)

## Customize Your Branding

Edit `references/branding.md` with your own brand colors, typography, and document structure. The skill reads this file to generate reports that match your brand identity.

## Example

Prompt:

```
Audit the creative performance of my Meta ad account.
Last 30 days, sales-objective only.
```

Output: Preflight Check confirmed Meta Ads MCP connected. Pulled 73 ads at ad level filtered to sales-objective campaigns. Derived personal-best benchmarks for 4 cells (VIDEO × COLD, VIDEO × WARM-HOT, STATIC × COLD, STATIC × WARM-HOT). Read every ad through the UCJ Metric Stack. Classified: 1 Winner (passes top-quartile on hook rate, hold rate, AND CTR simultaneously), 3 Workhorses, 0 Losers (no ad cleared the 4× CPA gate while underperforming), 69 Untested. Stage-Balance verdict: bottom-heavy funnel (58.7% retargeting / 41.3% cold) — recommend shift to 65:35 over 14 days. Per-ad diagnoses surfaced: 25 hook problems → Ad Hook Generator. 9 offer/CTA problems → Ad Copy Writer. 1 LP congruency problem → Landing Page Audit. 7 fatigued ads → Ad Fatigue Detector. 1 real winner → Creative Iteration Engine. Contrarian insight: the highest-spending ad in the account is a Workhorse at $65 CPA — the Real Winner is producing purchases at $24 CPA on one-fifth the budget.

## Who this is for

Media buyers, creative strategists, brand owners, and performance marketers who want to know exactly which ads to kill, scale, or refresh — and which downstream skill to use to actually do it. The audit is the entry point to the entire AI Skills for Media Buyers series.

This is skill 8 of 10 in the **AI Skills for Media Buyers** series by [Baweja Media](https://bawejamedia.com).

---

## Want Baweja Media to audit your ad account and explore opportunities to work together?

[→ Book a strategy call](https://webinar.sannidhyabaweja.com/vsl-lp-ind)

---

Built by [Baweja Media](https://bawejamedia.com) · MIT License
