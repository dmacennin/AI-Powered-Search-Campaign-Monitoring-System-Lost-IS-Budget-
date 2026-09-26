# Search Lost IS (Budget) Monitoring & Optimization Decision-Support System

**Proprietary decision-support system for Google Ads Search.**

Developed by [Dominic Mac-Ennin, CDMP, PCM®](https://www.linkedin.com/in/dominicmacennin) — Lead PPC Specialist.

This repository is **portfolio documentation only**. It is not source code and it is not a Cloud job. The operator guide, execution script, GAQL, prompts, and decision matrix are withheld.

This system stands alone. It is not an add-on to the keyword research or negative-keyword pipelines.

---

## What it does

Monitors Google Ads Search campaigns for **Search Impression Share Lost to Budget**, then produces a structured Slack / Teams briefing three times per week. Each flagged campaign is classified (how severe the constraint is, whether it is structural or a recent-change spike, and whether the limiter is budget, rank, both, or neither) and given **one primary recommended action**.

The system does not change budgets, bids, schedules, or keywords. Cards are decision support for an operator.

## The problem it solves

Lost IS (Budget) is a leading indicator that eligible Search demand is being throttled by daily budget. Most PPC reporting describes traffic already bought (CPC, CPA, ROAS). This metric describes traffic that never got a chance to be bought.

Without a repeatable read, operators either:

- chase the number daily and over-react to noise, or
- ignore it until month-end volume is already missed, or
- raise budget on a campaign that is actually rank-constrained or already over the monthly spend plan.

The system exists to make the same diagnosis, in the same order, every Monday, Wednesday, and Friday — with enough companion context that the operator does not have to rebuild the case from scratch in the Google Ads UI.

## How it works

The run is split on purpose: **scripts classify, the language model only writes.** Cut-offs, labels, diagnosis, and the action code are deterministic. The LLM receives an already-decided payload and turns it into an operator card. It does not invent tiers or reorder the report.

**Cadence**  
Monday / Wednesday / Friday — not daily. The gap is long enough to see whether a previous change moved the number, and short enough to cover weekday and weekend effects without living in the account.

**Two time windows**  
Every run uses a structural lookback (about a week) to decide the severity band, plus a shorter “since last report” window so the operator can see what changed after the previous check.

**Severity and depth**  
Campaigns are banded by how much eligible Search share is being lost to budget. Mild cases stay at campaign level unless the campaign is material to account spend. Clear and heavy cases drill to ad group so the operator can see *which parts* of the campaign are actually constrained.

**Structural vs post-change**  
A 72-hour change-history filter separates a persistent budget problem from a spike that followed a deliberate budget, bid, or keyword change. Post-change cases are monitored, not treated as a new strategy on the same day.

**Companion context (named, not fully specified here)**  
Each card carries the minimum context required to choose a lever:

- Monthly pacing vs intended spend (ahead / on track / behind)
- CPA vs target or a recent baseline
- Conversion volume (is this campaign actually producing?)
- Lost IS (Rank) — so budget-loss is not confused with a rank problem
- Peak-hour concentration, when losses cluster in a few hours rather than all day

**Diagnosis**  
Budget-loss and rank-loss are read together:

| Combined read | Meaning for the operator |
| --- | --- |
| Money problem | Budget is limiting delivery; rank is not |
| Rank problem | The ad is losing the auction; more budget alone will not fix it |
| Dual constraint | Budget and rank are both limiting delivery |
| Neither constraint | The auction is not the bottleneck (demand, eligibility, tracking, or offer) |

**One primary action**  
The system emits a single lead recommendation (increase budget, reallocate from weaker segments, small test, rank work first, efficiency first, monitor, or “not an auction problem”). Dayparting is supporting only, and only when losses are time-concentrated. If the monthly plan is already ahead, the system will not pretend extra budget exists — it points to reallocation instead.

## Architecture (conceptual)

```
Scheduler (Mon / Wed / Fri)
        |
        v
Cloud job
        |
        +------------------+
        |                  |
        v                  v
Google Ads API      Config Sheet
(performance)       (targets, account list)
        |                  |
        +------------------+
                 |
                 v
     Script classifies
     (severity, diagnosis, one action)
                 |
                 v
     LLM writes the card only
                 |
                 v
     Slack / Teams  +  run log
```

Multi-account shape:

```
One Cloud project
        |
        +-- Ads account A  -->  Slack / Teams A
        +-- Ads account B  -->  Slack / Teams B
        +-- Ads account C  -->  Slack / Teams C
```

The language model never assigns tiers or chooses the action. If it is unavailable, a raw structured fallback still carries the numbers and the action code.

## Output structure

| Surface | Contents |
| --- | --- |
| Slack / Teams channel (per account) | Priority-ordered cards: diagnosis, numbers beside every label, one primary action, optional supporting action |
| Monitoring block | Post-change spikes — watch, do not treat as new strategy |
| Config Sheet | Account list, monthly spend targets, CPA targets — the operator-edited surface |
| Run log | What ran, what was flagged, what failed |

Cards always show the figure next to the label (for example CPA vs target, not “Healthy” alone).

## Design principles

- Scripts classify; the language model only narrates. Deterministic rules own tiers, diagnosis, and action codes.
- One primary action per entity. Extra levers are supporting only.
- Numbers travel with labels. A status word without the figure is not an operator card.
- Campaign is the decision unit (where budget lives). Ad group is the diagnostic unit.
- v1 recommends. Humans apply every in-platform change.

## Technical stack

| Component | Role |
| --- | --- |
| Cloud Run + Cloud Scheduler | Multi-account runner (Mon / Wed / Fri) |
| Google Ads API | Performance, change history, hour-of-day, optional keyword pull |
| Google Sheet (config) | Targets and account list — operator-edited |
| LLM API (Claude or Grok) | Card narrative only |
| Slack / Teams | Operator inbox |

## What is intentionally not published

Exact tier cut-offs, the full action matrix, dayparting case rules, keyword-level reallocation helper logic, GAQL, prompts, and source code are withheld. Those live in the internal specification.

This write-up is the conceptual design only.

## Intellectual Property

This system is a proprietary work. All rights reserved.

© 2026 Dominic Andoh Mac-Ennin.

This repository is published for professional documentation and portfolio purposes. The system logic, classification frameworks, and specification documents are not open-source and are not licensed for use, reproduction, or adaptation without a separate written agreement.

## Licensing & Enquiries

Commercial licensing, white-label arrangements, and professional enquiries welcome.

**hello@dominicandohofficial.com**

---

*Dominic Mac-Ennin, CDMP, PCM® — Lead PPC Specialist*  
*Google Ads · Microsoft Ads · Meta Ads · AI-Powered SEM systems*