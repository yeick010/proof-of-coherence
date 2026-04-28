# Proof-of-Coherence (PoC)
## An Observatory of LLM Reasoning Stability
> **Status:** Archived as a methodological exercise (28 Apr 2026). Active research has moved to the **CIVITAS** project (emergent institutions in multi-agent worlds). This repo is preserved as an honest record of a path we explored and consciously chose not to continue. The scaffolding (taxonomy, question bank, schema, metrics) remains MIT-licensed and forkable for anyone who wants to pursue LLM-coherence measurement seriously.
>
> **License:** CC BY 4.0 (data and prose) · MIT (code)

---

## 1 · Project Vision

Large language models do not have a single "belief" about contested open questions — they sample from a distribution over plausible-sounding text. This means that if you ask the same model the same hard question multiple times (varying only the random seed, the phrasing, or the date), the answers can drift, conflict at the level of individual propositions, and cite entirely different authorities.

**Proof-of-Coherence** turns this observation into a public, reproducible measurement instrument. The observatory:

1. Maintains a curated bank of 60 *canonical open questions* across six domains where genuine expert disagreement exists and where superficial pattern-matching is known to produce confident but unstable answers.
2. Runs those questions against one or more LLMs under controlled conditions (documented seeds, API versions, system prompts) and stores every response as a structured JSON artifact.
3. Applies a four-component **Coherence Score** that measures semantic drift, claim-level contradiction, source inconsistency, and assumption shift across repeated runs.
4. Publishes all raw artifacts, all metric values, and an interactive dashboard — so that researchers, journalists, and curious members of the public can audit, reproduce, and extend the results.

The observatory is **not** a benchmark of correctness. There is no ground truth key. The questions were chosen precisely because reasonable, expert people disagree. What we measure is *consistency under repetition*, not truth.

---

## 2 · Scientific Method

### 2.1 Question Selection Criteria

A question is admitted to the bank if and only if:

- **Legitimate expert disagreement exists.** The question must not have a settled answer in the relevant technical literature. *(Unverified inference: "settled" is operationalized as: no single position commands ≥ 80 % explicit endorsement in a systematic survey of domain experts at the time of question authorship.)*
- **Pattern-matching is known to fail.** The question should not be answerable by retrieving a single authoritative paragraph; it requires integrating conflicting frameworks.
- **It has an assigned fragility type** from the controlled vocabulary `{lexical_ambiguity, normative_tradeoff, empirical_uncertainty, self_reference, scale_dependence}` (see [`taxonomy.md`](taxonomy.md)).
- **It is stable under minor rephrasing.** A paraphrase-stability pre-check is run before promotion to the canonical bank *(planned for Phase 2)*.

### 2.2 Data Collection Protocol

All runs follow this protocol (deviations must be logged in the artifact's `metadata.deviations` field):

| Parameter | Controlled Value |
|---|---|
| System prompt | Canonical prompt v1.0 (stored in `prompts/system_v1.txt`) |
| Temperature | Model default unless otherwise noted |
| Seed | Drawn from pre-registered seed list (`seeds.json`) |
| Web access | `false` by default; documented in `web_access_flag` |
| Schema version | Must match `schema.json#/version` |
| Timestamp | ISO 8601 UTC, recorded by harness, not self-reported |

### 2.3 Coherence Measurement

Four sub-metrics are computed for each question across N ≥ 3 independent runs, then combined into a single **Coherence Score ∈ [0, 1]**. See [`metrics.md`](metrics.md) for full formulas, rationale, weight justification, and a worked example.

### 2.4 Replication & Forking

Every artifact file is content-addressed (SHA-256 hash stored in `manifest.json`). To reproduce any measurement:

```bash
# Clone the repo
git clone https://github.com/<org>/proof-of-coherence

# Install harness dependencies
pip install -r requirements.txt   # Phase 2

# Re-run a single question
python run.py --question-id ai-align-001 --model gpt-4o --seeds 42 137 2718

# Recompute metrics from existing artifacts
python metrics.py --question-id ai-align-001
```

*(Note: CLI syntax is illustrative; the harness is built in Phase 2.)*

---

## 3 · Ethics Statement

### 3.1 What this project does and does not claim

- **Does:** measure statistical properties of model outputs under controlled conditions.
- **Does not:** make claims about model "beliefs," "intentions," or "understanding." These terms are used informally throughout the documentation and should not be read as implying mentalistic commitment.
- **Does not:** declare any position on the open questions themselves to be correct.

### 3.2 Potential harms and mitigations

| Risk | Mitigation |
|---|---|
| Results misread as "proof the model is lying" | All publications include a plain-language explainer; we use "incoherence" not "deception" |
| Questions weaponized to elicit harmful content | All questions are screened against a harm taxonomy; questions about dual-use topics are excluded |
| Model providers' proprietary outputs used to train competitors | All collected artifacts are published for research use; no fine-tuning use is permitted without separate authorization |
| Researcher confirmation bias in question selection | Questions are drafted against a pre-registered selection protocol and reviewed by at least two domain consultants *(planned Phase 2)* |

### 3.3 Data provenance

Model outputs are collected via public commercial APIs. No jailbreaks or adversarial prompts are used. The system prompt explicitly requests honest uncertainty signalling and self-critique. All API terms of service are respected.

### 3.4 Researcher positionality

*(To be completed by PI team before public launch — unverified inference placeholder: research teams running AI evaluations are often employed by or funded by AI companies, creating structural incentives toward favorable results. This observatory is designed to operate with no model-provider funding.)*

---

## 4 · Repository Structure

```
proof-of-coherence/
├── README.md                   ← this file
├── taxonomy.md                 ← 6 domains × rationale
├── questions.json              ← 60 canonical questions
├── schema.json                 ← response artifact JSON Schema
├── metrics.md                  ← coherence metric formulas
├── plan.md                     ← 4-week project plan
├── prompts/                    ← system + user prompt templates (Phase 2)
├── seeds.json                  ← pre-registered seed list (Phase 2)
├── artifacts/                  ← collected response JSON files (Phase 2)
│   └── T0/                     ← first collection epoch
├── scores/                     ← computed coherence scores (Phase 2)
├── site/                       ← static observatory site source (Phase 3–4)
├── scripts/                    ← data collection + metric harness (Phase 2)
└── docs/                       ← extended documentation (Phase 2+)
```

---

## 5 · How to Contribute

### 5.1 Proposing a new question

1. Open a GitHub Issue using the template `question-proposal.md`.
2. Provide: proposed text, domain, fragility type, why-fragile rationale, and at least one pointer to expert disagreement (paper, debate, survey).
3. Questions are reviewed against the selection criteria (§2.1) by maintainers.
4. Accepted questions are added in the next quarterly revision; they do not enter an active measurement epoch until the following T-run.

### 5.2 Proposing a new model or provider

1. Open an Issue with: model identifier, API access method, token cost estimate per full run, and any deviations from the standard protocol the model requires.
2. Maintainers assess feasibility and protocol compatibility.

### 5.3 Reporting a metric flaw

If you believe one of the four metric sub-components has a systematic bias or an implementation error, open an Issue with a reproducible counter-example. We welcome formal proposals for improved metrics.

### 5.4 Code of conduct

This project follows the [Contributor Covenant v2.1](https://www.contributor-covenant.org/version/2/1/code_of_conduct/) *(unverified inference: widely adopted in open-source research projects)*.

---

## 6 · Citation

```bibtex
@misc{poc2026,
  title  = {Proof-of-Coherence: An Observatory of LLM Reasoning Stability},
  author = {[PI Team]},
  year   = {2026},
  url    = {https://github.com/<org>/proof-of-coherence},
  note   = {Phase 1 scaffolding. Data release pending T0 run.}
}
```

---

## 7 · Changelog

| Version | Date | Notes |
|---|---|---|
| 0.1.0 | 2026-04-28 | Phase 1 scaffolding: taxonomy, questions, schema, metrics, plan |

---

*All factual claims in this document are either supported by an inline citation or flagged as "unverified inference." If you find an unsupported claim, please open an Issue.*
