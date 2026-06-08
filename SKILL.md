---
name: creative-performance-audit
description: "Audit a Meta/Facebook ad account's creative performance and generate a diagnostic report. Use when someone needs a creative KPI audit, wants to identify winning / losing / fatigued ads, needs hook rate / hold rate / first-frame rate / CTR analysis, wants to diagnose performance by funnel stage (TOFU / MOFU / BOFU) and format (video / static / carousel), or asks why specific ads are working. Applies the UCJ Metric Stack — Spend → Results → CPM → CTR → CPC → CPLV → CR — sorted by spend. Derives personal-best benchmarks per (format × stage × placement) cell from the account, not industry averages. Diagnoses each ad: hook problem, hold problem, CR problem, fatigue, budget-starved winner, untested. Outputs a branded Baweja Media DOCX. The audit is diagnosis only — names the structural mistake and hands off prescriptions to Creative Iteration Engine, Ad Fatigue Detector, and Landing Page Audit."
---

# Creative Performance Audit

You are a senior performance analyst running a creative audit on a Meta ad account. Your job is to look past surface metrics (spend, CPA) into the creative KPIs that explain WHY ads perform the way they do — hook rate, hold rate, first-frame rate, CTR — and produce a diagnostic verdict for every ad in the account.

**The audit's charter — diagnosis only.** Per the Audit Positioning Doctrine: "Most ad-account problems are simple structural mistakes, not algorithm or budget issues." The audit names the structural mistake. It does not prescribe solutions. Prescriptions live downstream in the Creative Iteration Engine (iterations), Ad Fatigue Detector (refresh), and Landing Page Audit (LP fixes).

---

## 1. Preflight Check + Setup Runbook

Always run this first. Detect what's available, report status, give exact install instructions for anything missing, and never block the user.

### 1.1 What to detect

| Capability | How to detect | Required for |
|---|---|---|
| **Meta Ads MCP** | `mcp__*__ads_get_ad_accounts` and `ads_get_ad_entities` available | Tier 1 data import |
| **Meta Ads MCP — audience metadata** | `ads_get_custom_audience` available for the target account | Stage classification (warm / hot detection) |
| **Meta Ads MCP — trend** | `ads_insights_performance_trend` available | Fatigue signal detection |
| **/watch skill** | `watch` skill in available skills | Optional — winner deep-dive video import |
| **Branding reference** | `references/branding.md` accessible | Output DOCX styling |

### 1.2 What to report

Use the same format as Creative Iteration Engine Section 1.2 — a clear status block with ✓ / ✗, recommended setup for anything missing, and a fallback offer. If everything is green, proceed to mode selection without asking permission.

### 1.3 Exact install instructions

Identical to Creative Iteration Engine Section 1.3 — Meta Ads MCP toggle in Cowork → Connectors, `/watch` from github.com/bradautomates/claude-video, etc.

### 1.4 Fallback ladder

| If missing | Fallback path |
|---|---|
| Meta Ads MCP | CSV upload from Ads Manager (last 30 days, ad-level). Required columns: Ad Name, Ad ID, Spend, Impressions, Clicks (link), CTR (link), CPM, CPC (link), Cost per Result, Results, Cost per LPV, Frequency, Start Date, Ad Set Name, Campaign Name, Campaign Objective, Placement, Creative Format, Reach, 3-sec video plays, 15-sec video plays, ThruPlays. If columns missing, walk the user through the UCJ Column Preset setup (Section 5.1). |
| Audience metadata | Auto-classify by ad set name + campaign objective heuristics (Section 4). Surface unclassifiable ads to user for labeling. |
| Trend tool | Compute fatigue from snapshot frequency + days-running heuristics. Note in the report that trend-based fatigue confirmation is unavailable. |
| Video metrics missing | Skip hook rate / hold rate / first-frame rate for those ads. Score on CTR / CPM / CPC only. Flag the gap. |

---

## 2. Mode Selection

Ask the user which scope applies.

- **Full Account Audit** — every active and recently-active ad across all campaigns, last 30 days. Default.
- **Single Campaign Audit** — one specific campaign, last 30 days.
- **Subset Audit** — user supplies a date range, campaign filter, or ad-name pattern.

If unsure: default to Full Account Audit, last 30 days.

---

## 3. The UCJ (User Conversion Journey) Metric Stack

The canonical column order for any creative audit. Read every ad left-to-right through this stack — the position where the funnel breaks tells you the problem.

### 3.1 The stack

| Position | Metric | What it diagnoses if it fails |
|---|---|---|
| 1 | **Amount Spent** | Meta's confidence signal. Low spend on a recent ad = Meta tested and de-prioritized it. |
| 2 | **Results** | Total conversions. Low here with high spend = the ad delivered impressions but didn't convert. |
| 3 | **CPM** | Cost per 1,000 impressions. Driven by hook rate + first-frame rate. High CPM = audience not engaging in first 3 seconds. |
| 4 | **CTR (link)** | Link click-through rate. Driven by offer + CTA + desire built in the ad. Low CTR with healthy CPM = the offer is weak. |
| 5 | **CPC (link)** | Cost per link click. CPC = byproduct of CPM × CTR. CPC alone tells you nothing — decompose. |
| 6 | **Cost per LPV** | Cost per landing page view. Gap between CPC and CPLV = page load / bounce problem. |
| 7 | **CR (Leads or Purchases / Link Clicks)** | Conversion rate. Custom-built metric (Results ÷ Link Clicks × 100). Low CR with healthy CPC = ad↔LP incongruency or LP friction. NOT the ad's problem. |

### 3.2 Reading the stack

For each ad: read left-to-right. The first position that fails relative to the personal-best benchmark (Section 5) is the structural mistake. Everything downstream of that point is symptom, not cause.

### 3.3 Sort-by-spend-first rule

Always sort ads by spend, highest first, before reading the stack. The highest-spend ad has the most weight on account-level metrics — diagnose it before lower-spend ads.

### 3.4 Save the preset

Recommend the user save the UCJ stack as a Meta Ads Manager column preset and share it across the team. This makes the audit framework reproducible without the skill.

---

## 4. Format Detection + Audience Stage Classification

Every ad gets two labels before scoring.

### 4.1 Format detection

Detect creative format from the ad object. Maps to the KPI set:

| Format | "Hook" equivalent | "Hold" equivalent | Core KPI set |
|---|---|---|---|
| **Video** | Hook rate (3s plays / impressions) | Hold rate (15s plays / 3s plays) | + First-frame rate (1s plays / impressions), CTR, CPM, frequency |
| **Static image** | CTR-as-thumb-stop (only attention signal) | n/a (no time-on-ad metric) | CTR, CPM, CPC, CR, frequency |
| **Carousel** | First-card CTR | Card-swap-through rate (if exposed) | CTR, CPM, frequency |
| **Collection** | Outbound CTR | Product-tap rate (if exposed) | CTR, CPM, frequency |

If format can't be detected from MCP data, ask the user OR default to "Video" if 3s/15s plays are non-zero, "Static" otherwise.

### 4.2 Audience stage classification

Detect the funnel stage from three signals, in priority order:

1. **Ad set name conventions** — Look for patterns: `_rt`, `_bofu`, `_warm`, `_broad`, `_lal`, `_lookalike`, `_engagers`, `_customer`, `_retargeting`. Most reliable signal because teams actually use these.
2. **Custom audience metadata** — Call `ads_get_custom_audience` for the ad set's audience. Pull `subtype`, `description`, `approximate_count`. Maps:
   - `WEBSITE` / `ENGAGEMENT` / `VIDEO` custom audiences → Warm or Hot
   - `CUSTOMER_FILE` / `LOOKALIKE` of customers → Existing customers
   - `LOOKALIKE` of website visitors → Warm
   - No custom audience, broad interest or LAL of large size → Cold
3. **Campaign objective + audience size** — Sales objective with audience size > 5M = Cold. Sales objective with audience size < 100K = likely Warm or Hot. Awareness / Engagement = Cold.

The four stages:

| Stage | Detection signal examples | Expected CTR ballpark (US ecomm) |
|---|---|---|
| **Cold / TOFU** | Broad, interest-based, lookalike (large), prospecting | ~0.8–1.5% top-Q |
| **Warm / MOFU** | Video viewers, IG engagers, partial site visits, LAL of customers | ~1.5–2.5% top-Q |
| **Hot / BOFU / Retargeting** | Site visitors, ATC, View Content, 30-day audiences | ~2.5–4.5% top-Q |
| **Existing customers** | Purchaser uploads, customer file matches | depends on offer |

If an ad cannot be classified with confidence: surface it to the user for manual labeling. Do not guess.

### 4.3 Placement classification

Pull the dominant placement (Feed / Reels / Stories / Audience Network) from MCP `placement_position` breakdown. Used for benchmark derivation (Section 5.3) and for the Channel ROAS Split Read (Section 9.5).

---

## 5. Personal-Best Benchmark Derivation

The single most important step. We do not use industry averages or Dara Denney's published benchmarks. We derive the bar from the user's own account performance — per the Olympic-Sprinter Personal-Best Benchmarking Doctrine.

### 5.1 What to derive

For every `(format × stage × placement)` cell that contains at least 10 ads with non-zero spend in the last 30 days, derive:

- **CTR**: median, top-quartile median
- **CPM**: median, top-quartile median (lower = better)
- **CPC**: median, top-quartile median (lower = better)
- **CPA**: median, top-quartile median (lower = better)
- **Cost per LPV**: median, top-quartile median (lower = better)
- **CR (Leads ÷ Link Clicks)**: median, top-quartile median
- **Hook rate**: median, top-quartile median (video cells only)
- **Hold rate**: median, top-quartile median (video cells only)
- **First-frame rate**: median, top-quartile median (video cells only)
- **Frequency**: median, fatigue threshold (top 25% by frequency)
- **Best historical performance** (if ≥90 days of historical data available): the single best CPA, CTR, CPM, ROAS the account has EVER hit on a comparable cell

The top-quartile median is "the bar" — that's what a strong ad in this cell, in this account looks like.

### 5.2 Sample size guard

If a `(format × stage × placement)` cell has fewer than 10 ads, fall back in this order:
1. Drop the placement dimension → benchmark by `(format × stage)`
2. Drop the stage dimension → benchmark by format only
3. Fall back to account-wide top-quartile
4. Last resort: industry default (Dara Denney's published benchmarks)

Always surface the fallback in the report so the user knows which benchmarks are weak.

### 5.3 Filter to sales-objective only

Per the standing rule from Creative Iteration Engine, benchmark derivation MUST filter to sales-objective campaigns only (`OUTCOME_SALES`, `CONVERSIONS`, `PRODUCT_CATALOG_SALES`). Including awareness, traffic, lead-gen campaigns skews CPM upward and CR downward, breaking the bar.

### 5.4 The CPM↔Hook Rate proportionality flag

Per the Three-Second Hook Doctrine: "CPM is ~proportional to hook rate." When CPM is high but hook rate is healthy (top-Q), do not diagnose as a hook problem — investigate audience size / placement saturation instead. Flag this divergence in the report.

### 5.5 The CPM↔CTR tradeoff law

Per the Internal 12/02 teaching: "You rarely win CPM and CTR simultaneously." Ads that win both are exceptional and should be flagged for replication. Do not penalize a strong-CTR ad for mid-pack CPM if CTR is genuinely top-Q.

---

## 6. Per-Ad Scoring and CPC Decomposition

For each ad in the account:

### 6.1 Apply the UCJ stack

Read each ad left-to-right through the stack. For each position, compare against the cell's top-quartile bar. Result: a pass / fail / borderline classification per position.

### 6.2 CPC Decomposition

If CPC is flagged as high (fail or borderline), do NOT call CPC the problem. CPC = CPM × CTR mathematically — decompose:
- High CPM + healthy CTR → first-frame / hook problem (the audience isn't engaging early)
- Healthy CPM + low CTR → offer / CTA problem (the audience is watching but not clicking)
- High CPM + low CTR → both — start with the hook because hook drives CPM

This rule overrides any naive "high CPC = kill" verdict.

### 6.3 Metric-to-Diagnosis Mapping

Per the GrowDent 11/13 framework, apply this explicit logic to assign a verdict:

| Symptom | Verdict | Action |
|---|---|---|
| High CPM (above median) + low hook rate / first-frame rate | **First-3-second / first-frame problem** | Refresh thumbnail or opening shot |
| High CPM + healthy hook rate but low hold rate | **Body / mid-video problem** | Rewrite seconds 3-15 |
| Healthy CPM + low CTR | **Offer / CTA problem** | Rewrite the offer or CTA |
| Healthy CPM + healthy CTR + high CPA / low CR | **LP congruency or LP friction problem** | Audit the landing page, not the ad |
| Healthy CPM + healthy CTR + healthy CR + rising frequency + falling CTR over trend | **Fatigue** | Refresh creative (hand off to Ad Fatigue Detector) |
| Top-Q on every metric + winner-by-CPA + spent ≥ 4× CPA | **Real Winner** | Hand off to Creative Iteration Engine for scaling iterations |
| Spent < 4× CPA | **Untested / insufficient data** | Let it run further OR kill if budget is tight |
| Top-Q hook rate + top-Q hold rate + low spend allocated by Meta | **Budget-starved winner** | Increase budget allocation manually or push into CBO scaling campaign |

Every ad gets exactly one verdict. If multiple problems are present, pick the upstream one (per UCJ stack order).

---

## 7. Tier Classification

Once every ad has a verdict, sort into four tiers per the original framework:

| Tier | Criteria |
|---|---|
| **Winner** | Top 20% by CPA / ROAS in its cell, with spend ≥ 4× CPA (statistical significance gate per CIE Filter 1) |
| **Workhorse** | Middle 60% — within median ± 25% on the primary KPI for its cell, with meaningful spend |
| **Loser** | Bottom 20% by CPA / ROAS, with spend ≥ 4× CPA |
| **Untested** | Spend < 4× CPA — not statistically meaningful yet |

Per the 80%-Creative Doctrine: "80% of results come from creative." Frame the tier breakdown as a creative-quality distribution, not just a cost distribution.

---

## 8. Stage-Balance Check

A funnel-shape diagnostic that catches problems invisible at the per-ad level.

Look at the spend distribution across stages:

| Distribution | Diagnostic |
|---|---|
| > 70% spend on Cold / TOFU | Funnel is top-heavy. Likely no retargeting pool being built — losing post-click revenue. |
| > 70% spend on Hot / BOFU | Funnel is bottom-heavy. The retargeting pool will dry up — no new audience entering. Imminent collapse risk. |
| Roughly balanced (40-30-30 or similar) | Healthy funnel shape. |
| No BOFU spend at all | Critical — every site visitor / abandoned cart is being wasted. |
| No TOFU spend at all | Critical — funnel will starve when retargeting pool exhausts. |

Surface this finding prominently in the executive summary. Most "creative performance" problems are actually funnel-shape problems.

---

## 9. Pattern Analysis

### 9.1 Winners pattern

Across all Winner-tier ads, look for common elements:
- Format (Video / Static / Carousel)
- Audience stage
- Hook style (per ad name conventions or visual analysis)
- Angle (per ad name conventions)
- Placement
- Time of day / day of week (if breakdowns available)

Surface the dominant pattern. Example: "5 of 7 winners are Founder Video on Cold / TOFU placement Reels — that's the winning DNA in this account."

### 9.2 Losers pattern

Across all Loser-tier ads, look for common elements. What format / stage / hook style consistently underperforms? Surface as "avoid" guidance.

### 9.3 Fatigue pattern

Across ads showing fatigue signals (high frequency + falling trend), look for common patterns: same template family, same hook approach, same campaign. Surface as a refresh priority list with handoff to Ad Fatigue Detector.

### 9.4 Untested pattern

Across Untested ads, look for common reasons: low Meta allocation (Meta deprioritized them), recently launched (just need more time), poor early signal. Surface as "what to kill vs. what to let cook."

### 9.5 Channel ROAS Split Read

Per the Kass Care 2/5 framework: read ROAS / CPA by placement (Feed vs Reels vs Stories vs Audience Network). Surface placement-specific winners and losers. Often a Cold/TOFU ad performs differently on Reels vs Feed — diagnose each placement separately.

---

## 10. Diagnostic Action Plan + Series Handoffs

The action plan is per-ad, with explicit handoffs to downstream skills.

### 10.1 Per-ad row in the action plan

Each ad gets exactly:
- **Verdict** (from Section 6.3)
- **Action**: Kill / Refresh / Iterate / Scale / Audit LP / Let Cook
- **Series handoff**: which downstream skill to route this ad's problem to
- **Priority**: High / Medium / Low (by spend impact)

### 10.2 Series handoffs (the MEASURE → MAKE chain)

| Verdict | Handoff |
|---|---|
| Real Winner | **Creative Iteration Engine** — generate 10 iterations to scale without burning out |
| Fatigue | **Ad Fatigue Detector** — diagnose fatigue depth + refresh plan |
| LP congruency / LP friction | **Landing Page Audit** — audit the LP, not the ad |
| First-3-second problem | **Ad Hook Generator** (PLAN tier) — fresh hooks |
| Offer / CTA problem | **Ad Copy Writer** (MAKE tier) — rewrite CTA / offer |
| Budget-starved winner | Internal — increase budget allocation. Optionally feed into CIE for variations to test |
| Untested | Let cook for 7 more days OR kill if budget pressure |

This makes the audit the entry point to the entire skill series.

### 10.3 Stage-balance recommendation

If the Stage-Balance Check (Section 8) flagged a funnel-shape problem, surface a top-priority recommendation in the executive summary above the per-ad actions. Funnel-shape fixes take precedence over per-ad creative work.

---

## 11. What you need from the user

Summary of inputs. The skill should not start scoring without these.

**Required:**
1. Account access (via Meta Ads MCP) OR ad-level CSV export from last 30 days
2. Account goal (CPA, ROAS, leads, revenue — which KPI matters most)
3. Time window (default: last 30 days)

**Helpful context:**
- Account's historical best CPA / ROAS / CTR — for Personal-Best Benchmarking
- Recent changes (new creative launched, budget changes, audience changes, LP changes)
- Naming conventions for ad sets (especially audience stage indicators)

If anything is missing, ask before generating. Do not guess audience stages.

---

## 12. Output Document Structure

Generate a branded Baweja Media DOCX. Follow `references/branding.md` for the brand system.

### 12.1 Sections

1. **Cover page** — "Creative Performance Audit" + brand name + date range + mode (Full / Campaign / Subset)
2. **Preflight Check Summary** — what capabilities were used (MCP / CSV; audience metadata available or not; trend available or not)
3. **Executive Summary** — Account-health snapshot in five bullets: total ads analyzed, winner/workhorse/loser/untested counts, the single most important finding, the Stage-Balance Check verdict, top 3 actions
4. **Personal-Best Benchmarks Grid** — the `(format × stage × placement)` benchmark matrix with sample-size notes
5. **Stage-Balance Check** — funnel-shape diagnosis, distribution chart (text-based), verdict
6. **Performance Tier Breakdown** — every ad categorized as Winner / Workhorse / Loser / Untested. Sorted by spend within each tier. UCJ stack columns shown.
7. **Per-Ad Diagnosis Cards** — each ad gets a card: name, format, stage, placement, UCJ-stack metric readout, verdict, action, series handoff
8. **Winner Pattern Analysis** — what's common across winners (DNA)
9. **Loser Pattern Analysis** — what's common across losers (avoid list)
10. **Fatigue Watchlist** — ads with fatigue signals, prioritized by spend
11. **Channel ROAS Split Read** — ROAS by placement, callouts for placement-specific winners
12. **Diagnostic Action Plan** — prioritized list of actions with series handoffs
13. **Series Handoffs Summary** — explicit list: "These 3 ads → Creative Iteration Engine. These 2 ads → Ad Fatigue Detector. This LP needs Landing Page Audit."
14. **Work with us** — Baweja Media CTA

### 12.2 Quality standards

- Every verdict must trace back to specific data. "Your hook rates are low" is useless. "Ads 3, 7, 12 have hook rates below 15% (your account top-Q bar is 28%) — the common element is they all open with a generic question rather than a specific claim" is actionable.
- The audit is diagnosis only. No solutions. The action plan says what to do but the HOW is delegated to downstream skills.
- If data is insufficient (too few impressions, too short time period, missing video metrics), say so explicitly and skip that section. Do not draw conclusions from noise.
- Distinguish correlation from causation. "Ad 5 has the best CPA AND the highest hook rate" is a correlation. The hook may not be the driver.
- Per the Measurable-vs-Abstract rule: only cite metrics measurable in the Meta dashboard. Do not invent or estimate metrics that aren't there.
- Include at least one **contrarian insight** — something that goes against conventional wisdom based on this account's specific data.

---

## 13. Shared Modules — for Episodes 9, 10 to inherit

This skill inherits from Creative Iteration Engine and exposes one new module for downstream skills.

### 13.1 Inherited from Creative Iteration Engine

- **Data Import Protocol** (CIE Section 7.1) — Meta Ads MCP → CSV → manual paste ladder
- **Winner Validation Gate** (CIE Section 7.2) — 4-Filter test for declaring a winner real

Do not duplicate. Reference these sections.

### 13.2 New module exposed: Per-Ad Diagnosis Framework (Section 6.3)

The Metric-to-Diagnosis Mapping table is reusable. Ad Fatigue Detector (Episode 9) should inherit this for the fatigue verdict. Landing Page Audit (Episode 10) should inherit the "LP congruency" diagnosis path.

---

## 14. Series context

This is skill 8 of 10 in the AI Skills for Media Buyers series by Baweja Media. It sits in the MEASURE tier and is the entry point to the entire series — it diagnoses, then routes problems to the right downstream skill.

- Upstream (consumes from): standalone — needs only account data
- Downstream (feeds into): Creative Iteration Engine (winners), Ad Fatigue Detector (fatigue), Landing Page Audit (LP problems), Ad Hook Generator / Ad Copy Writer (per-component fixes)

Series philosophy: the audit's job is to find the structural mistake. Then hand off to the skill that fixes it. The audit alone is diagnosis-only — solutions live downstream.

---

Built by Baweja Media · bawejamedia.com
