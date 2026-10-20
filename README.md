# Digest — Your Personal AI Podcast Generator

> Save the links you never have time to read. Get a ~10-minute audio briefing every morning.

**Status:** Design phase complete — *Ready for Design Freeze (conditional)* · **Target:** Beta, ~300 users

---

## Table of Contents

1. [What is Digest?](#1-what-is-digest)
2. [Why does it exist?](#2-why-does-it-exist)
3. [Who is it for?](#3-who-is-it-for)
4. [How does it work?](#4-how-does-it-work)
5. [Key Rules and Decisions](#5-key-rules-and-decisions)
6. [Architecture at a Glance](#6-architecture-at-a-glance)
7. [How the Project Was Built (SDLC)](#7-how-the-project-was-built-sdlc)
8. [Project Status](#8-project-status)
9. [Repository and Document Map](#9-repository-and-document-map)

---

## 1. What is Digest?

Digest is a **"read-it-later" app that talks instead of waiting**.

You save links (articles, newsletters, long posts). Each day, Digest:

- reads them,
- uses an LLM to pull out and summarize the key points,
- turns the summaries into **one personalized audio episode of about 10 minutes**.

You listen on your commute through a web player.

---

## 2. Why does it exist?

| Problem | Digest's answer |
|---|---|
| People bookmark dozens of articles every week. | Saving stays one click (browser extension sends the URL). |
| They rarely find time to sit down and read. | Reading is replaced by listening, during time you already have. |
| Long content is hard to digest. | An LLM extracts the key points and drops the filler. |
| AI summaries can invent facts. | Every summary is checked against the original text (grounding check, at least 90% factual consistency). |

---

## 3. Who is it for?

- **Primary users:** busy people who save more than they read and commute or have idle listening time.
- **Scale:** a closed **beta of about 300 users**.
- **Other stakeholders:**
  - **Product Owner / Sponsor:** Hadiffikri Firdaus Bin Muhamad Isa (final sign-off).
  - **Admin / support:** metadata-only access, never user content.
  - **Legal:** copyright (fair use of fetched text) and data privacy.

---

## 4. How does it work?

```
Save link ──► Fetch & extract ──► Summarize (LLM) ──► Fact-check ──► Narrate (TTS) ──► Publish episode
 (extension)   (sandboxed)         (via Gateway)       (judge)        (via Gateway)      (before 06:15 local)
```

**A day in the life**

1. **Save.** You save links any time through the extension or API.
2. **Build (03:00 local).** The system fetches each item, summarizes it, and checks the summary against the source.
3. **Publish (by 06:15 local).** One episode is ready, built from that day's items.
4. **Listen.** You play it in the web player. Summary text is kept as a text alternative (accessibility).
5. **Notified in-app** if something happens (see below).

**In-app notifications (5 types):** daily limit reached · item failed · item expired · export ready · build aborted.

---

## 5. Key Rules and Decisions

| Area | Rule |
|---|---|
| **Cost cap** | Hard limit of **$0.18 per user per day**. At 80% the system downgrades quality; at 100% it skips the build and shows "Daily limit reached". |
| **Thin day** | Fewer than 2 items or fewer than 1000 words: **no episode**; items carry over to the next day. |
| **Deadline** | **06:15 local** is a hard cutoff. A missed build is aborted and the user is told. 06:30 is the measurement point for the SLO. |
| **Retention** | Original text **30 days** · audio **90 days** · summaries **180 days** · unread carry-over **14 days**. |
| **Privacy** | Per-user encryption keys (crypto-shredding). Deleted data is removed within **24 hours**, and from backups within 35 days. |
| **Vendors** | LLM and TTS providers must be zero-data-retention (ZDR) approved, with at least two per type before launch. No unapproved fallback. |
| **Takedown** | Copyright takedowns go through an authenticated in-app form with **two-operator verification**. |
| **Auth** | Google/Apple sign-in, server-issued HttpOnly JWT (14-day sliding, 30-day absolute). |
| **Availability** | 99.5% for the web player and Save Item API. **RTO 4 hours, RPO 5 minutes.** |
| **Day 1 / empty queue** | A static "Welcome to Digest" audio plays. It is not an episode and costs nothing. |

---

## 6. Architecture at a Glance

- **Style:** modular monolith with a Postgres-backed job queue.
- **5 deployable services:** `api`, `worker`, `gateway`, `erasure`, `fetch-sandbox`.
- **Vendor Gateway:** the single enforcement point for budget, cutoff and ZDR rules. Only it talks to LLM/TTS vendors.
- **Fetch sandbox:** isolated service that downloads web pages (no JavaScript rendering in v1).
- **Data:** managed Multi-AZ Postgres (for high availability, not disaster recovery) with WAL point-in-time recovery. Encryption keys are stored outside the backup boundary.
- **Cloud:** AWS `ap-southeast-1`, multiple accounts (prod, staging, key-store, ledger, security, drill), 6 signed container images.
- **Observability:** Datadog APM.
- **Estimated infrastructure cost:** about **$910 / month**.

---

## 7. How the Project Was Built (SDLC)

The design was produced by specialist AI agent teams, run **one agent at a time**, with a **Product Owner approval gate** after every phase.

| # | Phase | Outcome |
|---|---|---|
| 1 | Requirements | Baseline requirements, gaps and priorities |
| 2 | Methodology & Planning | MVP scope, feasibility, risks, plan |
| 3 | System Modelling & Design | Use cases, ERD, sequence and component models |
| 4 | Validation & Consistency | Conditional Go, Corrections Register |
| 5 | System Architecture | Modular monolith, Gateway, sandbox |
| 6 | Database & Data | Crypto-shredding, erasure ledger, item status model |
| 7 | API & Integration | Endpoints, takedown flow, token scheme |
| 8 | Component & Interface | 5 deployables, 21 modules |
| 9 | Infrastructure & Deployment | AWS layout, network, cost |
| 10 | Design Validation & Quality | Design freeze review, ADR-001..022 |

Total: **10 teams, 59 agents**. Development teams exist but have **not** been run for Digest yet.

---

## 8. Project Status

**Verdict: Ready for Design Freeze — CONDITIONAL.** No non-functional requirement failed.

**Still open before launch**

- [ ] Signed ZDR contracts (at least 2 each for LLM and TTS) — **blocks launch**
- [ ] ERD diagram redrawn with the approved patches
- [ ] Product Owner signature record
- [ ] WCAG accessibility audit
- [ ] First timed restore drill
- [ ] Product Owner decisions D1–D6 (takedown backup wording, WCAG transcript, Datadog DPA/budget, fleet cost alert, meaning of 06:30, region)
- [ ] ADR-023..025 to be written

**Not started:** implementation, testing, security testing, release.

---

## 9. Repository and Document Map

All files are under `output/`.

| Document | What it contains |
|---|---|
| `digest_project_summary.md` | One-page summary of phases 1–10 |
| `digest_requirements_findings.md` | Phase 1 requirements findings |
| `digest/A05_gap_report.md` | Requirement gap report |
| `digest/A06_priority_report.md` | Requirement priorities |
| `digest_phase4_corrections_register.md` | **Absolute baseline** corrections |
| `digest_phase5_handoff.md` … `digest_phase8_handoff.md` | Architecture, data, API, components |
| `digest_phase9_handoff.md` | Infrastructure and deployment |
| `digest_phase10_signoff.md` | Design validation sign-off |

> Where documents disagree, the **Corrections Register** and later Product Owner decisions win over earlier phase outputs.
