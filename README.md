# Loan AI Tool — Origination Pipeline Telemetry

> **Prototype / Mock Application** — A single-page engineering and product intelligence dashboard for monitoring an AI-assisted mortgage origination pipeline. Built to demonstrate what actionable LLM observability looks like for a loan origination platform.

Live URL: deployed via GitHub Pages on push to `main`.

---

## Overview

This tool gives loan operations, engineering, and product teams a real-time view into where the automated origination pipeline is healthy, where it is failing, and exactly why any individual loan required human intervention. It is not a vanity-metrics dashboard — every number shown is tied to an action.

The app is a **static single-page application** (no backend, no build step) with two functional tabs:

| Tab | Purpose |
|-----|---------|
| **Pipeline Telemetry** | System-wide pipeline health, KPIs, SLA breach alerts, velocity chart, manual intervention log |
| **Automated Origination Engine** | Per-loan exception review workflow — document viewer + LLM confidence scoring + human approval/rejection |

---

## Features

### Tab 1 — Pipeline Telemetry (Data Research Dashboard)

#### Filter Bar
- **Date Range** selector: Last 7 Days / Last 30 Days / Year to Date
- **Loan Type** selector: All / Conventional / FHA / VA
- **LLM Model Version** selector: v2.3-stable / v2.4-turbo / v3.0-beta

#### Global Alert & Recommendation Banner
- Full-width red warning banner that fires when a pipeline stage exceeds SLA
- Displays: bottleneck stage, current average days, % over SLA, and root cause diagnosis
- Mock example: _"Income & Asset Verification averaging 11 days (175% over SLA) — 82% failure rate on self-employed tax return extraction"_
- Action buttons: **Export Failed Documents for LLM Evals** and **View Ops Tickets** (links to Tab 2)

#### KPI Cards (4 horizontal)
| Metric | Current Value | Trend |
|--------|--------------|-------|
| Avg Time to Close | 32 Days | ↓ 2 days MoM |
| Fully Automated Origination Rate | 14% | ↑ 1.5% MoM |
| Avg Cost per Loan Originated | $8,450 | Goal: <$5,000 |
| System-Wide LLM Hallucination Rate | 1.2% | ↑ 0.2% MoM |

#### Pipeline Velocity Chart
- **Chart.js** grouped bar chart showing all origination stages on the X-axis
- Stages: Intake → Disclosures → Verification → Underwriting → Title → Close
- **Gray bars** = Target SLA days per stage
- **Blue bars** = Actual average completion time (on track)
- **Red bars** = Actual time when the stage is breaching SLA (auto-colored when actual > 1.5× target)
- Interactive tooltips on hover

#### Manual Intervention Log (HITL Tracker)
A scannable table of loans that fell out of the automated pipeline and require human review:

| Column | Description |
|--------|-------------|
| Loan ID | Monospace loan identifier (e.g. `#LC-992`) |
| Stage | Current stuck stage (Income, Conditions, Underwriting) |
| LLM Confidence Score | Color-coded badge: red (<75%), yellow (75–89%), green (≥90%) |
| Action | "Review" button that navigates to Tab 2 for that loan |

---

### Tab 2 — Automated Origination Engine (AI Assisted Processing)

Opens in **Exception Review Mode** for a specific loan (`#LC-992` in the mock). This tab represents what a loan processor sees when the LLM pipeline escalates a case.

#### Integration Stage Matrix
Six-step pipeline stepper showing which third-party integrations have completed vs. are blocked:

| Stage | Integration | Status |
|-------|------------|--------|
| Identity & Fraud | Socure API | ✅ Complete |
| Credit Pull | Equifax API | ✅ Complete |
| Income & Employment | Loancrate LLM | ⚠️ Exception (pulsing) |
| Asset Verification | Plaid API | ✅ Complete |
| Property Valuation | CoreLogic AVM | ✅ Complete |
| Underwriting | Fannie Mae DU | ⏸ Paused (greyed out) |

#### HITL Review Panel (split layout)

**Left — Document Viewer**
- Renders a mock Schedule C (Form 1040) tax document
- Highlights the specific field the LLM was attempting to extract (Line 31: Net Profit)
- Animated yellow **bounding box overlay** shows the exact region the LLM targeted
- Zoom controls (UI only in prototype)

**Right — Exception Handling**
- Displays the LLM's confidence score for the extraction (42% in the mock — below the 90% autonomous threshold)
- Shows the required field (`Schedule C – Line 31, Net Profit`)
- Displays the LLM-extracted value (`$84,250.00`) in an **editable input field** so the reviewer can correct it if the extraction was wrong
- Two action buttons:
  - **Approve Extraction & Resume DU API** — approves the value and returns to Tab 1
  - **Reject & Request Clearer Document** — rejects and flags for re-submission

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Markup | Vanilla HTML5 |
| Styling | Tailwind CSS (CDN), custom CSS |
| Charts | Chart.js (CDN) |
| Fonts | Inter via Google Fonts |
| Theme | Dark mode, glassmorphism panel style |
| Deployment | GitHub Pages via GitHub Actions |
| Build step | None — fully static |

---

## Deployment

The repo uses a GitHub Actions workflow (`.github/workflows/deploy.yml`) that:

1. Triggers on every push to `main` (and supports manual `workflow_dispatch`)
2. Uploads the entire repo root as the Pages artifact
3. Deploys to GitHub Pages using `actions/deploy-pages`

**Required one-time setup in the GitHub repo:**
> Settings → Pages → Source → set to **"GitHub Actions"**

---

## Project Structure

```
loan-ai-tool/
├── index.html                    # Entire application (single file)
└── .github/
    └── workflows/
        └── deploy.yml            # GitHub Pages deployment workflow
```

---

## Roadmap / Planned Features

- [ ] Real filter interactivity (loan type filter wires to table and chart)
- [ ] Additional loan cases in the Manual Intervention Log
- [ ] Cost-to-Originate Calculator widget with per-category cost breakdown
- [ ] AI Assisted Processing tab — full document queue, not just single loan
- [ ] Date range filter wired to dynamic chart data
- [ ] LLM model version toggle affects confidence score display
- [ ] Export functionality for failed document sets

---

## Context

This prototype was designed around the goal of eliminating a **$11,000/loan origination fee** by automating as much of the document extraction, income verification, and underwriting pipeline as possible. The dashboard surfaces where automation is working, where it is failing, and gives human reviewers the minimum viable context to resolve exceptions quickly and get loans back into the automated pipeline.
