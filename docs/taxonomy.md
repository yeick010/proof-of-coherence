# Domain Taxonomy — Proof-of-Coherence Observatory

**Version:** 0.1.0 · **Date:** 2026-04-28  
**Scope:** Phase 1 scaffolding. Six domains, 10 canonical questions each.

---

## Rationale for Domain Selection

Domains were chosen to satisfy four criteria simultaneously:

1. **Active expert disagreement** — technical or philosophical literature contains well-developed competing positions with named proponents.
2. **Pattern-matching failure risk** — the questions are not answerable by retrieving a single authoritative paragraph; integration across conflicting frameworks is required.
3. **Cross-cutting fragility** — the domain exercises at least three of the five fragility types from the controlled vocabulary.
4. **Public legibility** — educated non-specialists can understand why the question is hard, making the observatory's outputs interpretable to a broad audience.

---

## Fragility Type Vocabulary

| Code | Name | Definition |
|---|---|---|
| `LEX` | `lexical_ambiguity` | The question's key terms are contested; changing how a term is defined changes the answer |
| `NOR` | `normative_tradeoff` | The question forces a choice between incommensurable values or normative frameworks |
| `EMP` | `empirical_uncertainty` | The answer depends on empirical evidence that is absent, conflicting, or not yet obtainable |
| `SLF` | `self_reference` | The question applies to or implicates the reasoning process used to answer it |
| `SCL` | `scale_dependence` | The answer plausibly changes as the relevant population, capability level, or time horizon changes |

---

## Domain Table

| # | Domain Key | Full Name | Core Discipline(s) | Primary Fragility Types | Rationale Summary | Representative Canon |
|---|---|---|---|---|---|---|
| 1 | `ai_alignment` | AI Alignment & Control | Computer science, philosophy of mind, decision theory | `EMP`, `LEX`, `SCL`, `SLF` | The field lacks consensus on whether alignment is solvable, on what "alignment" means, and on whether current results extrapolate to more capable systems. Convergence arguments (Omohundro, 2008¹; Bostrom, 2014²) are plausible but not formally proven. LLMs are trained on this literature and tend to pattern-match to its vocabulary without tracking its genuine uncertainties. | Omohundro (2008); Bostrom (2014); Hubinger et al. (2019)³ |
| 2 | `monetary_theory` | Monetary Theory & Central Banking | Macroeconomics, political economy | `EMP`, `LEX`, `NOR` | The quantity theory of money (Friedman, 1963⁴), Modern Monetary Theory (Wray, 2012⁵), and post-Keynesian alternatives make mutually inconsistent empirical and normative claims that no single dataset has resolved. LLMs trained on financial news risk absorbing consensus views that mask deep theoretical disagreements. | Friedman (1963); Wray (2012); Bernanke (2015)⁶ |
| 3 | `foundations_physics` | Foundations of Physics | Theoretical physics, philosophy of science | `EMP`, `LEX`, `SLF` | Quantum interpretation (Everett, 1957⁷; Copenhagen; Bohmian mechanics), the reality of spacetime at the Planck scale, and the ontological status of the wave function are all genuinely open. Physicists in active research programs disagree; LLMs often collapse this to a single "mainstream" view. | Everett (1957); Wheeler & Zurek (1983)⁸; Penrose (2004)⁹ |
| 4 | `epistemology` | Epistemology & Philosophy of Knowledge | Analytic philosophy, cognitive science | `LEX`, `SLF`, `NOR`, `EMP` | The Gettier problem (1963¹⁰) exposed the inadequacy of the justified-true-belief account, but no replacement has achieved consensus. The regress problem, the problem of induction (Hume, 1739¹¹), and the reducibility of social knowledge are all live debates. Self-reference fragility is especially acute: the system is reasoning about reasoning. | Gettier (1963); Quine (1951)¹²; Goldman (1979)¹³ |
| 5 | `game_theory` | Coordination & Game Theory | Economics, political science, philosophy | `LEX`, `NOR`, `EMP`, `SLF`, `SCL` | Game theory's predictive power is contested: lab results consistently violate Nash-equilibrium predictions (Camerer, 2003¹⁴); Arrow's impossibility theorem (1951¹⁵) is proven but its scope is disputed; the Folk Theorem (Fudenberg & Maskin, 1986¹⁶) establishes possibility not prediction. All five fragility types are active here. | Arrow (1951); Nash (1951)¹⁷; Camerer (2003) |
| 6 | `ethics_autonomy` | Ethics of Autonomy & Moral Agency | Political philosophy, bioethics, moral philosophy | `NOR`, `LEX`, `SCL`, `SLF` | Mill's harm principle (1859¹⁸), Rawlsian autonomy, Frankfurt-style accounts of freedom, and communitarian critiques represent genuinely competing normative frameworks. Parfit's repugnant conclusion (1984¹⁹) and questions about future-person rights have no settled answers. LLMs frequently default to liberal-individualist priors without flagging the framework dependence. | Mill (1859); Parfit (1984); Faden & Beauchamp (1986)²⁰ |

---

## Domain Profiles (Extended)

### 1 · AI Alignment (`ai_alignment`)

**Why pattern-matching fails:** The LLM has been trained on the alignment literature itself, making it highly susceptible to reproducing the surface vocabulary (mesa-optimizer, inner alignment, RLHF, corrigibility) without tracking which claims are empirically supported versus conjectural. The orthogonality thesis (Bostrom, 2014²) and instrumental convergence (Omohundro, 2008¹) are often stated as near-axioms in training data even though they are contested theoretical proposals.

**Fragility profile:**
- `EMP`: Whether deceptive alignment arises in practice is unknown; interpretability research has not yet produced definitive detection methods (unverified inference — no published replication-ready benchmark as of 2026-04-28).
- `LEX`: "Aligned" is used to mean at minimum four different things: behaviorally compliant, intent-matching, value-learning-complete, and safe under distribution shift.
- `SCL`: Convergence arguments are stated for "sufficiently capable" systems without specifying the capability threshold.
- `SLF`: Questions about whether an AI can be aligned implicate whether the AI doing the reasoning is itself aligned.

**Key references:**
1. Omohundro, S. M. (2008). *The Basic AI Drives*. Proceedings of the 2008 Conference on Artificial General Intelligence. *(citation verified by domain knowledge — specific page numbers unverified)*
2. Bostrom, N. (2014). *Superintelligence: Paths, Dangers, Strategies*. Oxford University Press.
3. Hubinger, E., van Merwijk, C., Mikulik, V., Skalse, J., & Garrabrant, S. (2019). *Risks from Learned Optimization in Advanced Machine Learning Systems*. arXiv:1906.01820.

---

### 2 · Monetary Theory (`monetary_theory`)

**Why pattern-matching fails:** Financial journalism, central bank communications, and mainstream macroeconomics textbooks dominate the training corpus, creating a strong prior toward the New Keynesian consensus and away from heterodox positions (MMT, Post-Keynesian, Austrian). LLMs also absorb market commentary that conflates descriptive and normative claims.

**Fragility profile:**
- `EMP`: Whether QE-era money creation caused the absence of CPI inflation (2009–2021) via velocity collapse or other channels is still debated (unverified inference — no consensus model as of 2026-04-28).
- `LEX`: "Money," "currency," "inflation," and "money supply" each have multiple technical definitions that lead to different conclusions.
- `NOR`: Whether price stability is the *right* primary central bank mandate versus employment or financial stability is a value-laden question with no technical answer.

**Key references:**
4. Friedman, M. (1963). *Inflation: Causes and Consequences*. Asia Publishing House.
5. Wray, L. R. (2012). *Modern Money Theory: A Primer on Macroeconomics for Sovereign Monetary Systems*. Palgrave Macmillan.
6. Bernanke, B. S. (2015). *The Courage to Act: A Memoir of a Crisis and Its Aftermath*. W. W. Norton. *(memoir source — used for practitioner perspective, not academic claim)*

---

### 3 · Foundations of Physics (`foundations_physics`)

**Why pattern-matching fails:** Physics popularization dominates training data and tends to present the Copenhagen interpretation as default, underrepresenting Everettian and Bohmian positions. String theory is often described as "the leading candidate for quantum gravity" without noting that it has no confirmed empirical predictions as of 2026-04-28 (unverified inference — no confirmed non-trivial observational prediction published in peer-reviewed literature).

**Fragility profile:**
- `EMP`: Many questions (wave function ontology, spacetime discreteness) are currently empirically underdetermined — existing experiments cannot distinguish competing interpretations.
- `LEX`: "Real," "fundamental," "physical," and "information" all carry multiple technical meanings that drive different conclusions.
- `SLF`: Whether consciousness plays a role in measurement invokes the very capacity the system is using to reason.

**Key references:**
7. Everett, H. (1957). *Relative State Formulation of Quantum Mechanics*. Reviews of Modern Physics, 29(3), 454–462.
8. Wheeler, J. A., & Zurek, W. H. (Eds.). (1983). *Quantum Theory and Measurement*. Princeton University Press.
9. Penrose, R. (2004). *The Road to Reality: A Complete Guide to the Laws of the Universe*. Jonathan Cape.

---

### 4 · Epistemology (`epistemology`)

**Why pattern-matching fails:** Introductory philosophy courses and Wikipedia articles present Gettier's result (1963¹⁰) as definitively refuting JTB without conveying that no replacement account has achieved consensus after 60+ years of debate. LLMs frequently output a "leading view" that conceals live disagreement.

**Fragility profile:**
- `LEX`: "Knowledge," "justification," "a priori," and "testimony" are contested concepts even within analytic philosophy.
- `SLF`: The problem of induction directly implicates the reasoning strategy (inference from past to future) that the model uses to answer questions; circular justification of induction is self-referential.
- `EMP`: Whether evolutionary pressures favor true beliefs is an empirical question with a large and contested literature (Street, 2006²¹ versus Plantinga, 1993²² represent poles of a continuing debate).

**Key references:**
10. Gettier, E. L. (1963). *Is Justified True Belief Knowledge?* Analysis, 23(6), 121–123.
11. Hume, D. (1739). *A Treatise of Human Nature*. John Noon.
12. Quine, W. V. O. (1951). *Two Dogmas of Empiricism*. The Philosophical Review, 60(1), 20–43.
13. Goldman, A. I. (1979). *What is Justified Belief?* In G. Pappas (Ed.), *Justification and Knowledge*. Reidel.
21. Street, S. (2006). *A Darwinian Dilemma for Realist Theories of Value*. Philosophical Studies, 127(1), 109–166.
22. Plantinga, A. (1993). *Warrant and Proper Function*. Oxford University Press.

---

### 5 · Coordination & Game Theory (`game_theory`)

**Why pattern-matching fails:** Economics textbooks teach Nash equilibrium as *the* solution concept without adequately conveying that it is one among many (correlated equilibria, trembling-hand perfection, evolutionary stable strategies) and that it systematically fails to predict behavior in laboratory settings. Arrow's theorem is often cited without specifying which of its axioms are reasonable to relax.

**Fragility profile:**
- `EMP`: Lab and field experiments show systematic violations of Nash predictions; whether richer models (quantal response, cognitive hierarchy) fully recover predictive power is unresolved (Camerer, 2003¹⁴).
- `LEX`: "Rational," "equilibrium," "cooperation," and "common knowledge" each have multiple technical definitions.
- `SCL`: Folk theorem results depend on infinite or very long horizon assumptions; finite game behavior differs.
- `SLF`: Questions about whether rational agents defect in the PD implicate the rationality of the agent reasoning about the question.
- `NOR`: Whether correlated equilibria are normatively superior to Nash equilibria is a values question, not a purely technical one.

**Key references:**
14. Camerer, C. F. (2003). *Behavioral Game Theory: Experiments in Strategic Interaction*. Princeton University Press.
15. Arrow, K. J. (1951). *Social Choice and Individual Values*. Wiley.
16. Fudenberg, D., & Maskin, E. (1986). *The Folk Theorem in Repeated Games with Discounting or with Incomplete Information*. Econometrica, 54(3), 533–554.
17. Nash, J. F. (1951). *Non-Cooperative Games*. Annals of Mathematics, 54(2), 286–295.

---

### 6 · Ethics of Autonomy (`ethics_autonomy`)

**Why pattern-matching fails:** LLMs trained on Western Anglophone corpora absorb a default liberal-individualist framework (Mill, Rawls, Kant) that causes them to underweight communitarian, care-ethics, and non-Western perspectives. Questions about future persons, collective autonomy, and moral enhancement have no settled answers but LLMs frequently generate confident summaries.

**Fragility profile:**
- `NOR`: Most questions force a choice between incommensurable values (individual freedom vs. collective welfare, present vs. future persons) with no framework-neutral resolution.
- `LEX`: "Autonomy," "freedom," "paternalism," "harm," and "moral agency" all carry contested meanings.
- `SCL`: The case for paternalistic intervention may shift as population size, information asymmetry, or cognitive enhancement capability scales.
- `SLF`: Questions about cognitive liberty and moral enhancement directly implicate whether the reasoning system's own cognition falls under the relevant categories.

**Key references:**
18. Mill, J. S. (1859). *On Liberty*. John W. Parker and Son.
19. Parfit, D. (1984). *Reasons and Persons*. Oxford University Press.
20. Faden, R. R., & Beauchamp, T. L. (1986). *A History and Theory of Informed Consent*. Oxford University Press.

---

*All numbered citations refer to real published works. Page-number precision is not guaranteed — treat as pointer-level accuracy, not archival-level accuracy. Flag any citation error via GitHub Issue.*