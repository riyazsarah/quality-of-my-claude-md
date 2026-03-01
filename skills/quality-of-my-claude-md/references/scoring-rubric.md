# Scoring Rubric

Use during Phase 5 (Scoring) to compute quality scores for each context file.

## Dimensions and Weights

| # | Dimension | Weight | What It Measures |
|---|-----------|--------|-----------------|
| 1 | Signal-to-Noise Ratio | 25% | Percentage of instructions providing unique, non-discoverable value |
| 2 | Redundancy Score | 20% | Inverse of percentage of instructions duplicated in configs/README/CI |
| 3 | Specificity | 20% | How actionable and specific vs. vague instructions are |
| 4 | Guardrail Coverage | 15% | Presence and quality of critical safety guardrails |
| 5 | Conciseness | 10% | Word efficiency; penalizes verbose instruction blocks |
| 6 | Freshness | 10% | Percentage of instructions referencing valid, existing entities |

**Total weight: 100%**

---

## Dimension 1: Signal-to-Noise Ratio (25%)

Measures what percentage of instructions provide value the agent cannot get elsewhere.

| Score | Criteria |
|-------|----------|
| 0 | Every instruction is redundant, boilerplate, or discoverable |
| 25 | <25% of instructions provide unique value |
| 50 | ~50% of instructions provide unique value |
| 75 | ~75% of instructions provide unique value |
| 100 | Every instruction provides unique, non-discoverable value |

**Calculation:** `(instructions without any anti-pattern flags / total instructions) * 100`

Instructions flagged with AP-001 (redundancy), AP-004 (boilerplate), AP-005 (discoverable) reduce the signal.

---

## Dimension 2: Redundancy Score (20%)

Inverse measure of how much content is duplicated with config files, README, or CI.

| Score | Criteria |
|-------|----------|
| 0 | >80% of instructions are redundant with project configs |
| 25 | 60-80% redundant |
| 50 | 40-60% redundant |
| 75 | 20-40% redundant |
| 100 | <5% redundant — almost nothing overlaps with configs |

**Calculation:** `100 - (instructions flagged AP-001 or AP-005 / total instructions) * 100`

Only counts instructions where codebase probing confirmed the redundancy (not just suspected). In `--quick` mode (where Phase 3 is skipped), use Phase 2 static analysis flags at Medium confidence for this dimension, and note the lower confidence in the score card.

---

## Dimension 3: Specificity (20%)

Measures how actionable and concrete each instruction is.

| Score | Criteria |
|-------|----------|
| 0 | All instructions are vague ("be careful", "follow best practices") |
| 25 | Most instructions lack specific actions or verifiable conditions |
| 50 | Mix of specific and vague instructions |
| 75 | Most instructions are actionable with clear conditions |
| 100 | Every instruction has a concrete action verb and verifiable condition |

**Calculation:** `(instructions NOT flagged AP-002 / total non-boilerplate instructions) * 100`

An instruction is specific when it contains: (1) an action verb (use, run, avoid, prefer, never) AND (2) a concrete subject (a tool name, file path, pattern name, threshold).

---

## Dimension 4: Guardrail Coverage (15%)

Measures presence and quality of critical safety guardrails.

**Essential guardrail categories** (5 total):

| Category | What to Look For |
|----------|-----------------|
| Secrets protection | Instructions about .env, API keys, credentials, never committing secrets |
| Destructive operations | Instructions about force-push, rm -rf, dropping databases |
| Test requirements | Instructions about running tests before commits/PRs |
| PII handling | Instructions about personal data, logging, GDPR |
| File safety | Instructions about deletion safety, backup, reversibility |

| Score | Criteria |
|-------|----------|
| 0 | Zero guardrail categories covered |
| 25 | 1 of 5 categories covered |
| 50 | 2-3 of 5 categories covered |
| 75 | 4 of 5 categories covered |
| 100 | All 5 categories covered with specific, actionable rules |

**Calculation:** `(categories covered / 5) * 100`, with a bonus of +10 (capped at 100) if all covered guardrails are specific (not vague).

---

## Dimension 5: Conciseness (10%)

Measures word efficiency — penalizes unnecessarily verbose instruction blocks.

| Score | Criteria |
|-------|----------|
| 0 | Average instruction length >100 words |
| 25 | Average instruction length 60-100 words |
| 50 | Average instruction length 40-60 words |
| 75 | Average instruction length 20-40 words |
| 100 | Average instruction length <20 words — terse and focused |

**Calculation:** Based on average words per instruction block. Instructions flagged AP-006 (verbose) reduce the score further by 5 points each (minimum 0).

---

## Dimension 6: Freshness (10%)

Measures what percentage of instructions reference entities that currently exist.

| Score | Criteria |
|-------|----------|
| 0 | >50% of reference-containing instructions point to non-existent entities |
| 25 | 30-50% stale references |
| 50 | 15-30% stale references |
| 75 | 5-15% stale references |
| 100 | All referenced files, deps, tools, and paths exist (or no references to check) |

**Calculation:** `100 - (instructions flagged AP-008 / instructions containing file paths or dep names) * 100`

If no instructions contain references to check, default to 100 (no staleness possible).

---

## Overall Score and Grade

**Overall score** = weighted average:

```
overall = (signal_to_noise * 0.25) + (redundancy * 0.20) + (specificity * 0.20)
         + (guardrail_coverage * 0.15) + (conciseness * 0.10) + (freshness * 0.10)
```

**Grade mapping:**

| Range | Grade | Interpretation |
|-------|-------|---------------|
| 90-100 | A | Excellent — file provides high value with minimal waste |
| 80-89 | B | Good — solid file with minor improvements possible |
| 70-79 | C | Adequate — noticeable redundancy or gaps, worth improving |
| 60-69 | D | Below average — significant issues, improvements recommended |
| <60 | F | Poor — file may be hurting more than helping, major revision needed |

---

## Interpretation Guidance

Include these notes in the score card output:

- **Grade A-B:** "Your context file is well-optimized. Focus on the specific findings below to fine-tune."
- **Grade C:** "Your context file has room for improvement. The proposed diff removes redundancy and strengthens guardrails."
- **Grade D-F:** "Your context file may be increasing inference costs without proportional benefit. Consider applying the proposed diff or starting fresh with the guardrails-first approach."

For files scoring below 60, emphasize that the ETH Zurich research shows poorly written context files can reduce agent success rates by ~2% while increasing costs 20%+.
