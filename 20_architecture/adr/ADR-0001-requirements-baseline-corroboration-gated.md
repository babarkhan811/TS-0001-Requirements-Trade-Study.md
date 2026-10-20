# ADR-0001: Freeze the split-latency, corroboration-gated requirement baseline (B3) at Gate 1

| Field | Value |
| --- | --- |
| Status | Proposed (Accepted once the sponsor answers DV-01 and CQ-01) |
| Date | TODO |
| Decision owner | Babar Yousaf, systems-engineering lead (TODO: confirm) |
| Trade study | TS-0001 (20_architecture/trade_studies/TS-0001-requirements-baseline.md) |
| Requirements affected | PR-01 → REQ-0103a/b; new REQ-0113 (derived from FR-13); REQ-0112 quantifies US-05 |

> TODO: renumber to the next free ADR number and update the TS-0001 header.

## Context

PR-03 asks for ≥ 0.90 sensitivity at ≤ 1 false positive per 1,000 criticality-high device-days. PR-01 asks for candidate findings ≤ 60 s p95.

A 60 s cap limits the evidence window to 45 s, giving 1,920 decisions per device-day. Each decision may then be a false positive with probability no more than 5.2 × 10⁻⁷. No evidence shows any candidate detector can do this (TS-0001 §2.3, §3.1).

SF-07a and FR-20 (5-min surfacing), FR-11 (low-confidence escalation) and SF-02b (no rate-limiting of life-sustaining findings) bound the alternatives.

## Decision

Adopt baseline B3:
- Tier-1 findings ≤ 60 s p95; tier-2 findings ≤ 4 min p95. SF-07a's 5-minute surfacing is preserved.
- A tier-2 anomaly on a criticality-high device becomes a finding only if its score is high on its own, or a lower score is corroborated by an independent evidence-source class for the same resolved device in the same window. This follows the FR-05b precedent of corroborating with VEX.
- PR-03's numbers are unchanged.

## Alternatives considered

| Baseline | Score | Outcome |
| --- | --- | --- |
| B0 as drafted | 61.7 | Not demonstrated feasible |
| B1 relaxed FP | 56.2 | 150 analyst-min/day |
| B2 latency split only | 66.2 | Close second; fallback |
| B4 Alerts + Leads (previous draft) | — | Failed SF-07a, FR-11, SF-02b |

B3 scored 70.3 and led in 11 of 12 ±25 % weight swings.

## Consequences

- **Positive:** the detector's allowed false-positive rate loosens about 125-fold; tier-2 compute falls 5-fold and stays on CPU; analyst load stays about 30 min/site/day.
- **Negative:** one deviation (DV-01); B3 depends on three unmeasured inputs (p_c, q, detector ROC); its correctness depends on cross-source identity resolution (FR-12).

## Reversal triggers

R1–R9 in TS-0001 §9. Most important:
- **EXP-01a:** if p_c ≥ 0.05, collapse to B2.
- If the sponsor rules corroboration is an SF-02b filter, adopt B2.
