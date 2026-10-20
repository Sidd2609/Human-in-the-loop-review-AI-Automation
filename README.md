# Human in the Loop Review in AI Automation

An interactive showcase of how an AI content pipeline decides which outputs need human review and which can proceed automatically, so review stays real and never becomes the bottleneck.

Open `hitl_review_showcase.html` in any modern browser. It is a single self-contained file with no build step, no dependencies and no network requests.

## The problem

Fully automated AI at scale will make errors, and humans cannot check every output. Review everything and you get a bottleneck. Ask too much of reviewers and they approve without checking. The goal is to spend a fixed review budget on the outputs that carry the most risk.

**Core principle:** route by risk, not by default. Hard gates override scores. Reviewers are shown evidence, not verdicts.

## How shortlisting works

1. **Hard gates.** Rules that block, hold or force a tier regardless of confidence: unverified price or entitlement claims, off-limits targeting attributes, regulated terms, policy violations, active incidents, new templates.
2. **Risk score.** `Risk = 100 × P(error) × Impact`.
   - `P(error)` combines weighted uncertainty signals using noisy-OR.
   - `Impact` is the largest of reach, money, reversibility, entity risk and visibility.
3. **Tiers.** Auto-approve, auto-approve with sampling, reviewer queue, specialist review, block. Queued items are tagged High, Medium or Low.
4. **Sampling.** Random QA samples of auto-approved outputs, plus seeded known errors to test reviewer attention.

## What the page includes

- Guided walkthrough: the problem, a five-stage pipeline, and a three-layer shortlisting funnel
- Single-output scorer with a live gate trace
- Mock reviewer queue (evidence, not a verdict; reasons required to edit or reject)
- Kill-switch demo: suppress a single campaign (or all campaigns), hold its scheduled sends, notify owners, then lift and re-score
- Sample PRD outline
- Simulator with Airline and Retail presets. Editable inputs:
  - volume and reviewer capacity
  - tier thresholds
  - sampling and seeded-error rates
  - signal weights
  - impact scales

All sections update live as inputs change.

## Illustrative data

All thresholds, weights, volumes, the 600-output synthetic population and the error estimates are illustrative defaults, not calibrated or measured values. The "errors reaching customers" figures assume reviewers catch every error they see. Real values should come from a shadow-mode period where humans review everything.

## Files

| File | Purpose |
|---|---|
| `hitl_review_showcase.html` | The showcase page |
| `prd.md` | Product requirements document |

## Status

The page has not yet been browser-tested end to end.
