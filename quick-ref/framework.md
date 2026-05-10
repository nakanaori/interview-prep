# 5-Step Interview Framework

Use this structure for every system design question.

---

## The 5 Steps

| Step | Time | Focus |
|---|---|---|
| **1. Clarify Requirements** | 3-5 min | Functional vs non-functional, scale |
| **2. Estimation** | 3-5 min | QPS, storage, bandwidth |
| **3. High-Level Design** | 15-20 min | API, data model, architecture |
| **4. Deep Dive** | 15-20 min | Interviewer picks a component |
| **5. Wrap-up** | 5 min | Bottlenecks, monitoring, what you'd add |

---

## Step 1 — 7 Questions to Extract Requirements

Works for any prompt. Ask these to the interviewer.

**Functional (what does it do):**
1. **What triggers it?** — What event starts the system?
2. **What can the user control?** — Preferences, settings, customization?
3. **What can the user see?** — History, dashboard, status?
4. **What happens when things go wrong?** — Edge cases, failures?

**Non-functional (how well does it do it):**
5. **How many?** (scale) — Users, transactions, data volume?
6. **How fast?** (latency) — Response time, real-time or delayed?
7. **What can't go wrong?** (reliability) — Consistency, duplication, data loss?

---

## Step 2 — Back-of-Envelope Estimation

> Quick math: **total per day ÷ 100,000 = average QPS**. Multiply by **10x for peak**.

Example: 5M transactions/day → 50 QPS avg → 500 QPS peak

---

## Step 5 — Wrap-up Template

> *"With more time I'd add monitoring for [key metrics], [one missing feature], and [one edge case]."*

Always mention:
- **Monitoring** — latency SLA, error rates, queue lag
- **One missing feature** — something you scoped out
- **One safety concern** — rate limiting, fraud detection, etc.
