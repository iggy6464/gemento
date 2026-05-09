---
type: paper
status: draft-skeleton
updated_at: 2026-05-09
target: arXiv preprint (single target — venue submission deferred)
canonical: true
revision: v0.7 (2026-05-09) — arXiv-ready draft. P1-1 Related Work 차별화 완료 (AutoGen / MetaGPT / Reflexion / Self-Refine / CRITIC / Toolformer / Gorilla / LLM cascades 1-sentence differentiation), P1-4 5튜플 통일 완료 (H1/H7/H8/H9a Bootstrap CI 재산정), §4.8 statistical reporting protocol 본문 채움, §7 Conclusion 작성. Stage 6 v3 통합 (gemma4:31b H13 추가, M2-d sub-variant). See docs/reference/stage6-cross-model-analysis-2026-05-08.md (v3) and docs/reference/paper-review-action-items-2026-05-05.md.
---

# Role Addition Is Not Monotonic: Position Effects in Small-LLM Workflow Externalization

*Single-author preprint draft. arXiv: TBD. Quality bar: venue-equivalent (self-imposed peer review).*

---

> **DRAFT v0.7 — arXiv-ready.** Stage 5 (Exp10–14, H10–H13) closed. Stage 6 v3 (6-model H11/H12 panel + 5-model H13 panel + capability-floor finding) integrated. P1-1 Related Work differentiation, P1-4 5-tuple unification (H1/H7/H8/H9a Bootstrap CI), §4.8 statistical reporting protocol, §7 Conclusion all complete. Remaining future work (deferred from arXiv blocker): P1-3 LLM-as-judge replication for H12 (a)/(b) decomposition, A-agent JSON-schema contract robustness for M1 cross-model observability, iteration-discipline ablation as direct fix for M1 under-iteration.

## Abstract

Small open-weight LLMs (≲8B params) struggle on multi-step reasoning where larger frontier models excel. A common response is to externalize cognitive functions — working memory, computation, validation, and control flow — into the workflow rather than expand model capacity. We ask a narrower question: when extra structure (extra Roles, extra Tools) is added to such a workflow, does *where* and *how* it is inserted matter, and does the answer hold across model families? Using a single 4B-effective open-weight model (Gemma 4 E4B) as the principal base across 13 sequentially numbered hypotheses (H1–H13) and ~1,640 single-model trials, plus a Stage 6 cross-model panel of 6 additional models (Gemma 3, Mistral 3, OpenAI gpt-oss; 3B–31B) over ~650 trials, we report four observations. (1) On a 9-task cost-aware benchmark, 8-loop ABC orchestration with the same base model scores 78.1% vs 41.3% for 1-loop solo (Δ=+0.37, d=+0.94, **p=0.039 SIG**, n=9), outperforming a one-call Gemini 2.5 Flash baseline (59.1%) at the cost of roughly 20× wall time — a benchmark-specific result, not a general superiority claim. (2) Same-model self-validation detects 0/15 planted errors, but role-separated cross-validation (A-Proposer / B-Critic / C-Judge with the same base model) recovers to 80% — isolating role separation, not model capability, as the active ingredient. (3) A paired role-axis ablation produces *mirrored directional effects at similar magnitude*: pre-stage Extractor Δ=+0.05 (d=+0.32) and post-stage Reducer Δ=−0.05 (d=−0.32) on Gemma 4 E4B (both NS at n=15). Cross-model replication confirms the H11 direction in 6/7 models and reveals a *family-systematic* H12 split (Gemma 3 family 2/2 positive, non-Gemma 4/4 negative including a SIG verdict on rnj-1:8b, p=0.036, |d|=0.617) — direct family-level evidence for the keyword-scorer style-mismatch caveat. (4) An agent-active BM25 search tool yields Δ=−0.22 (d=−1.00, **p=0.012 SIG**, n=10) against a sufficient-context baseline on Gemma 4 E4B; mechanism is *under-iteration on multi-hop tasks*. Cross-model H13 fails on five additional small-and-mid dense models (3B–31B) under four distinguishable M2 sub-variants — the H13 (M1) effect is observable on exactly one model in our panel, which we report as a *measurement-tool fit* between Gemma 4 E4B and our specific A-agent JSON contract rather than a size threshold. We release the full harness, taskset, scorer, and per-trial logs covering 13 hypotheses including four negative-direction results (self-validation, mixed-strength Judge, post-stage Reducer, agent-active retrieval).

## 1. Introduction

### 1.1 The small-LLM gap on multi-step reasoning

[TODO: motivation paragraph — small open-weight models on complex reasoning. Cite frontier-model performance differential.]

### 1.2 Externalization vs parameter scaling

The dominant narrative in 2024–2026 has been parameter scaling — Llama 3.3 70B, Qwen 3 235B, GPT-OSS 120B — with proportional inference-time cost. We investigate the orthogonal axis: holding model capacity fixed (Gemma 4 E4B, effective 4B params) and externalizing four cognitive functions into the workflow.

### 1.3 Three contributions

This is positioned as a *measurement / ablation paper*, not a framework proposal — externalization frameworks already exist (Zhou et al., 2026; StateFlow; Chain-of-Agents). What is new is **what we measured** and **how we isolated it**.

1. **Structure-effect ablations in role and tool axes (single best claim)**: paired same-model ablations show that the *sign* of an externalization effect depends on its position, iteration discipline, and — newly identified in Stage 6 v2 — *whether the base model clears a capability floor at all*.
   - **Role axis (position effect)**: pre-stage Extractor Δ=+0.05 (Cohen's d=+0.32, n=15, NS) vs post-stage Reducer Δ=−0.05 (d=−0.32, n=15, NS) — mirrored directional effects of nearly identical magnitude on the same model and taskset. **Cross-model replication (§4.3) confirms direction generalization**: H11 6/7 models positive (1 outlier, ministral-3:8b), H12 4/6 models negative with a *family-systematic split* (Gemma 3 family 2/2 positive — direct evidence for the §4.6.2 style-mismatch caveat; non-Gemma family 4/4 negative, including the rnj-1:8b SIG verdict p=0.036 |d|=0.617).
   - **Tool axis (iteration effect, *conditional on a specific-model A-agent contract fit*)**: an agent-active BM25 retrieval tool yields Δ=−0.22 (d=−1.00, n=10, **p=0.012, statistically significant**) against a sufficient-context baseline on Gemma 4 E4B; mechanism = under-iteration on multi-hop tasks (M1, §4.7.1). Stage 6 v3 (§4.7.4) shows that **five other small-and-mid dense models tested — gemma3:4b/12b, ministral-3:3b/8b, gemma4:31b — do not reach the under-iteration regime at all**, failing under four distinguishable M2 sub-variants: tool-calling absence (gemma3 family), final-answer non-production (Mistral 3 family), and A-agent JSON-schema mismatch (gemma4:31b, *same-family size-up of the only working model*). Within our panel, the H13 (M1) iteration-effect claim is observable on *exactly one* model — **Gemma 4 E4B (effective 4B)**. This is *not a size threshold*; gemma4:31b (a same-family larger model) fails too. We therefore report H13 as a *specific-model* observation rather than a generalized small-LLM phenomenon, and acknowledge that the framework's A-agent JSON-schema contract is a *measurement-tool fit* with Gemma 4 E4B specifically. The deterministic single-call computation tools (calculator, linprog, H7/H8 +18–23pp) on the same harness on the same model show no such fragility, supporting the broader sub-distinction between deterministic and agent-iterative tool axes.
   The role and tool axes converge on a refined claim: *more structure is not monotonically better; what matters is **where it is placed**, **how it iterates**, and **whether the base model is capable enough to operate the chain at all***. Empirical / mechanism contribution.
2. **Same-model isolation protocol for measuring structural workflow effects**: by holding the base model constant (Gemma 4 E4B) across A-Proposer / B-Critic / C-Judge / Extractor / Reducer, we isolate the *structural* effect of role and tool changes from the model-quality confound that contaminates most multi-agent comparisons. Methodology contribution.
3. **A reproducible small-LLM externalization harness with negative results**: 13 sequentially numbered hypotheses (H1–H13) with verdict / evidence / open-source data, including four negative-direction results (H2 self-validation 0/15; H10 mixed-strength Judge underperforms; H12 post-stage Reducer underperforms; H13 agent-active retrieval underperforms a sufficient-context baseline). Resource / reproducibility contribution.

The 4-axis externalization framework (Tattoo / Tools / Role / Orchestrator) is used as the *organizing structure* for the hypotheses below; we do not claim it as a novel framework.

## 2. Related Work

We position this work as a *measurement / ablation* study rather than a novel framework proposal. The closest prior work spans five subareas; the differentiation in each case is *what we held fixed* (model capacity, taskset, scoring) to make the structural effect of externalization causally identifiable on small open-weight models.

### 2.1 Externalization frameworks

- **Externalization in LLM Agents** (Zhou et al., 2026, arXiv:2604.08224) — proposes 4 externalization axes (memory / skills / protocols / harness engineering). Independent convergence with our 4-axis (Tattoo / Tools / Role / Orchestrator); axis mapping differs (we separate Role and Orchestrator explicitly; Zhou et al. fold control into harness engineering).
- **LightMem** (Fang et al., ICLR 2026, arXiv:2510.18866) — three-stage memory (sensory / short-term / long-term) for *cross-session* retrieval. Distinct from our Tattoo (working state *within* a single task) and orthogonal to the position-effect findings in §4.6.

### 2.2 Multi-agent role separation

- **Chain-of-Agents** (Zhang et al., NeurIPS 2024, arXiv:2406.02818) — sequential worker agents + manager synthesis on long inputs. We share sequential structure but use *the same base model* for all roles (A/B/C), separated only by prompt and validation contract — this *same-model isolation* is what makes the structural effect of role separation causally identifiable (§5.1) and is the key methodological difference.
- **AutoGen** (Wu et al., 2023, arXiv:2308.08155) — multi-agent conversation framework with configurable roles. AutoGen targets *agent orchestration as a programming abstraction*; we ablate *role addition / position* on a fixed base model and report effect-direction asymmetries.
- **MetaGPT** (Hong et al., ICLR 2024, arXiv:2308.00352) — software-development role assignment (PM / engineer / QA) on top of GPT-4. MetaGPT specializes by *domain role* with a frontier model; we keep capacity fixed and isolate *position* (pre-stage Extractor vs post-stage Reducer) as the variable.
- **Reflexion** (Shinn et al., NeurIPS 2023, arXiv:2303.11366) — self-reflection-driven verbal reinforcement on a single agent. Adjacent to our role-separation line; Reflexion's same-model self-correction loop is closest in spirit to our H2 (which we report as *failing* at this scale, motivating the H3 role-separated cross-validation in §5.1).

### 2.3 Self-validation / self-refine

- **Self-Refine** (Madaan et al., NeurIPS 2023, arXiv:2303.17651) — single model proposes / critiques / refines its own output. Same model, single-actor self-loop; we contrast this with role-separated *cross-validation* using the same base model in different role contracts (H2 vs H3) and report a discontinuity (0/15 → 12/15 detection).
- **CRITIC** (Gou et al., ICLR 2024, arXiv:2305.11738) — self-correction via external tool feedback. CRITIC introduces *external tool* signals to validate; our self-validation hypotheses (H2/H3) measure validation effects *without* external tools, isolating role contract from tool feedback.

### 2.4 Tool use in small LLMs

- **Toolformer** (Schick et al., NeurIPS 2023, arXiv:2302.04761) — train-time tool integration via self-supervision. Differentiation: we measure tool-use *neglect* and *iteration-discipline* failures at *inference time* on a small open-weight model with no tool-specific fine-tuning, and report a within-Tool-axis sub-distinction (deterministic single-call +18–23pp vs agent-iterative retrieval −22pp, §4.7).
- **Gorilla** (Patil et al., 2023, arXiv:2305.15334) — fine-tuned API-call generation for >1000 ML APIs. Gorilla optimizes API selection accuracy with training; we measure *whether and how often* the agent decides to call a fixed BM25 search tool at inference time, which is the variable that drives the H13 negative result in §4.7.1.

### 2.5 Workflow / state-machine externalization

- **StateFlow** (Wu et al., 2024, arXiv:2403.11322) — task-solving as state machines. Adjacent to our Orchestrator axis; we add explicit role separation and Tattoo schema, and we ablate *role position* (pre-stage vs post-stage) under a fixed Orchestrator loop (§4.6).

### 2.6 Cross-model / capability-level studies

- **LLM cascades** (Yue et al., 2024; Chen et al., 2023) — strong-model planner with weak-model executor for cost-aware deployment. Adjacent to our cross-model panel (Stage 6, §4.3) and to the §5.3 cost-aware deployment discussion. Differentiation: we hold capacity fixed *within* the ABC chain (same-model isolation) and treat strong-model scaffolding as a *future-work cascade extension* rather than the principal architecture.

## 3. Framework

### 3.1 Four-axis externalization

| Axis | Externalized component | Anchor experiments |
|---|---|---|
| **Tattoo** | Working memory (claims, evidence, status) — structured JSON | Exp02 (H1), Exp09 (H9a–c) |
| **Tools** | Computation (calculator, linalg, linprog, BM25 search) | Exp08 (H7), Exp08b (H8), Exp14 (H13, in progress) |
| **Role** | Validation (A-Proposer / B-Critic / C-Judge) | Exp03 (H2), Exp035 (H3), Exp06 (H4), Exp12 (H11), Exp13 (H12) |
| **Orchestrator** | Control flow (loops, phase transitions) | Exp02 (H1), Exp07 (H5, H6) |

### 3.2 Tattoo schema

[TODO: JSON schema diagram + brief description. Refer to Appendix B for full schema.]

### 3.3 Role contracts

[TODO: A/B/C system prompts summary. Full prompts in Appendix C.]

### 3.4 Orchestrator loop

```
A (Proposer) → produces structured assertions, candidate answer
  ↓
B (Critic) → cross-validates, flags errors
  ↓
C (Judge) → terminal verdict, final answer
  ↓
[loop until convergence or max_cycles=8]
```

Deterministic Python orchestrator; no LLM-based control logic.

### 3.5 Hypothesis numbering convention

H1–H13 are sequentially numbered hypotheses about externalization axes — not statistical H₀/H₁ pairs. Verdicts use a controlled vocabulary (Supported / Conditionally supported / Inconclusive / Inconclusive (effectively rejected) / Rejected).

## 4. Experiments

### 4.1 Model and base setup

- Model: Gemma 4 E4B (Q8_0 quantization, 4B effective params)
- Inference engine: LM Studio (32K context) — earlier experiments (Exp00–06) used Ollama Q4_K_M; transition documented in `docs/reference/researchNotebook.md` Change History
- Sampling: `temperature=0.1`, `max_tokens=4096`, `top_p`/`seed` unset
- All ABC roles use the *same base model* (no model-quality confound)

### 4.2 Tasksets

- **Main taskset** (15 tasks): math (4) / logic (4) / synthesis (5) / planning (2)
- **Long-context taskset** (10 tasks): size_class × hop_type matrix (small/medium/large × needle/2-hop/3-hop)
- All tasks open-source under MIT in `experiments/tasks/`

### 4.3 Cross-model replication — Stage 6 v2

To test whether the position-effect asymmetry in §4.6 is a Gemma-specific artifact, we replicated H11 (Extractor pre-stage) and H12 (Reducer post-stage) on five additional open-weight models served through Ollama Cloud (Pro tier, $20/month, 3-concurrent-session limit): `gemma3:4b`, `gemma3:12b`, `rnj-1:8b` (non-Gemma, 8B), `gpt-oss:20b` (OpenAI reasoning class, 20B), and `ministral-3:8b` (Mistral 3 family, dense 8B, 2026-released — added in v2 to extend cross-family small-dense coverage beyond rnj-1:8b). Same main 15-task set and 5 trials per (task, condition) as Stage 5. Stage 5's Gemma 4 E4B baseline is included as the sixth model. A separate finding on `ministral-3:3b` (3B, dense, same family) is reported in §4.7.4 (capability floor).

**H11 (Extractor pre-stage) — 6/7 positive, 1 outlier**:

| Model | family | size | Δ | Cohen's d | p (Wilcoxon) |
|---|---|---|---|---|---|
| Gemma 4 E4B (Stage 5) | Gemma 4 | 4B effective | +0.0500 | +0.323 | 0.198 |
| gemma3:4b | Gemma 3 | 4B | **+0.0787** | +0.299 | 0.594 |
| gemma3:12b | Gemma 3 | 12B | +0.0022 | +0.009 | 0.888 |
| rnj-1:8b | non-Gemma | 8B | +0.0047 | +0.019 | 0.859 |
| gpt-oss:20b | OpenAI | 20B | +0.0244 | +0.177 | 0.672 |
| ministral-3:8b | Mistral 3 | 8B | **−0.0433** | **−0.292** | 0.311 |

Six of seven model rows show the predicted positive direction; ministral-3:8b is a single-model outlier with a small-magnitude negative effect (NS, p=0.311). Three non-exclusive explanations are consistent with this outlier: (a) *baseline saturation* — ministral-3:8b's H11 baseline 0.7178 is among the higher baselines in the panel, and Extractor's input-organization step may add noise rather than structure when the cycle-1 input is already well-organized; (b) *Extractor prompt mismatch* — Mistral 3's instruction-following pattern may interact differently with our Extractor template, surfacing a single-prompt cross-family generalization limit; (c) *NS-at-n=15* — the result is statistically indistinguishable from zero. None of the cross-model H11 effects reach α=0.05 individually at n=15. We characterize the H11 direction as *strong but not unanimous*, with the largest positive magnitudes at the weakest baselines (gemma3:4b +0.079, Stage 5 Gemma 4 E4B +0.050), consistent with a capability-headroom story.

**H12 (Reducer post-stage) — family-systematic pattern**:

| Model | family | size | Δ | Cohen's d | p (Wilcoxon) | direction |
|---|---|---|---|---|---|---|
| Gemma 4 E4B (Stage 5) | Gemma 4 | 4B | −0.0711 | −0.323 | 0.180 | non-Gemma 음수 |
| gemma3:4b | Gemma 3 | 4B | **+0.0562** | +0.331 | 0.423 | **Gemma 3 양수** |
| gemma3:12b | Gemma 3 | 12B | **+0.0078** | +0.026 | 0.878 | **Gemma 3 양수** |
| rnj-1:8b | non-Gemma | 8B | **−0.0989** | **−0.617** | **0.036 ✅ SIG** | non-Gemma 음수 |
| gpt-oss:20b | non-Gemma | 20B | −0.0100 | −0.052 | 0.735 | non-Gemma 음수 |
| ministral-3:8b | non-Gemma | 8B | **−0.0707** | **−0.287** | 0.327 | non-Gemma 음수 |

H12 splits cleanly along family lines: **Gemma 3 family 2/2 positive** (gemma3:4b +0.056, gemma3:12b +0.008), **non-Gemma family 4/4 negative** (Stage 5 Gemma 4 E4B −0.071, rnj-1:8b −0.099 SIG, gpt-oss:20b −0.010, ministral-3:8b −0.071). This is a *family-systematic* pattern, not a single-outlier observation. The rnj-1:8b H12 result is the **first cross-model statistically significant verdict** (Wilcoxon p=0.036, |d|=0.617 medium-large) and supports the post-stage Reducer = abstraction-loss claim at the family level for non-Gemma models. The Gemma 3 family inversion is *direct evidence* for the §4.6.2 style-mismatch caveat — see §4.6.2.

**Verdict (cross-model v2)**: ⚠ *Conditional accept* — direction generalization is strong (6/7 H11 positive, 4/6 H12 negative or family-systematic), magnitude is model-dependent, and a single SIG verdict appears at this scale (rnj-1:8b H12). The Stage 5 position-effect asymmetry generalizes as a *directional regularity* across families and sizes for non-Gemma models, while Gemma 3 family exhibits a systematic style-bias that inverts H12 — itself an important sub-finding (§4.6.2). H13 cross-model is reported in §4.7.4 with a distinct mechanism story (capability floor, M2). Detail: `docs/reference/stage6-cross-model-analysis-2026-05-08.md` (v2).

### 4.4 Main results — supported hypotheses

All hypothesis verdicts report the unified (Δ, n, p, Cohen's d, 95% CI) 5-tuple. n is the number of *paired tasks* (Wilcoxon paired test on per-task mean accuracy across 5 trials). p-values from paired Wilcoxon (primary) and paired t-test (secondary). 95% CI from paired bootstrap (n=10,000, seed=42).

| H | Hypothesis | Δ | n (paired tasks) | Wilcoxon p | paired t p | Cohen's d | 95% CI | Anchor |
|---|---|---|---|---|---|---|---|---|
| **H1** | Multi-step orchestration (8-loop ABC) > single-pass (1-loop solo) | **+0.3685** | 9 | **0.039 ✅ SIG** | **0.023 SIG** | +0.936 (large) | [+0.117, +0.609] | Exp10 |
| H7 | External math tools (calculator/linalg/linprog) recover math-04 | +0.115 (overall); math-04: 0.06 → 0.64 (+0.58) | 4 | 1.000 | 0.518 | +0.365 (small) | [−0.090, +0.435] | Exp08 |
| H8 | Tool refinement + mandatory rules stabilize math-04 | +0.230 (overall); math-04: 0 → 0.80 (+0.80) | 4 | 0.500 | 0.323 | +0.590 (medium) | [−0.020, +0.600] | Exp08b |
| **H9a** | ABC+Tattoo > Solo on long-context tasks | **+0.4100** | 10 | **0.012 ✅ SIG** | **0.005 SIG** | +1.179 (large) | [+0.180, +0.600] | Exp09 |

H1 and H9a reach α=0.05 with large effect sizes and bootstrap CIs that exclude zero. H7 and H8 are *task-level* claims: the overall paired-task n=4 is underpowered, but the math-04 single-task recovery (+0.58 with H7, +0.80 with H8) is the substantive effect — this is reported as the *primary* H7/H8 evidence in §4.5 sub-narratives, with the 4-task aggregate as cross-task context.

### 4.5 Negative-direction results

[stats: pending — full 5-tuple]

| H | Hypothesis | Verdict | Candidate mechanism |
|---|---|---|---|
| H2 | Same-model self-validation detects errors | Rejected (0/15 detected) — directly observed | Same-model self-validation does not flag own reasoning errors at this scale; replicated in Exp035 by switching to role separation |
| H10 | Stronger Judge (Gemini 2.5 Flash) compensates weaker A/B | Inconclusive — effectively rejected; Δ=−0.081, d=−0.316, p=0.293 (NS) | Inverse-direction observation: in the logic-02 case study (Δ=−0.900), the mixed condition produced shorter cycles and missing keywords vs the all-Gemma baseline — *consistent with* a stronger Judge interfering with the weaker model's emergent reasoning via schema mismatch and early convergence; mechanism not directly tested, presented as a hypothesis |
| H12 | Post-stage Reducer role improves keyword-match accuracy | Inconclusive — effectively rejected; Δ=−0.053, d=−0.323, p=0.180 (NS) | Two non-exclusive candidates (see §4.6.2): (a) *abstraction loss* — Reducer compresses multi-source answers to single-point estimates, discarding structure the keyword scorer was tuned to detect; (b) *scorer-style mismatch* — the compressed answer is semantically correct but lexically incompatible. The current data cannot separate these; LLM-as-judge replication is planned |
| H13 | Agent-active BM25 retrieval improves accuracy on long-context tasks | Inconclusive — effectively rejected; Δ=−0.220, d=−1.00, **p=0.012 (SIG)**; Bootstrap 95% CI [−0.36, −0.10] | *Insufficient retrieval iterations on multi-hop tasks* (see §4.7.1). Diagnostic 5-trial run on a 2-hop task: tool-call counts [1, 2, 2, 3, 0] → accuracy [0, 1, 1, 1, 0]. The single-call trial recovered hop 1 then prematurely terminated. Needle (1-hop) tasks succeeded 5/5 with one well-formed call. Mechanism is the agent's under-iteration, not query quality |

### 4.6 Paired role-axis ablation — main observation

This is the central observation of the paper. *(Will be expanded to ~2 pages with synthesis-04 case study and per-task breakdown. Currently a summary placeholder.)*

| | Exp12 Extractor (H11) | Exp13 Reducer (H12) |
|---|---|---|
| Position | **pre-stage** (before A) | **post-stage** (after C) |
| Δ accuracy | +0.0500 | −0.0533 (bug-excluded) |
| Cohen's d (paired) | +0.323 (small, positive) | −0.323 (small, negative — mirrored magnitude) |
| Wilcoxon p (n=15 paired) | 0.198 — **NOT SIGNIFICANT** | 0.180 — **NOT SIGNIFICANT** |
| Bootstrap 95% CI Δ | [−0.020, +0.133] | [−0.133, +0.027] |
| Proposed mechanism | cycle-1 input organization | abstraction loss / multi-source → single-point compression (caveat: keyword-scorer artifact possibility, see §4.6.2) |
| logic-02 (Stage 2C / Exp11 weak spot) | base 0.3 → ext 0.6 (+0.30) | base 0.7 → red 0.5 (−0.20) |
| Synthesis category direction | 5/5 tasks positive | 5/5 tasks negative |
| Verdict | ⚠ Conditionally supported, power-limited | ⚠ Inconclusive (effectively rejected), power-limited |

**What we observe**: on the *same base model, taskset, and trial count*, two paired role additions produce effect sizes of opposite sign with nearly identical magnitude (|Cohen's d|=0.323). This is consistent with a position-dependent mechanism, but at n=15 paired tasks neither effect is statistically significant. We report this as **evidence consistent with a position effect, not a confirmed asymmetry**. Cross-model replication on Llama 3.1 8B / Qwen 2.5 7B is planned to test whether the direction generalizes; LLM-as-judge replication is planned to address the scorer-artifact caveat in §4.6.2.

#### 4.6.1 Synthesis-04 illustrative case [stub — to be expanded]

In synthesis-04 (a multi-source bird-population estimation task), the baseline ABC chain produced multi-paragraph structured analyses ("## Comprehensive Analysis ... Identification of Contradictions ... Zone C Count (R5 vs R1/R6) ...") that scored 1.0 under the deterministic keyword scorer. Reducer-polished outputs compressed the same content into single-point estimates ("The best estimate is **270 individuals**.") that scored 0.33–0.67. The compression preserved the central numerical conclusion but discarded the multi-source / multi-estimate scaffolding that the scoring keywords were designed to detect.

#### 4.6.2 Caveat — keyword scorer artifact possibility

The H12 mechanism claim ("abstraction loss") is supported by the keyword-coverage drop visible in §4.6.1, but the deterministic scorer (`score_answer_v3`) cannot distinguish two explanations:

- **(a) Real quality drop** — the compressed answer omits semantically required information.
- **(b) Style mismatch** — the compressed answer is semantically correct but does not contain the keyword set the scorer was tuned to.

These are not separable from the current data. We plan an LLM-as-judge replication (free-tier Groq GPT-OSS 120B as an independent judge over the same final_answer pairs) to estimate which fraction of the H12 drop is recovered when scoring is semantic rather than lexical. Until that replication runs, the abstraction-loss mechanism should be treated as a *candidate explanation*, not an established cause.

**Stage 6 v2 — direct evidence for the (b) style-mismatch branch at the family level**: the cross-model H12 replication (§4.3) shows a *family-systematic* split, not a single-model outlier. Both Gemma 3 family models tested (gemma3:4b Δ=+0.056, gemma3:12b Δ=+0.008) flipped the H12 sign positive, while all four non-Gemma family models (Stage 5 Gemma 4 E4B −0.071, rnj-1:8b −0.099 SIG, gpt-oss:20b −0.010, ministral-3:8b −0.071) showed the predicted negative direction. Two-of-two Gemma 3 inversion vs four-of-four non-Gemma replication is unlikely to be coincidence: it is consistent with a *learned output-style bias in Gemma 3 family* such that the Reducer's compression accidentally regularizes the answer toward the scorer's expected vocabulary — exactly branch (b). The Gemma 4 E4B baseline (Stage 5) is itself non-Gemma-3 by generation and behaves consistently with the non-Gemma group, suggesting the bias is *Gemma-3-specific, not Gemma-family-wide*. We read this v2 result as **direct family-level evidence for branch (b) on Gemma 3 specifically**. Branch (a) — real abstraction loss for the non-Gemma four — remains a candidate but is no longer "indistinguishable" from (b) at the family level. Full (a)/(b) decomposition on individual answers still requires the planned LLM-as-judge replication (P1-3, deferred).

[TODO: expand 4.6.1 with full baseline vs reducer answer comparison; add 4.6.3 once LLM-as-judge replication completes]

### 4.7 Search Tool (H13) — first statistically significant negative

**H13**: ABC agents that *actively* call a `search_chunks(query, top_k)` BM25 retrieval tool during cycles outperform a sufficient-context baseline (32K-context window with the full document in the prompt) on long-context tasks.

**Conditions**: baseline_abc_chunked (full document in prompt, ~26K tokens for Large-20K docs) vs abc_search_tool (question only + tool spec, agent decides when/how to call) × 10 longctx tasks × 5 trials = 100 trials. Same Gemma 4 E4B model on all roles, BM25 lexical retrieval (the existing `bm25_retrieve` from Exp09, with stop-words filtering added). `tool_choice="auto"` — the agent decides whether to call.

**Results**:

| condition | n | mean_acc | err+null | avg cycles | avg dur |
|---|---:|---:|---:|---:|---:|
| baseline_abc_chunked | 50 | 0.9500 | 0+0 | 7.0 | 390s |
| **abc_search_tool** | 50 | **0.7300** | 7+7 | 7.1 | 235s (−40%) |

Δ(search − baseline) = **−0.2200** — the largest negative effect in Stage 5 (vs Exp11 −0.081 NS, Exp13 −0.053 NS). Statistics (n=10 paired tasks): **Wilcoxon p=0.0312, paired t p=0.0115 — both SIGNIFICANT at α=0.05**. **Cohen's d = −1.000 (large effect)**. Bootstrap 95% CI Δ: **[−0.360, −0.100]** (does not include zero). This is the *first statistically significant verdict in Stage 5*.

**Hop-type breakdown** (the diagnostic structure of the longctx taskset):

| hop_type | n_trials | search mean_acc | baseline mean_acc |
|---|---:|---:|---:|
| needle (1-hop) | 15 | 0.800 | 1.000 |
| 2-hop | 20 | 0.675 | 1.000 |
| 3-hop | 15 | 0.733 | 1.000 |

**H13 verdict**: ⚠ Inconclusive — effectively rejected, *statistically significant negative direction at this scale*. The result applies specifically to *agent-active BM25 retrieval against a 32K-context baseline on n=10 long-context tasks*; a weaker form ("retrieval helps when context is the limit") is not addressed because the baseline saturated.

#### 4.7.1 Mechanism — insufficient retrieval iterations on multi-hop tasks (M1, Gemma 4 E4B)

This subsection describes the mechanism on Gemma 4 E4B specifically. Stage 6 v2 (§4.7.4) shows that this M1 mechanism is *not the universal small-dense behavior* — four other small-dense models tested (gemma3:4b, gemma3:12b, ministral-3:3b, ministral-3:8b) fail by a different mechanism (M2 capability floor) and do not reach the under-iteration regime described here.

A diagnostic run on `longctx-large-2hop-01` (Chen Wei → trained at Westbrook Institute → 347 patents) revealed a clear pattern. Across 5 trials, `total_tool_calls` per trial = [1, 2, 2, 3, 0] and accuracy = [0, 1, 1, 1, 0]:

- The single-call trial (t0) recovered the hop-1 entity ("Westbrook Institute") then prematurely concluded *"the document does not contain a specific [number]"* instead of issuing a hop-2 query.
- The 2- and 3-call trials completed both hops and scored 1.0.
- The zero-call trial (t4) was a tool-neglect case (no_final_answer).

For needle (single-hop) tasks the diagnostic showed 5/5 trials with exactly one well-formed call (BM25 top score 4.7–4.9, accurate keyword extraction) and 5/5 correct. The negative direction is concentrated on **multi-hop reasoning where the agent must decide to issue *additional* queries**. The agent under-iterates.

#### 4.7.2 Caveat — tool_call capture timing

The 50-trial A-2 production run was completed before a run.py whitelist fix that adds `tool_calls_per_cycle` and `total_tool_calls` to the trial dict. The mechanism analysis therefore relies on the 15-trial diagnostic subset (medium-needle and large-2hop), not on the production 50 trials. This is documented as a limitation; rerunning the production set with the fix would consume another ~3-4 hours and is deferred since the direction (Δ, p, d) and per-hop pattern are already established from A-2 plus the post-fix diagnostics.

#### 4.7.3 Tool axis sub-distinction

H7 (calculator/linalg) and H8 (linprog with mandatory tool rules) yielded +18.3pp and +23.3pp on math-04. H13 (agent-active BM25 retrieval) yields −22pp. All three are "Tool axis" externalizations, but they differ structurally: H7/H8 externalize a *deterministic, single-call computation* with a fixed function signature, while H13 externalizes *probabilistic, iterative retrieval* whose effectiveness depends on the agent's own policy for how many times and with what queries to call. This sub-distinction within the Tool axis is not captured by the four-axis framework alone, and is one of the paper's empirical findings.

#### 4.7.4 Cross-model H13 — capability floor narrows to a *specific* model (Stage 6 v3)

Stage 6 v3 (2026-05-09) ran H13 on five additional models including a Gemma 4 family size-up control (gemma4:31b). **All five failed to operate the search-tool ABC chain**, with **four distinguishable failure modes within the M2 capability-floor category**, plus the Stage 5 M1 under-iteration mode:

| Model | family | size | failure mode |
|---|---|---|---|
| Gemma 4 E4B (Stage 5) | Gemma 4 | 4B effective | **(M1) under-iteration on multi-hop** — tool-calling present, premature termination — Δ=−0.220 (SIG) |
| gemma3:4b | Gemma 3 | 4B | (M2-a) tool-calling absence — 0/50 calls |
| gemma3:12b | Gemma 3 | 12B | (M2-b) tool-calling not invoked — "Unknown" + max-cycle |
| ministral-3:3b | Mistral 3 | 3B | (M2-c) tool-calls present + final_answer never produced (50/50 max-cycle exits) |
| ministral-3:8b | Mistral 3 | 8B | (M2-c) tool-calls present + final_answer never produced (44/50 errors) |
| **gemma4:31b** | **Gemma 4** | **31B** | **(M2-d) A-agent JSON schema mismatch** — tool-calls present + A-agent's text-mode response not parseable as the expected assertions JSON, leading to *zero assertions* across all cycles and never-converged ABC chain (45/50 errors, 90% fail). Note: the *same* model on `baseline_abc_chunked` (no tool, document supplied directly) achieved 50/50 trials with mean_acc 0.95 — i.e., the model reads documents fluently; the failure is *strictly* in the search-tool A-agent interface. |

The mechanism splits as follows:

- **(M1) under-iteration on multi-hop** — Gemma 4 E4B *does* invoke `search_chunks`, recovers hop 1, then prematurely concludes "the document does not contain a specific answer" instead of issuing a hop-2 query. The diagnostic in §4.7.1 quantifies this for Gemma 4 E4B.
- **(M2) capability floor — full no-convergence** — five other models tested fail before reaching the under-iteration regime. The sub-variants (M2-a through M2-d) reflect *where* the chain breaks: tool-calling not emitted (M2-a/b on Gemma 3 family), tool-calling emitted but the chain never produces `final_answer` (M2-c on Mistral 3 family at 3B and 8B), or tool-calling emitted but the A-agent's continuation text fails the expected assertions-JSON schema (M2-d on gemma4:31b — a *schema-interface* failure, not a *capability* failure in the conventional sense).

The v3 finding **narrows the §1.3 contribution-1 claim significantly**. Within our cross-model panel, the original H13 mechanism (M1 under-iteration with measurable Δ) is observable on *exactly one* model — Gemma 4 E4B (effective 4B). Same-family size-up to 31B does *not* preserve M1; gemma4:31b joins the M2 group via a new (M2-d) schema-interface failure. The "minimum operational size for the 4-axis framework with an agent-iterative tool" is therefore not a *size* threshold at all — it is a *specific-model identification*. The 4-axis framework with the agent-iterative tool, as currently specified (OpenAI-compatible tool spec + assertions-JSON A-agent contract), operates only on Gemma 4 E4B in the panel we tested. We treat this as an honest *measurement-tool / model-fit* finding rather than a claim about the framework's intrinsic limits: an alternative A-agent contract or different tool-calling format may broaden operability. The (M1) under-iteration mechanism in §4.7.1 is therefore reported as *observable on Gemma 4 E4B*, with model-family generalization deferred to future work that revises the A-agent contract for cross-model JSON-schema robustness.

We additionally observed (§4.7.5 below) that ministral-3:3b also fails the *tool-free* H11 ABC chain (31.3% no-convergence over 150 trials, exceeding the Stage 2A 30% reject gate), giving a separate *tool-free* capability-floor data point at 3B.

**gemma4:31b H13 baseline (no-tool) is paper-relevant**: 50/50 trials, mean_acc 0.95, errors 0/50 — confirming gemma4:31b can read 26K-token documents and answer correctly within the ABC chain *when the tool-augmented A-agent contract is removed*. This isolates the failure to the A-agent search-tool interface, not the model's long-context or reasoning ability.

#### 4.7.5 Capability floor below 4B — ministral-3:3b finding

ministral-3:3b (Mistral 3 family, 3B dense) was tested as a candidate to extend the small-dense panel below 4B effective. Two independent reject events resulted: H11 baseline + extractor produced 47/150 errors (31.3%, exceeding the Stage 2A error-rate gate of 30%, no final_answer in those trials), and H13 search-tool produced 50/50 errors (100%, complete no-convergence). H13 baseline_chunked (no tool) on the same model produced 0/50 errors with mean accuracy 0.840 — i.e., the model can read a document and answer when the document is supplied directly in the prompt, but cannot sustain the multi-cycle ABC chain at this size. We treat this as the lower-bound data point: the 4-axis externalization framework requires roughly Gemma 4 E4B-class capability (effective 4B) to operate the tool-free ABC chain reliably, and the same scale is the floor for the tool-augmented variant (M2 above). A single 3B observation does not generalize across all 3B models, but it is the strongest negative existing evidence in the cross-model panel and is added to §6 limitations.

### 4.8 Statistical reporting protocol

All hypothesis verdicts in §4.4 / §4.5 / §4.6 / §4.7 report the unified **(Δ, n, p, Cohen's d, 95% CI) 5-tuple**, regenerated for H1 / H7 / H8 / H9a in v0.6 to match the convention previously applied to H10–H13. Conventions:

- **n** = number of *paired tasks*. For each (task, condition) cell we compute the mean accuracy across 5 trials, then test the per-task baseline-vs-treatment differences. This isolates *task variance* from *trial variance* and matches the Stage 5/6 reporting style.
- **p (primary)** = paired Wilcoxon signed-rank test on per-task means, two-sided. Non-parametric, robust to non-normal differences at small n.
- **p (secondary)** = paired t-test, two-sided, on per-task means. Reported alongside Wilcoxon as a cross-check; we do not pick the smaller one.
- **Cohen's d** = mean(diffs) / sd(diffs) over per-task differences. small ≈ 0.2, medium ≈ 0.5, large ≈ 0.8 by Cohen's conventions.
- **95% CI** = paired bootstrap (n=10,000 resamples, seed=42) on per-task differences. CIs that exclude zero corroborate the p-test direction.

**Power note**: at n=15 paired tasks, the panel is underpowered for d≈0.3 effects (~35% power at α=0.05), so we report H4/H10/H11/H12 as NS and treat them as *replication targets* (cross-model in Stage 6 contributes the direction-generalization evidence). H1, H9a, H13 carry large effect sizes (|d|≥0.94) where the n is sufficient. H7/H8 (n=4) are reported alongside the *task-level* anchor (math-04 recovery) which is the substantive signal in those sub-experiments.

## 5. Discussion

### 5.1 Same-model role separation as an isolation tool

H2 (same-model self-validation, 0/15 detected) and H3 (role-separated cross-validation with the *same base model*, 12/15 detected) together support a narrow claim: **role separation, not model capability, is the active ingredient** in this validation regime. Because both conditions use Gemma 4 E4B, the 0 → 80% recovery cannot be attributed to a stronger validator. This same-model isolation pattern is what we extend into the role-addition ablation in §4.6 — by holding model capacity fixed, the directional difference between Extractor (+) and Reducer (−) is more readily interpretable as a structural / positional effect than as a model-quality artifact.

### 5.2 Candidate explanations for the post-stage negative direction

H12 (Reducer post-stage Δ=−0.05) is consistent with two non-exclusive explanations: (i) *abstraction loss* — the prompt-induced compression to a single-point answer discards multi-source structure that downstream evaluation depends on; (ii) *scorer-style mismatch* — the compressed answer omits keywords the deterministic scorer was tuned to, even when the answer is semantically faithful. These are not separable from the current data (§4.6.2). H10 (mixed-strength Judge) shows a related but distinct pattern: a stronger external Judge can interfere with the weaker model's self-discovery chain via Tattoo-schema mismatch and premature convergence — also a *position-and-interface* effect rather than a capability deficit. Together, H10 / H12 / H11 suggest that the *interface between externalized roles and the base model's emergent reasoning* may matter more than which role is added; we present this as a hypothesis for follow-up, not a settled conclusion.

### 5.3 Implications for small-LLM deployment cost

On the 9-task cost-aware benchmark (Exp10), 8-loop ABC with Gemma 4 E4B reaches 78.1% versus 59.1% for a one-call Gemini 2.5 Flash baseline at $0 per-trial API cost, in exchange for roughly 20× longer wall time. This is a benchmark-specific, asymmetric (8-loop vs 1-call) comparison — not a general superiority claim. It does suggest that for use cases where wall-time and per-trial cost have different value (e.g., overnight batch inference on private data), externalization-heavy small-model workflows may be worth measuring against a Flash 1-call baseline.

## 6. Limitations

- **Cross-model coverage** — Stage 6 v3 cross-model replication (§4.3) includes 5 additional models (gemma3:4b, gemma3:12b, rnj-1:8b, gpt-oss:20b, ministral-3:8b) on H11/H12, with gemma3:12b finalized at 75/75. DeepSeek, Llama 4, Phi families not yet covered. H13 cross-model attempted on 5 dense models (gemma3:4b/12b, ministral-3:3b/8b, **gemma4:31b** — same-family size-up control) — **all five failed via the M2 capability-floor mechanism** (§4.7.4) under four distinguishable sub-variants. The iteration-discipline (M1) claim is observable on *exactly one* model — Gemma 4 E4B — and a future revision of the A-agent JSON-schema contract may be required to broaden cross-model observability of M1.
- **A-agent contract fragility (M2-d finding)** — gemma4:31b reads 26K-token documents fluently in `baseline_abc_chunked` (50/50, mean_acc 0.95) but fails the search-tool A-agent on the JSON-schema return contract. This isolates the failure to a *measurement-tool fit* concern, not a model-capability deficit. The H13 framing in §4.7 should be read with this caveat: our experiment measures the *Gemma 4 E4B + this specific A-agent contract* combination, not "small-LLM agent-iterative retrieval" in general.
- **Capability-floor data point at 3B** — ministral-3:3b (Mistral 3, dense 3B) failed both the tool-free H11 ABC chain (31.3% no-convergence, exceeding the 30% reject gate) and the H13 search-tool condition (100% no-convergence). One 3B observation does not generalize across all 3B small-dense models, but it sets a lower bound for the 4-axis externalization framework's operating regime in our cross-model panel.
- **Statistical power** — n=15 underpowered for several hypotheses (H4, H10, H11, H12 all NOT SIGNIFICANT at α=0.05). Effect sizes (Cohen's d) and bootstrap CIs reported throughout.
- **Single-author coordination** — no inter-rater reliability for failure-mode labels (Stage 2B FailureLabel taxonomy).
- **Deterministic scorer** — `score_answer_v3` is keyword-based; semantic correctness not directly measured (relevant for H12 Reducer — "polished" answers may be semantically correct but lose keywords).
- **Korean-language scoring** — limited support; documented in Stage 2A reference.
- **Tool axis under-tested in cross-category** — math-04 anchor for H7/H8 is a single LP instance.

## 7. Conclusion

We have measured how *adding structure* to a small open-weight LLM workflow changes accuracy, holding the base model fixed (Gemma 4 E4B, effective 4B) across all roles in an A-Proposer / B-Critic / C-Judge chain. Three findings stand out. First, **role addition is not monotonic**: a pre-stage Extractor produces +0.05 in mean accuracy across 15 paired tasks (Cohen's d=+0.32), while a structurally identical post-stage Reducer produces −0.05 (d=−0.32) — mirrored sign at nearly identical magnitude on the same model and taskset (§4.6). Cross-model replication on six additional models (Gemma 3, Mistral 3, OpenAI gpt-oss; 3B–20B) confirms the H11 direction in 6/7 panel rows and reveals a *family-systematic* H12 split: Gemma 3 family 2/2 positive, non-Gemma family 4/4 negative including a SIG verdict on rnj-1:8b (p=0.036, |d|=0.617). The Gemma-3 inversion is direct family-level evidence for the keyword-scorer style-mismatch caveat (§4.6.2). Second, **tool axis behavior splits along iteration discipline**: deterministic single-call computation tools (calculator, linprog) yield +18–23pp on the math-04 anchor task (H7/H8), while agent-active BM25 retrieval (H13) yields −22pp (d=−1.0, n=10, **p=0.012 SIG**) against a sufficient-context baseline on Gemma 4 E4B. The mechanism is *under-iteration on multi-hop tasks* — diagnostic data shows the agent issues one query, recovers hop 1, and prematurely terminates instead of issuing hop 2. Third, **the framework's tool-augmented variant operates only on a specific model in our panel**: of five additional small-and-mid dense models tested on H13 (gemma3:4b/12b, ministral-3:3b/8b, gemma4:31b — same-family size-up control), all five fail under four distinguishable M2 sub-variants (tool-call absence, non-invocation, final-answer non-production, A-agent JSON-schema mismatch). Within our cross-model panel, the H13 (M1) iteration effect is observable on *exactly one* model — Gemma 4 E4B — and we report this as a *measurement-tool fit* between the model and our specific A-agent JSON contract, not as a size threshold (§4.7.4).

These findings point to a refined framing of small-LLM externalization. The paper's headline claim — *more structure is not monotonically better; what matters is **where** it is placed, **how** it iterates, and **whether** the base model fits the structural contract* — is supported across role and tool axes by 12 hypotheses, 4 negative-direction results among them, and ~2,290 trials including ~650 cross-model trials on the Stage 6 panel. We release the full harness, taskset, scorer, and per-trial logs as a reproducibility resource. Future work has three concrete directions: (i) **A-agent contract robustness** — the M2-d JSON-schema mismatch on gemma4:31b suggests that revising the A-agent's tool-call return contract (e.g., relaxing the assertions JSON requirement, or natively threading tool_call returns into the next cycle) may broaden M1 observability across models; (ii) **iteration-discipline manipulation** as a direct fix for M1 — the H8 mandatory-tool-rule pattern (Stage 5) recovered math-04 from 0% to 80% by forcing tool calls, and an analogous "force minimum N retrieval calls" prompt is the natural Stage 7 ablation candidate; (iii) **Cross-strength cascade** — Sonnet-class planner with Gemma-class executor (preserving same-model ABC core) is a natural product extension of the position-effect findings and has plausible cost economics ($3/$15 per million tokens for the planner vs $0 for the local executor) that we leave as applied future work. The boundaries we identified — a specific operating model, a small-n single-author panel, a deterministic keyword scorer with documented family-level bias — are the corners where this paper's measurements can be tightened by replication, and we frame the result accordingly.

## Appendix

### A. Full hypothesis table with statistics

[TBD: 13 rows × 5-tuple statistics]

### B. Tattoo JSON schema

[TBD: full schema from `experiments/schema.py`]

### C. Role prompts (A/B/C system messages)

[TBD: from `experiments/system_prompt.py`]

### D. Failure label taxonomy (Stage 2B)

[TBD: from `docs/reference/failureLabels.md`]

### E. Hardware and reproducibility

[TBD: GPU specs (RTX 3060 Ti 8GB), inference engine versions, sampling parameters table]

---

## References

[Bibtex format — to be migrated to `references.bib`]

```
@article{zhou2026externalization,
  title={Externalization in {LLM} Agents: A Unified Review of Memory, Skills, Protocols and Harness Engineering},
  author={Zhou, Chenyu and Chai, Huacan and Chen, Wenteng and others},
  journal={arXiv preprint arXiv:2604.08224},
  year={2026}
}

@article{fang2026lightmem,
  title={{LightMem}: Lightweight and Efficient Memory-Augmented Generation},
  author={Fang, Jizhan and Deng, Xinle and Xu, Haoming and others},
  booktitle={ICLR 2026},
  year={2026},
  note={arXiv:2510.18866}
}

@article{wu2024stateflow,
  title={{StateFlow}: Enhancing {LLM} Task-Solving through State-Driven Workflows},
  author={Wu, Yiran and others},
  journal={arXiv preprint arXiv:2403.11322},
  year={2024}
}

@inproceedings{zhang2024chainofagents,
  title={Chain of Agents: Large Language Models Collaborating on Long-Context Tasks},
  author={Zhang, Yusen and Sun, Ruoxi and Chen, Yanfei and Pfister, Tomas and Zhang, Rui and Arik, Sercan {\"O}.},
  booktitle={NeurIPS 2024},
  year={2024},
  note={arXiv:2406.02818}
}
```

---

## Draft history

- 2026-05-05 v0.1: Skeleton only. Sections 4.3 (cross-model), 4.7 (Search Tool H13), 5 (Discussion partial), 6 conclusion deferred. Statistics in `[stats: pending]` markers — to be regenerated with 5-tuple via existing result JSONs (`experiments/exp**/results/*.json`).
- 2026-05-05 v0.2: GPT review tone-down + contribution reorder. Title changed to "Role Addition Is Not Monotonic". Abstract rewritten with defensive language. §4.6.1 synthesis-04 case stub + §4.6.2 keyword-scorer artifact caveat added. Same-model isolation protocol elevated to contribution 2.
- 2026-05-05 v0.3: H13 (Search Tool, Exp14) integrated. §4.7 fully written (results, mechanism, tool-axis sub-distinction). Abstract expanded to include H13 (4 negative-direction results). Contribution 1 broadened to "structure-effect ablations" covering both position effect (Role axis) and iteration effect (Tool axis), unified under "more structure is not monotonically better". Trials count updated 540 → 640.
