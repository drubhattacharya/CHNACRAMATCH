# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A browser-based dashboard for healthcare and banking compliance professionals. It analyzes Community Health Needs Assessment (CHNA) documents to identify Community Reinvestment Act (CRA) eligible opportunities, focusing on Non-Emergency Medical Transportation (NEMT) for low-to-moderate-income (LMI) populations.

The application is split across three files with no build step:
- `index.html` (~550 lines) — HTML markup only
- `styles.css` (~150 lines) — all CSS
- `app.js` (~1,850 lines) — all JavaScript

There are no subdirectories and no build artifacts.

## Running the Application

Open `index.html` directly in a browser, or serve locally:
```bash
python3 -m http.server 8080
```

There is no build step, no package manager, no tests, and no linter.

## Dependencies (CDN-loaded, no installation)

- **PDF.js 3.11.174** — client-side PDF text extraction
- **Chart.js 4.4.1** — bar and line charts

All processing is local-only; no data leaves the browser.

## Architecture

**Vanilla HTML/CSS/JS** — no framework, no modules. `app.js` is loaded as a plain `<script>` at the bottom of `<body>` so all functions are global. A global `state` object is the single source of truth:
```
state.docs[]        — parsed documents (name, type, pages, textByPage[])
state.evidence[]    — extracted disparity snippets
state.findings[]    — Tier 1 materiality-scored results
state.opportunities[] — Tier 2 CRA opportunities
state.model         — Tier 3 ROI scenario cache
state.chnaEval[]    — per-document CHNA/IS gap analysis
```

### Six-Tier Data Pipeline

Each tier gates on the previous:
1. **CHNA Assessment** — PDF/TXT parsing, keyword/regex signal extraction, materiality scoring (0–100)
2. **CRA Readiness** — maps disparities to CRA criteria (12 CFR 25.04(c)(3)); gates Tier 3 at score ≥ 60
3. **ROI & Break-even** — NEMT no-show financial model with optional SDOH coding layer, 3-year projections
4. **Implementation** — auto-generated 30-60-90 day plan from Tier 2/3 data
5. **Outcomes** — quarterly report template populated from Tier 3 figures
6. **Evaluation** — formal evaluation plan anchored to CHNA evidence and CRA metrics

An **Application Drafts** tab generates 10 audit-ready document types from Tier 1–3 state.

### Key Code Patterns

- `render_*()` functions read from `state` and update the DOM
- `scanDoc()` → `computeMateriality()` → `buildOpportunities()` is the main processing pipeline
- `findPercentNear(text, keyword)` finds percentages within 160 chars of a keyword via regex
- CRA regulatory criteria live in `CRA_CRITERIA`; CHNA checklists in `CHNA_ELEMENTS`, `IS_ELEMENTS`, `WRITTEN_COMMENT_ELEMENTS`
- Tab switching via `setView(view)` which toggles `.active` class and forces Chart.js canvas resize
- `render_roi()` delegates to `roi_inputs()` and `roi_calc()` for testable separation

### Materiality Scoring Formula

```
score = magnitude (0–60) + concentration (0–25) + prominence (0–15)
        + transportation amplification bonus (0–10, if 65–74 age differential detected)
```

## Domain Glossary

- **CHNA**: Community Health Needs Assessment (IRS-required for nonprofit hospitals)
- **CRA**: Community Reinvestment Act (federal bank regulation)
- **LMI**: Low-to-Moderate Income
- **AA**: Assessment Area (CRA geographic zone)
- **NEMT**: Non-Emergency Medical Transportation
- **SDOH**: Social Determinants of Health
- **Z-codes**: ICD-10-CM codes for social determinants (Z59.*, Z60.*, etc.)
