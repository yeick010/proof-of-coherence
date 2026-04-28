# Project Plan — Proof-of-Coherence Observatory

**Version:** 0.1.0 · **Date:** 2026-04-28  
**Horizon:** 4 weeks (Weeks 1–4), ending with public static site + GitHub repo + report  
**Phase:** Phase 1 (Scaffolding) is complete. This plan covers Phases 2–4.

---

## Milestone Summary

| Week | Phase | Theme | Key Deliverables | Go/No-Go Gate |
|---|---|---|---|---|
| 1 | 2 — Infrastructure | Harness + Pilot Prep | Working collection harness; 20-Q pilot spec; seed registry | Harness validates schema on ≥ 1 test response |
| 2 | 2 — T0 Pilot | Pilot Run | T0 artifact set (20 Q × N models × 5 seeds); validated metrics on pilot | Pilot CS values computed for ≥ 3 models without schema errors |
| 3 | 3 — Full T0 + Analysis | Full Run + Dashboard | T0 artifacts for all 60 Q; coherence scores; static dashboard prototype | CS computed for all 60 questions; dashboard renders locally |
| 4 | 4 — Publication | Public Release | GitHub repo public; static site deployed; written report | Site live; all artifacts content-addressed; report peer-reviewed by 2 co-investigators |

---

## Week 1 — Infrastructure & Pilot Preparation

**Theme:** Build the collection harness, validate the schema, select the pilot subset, and register seeds.

### Day 1–2: Repository & Environment

| Task | Owner | Output | Done When |
|---|---|---|---|
| Create GitHub repository (initially private) | PI | Repo at `github.com/<org>/proof-of-coherence` | Repo exists; Phase 1 files committed |
| Write `requirements.txt` with pinned versions | Eng | `requirements.txt` (Python 3.12) | `pip install -r requirements.txt` succeeds in clean venv |
| Write `pyproject.toml` and lint config (`ruff`, `mypy`) | Eng | `pyproject.toml` | `ruff check .` passes on scaffolding files |
| Register API credentials in `.env.example`; document secret management | PI | `.env.example` | No credentials committed to repo |

### Day 2–3: Collection Harness

| Task | Owner | Output | Done When |
|---|---|---|---|
| Implement `scripts/collect.py` with CLI: `python collect.py --question-id <id> --model <model> --seed <seed>` | Eng | `scripts/collect.py` | Produces a schema-valid JSON artifact for 1 test question |
| Implement schema validation (jsonschema library) run after every collection | Eng | Validation integrated in `collect.py` | Invalid artifact causes non-zero exit + error log |
| Write `prompts/system_v1.txt` — canonical system prompt | PI + Eng | `prompts/system_v1.txt` | Prompt reviewed and versioned |
| Write `prompts/user_v1.txt` — canonical user prompt template | PI + Eng | `prompts/user_v1.txt` | Template reviewed and versioned |
| Write `prompts/prop_extract_v1.txt` — proposition extraction prompt | Eng | `prompts/prop_extract_v1.txt` | Produces ≥ 3 propositions for 3 test answers |
| Implement `scripts/validate.py` — batch artifact validation | Eng | `scripts/validate.py` | Validates all artifacts in `artifacts/` directory |

**System Prompt Design Principles** (to be encoded in `prompts/system_v1.txt`):
1. Instruct the model to produce a structured response matching the schema fields.
2. Require explicit `declared_assumptions` — "Make hidden assumptions explicit."
3. Require non-trivial `self_critique` — "Identify at least two ways your answer could be wrong."
4. Require honest uncertainty in `confidence` — "Do not default to 0.5 or 1.0."
5. Require source citations or explicit "unverified inference" flags.
6. Forbid appeals to authority as a substitute for reasoning.

### Day 3–4: Seed Registry & Pilot Subset

| Task | Owner | Output | Done When |
|---|---|---|---|
| Generate and commit `seeds.json` — 20 pre-registered seeds per question, drawn from CSPRNG | Eng | `seeds.json` | Seeds committed; generation script reproducible |
| Select 20-question pilot subset from `questions.json` | PI | `pilot_subset.json` (list of 20 question IDs) | Selection documented with rationale; balanced across domains and fragility types |
| Define model roster for T0 | PI | `models.json` (list of model IDs and API endpoints) | ≥ 3 models; at least 2 providers |

**Pilot Subset Criteria:** Select questions tagged `"pilot_candidate": true` in `questions.json`. There are 20 such questions, one per 2 domains (3–4 per domain), covering all 5 fragility types. This is intentional — pilot_candidate flags were set during scaffolding to ensure a balanced 20-question pilot.

### Day 5–7: Metrics Harness

| Task | Owner | Output | Done When |
|---|---|---|---|
| Implement `scripts/metrics.py` — compute SD, CC, SR, AD, CS for a question-epoch pair | Eng | `scripts/metrics.py` | Produces correct scores for the worked example in `metrics.md` ± 0.01 |
| Set up local NLI model (`cross-encoder/nli-deberta-v3-large`) | Eng | Model cached; inference tested | NLI returns entailment/neutral/contradiction for 5 test pairs |
| Set up embedding pipeline (OpenAI `text-embedding-3-large`) | Eng | Embedding pipeline in `scripts/embed.py` | Returns 3072-dim unit vectors for 3 test strings |
| Write `scores/manifest_schema.json` — schema for the scores manifest | Eng | `scores/manifest_schema.json` | Manifests validate against schema |

**Week 1 Go/No-Go Gate:** Run the harness on question `ai-align-001` with seed 42 on at least one model. The artifact must pass schema validation. The metrics harness must compute CS without errors.

---

## Week 2 — T0 Pilot Run

**Theme:** Execute the full T0 collection on the 20-question pilot, compute coherence scores, and diagnose data quality issues.

### Day 8–9: Pilot Run Execution

| Task | Owner | Output | Done When |
|---|---|---|---|
| Run `collect.py` for all 20 pilot questions × all models in `models.json` × 5 seeds | Eng | Artifacts in `artifacts/T0/` | All 20×N×5 artifacts exist and pass schema validation |
| Log all API errors, rate limits, and unexpected finish_reasons | Eng | `artifacts/T0/run_log.jsonl` | All anomalies recorded |
| Compute SHA-256 hashes of all artifacts; write `artifacts/T0/manifest.json` | Eng | `artifacts/T0/manifest.json` | All artifacts content-addressed |

**Resource estimate (unverified inference):** For 3 models × 20 questions × 5 seeds = 300 calls. At max_tokens = 2048 and average output ~800 tokens, estimated cost is approximately $30–$60 USD at 2026 API pricing depending on models selected. This is an estimate; actual cost must be tracked and reported.

### Day 9–10: Proposition Extraction & NLI

| Task | Owner | Output | Done When |
|---|---|---|---|
| Run proposition extraction on all pilot artifacts | Eng | `computed_metrics.atomic_propositions` populated in each artifact | All artifacts have ≥ 3 propositions |
| Run NLI contradiction check for all cross-run proposition pairs per question | Eng | CC scores computed for all pilot questions | CC values stored in `scores/T0_pilot_scores.json` |
| Manually audit 5% of NLI outputs for false positive/negative rate | PI | Audit log in `docs/nli_audit_T0.md` | Estimated FP/FN rate documented |

### Day 10–11: Full Metric Computation

| Task | Owner | Output | Done When |
|---|---|---|---|
| Run embedding pipeline on all pilot artifacts | Eng | Embedding vectors cached; SHA-256 hashes stored | All embeddings computed |
| Compute SD, SR, AD for all pilot questions | Eng | All four sub-metrics in `scores/T0_pilot_scores.json` | Scores validate against `scores/manifest_schema.json` |
| Compute CS for all pilot questions × all models | Eng | CS table in `scores/T0_pilot_scores.json` | CS values in [0, 1] for all entries |

### Day 11–12: Pilot Analysis

| Task | Owner | Output | Done When |
|---|---|---|---|
| Rank pilot questions by CS (ascending = most incoherent) | PI | `docs/pilot_ranking.md` | Ranking table with CS, sub-metric breakdown, fragility type |
| Identify top-3 most incoherent questions; write qualitative analysis | PI | `docs/pilot_qualitative.md` | ≥ 500 words per question; concrete propositions cited |
| Identify any schema or prompt issues that produced malformed artifacts | Eng | `docs/pilot_issues.md` | All issues categorized; fixes proposed |
| Check for systematic model-level differences in CS distribution | PI | Table in `docs/pilot_analysis.md` | Per-model CS mean ± SD reported |

### Day 12–14: Prompt & Schema Iteration

| Task | Owner | Output | Done When |
|---|---|---|---|
| Revise `prompts/system_v1.txt` if ≥ 30% of artifacts have trivial self_critique | PI + Eng | `prompts/system_v1.1.txt` (if revised) | Revision documented with rationale |
| Revise schema if pilot surfaces needed fields or field type mismatches | Eng | `schema_v0.1.1.json` (if revised) | Schema version bump with changelog |
| Document all prompt and schema changes in `docs/CHANGELOG.md` | Eng | `docs/CHANGELOG.md` | All changes recorded |

**Week 2 Go/No-Go Gate:** CS scores computed for all 20 pilot questions for ≥ 3 models. At least one question has CS < 0.70 (demonstrating that the metric detects real incoherence). Pilot analysis drafted.

---

## Week 3 — Full T0 Run & Dashboard

**Theme:** Run the complete 60-question collection, compute all scores, and build the static observatory dashboard.

### Day 15–16: Full T0 Collection

| Task | Owner | Output | Done When |
|---|---|---|---|
| Extend `collect.py` to batch-process all 60 questions | Eng | `artifacts/T0/` complete with 60×N×5 artifacts | All artifacts pass schema validation |
| Implement retry logic for API failures (exponential backoff, max 3 retries) | Eng | Updated `collect.py` | Zero unrecovered API failures in run log |
| Track and report total API cost | PI | `docs/T0_cost_report.md` | Actual cost documented |

### Day 16–18: Full Metric Computation & QA

| Task | Owner | Output | Done When |
|---|---|---|---|
| Run full metric pipeline on all 60 questions × all models | Eng | `scores/T0_scores.json` | All CS values computed |
| Run sensitivity analysis: perturb weights by ±0.10 | Eng | `scores/T0_weight_sensitivity.json` | Top-10 incoherence rankings stable? Documented |
| Bootstrap 95% CIs for all CS values (B = 1000 resamples) | Eng | CIs in `scores/T0_scores.json` | Flag questions with CI width > 0.20 |
| Compute domain-level aggregate CS and sub-metric distributions | PI | `docs/T0_domain_summary.md` | Summary table with mean, SD, min, max per domain |

### Day 18–20: Static Dashboard (Site)

| Task | Owner | Output | Done When |
|---|---|---|---|
| Choose static site stack: Next.js + shadcn/ui (exported) or plain HTML/CSS/JS | PI + Eng | Decision documented in `site/README.md` | Stack chosen; scaffolded |
| Implement question-level page: CS display, sub-metric breakdown, per-run answer comparison | Eng | `site/src/` | Question page renders for ai-align-001 |
| Implement domain overview page: scatter plot of CS vs. fragility_type; model comparison | Eng | `site/src/` | Domain page renders with real data |
| Implement search/filter by domain, fragility type, CS range | Eng | `site/src/` | Filter works for ≥ 3 filter combinations |
| Write content for site landing page: project overview, how to read the dashboard, ethics note | PI | `site/content/landing.md` | Landing page reviewed by co-investigator |
| Test dashboard locally with full T0 score data | Eng | — | All 60 question pages render without errors |
| Export static site; test all navigation paths | Eng | `site/out/` | Zero 404 errors in local static build |

### Day 20–21: Report Drafting

| Task | Owner | Output | Done When |
|---|---|---|---|
| Draft `report/report.md` — methodology, T0 findings, limitations, future work | PI | `report/report.md` ≥ 3000 words | Draft complete; all factual claims sourced |
| Produce figures: CS heatmap (questions × models), sub-metric distributions, CS vs. confidence scatter | Eng | `report/figures/` (SVG preferred) | All figures readable at 1200px width |

**Week 3 Go/No-Go Gate:** Static dashboard renders locally with all 60 questions. Full T0 scores computed and validated. Report draft complete.

---

## Week 4 — Public Release

**Theme:** Finalize, harden, deploy the static site; make the GitHub repo public; publish the report.

### Day 22–23: Final QA & Hardening

| Task | Owner | Output | Done When |
|---|---|---|---|
| Final schema validation pass on all T0 artifacts | Eng | Zero validation errors | `scripts/validate.py --dir artifacts/T0` exits 0 |
| Add `manifest.json` with SHA-256 hashes of all artifacts; sign with GPG if available | Eng | `artifacts/T0/manifest.json` | Manifest verified against artifacts |
| Code review of scripts harness (2-person review) | PI + Eng | Review notes in PR | PR approved |
| Accessibility audit of static site (WCAG 2.1 AA minimum) | Eng | `docs/accessibility_audit.md` | 0 critical failures; all issues documented |
| Cross-browser test: Chrome, Firefox, Safari (latest stable) | Eng | `docs/browser_test.md` | Core functionality works in all 3 |

### Day 23–24: Report Finalization

| Task | Owner | Output | Done When |
|---|---|---|---|
| Incorporate pilot analysis and full T0 findings into report | PI | `report/report.md` finalized | Report reviewed by 2 co-investigators |
| Write executive summary (≤ 500 words; non-specialist audience) | PI | `report/executive_summary.md` | Summary reviewed |
| Export report to PDF; add to repo and site | Eng | `report/poc_t0_report.pdf` | PDF links from site landing page |
| Write `CITATION.cff` for repo | PI | `CITATION.cff` | GitHub renders citation |

### Day 24–25: Documentation & Contribution Guide

| Task | Owner | Output | Done When |
|---|---|---|---|
| Finalize `README.md` with deployment instructions, quickstart, and CI badge | Eng | Updated `README.md` | README renders correctly on GitHub |
| Write `CONTRIBUTING.md` — question proposal, model proposal, metric improvement workflows | PI | `CONTRIBUTING.md` | CONTRIBUTING.md reviewed |
| Write `docs/structure.md` — full directory tree with explanations | Eng | `docs/structure.md` | All directories and key files documented |
| Write `docs/prompt_design.md` — rationale for system and user prompt choices | PI | `docs/prompt_design.md` | Reviewed by co-investigator |

### Day 25–26: Deployment

| Task | Owner | Output | Done When |
|---|---|---|---|
| Deploy static site to GitHub Pages or Netlify (decision: prefer Netlify for custom domain) | Eng | Public URL documented | Site loads at public URL |
| Set up GitHub Actions CI: on push to `main`, validate all artifacts and re-run site build | Eng | `.github/workflows/ci.yml` | CI passes on main branch |
| Configure `CODEOWNERS` and branch protection on `main` | PI | `CODEOWNERS`; branch protection settings | Direct push to main disabled |

### Day 26–28: Release & Announcement

| Task | Owner | Output | Done When |
|---|---|---|---|
| Make GitHub repository public | PI | Public repo visible | Confirmed public; no credentials in history (run `git-secrets` scan) |
| Tag release `v0.1.0-T0` with annotated tag | Eng | GitHub release `v0.1.0-T0` | Release includes PDF report and score files |
| Write release announcement (≤ 800 words); post to relevant communities | PI | Announcement text in `docs/announcement.md` | Announcement reviewed; communities TBD |
| Open "Phase 2 Planning" GitHub Issue for T1 run and metric improvements | PI | GitHub Issue | Issue created with reference to open questions below |

---

## Open Questions & Risks

| # | Question / Risk | Severity | Mitigation |
|---|---|---|---|
| 1 | **NLI false positive rate:** The NLI model may over-detect contradiction in hedged, normative, or complex propositions, inflating CC. | High | Manual audit of 5% of NLI outputs in Week 2; publish calibration data |
| 2 | **Assumption self-report quality:** Models may give consistent but shallow or empty `declared_assumptions`, making AD uninformative. | Medium | Analyze AD distribution; if median AD < 0.1, revise system prompt to elicit richer assumption disclosure |
| 3 | **Model API non-determinism:** Some APIs (Anthropic) may not support reproducible seeding, making exact replication impossible. | Medium | Document in `run_metadata.deviations`; report harness-level seed as approximation |
| 4 | **API cost overrun:** Full T0 run cost may exceed estimate if models have long output. | Low | Set `max_tokens = 2048` hard limit; monitor spend during pilot |
| 5 | **Question recall bias:** Questions were authored with awareness of the alignment/physics literature; may not cover heterodox positions. | Medium | Domain consultant review planned for T1 question revision cycle |
| 6 | **Weight justification:** Default weights (0.35, 0.40, 0.10, 0.15) are principled but not empirically grounded. | Medium | Sensitivity analysis in Week 3; report rankings as stable/unstable under weight perturbation |
| 7 | **Publication misuse:** Results may be misread as model ranking or "AI safety report card." | High | Prominent plain-language explainer on site landing page; embargo announcement until site is live |

---

## Roles & Responsibilities

| Role | Responsibilities |
|---|---|
| **PI (Principal Investigator)** | Question quality, domain review, report writing, ethics, announcement, final approval of public release |
| **Eng (Engineer)** | Harness implementation, metric computation, schema validation, site build, CI, deployment |
| **Co-Investigator (Domain Consultant, ×2)** | Review question quality for ≥ 2 assigned domains; review report methodology section before publication |

*(Co-Investigators to be confirmed before Phase 2 start.)*

---

## Success Criteria

The project is complete at the end of Week 4 if and only if:

- [ ] All 60 questions have CS scores from ≥ 3 models × 5 seeds for epoch T0  
- [ ] All artifacts are content-addressed and schema-valid  
- [ ] Static site is publicly accessible and renders all 60 question pages without errors  
- [ ] GitHub repository is public with MIT + CC BY 4.0 license files  
- [ ] Report is published (PDF in repo + rendered on site)  
- [ ] `CONTRIBUTING.md` enables external question/model proposals  
- [ ] CI passes on the public main branch  

---

*Dates are relative to Phase 2 kick-off (TBD). All estimates are unverified inferences about execution time; actual duration will depend on team size, API reliability, and prompt iteration cycles.*