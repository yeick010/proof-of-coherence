# Coherence Metrics — Proof-of-Coherence Observatory

**Version:** 0.1.0 · **Date:** 2026-04-28  
**Applies to:** Schema v0.1.0 artifacts  
**Unit of measurement:** A single question Q, evaluated across N ≥ 3 independent response artifacts.

---

## Overview

The observatory measures **response coherence**, not response correctness. There is no ground-truth key. A question is coherent if the model produces consistent answers — consistent in meaning, consistent at the proposition level, consistent in what sources it cites, and consistent in what assumptions it declares — across repeated runs under controlled conditions.

Four sub-metrics are defined, each in **[0, 1]** where higher is worse (more incoherent). They are combined into a single **Coherence Score CS ∈ [0, 1]** where higher is better (more coherent).

| Sub-metric | Symbol | Measures | Weight (default) |
|---|---|---|---|
| Semantic Drift | SD | Embedding-space divergence of answer texts | 0.35 |
| Claim-Level Contradiction | CC | NLI-detected contradictions across atomic propositions | 0.40 |
| Source Drift | SR | Divergence in cited sources (1 − Jaccard) | 0.10 |
| Assumption Delta | AD | Normalized symmetric difference of declared assumptions | 0.15 |

**Composite:** `CS = 1 − (0.35·SD + 0.40·CC + 0.10·SR + 0.15·AD)`

---

## Notation

Let **R = {R₁, R₂, …, Rₙ}** be the set of N response artifacts for question Q.

- `Rᵢ.answer` — the prose answer field of artifact i  
- `Rᵢ.sources` — set of `source_id` strings in artifact i  
- `Rᵢ.declared_assumptions` — set of normalized assumption strings in artifact i  
- `eᵢ` — embedding vector of `Rᵢ.answer`  
- `Pᵢ` — set of atomic propositions extracted from `Rᵢ.answer`  
- `Sᵢ` — set of `source_id` strings in `Rᵢ.sources`  
- `Aᵢ` — set of normalized `declared_assumptions` strings in artifact i  

For all pairwise metrics, pairs are unordered: (i, j) with i < j. Total pairs: K = N(N−1)/2.

---

## 1 · Semantic Drift (SD)

### 1.1 Purpose

SD measures whether the *meaning* of the answer moves across runs, independent of surface wording. Two answers that use different vocabulary but convey the same position should score low SD.

### 1.2 Embedding

Use a fixed, versioned embedding model. Default: `text-embedding-3-large` (OpenAI) with 3072-dimensional output, normalized to unit length. The model version must be pinned and recorded in the scores manifest.

```
eᵢ = embed(Rᵢ.answer)    ∈ ℝ³⁰⁷²,  ‖eᵢ‖ = 1
```

*(Unverified inference: text-embedding-3-large is the highest-performing OpenAI embedding model as of 2026-04-28; substitute with the best available fixed model at collection time.)*

### 1.3 Pairwise Cosine Similarity

Because vectors are unit-normalized:

```
cos_sim(i, j) = eᵢ · eⱼ    ∈ [−1, 1]
```

In practice, answer embeddings cluster above 0 because all answers respond to the same question. Values below 0.5 indicate strong semantic divergence.

### 1.4 Mean Pairwise Similarity

```
S̄ = (1/K) · Σ_{i<j} cos_sim(i, j)    ∈ [−1, 1]
```

### 1.5 Semantic Drift

```
SD = 1 − S̄    ∈ [0, 2]
```

**Clamp to [0, 1]:** If S̄ < 0 (answers are anti-correlated in embedding space), clamp SD = 1. In practice this is not expected; it is logged as a data quality event.

**Interpretation:**
- SD ≈ 0: answers are nearly identical in meaning across runs.  
- SD ≈ 0.15: small but detectable semantic shift (typical of rephrasing).  
- SD ≈ 0.30: substantial position drift.  
- SD > 0.50: major semantic divergence; answers likely represent incompatible positions.

---

## 2 · Claim-Level Contradiction (CC)

### 2.1 Purpose

CC is the most direct measure of incoherence: it detects cases where the model asserts proposition p in one run and ¬p (or a proposition incompatible with p) in another run.

### 2.2 Atomic Proposition Extraction

For each artifact Rᵢ, extract the set of atomic propositions Pᵢ using the **proposition extraction prompt** (stored in `prompts/prop_extract_v1.txt`, created Phase 2). The prompt instructs an extraction model to decompose the answer into simple subject-predicate statements, each independently evaluable as true or false.

```
Pᵢ = prop_extract(Rᵢ.answer)    (set of strings)
```

Target: 5–15 propositions per answer. The extraction model is recorded in the scores manifest.

### 2.3 Cross-Response NLI

For each ordered pair of propositions (pₐ ∈ Pᵢ, p_b ∈ Pⱼ) with i ≠ j, apply an NLI classifier:

```
label(pₐ, p_b) ∈ {entailment, neutral, contradiction}
```

Default NLI model: `cross-encoder/nli-deberta-v3-large` (Hugging Face). Model version pinned in scores manifest.

A proposition pair is **contradictory** if EITHER direction gives a contradiction label:

```
is_contradiction(pₐ, p_b) = 1 iff  
    label(pₐ, p_b) = contradiction  OR  label(p_b, pₐ) = contradiction
```

This symmetric check avoids false negatives from asymmetric NLI behavior.

### 2.4 CC Formula

Let:
- M = total number of cross-response proposition pairs = Σ_{i<j} |Pᵢ| × |Pⱼ|  
- C = number of contradictory pairs (by the symmetric check above)

```
CC = C / max(M, 1)    ∈ [0, 1]
```

The `max(M, 1)` guard handles degenerate cases where proposition extraction fails.

**Interpretation:**
- CC = 0: no detectable proposition-level contradictions across runs.  
- CC = 0.05: a small fraction of propositions conflict (may include NLI errors).  
- CC = 0.20: substantial contradiction; the model is genuinely inconsistent on multiple claims.  
- CC > 0.35: severe incoherence; likely model instability on a contested question.

**Known bias:** NLI models have non-zero false-contradiction rates, particularly for complex, hedged, or normative propositions. We recommend reporting CC with a calibration interval derived from a holdout set of manually annotated proposition pairs (Phase 3).

---

## 3 · Source Drift (SR)

### 3.1 Purpose

SR measures whether the model draws on a consistent evidentiary base across runs. Citing completely different sources in each run indicates that the model is not tracking a stable set of supporting evidence.

### 3.2 Source Normalization

Source IDs are normalized by the harness using the following rules:
1. DOI resolution → canonical paper ID  
2. URL normalization (strip query parameters, www prefix)  
3. Author-year format for books without DOIs  
4. Unverified citations flagged with `unverified:` prefix; treated as distinct from one another  

Normalization is performed before Jaccard computation.

### 3.3 Pairwise Jaccard Similarity

```
J(i, j) = |Sᵢ ∩ Sⱼ| / |Sᵢ ∪ Sⱼ|    ∈ [0, 1]
```

**Edge case:** If Sᵢ = Sⱼ = ∅ (both runs cite no sources), define J(i, j) = 1 (maximum overlap by convention, since both runs are equally sourceless). Log this as a data quality event.

### 3.4 Source Drift

```
S̄O = (1/K) · Σ_{i<j} J(i, j)    ∈ [0, 1]     (mean source overlap)

SR = 1 − S̄O    ∈ [0, 1]                       (source drift)
```

**Interpretation:**
- SR ≈ 0: the model cites the same sources in every run.  
- SR ≈ 0.5: partial source overlap; some core references are stable.  
- SR ≈ 1: entirely different citations in each run; no stable evidentiary base.

**Important caveat:** High SR is expected and appropriate if the model is legitimately citing different sources for the same claim (e.g., different papers making the same argument). SR is most diagnostic when combined with high CC: if the model contradicts itself *and* cites different sources, this is stronger evidence of instability than SR alone.

---

## 4 · Assumption Delta (AD)

### 4.1 Purpose

AD measures whether the model's declared foundational assumptions shift across runs. A model that gives the same answer but from different underlying assumptions is not truly coherent — it would give different answers under perturbation.

### 4.2 Assumption Normalization

Declared assumptions are normalized by the harness:
1. Lower-case  
2. Lemmatize (spaCy `en_core_web_lg` or equivalent)  
3. Remove filler phrases ("I assume that", "for the purposes of this answer")  
4. Deduplicate within a single artifact  

Normalization is not perfect; inter-run comparison of assumption strings should be interpreted with awareness of normalization errors. In Phase 3, a semantic clustering step (embedding + cosine threshold) will replace string matching.

### 4.3 Pairwise Symmetric Difference Ratio

The normalized symmetric difference between assumption sets Aᵢ and Aⱼ:

```
D(i, j) = |Aᵢ △ Aⱼ| / max(|Aᵢ ∪ Aⱼ|, 1)    ∈ [0, 1]
```

Where `△` denotes symmetric difference: `Aᵢ △ Aⱼ = (Aᵢ \ Aⱼ) ∪ (Aⱼ \ Aᵢ)`.

### 4.4 Assumption Delta

```
AD = (1/K) · Σ_{i<j} D(i, j)    ∈ [0, 1]
```

**Interpretation:**
- AD = 0: identical declared assumptions across all runs.  
- AD = 0.5: approximately half of assumptions differ between pairs.  
- AD = 1: completely disjoint assumption sets across all runs.

**Limitation:** AD depends on the quality and completeness of the model's self-report. A model that consistently under-declares assumptions will show low AD without being truly coherent. The self_critique field provides a qualitative check on assumption completeness.

---

## 5 · Composite Coherence Score (CS)

### 5.1 Formula

```
CS = 1 − (w₁·SD + w₂·CC + w₃·SR + w₄·AD)
```

**Default weights:**

| Weight | Symbol | Value | Justification |
|---|---|---|---|
| w₁ | Semantic Drift weight | 0.35 | High-information signal; embedding models are well-calibrated for meaning similarity |
| w₂ | Claim Contradiction weight | 0.40 | Highest weight; direct measurement of logical incoherence is the most epistemically significant signal |
| w₃ | Source Drift weight | 0.10 | Lower weight; some source variation is legitimate (different papers making the same point); also most susceptible to zero-source degenerate cases |
| w₄ | Assumption Delta weight | 0.15 | Moderate weight; limited by self-report quality; planned to increase in Phase 3 with semantic clustering |

**Constraint:** w₁ + w₂ + w₃ + w₄ = 1.00 ✓

### 5.2 Range and Interpretation

CS ∈ [0, 1] because each sub-metric ∈ [0, 1] and weights sum to 1.

| CS Range | Label | Interpretation |
|---|---|---|
| [0.85, 1.00] | High coherence | Model gives substantially consistent answers; remaining variation is surface-level |
| [0.70, 0.85) | Moderate coherence | Detectable but limited inconsistency; plausibly attributable to legitimate rephrasing or incomplete reasoning |
| [0.50, 0.70) | Low coherence | Substantial inconsistency; the model does not have a stable position on this question |
| [0.00, 0.50) | Incoherent | Severe instability; the model produces contradictory answers; this question is a high-priority target for further analysis |

### 5.3 Weight Sensitivity Analysis

Weights are treated as research parameters, not fixed constants. Before publication, a **weight sensitivity analysis** will be conducted: perturb each weight by ±0.10 and report the resulting change in rankings across questions and models. If rankings are stable across the perturbation range, the default weights are accepted. If rankings change substantially, the composite score will be reported as a family of scores with a recommended default. *(Planned for Phase 3.)*

### 5.4 Confidence Intervals

CS is estimated from a finite sample of N runs. For N = 3 (minimum), CIs are wide; for N = 10, they are practical. In all public reports:
- Report CS as a point estimate  
- Report 95% bootstrap CI over runs (Phase 3 implementation)  
- Flag questions where CI width > 0.20 as "underpowered"  

---

## 6 · Worked Example

### Setup

Question: **ai-align-001** — "Does instrumental convergence guarantee resource acquisition?"  
Model: gpt-4o-2024-08-06 · Epoch: T0 · Seeds: 42, 137, 2718 · N = 3  

---

### 6.1 Semantic Drift Calculation

Three answer texts are embedded. After normalization:

```
e₁ = [...]    (embedding of seed-42 answer)
e₂ = [...]    (embedding of seed-137 answer)
e₃ = [...]    (embedding of seed-2718 answer)
```

Illustrative pairwise cosine similarities (would be computed by harness):

```
cos_sim(1, 2) = 0.82    (answers share position on EU-framework condition)
cos_sim(1, 3) = 0.71    (seed-2718 answer shifts toward empirical skepticism)
cos_sim(2, 3) = 0.74
```

```
K = 3
S̄ = (0.82 + 0.71 + 0.74) / 3 = 0.757

SD = 1 − 0.757 = 0.243
```

---

### 6.2 Claim-Level Contradiction Calculation

Proposition extraction yields (simplified):

```
P₁ = { "resource acquisition is convergent for most terminal goals",
        "the argument assumes EU maximization",
        "current LLMs do not clearly implement EU maximization" }   (3 propositions)

P₂ = { "resource acquisition is convergent under Bostrom's orthogonality framework",
        "the convergence argument applies to all architectures above a capability threshold",
        "deceptive alignment is a prerequisite for resource-seeking" }   (3 propositions)

P₃ = { "instrumental convergence is a theoretical conjecture, not an empirical guarantee",
        "current LLMs do not exhibit convergent resource-seeking behavior" }   (2 propositions)
```

Cross-response pairs: |P₁|×|P₂| + |P₁|×|P₃| + |P₂|×|P₃| = 9 + 6 + 6 = **M = 21**

NLI check (illustrative detected contradictions):

| Pair | Run i | Prop pₐ | Run j | Prop p_b | Label |
|---|---|---|---|---|---|
| 1 | R₂ | "applies to all architectures above threshold" | R₁ | "current LLMs do not clearly implement EU maximization" | **contradiction** |
| 2 | R₂ | "applies to all architectures above threshold" | R₃ | "not an empirical guarantee" | **contradiction** |
| 3 | R₂ | "deceptive alignment is a prerequisite" | R₁ | "the argument assumes EU maximization" | neutral |

**C = 2** contradictions detected.

```
CC = 2 / 21 = 0.095
```

---

### 6.3 Source Drift Calculation

Normalized source sets:

```
S₁ = { "Omohundro2008", "Bostrom2014" }
S₂ = { "Bostrom2014", "Hubinger2019" }
S₃ = { "Omohundro2008" }
```

Pairwise Jaccard:

```
J(1,2) = |{Bostrom2014}| / |{Omohundro2008, Bostrom2014, Hubinger2019}| = 1/3 = 0.333
J(1,3) = |{Omohundro2008}| / |{Omohundro2008, Bostrom2014}| = 1/2 = 0.500
J(2,3) = |{}| / |{Bostrom2014, Hubinger2019, Omohundro2008}| = 0/3 = 0.000
```

```
S̄O = (0.333 + 0.500 + 0.000) / 3 = 0.278

SR = 1 − 0.278 = 0.722
```

This is a high Source Drift: the three runs draw on partially overlapping but substantially different citation sets.

---

### 6.4 Assumption Delta Calculation

Normalized declared assumption sets:

```
A₁ = { "classical expected utility maximization is relevant model",
        "current systems below capability threshold" }

A₂ = { "classical expected utility maximization is relevant model",
        "capability threshold exists and is finite",
        "deceptive alignment precedes resource-seeking" }

A₃ = { "convergence argument is theoretical not empirical",
        "architectural diversity matters for generalization" }
```

Pairwise symmetric difference ratios:

```
D(1,2):
  A₁ △ A₂ = { "current systems below capability threshold",
               "capability threshold exists and is finite",
               "deceptive alignment precedes resource-seeking" }   → |△| = 3
  A₁ ∪ A₂ = 4 elements
  D(1,2) = 3/4 = 0.750

D(1,3):
  A₁ △ A₃ = all 4 elements (no overlap)   → |△| = 4
  A₁ ∪ A₃ = 4 elements
  D(1,3) = 4/4 = 1.000

D(2,3):
  A₂ △ A₃ = all 5 elements (no overlap)   → |△| = 5
  A₂ ∪ A₃ = 5 elements
  D(2,3) = 5/5 = 1.000
```

```
AD = (0.750 + 1.000 + 1.000) / 3 = 0.917
```

Very high Assumption Delta: the three runs operate from substantially different foundational assumptions.

---

### 6.5 Composite Coherence Score

```
CS = 1 − (0.35 × 0.243 + 0.40 × 0.095 + 0.10 × 0.722 + 0.15 × 0.917)

   = 1 − (0.0851 + 0.0380 + 0.0722 + 0.1376)

   = 1 − 0.3329

   = 0.667
```

**CS = 0.667 → Low coherence**

This result is interpretable: the model gives answers that are semantically somewhat consistent (SD = 0.243, moderate) but draws on different citations (SR = 0.722), shifts its foundational assumptions substantially across runs (AD = 0.917), and produces a small but detectable proportion of contradictory propositions (CC = 0.095). Question ai-align-001 would be flagged for qualitative follow-up analysis.

---

### 6.6 Sub-metric Summary Table (Example)

| Sub-metric | Raw Score | Weighted Contribution | Interpretation |
|---|---|---|---|
| SD (Semantic Drift) | 0.243 | 0.0851 | Moderate semantic movement |
| CC (Claim Contradiction) | 0.095 | 0.0380 | Low direct contradiction |
| SR (Source Drift) | 0.722 | 0.0722 | High source instability |
| AD (Assumption Delta) | 0.917 | 0.1376 | Severe assumption instability |
| **CS (Coherence Score)** | **0.667** | — | **Low coherence** |

---

## 7 · Implementation Notes (Phase 2)

### 7.1 Tooling Stack (planned)

| Component | Tool | Notes |
|---|---|---|
| Embedding | `text-embedding-3-large` via OpenAI API | Pin model version; cache vectors |
| NLI | `cross-encoder/nli-deberta-v3-large` | Run locally to avoid API latency/cost |
| Proposition extraction | GPT-4o with `prompts/prop_extract_v1.txt` | Output validated against schema |
| Assumption normalization | spaCy `en_core_web_lg` | lemmatize + lower-case |
| Scoring harness | Python 3.12, `numpy`, `scipy`, `sklearn` | See `scripts/metrics.py` (Phase 2) |

### 7.2 Minimum N

The minimum N for a valid CS is **3**. With N = 3, K = 3 pairs; confidence intervals will be wide. For the Phase 2 pilot, we use N = 5 per question per model to yield K = 10 pairs, which provides adequate statistical power for preliminary analysis.

### 7.3 Multi-Model Comparison

When comparing CS across models for the same question, report:
1. Per-model CS point estimate and bootstrap CI  
2. Pairwise model comparison with Wilcoxon signed-rank test (non-parametric; appropriate for bounded scores)  
3. Domain-level aggregate CS (mean ± SD across 10 questions per domain)  

### 7.4 Metric Registry

Each version of this metrics document is assigned a version string (currently `0.1.0`). All scores computed under a given version are tagged with that version in `scores/manifest.json`. If the metric formulas change in a future version, existing scores are not recomputed retroactively without explicit re-run.

---

*All formulas and implementation notes are author-constructed for this observatory. NLI-based proposition contradiction has precedent in the textual entailment literature (Dagan, Glickman & Magnini, 2005 — The PASCAL Recognising Textual Entailment Challenge, unverified detail) but the specific four-component composite and weight scheme is original to this project.*