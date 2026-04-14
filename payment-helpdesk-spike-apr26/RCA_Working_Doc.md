# Root Cause Analysis — Payment Helpdesk Spike

**April'26 vs March'26 | Wiom**

---

## TLDR

_Filled last. 3-4 lines that a stakeholder can read in 10 seconds and know the headline._

> **To be filled after all analysis is complete.**
> Format: Universe size. Spike size. Main takeaway in one sentence. Top root cause in one sentence.

---

## Status Tracker

| Phase | Description | Status | Key Finding (one-liner) |
|-------|-------------|--------|------------------------|
| Phase 1 | Observation Data Base | DONE | Mar n=6,592 (30d), Apr n=4,998 (13d). Daily avg: 219.7 → 384.5 = 75% spike. |
| Phase 2 | Headline Numbers — Quantify the Spike | NOT STARTED | — |
| Phase 3, Cut 1 | By Issue Type + Txn Data Verification | NOT STARTED | — |
| Phase 3, Cut 2 | By Payment Journey + Txn Data Verification | NOT STARTED | — |
| Phase 3, Cut 3 | By Customer Cohort (PayG vs Non-PayG) + Txn Data Verification | NOT STARTED | — |
| Phase 3, Cut 4 | By Payment Method + Txn Data Verification | NOT STARTED | — |
| Phase 3, Cut 5 | By Timeline — Correlation with Events + Txn Data Verification | NOT STARTED | — |
| Phase 3, Cut 6 | By Repeat vs First-time Complainants | NOT STARTED | — |
| Phase 3, Cut 7 | By Retry Behaviour + Txn Data Verification | NOT STARTED | — |
| Phase 3, Cut 8 | By Refund-Related vs Transaction-Related + Txn Data Verification | NOT STARTED | — |
| Phase 3.5 | Qualitative Validation (VOC Analysis) | NOT STARTED | — |
| Phase 4 | Root Cause Chains | NOT STARTED | — |
| Phase 4.5 | What is NOT a Root Cause | NOT STARTED | — |
| Phase 5 | Payment Funnel per Journey (Metabase + Juspay) | NOT STARTED | — |
| Phase 6 | Final RCA Assembly | NOT STARTED | — |

**Approach note:** Transaction data cross-verification (Metabase/Juspay) is done within each cut — not as a separate phase. Each cut's finding is written only after both ticket data AND transaction data confirm the pattern.

**Last Updated:** 15 Apr 2026
**Overall Status:** Approach finalized. Awaiting raw ticket data.

---

## Context & Problem Statement

### Why this RCA exists

A significant spike in payment-related helpdesk tickets was observed in April 2026 compared to March 2026. This RCA investigates the root causes behind the spike.

### The question this RCA answers

**Why did payment-related helpdesk tickets spike in April 2026 compared to March 2026 — and what is structurally driving the increase?**

### Scope

- **In scope:** All payment-related helpdesk tickets across all payment journeys (Booking, Install, Renewal, Wiom Net), all payment methods (UPI, Cards, Net Banking, QR), and all customer cohorts (PayG, Non-PayG).
- **Out of scope:** Non-payment helpdesk tickets (connectivity, onboarding, app issues unrelated to payments). These will be flagged under Cross-Theme Flags if encountered during investigation.

---

## Definitions

_Key terms used in this document. Ensures every reader interprets findings the same way._

| Term | Definition |
|------|-----------|
| **PayG** | Pay-as-you-go customer — no long-term commitment, pays per recharge cycle |
| **Non-PayG** | Plan-based / subscribed customer — committed plan with recurring billing |
| **Juspay** | Payment orchestration layer that routes transactions to multiple payment gateways (PGs) |
| **PG (Payment Gateway)** | Backend service that processes the actual payment (e.g., Paytm, Razorpay, PhonePe, PayU) |
| **UPI Auto-pay** | Recurring UPI mandate — automatic debit from customer's bank account on renewal date |
| **Manual Payment** | Customer-initiated payment in the Wiom app (UPI, card, net banking, or QR) |
| **QR Code Payment** | Customer generates a QR code in the app; they or someone else scans it to pay |
| **Booking Payment** | One-time payment made when a new customer books a Wiom connection |
| **Install Payment** | One-time Rs.300 payment made at the time of device installation |
| **Renewal Payment** | Payment made to continue/recharge an existing Wiom plan |
| **Wiom Net** | Pay-per-use public WiFi service |
| **VOC** | Voice of Customer — verbatim customer complaint or feedback from helpdesk |
| **Ticket** | A single helpdesk complaint raised by a customer |
| **Transaction** | A single payment attempt recorded in Juspay/PG systems |
| **Spike** | The incremental increase in April tickets over March baseline (April count minus March count) |
| **USER_DROPPED** | Juspay error when payment session times out without the user completing payment |
| **BUSINESS_ERROR** | Juspay error from bank-side or business logic (e.g., insufficient balance, mandate expired) |

_Additional terms will be added as they emerge during analysis._

---

## Payment Flow Architecture

_How the payment system works at Wiom — context for readers who don't know the internals._

### Payment Journeys

```
1. BOOKING PAYMENT (one-time)
   New customer → Booking flow in app → Checkout screen → Payment via Juspay → Booking confirmed

2. INSTALL PAYMENT (one-time, Rs.300)
   Device installation scheduled → Technician arrives → Customer pays Rs.300 → Installation proceeds

3. RENEWAL — MANUAL (one-time per cycle)
   Plan nearing expiry → Customer opens app → Selects plan → Checkout screen → Payment via Juspay → Plan renewed

4. RENEWAL — UPI AUTO-PAY (recurring)
   Customer has active UPI mandate → System triggers auto-debit on renewal date → Bank processes debit → Plan renewed

5. WIOM NET (pay-per-use)
   Customer at public WiFi location → Connects to Wiom Net → Pays for session → WiFi access granted
```

### Payment Methods Available

| Method | Type | Used In |
|--------|------|---------|
| UPI (direct) | Digital — customer pays from their UPI app | All journeys |
| Cards (debit/credit) | Digital — customer enters card details | All journeys |
| Net Banking | Digital — customer logs into bank website | All journeys |
| QR Code | Digital — QR generated in app, scanned by customer or shared with someone else to pay | All journeys |
| UPI Auto-pay (mandate) | Recurring — automatic debit, no customer action needed per cycle | Renewal only |

### Payment Stack

```
Customer App → Wiom Backend → Juspay (orchestrator) → Payment Gateway (Paytm/Razorpay/PhonePe/PayU/others) → Bank
```

Juspay sits in the middle and routes each transaction to a PG. Multiple PGs are active. Juspay decides which PG handles which transaction based on routing rules.

---

## SECTION 1: Theme Overview

| Field | Value |
|-------|-------|
| Theme Name | Payment-Related Helpdesk Spike |
| Owner(s) | Yash Gupta |
| Failure Zone | Customer Payment Attempt → Successful Payment Completion |
| Tickets Impacted (March) | _To be filled_ |
| Tickets Impacted (April) | _To be filled_ |
| Spike Size (absolute + %) | _To be filled_ |
| Date of Submission | _To be filled_ |

---

## SECTION 2: Observation Data Base

_Every finding is only as reliable as the data underneath it._

| Parameter | Value |
|-----------|-------|
| Ticket Cohort Period | March 1–30, 2026 and April 1–13, 2026 |
| Observation Window | Data pulled on 15 Apr 2026 |
| Filters / Exclusions | None applied yet. May be added as analysis progresses. |
| Sample Size | March: n = 6,592 (30 days) / April: n = 4,998 (13 days) |
| Comparison Method | Per-day average comparison. March avg: 219.7 tickets/day. April avg: 384.5 tickets/day. 75% spike in daily ticket rate. |
| Data Source(s) | Helpdesk raw export (Kapture CRM). Additional sources (Metabase, CleverTap, Juspay) to be added as investigation deepens. |
| Known Limitations | 1. April data covers only 13 days — full month trend not yet visible. 2. March 4 was Holi — excluding it from per-day average would push March baseline slightly higher. |

---

## SECTION 3: What the Data Shows

### Headline Numbers

_Total payment tickets March vs April. Day-by-day trend. Spike shape._

> **To be filled after Phase 2**

---

### Cut 1: By Issue Type

_Which categories of payment issues are driving the spike?_

| Issue Type | March Count | April Count | Change (abs) | Change (%) | % of Total Spike |
|------------|-------------|-------------|--------------|------------|-----------------|
| Payment failed / transaction declined | | | | | |
| Payment deducted but service not activated | | | | | |
| Double charge / extra deduction | | | | | |
| Refund not received / delayed refund | | | | | |
| Auto-pay (UPI mandate) failed | | | | | |
| Auto-pay deducted unexpectedly | | | | | |
| QR code payment issue | | | | | |
| Payment screen error / app crash during payment | | | | | |
| Other | | | | | |
| **TOTAL** | | | | | |

**Finding:** _To be filled_
**Sample Size:** n = ___
**Source:** Helpdesk raw export

---

### Cut 2: By Payment Journey

_Which journey is causing the most pain?_

| Payment Journey | March Count | April Count | Change (abs) | Change (%) | % of Total Spike |
|----------------|-------------|-------------|--------------|------------|-----------------|
| Booking payment | | | | | |
| Install payment (Rs.300) | | | | | |
| Renewal — manual (in-app) | | | | | |
| Renewal — UPI auto-pay | | | | | |
| Wiom Net (public WiFi) | | | | | |
| Unknown / Unidentifiable | | | | | |
| **TOTAL** | | | | | |

**Finding:** _To be filled_
**Sample Size:** n = ___
**Source:** Helpdesk raw export

---

### Cut 3: By Customer Cohort (PayG vs Non-PayG)

| Cohort | March Count | April Count | Change (abs) | Change (%) | % of Total Spike |
|--------|-------------|-------------|--------------|------------|-----------------|
| PayG | | | | | |
| Non-PayG | | | | | |
| Unknown | | | | | |
| **TOTAL** | | | | | |

**Finding:** _To be filled_
**Sample Size:** n = ___
**Source:** Helpdesk raw export + Metabase (for cohort mapping)

---

### Cut 4: By Payment Method

| Payment Method | March Count | April Count | Change (abs) | Change (%) | % of Total Spike |
|---------------|-------------|-------------|--------------|------------|-----------------|
| UPI | | | | | |
| Cards | | | | | |
| Net Banking | | | | | |
| QR Code | | | | | |
| Unknown / Not mentioned | | | | | |
| **TOTAL** | | | | | |

**Finding:** _To be filled_
**Sample Size:** n = ___
**Source:** Helpdesk raw export

---

### Cut 5: By Timeline — Correlation with Events

_Day-by-day ticket trend overlaid with known events._

| Date | Ticket Count | Notable Event (if any) |
|------|-------------|----------------------|
| _To be filled day by day_ | | |

**Known events to check:**
- [ ] App version releases in March / April
- [ ] Juspay / PG planned or unplanned downtime
- [ ] PG routing changes
- [ ] Plan or pricing changes
- [ ] Bulk renewal dates
- [ ] Total transaction volume changes (growth vs failure rate increase)

**Finding:** _To be filled_
**Source:** Helpdesk raw export + Juspay dashboard + Engineering release logs

---

### Cut 6: By Repeat vs First-time Complainants

| Complainant Type | March Count | April Count | Change (abs) | Change (%) |
|-----------------|-------------|-------------|--------------|------------|
| First-time complainant (no prior payment ticket) | | | | |
| Repeat complainant (had prior payment ticket) | | | | |
| Unknown | | | | |

**Finding:** _To be filled_
**Sample Size:** n = ___
**Source:** Helpdesk raw export (customer ID cross-check)

---

### Cut 7: By Retry Behaviour (from VOCs)

_Did customers attempt multiple times before raising a ticket?_

| Retry Pattern | April Count | % of April Tickets | Interpretation |
|--------------|-------------|-------------------|----------------|
| Multiple retries mentioned in VOC | | | Intermittent / flaky failure |
| Single attempt, then raised ticket | | | Hard failure or UX confusion |
| Not identifiable from VOC | | | Instrumentation gap |

**Finding:** _To be filled_
**Sample Size:** n = ___
**Source:** VOC text analysis

---

### Cut 8: By Refund-Related vs Transaction-Related

_Separate "money went but service didn't come" from "money didn't go at all"_

| Category | March Count | April Count | Change (abs) | Change (%) | % of Total Spike |
|----------|-------------|-------------|--------------|------------|-----------------|
| **Transaction failure** (money didn't go) | | | | | |
| **Post-payment failure** (money went, service didn't activate) | | | | | |
| **Refund issue** (refund not received / delayed) | | | | | |
| Unknown | | | | | |
| **TOTAL** | | | | | |

**Finding:** _To be filled_
**Sample Size:** n = ___
**Source:** Helpdesk raw export + VOC analysis

---

## SECTION 3.5: Qualitative Validation (VOC Analysis)

_Quantitative cuts tell us WHAT happened. VOCs tell us what the customer EXPERIENCED. This section structures VOC evidence per issue type — not as anecdotes, but as patterns with sample sizes._

### Methodology

- **Sample:** _To be filled (e.g., "Read all April payment VOCs, n = ___")_
- **Approach:** Group VOCs by issue type from Cut 1. For each group, identify recurring phrases, patterns, and customer sentiment.
- **Rule:** One VOC is context. A pattern across 5+ VOCs is evidence. State the sample size for every pattern claimed.

### VOC Patterns by Issue Type

| Issue Type | VOC Sample Size | Recurring Pattern | Representative Quotes (2-3) |
|------------|----------------|-------------------|---------------------------|
| _To be filled per issue type_ | n = ___ | | |
| | | | |
| | | | |
| | | | |
| | | | |

### Key Qualitative Insights

1. _To be filled_
2. _To be filled_
3. _To be filled_

---

## SECTION 4: Root Cause Chains

_For each major finding from Section 3, build the chain. Each root cause labelled A, B, C. Start from the observable failure. Keep asking "why" until you hit a structural/systemic cause. Stop when you reach a system design decision, not a human behaviour._

### Root Cause A

> **OBSERVABLE FAILURE**
> _What is happening? State precisely with numbers._
> _To be filled_

> **WHY?**
> _First-level reason. With data._
> _To be filled_

> **WHY?**
> _Go deeper. With data._
> _To be filled_

> **WHY?**
> _Keep going until you hit a system design decision._
> _To be filled_

> **STRUCTURAL ROOT CAUSE**
> _The systemic reason. A statement about how the system is designed, not about how people behave._
> _To be filled_

**Classification:** _System Design Flaw / Coverage Gap / Product Bug / Instrumentation Gap / Process Gap_

---

### Root Cause B

> **OBSERVABLE FAILURE**
> _To be filled_

> **WHY?**
> _To be filled_

> **WHY?**
> _To be filled_

> **STRUCTURAL ROOT CAUSE**
> _To be filled_

**Classification:** _To be filled_

---

### Root Cause C

> **OBSERVABLE FAILURE**
> _To be filled_

> **WHY?**
> _To be filled_

> **WHY?**
> _To be filled_

> **STRUCTURAL ROOT CAUSE**
> _To be filled_

**Classification:** _To be filled_

---

## SECTION 4.5: What is NOT a Root Cause

_Equally important as identifying root causes is ruling out what is NOT causing the spike. This prevents stakeholders from derailing discussions with unrelated hypotheses. Every ruled-out cause must have evidence for why it was eliminated._

| # | Hypothesis Tested | Evidence That Rules It Out | Source |
|---|------------------|---------------------------|--------|
| 1 | _e.g., "Payment gateways degraded"_ | _e.g., "All PGs show similar success rates in March and April — no single PG degradation observed"_ | _Juspay dashboard_ |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 | | | |

---

## SECTION 5: Cross-Verify with Transaction Data (Metabase + Juspay)

_Once suspects identified from ticket analysis, prove or disprove with hard transaction data._

### 5.1 Transaction Success/Failure Rates

| Journey | March Success Rate | April Success Rate | Change | Source |
|---------|-------------------|-------------------|--------|--------|
| Booking | | | | |
| Install | | | | |
| Renewal — manual | | | | |
| Renewal — UPI auto-pay | | | | |
| Wiom Net | | | | |

### 5.2 Top Failure Error Codes

| Error Code | Error Description | March Count | April Count | Change | Journey Affected |
|-----------|------------------|-------------|-------------|--------|-----------------|
| | | | | | |
| | | | | | |
| | | | | | |

**Source:** Juspay dashboard

### 5.3 PG-wise Performance

| Payment Gateway | March Success Rate | April Success Rate | Change | Volume Share |
|----------------|-------------------|-------------------|--------|-------------|
| | | | | |
| | | | | |
| | | | | |

**Source:** Juspay dashboard

### 5.4 Volume vs Failure Rate

_Critical distinction: Did total transactions increase (organic growth → proportional ticket increase) or did failure RATE increase (something broke)?_

| Metric | March | April | Change |
|--------|-------|-------|--------|
| Total transactions | | | |
| Failed transactions | | | |
| Failure rate (%) | | | |
| Total helpdesk tickets | | | |
| Ticket-to-failure ratio | | | |

**Finding:** _To be filled — "Volume-driven spike" vs "Failure-rate-driven spike"_

### 5.5 App Version Analysis

| App Version | March Failures | April Failures | Notes |
|------------|---------------|---------------|-------|
| | | | |
| | | | |

### 5.6 Auto-pay Mandate Health

| Mandate Status | March Count | April Count | Change |
|---------------|-------------|-------------|--------|
| Active mandates | | | |
| Expired mandates | | | |
| Failed mandate debits | | | |
| Mandate creation failures | | | |

---

## SECTION 5.5: Payment Funnel per Journey

_Step-by-step funnel with conversion rates at each step. Modelled after the Payments Funnel RCA approach. Built from Metabase + Juspay data._

### Booking Payment Funnel

| Step | March (Unique Users) | March Step Conv. | April (Unique Users) | April Step Conv. | Change |
|------|---------------------|-----------------|---------------------|-----------------|--------|
| Checkout page loaded | | | | | |
| Payment method selected | | | | | |
| Juspay session initiated | | | | | |
| Payment success | | | | | |
| Payment failed | | | | | |

### Install Payment Funnel

| Step | March | March Conv. | April | April Conv. | Change |
|------|-------|------------|-------|------------|--------|
| _To be filled if data available_ | | | | | |

### Renewal — Manual Payment Funnel

| Step | March (Unique Users) | March Step Conv. | April (Unique Users) | April Step Conv. | Change |
|------|---------------------|-----------------|---------------------|-----------------|--------|
| Checkout page loaded | | | | | |
| Payment method selected | | | | | |
| Juspay session initiated | | | | | |
| Payment success | | | | | |
| Payment failed | | | | | |

### Renewal — UPI Auto-pay Funnel

| Step | March | March Conv. | April | April Conv. | Change |
|------|-------|------------|-------|------------|--------|
| Mandate debit triggered | | | | | |
| Debit successful | | | | | |
| Debit failed | | | | | |
| Retry triggered (if any) | | | | | |

### Wiom Net Payment Funnel

| Step | March | March Conv. | April | April Conv. | Change |
|------|-------|------------|-------|------------|--------|
| _To be filled if data available_ | | | | | |

**Source:** Metabase + Juspay
**Known Limitations:** _To be filled (e.g., instrumentation gaps, funnel events not tracked)_

---

## SECTION 6: Evidence Log

_Every piece of evidence that supports the root cause chains._

| Evidence # | Type | Description | Key Insight | Supports Root Cause | Link / Reference |
|-----------|------|-------------|-------------|-------------------|-----------------|
| 1 | _Data cut / VOC pattern / Juspay screenshot / Metabase query / PG dashboard_ | | | A / B / C | |
| 2 | | | | | |
| 3 | | | | | |
| 4 | | | | | |
| 5 | | | | | |
| 6 | | | | | |
| 7 | | | | | |
| 8 | | | | | |

---

## SECTION 7: What We Don't Know

_Unknowns are as important as knowns. List gaps honestly._

| # | Unknown / Gap | Why We Can't Answer It | What Would Help |
|---|--------------|----------------------|----------------|
| 1 | | | |
| 2 | | | |
| 3 | | | |

---

## SECTION 8: Cross-Theme Flags

_Issues found during investigation that belong to non-payment themes. Don't investigate — just pass the signal._

| Observation | Likely Relevant To | Brief Note |
|------------|-------------------|------------|
| | | |

---

## SECTION 9: Summary — Ranked Root Causes

_Top 3 by volume impact. What is broken, why, and how much it costs._

| Rank | Root Cause Statement | Failure Type | Tickets Impacted |
|------|---------------------|-------------|-----------------|
| 1 | _To be filled_ | | n = ___ |
| 2 | _To be filled_ | | n = ___ |
| 3 | _To be filled_ | | n = ___ |

---

## SECTION 9.5: Triage Classification

_Not all root causes need the same response. Classify each finding._

### Expected (not a problem — no action needed)

| Finding | Why it's expected | Tickets in this bucket |
|---------|------------------|----------------------|
| _e.g., "Ticket volume grew proportionally with transaction volume — no failure rate increase"_ | _Organic growth, not a system issue_ | n = ___ |
| | | |

### Fixable (real issues — action required)

| Finding | Why it's fixable | Tickets in this bucket | Severity |
|---------|-----------------|----------------------|----------|
| | | n = ___ | P0 / P1 / P2 |
| | | | |
| | | | |

### Needs No Action (already resolved or one-time)

| Finding | Why no action needed | Tickets in this bucket |
|---------|---------------------|----------------------|
| _e.g., "PG-X had 2-hour downtime on Apr 3, now resolved"_ | _One-time incident, already recovered_ | n = ___ |
| | | |

---

## SECTION 10: Recommended Actions

_Prioritized by customer impact. P0 = immediate (this week), P1 = next sprint, P2 = backlog._

### P0 — Immediate (stop the bleeding)

| # | Action | Root Cause Addressed | Expected Impact | Owner |
|---|--------|---------------------|----------------|-------|
| 1 | _To be filled_ | Root Cause A / B / C | | |
| 2 | | | | |

### P1 — Next Sprint

| # | Action | Root Cause Addressed | Expected Impact | Owner |
|---|--------|---------------------|----------------|-------|
| 1 | _To be filled_ | | | |
| 2 | | | | |

### P2 — Backlog (systemic / long-term)

| # | Action | Root Cause Addressed | Expected Impact | Owner |
|---|--------|---------------------|----------------|-------|
| 1 | _To be filled_ | | | |
| 2 | | | | |

---

## SECTION 11: Data Sources & Definitions Appendix

### Data Sources Used

| Source | Table / System | Used For | Access |
|--------|---------------|----------|--------|
| Helpdesk raw export | _Tool name / export file_ | Ticket-level analysis (all 8 cuts) | Shared by ops team |
| Juspay Dashboard | Juspay merchant portal | PG-wise success rates, error codes, downtime logs | Dashboard access |
| Metabase | _Specific database(s)_ | Transaction-level verification, funnel data, mandate data | Creds in WiomPulse/.env |
| Engineering release logs | _Source_ | App version releases, deployment dates | Engineering team |
| CleverTap | _If used_ | Funnel events, user-level conversion | _Access TBD_ |

### Known Limitations

| # | Limitation | Impact on Analysis |
|---|-----------|-------------------|
| 1 | _To be filled as discovered_ | |
| 2 | | |
| 3 | | |

---

## Progress Log

| Date | What was done | What's next |
|------|--------------|-------------|
| 15 Apr 2026 | Approach finalized. Working doc created with all sections. 8 data cuts agreed. RCA template and 2 sample RCAs reviewed. | Awaiting raw ticket data for March + April to begin Phase 1 and 2 |
| | | |
