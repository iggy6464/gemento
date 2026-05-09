---
type: result
status: in_progress
updated_at: 2026-05-09
experiment: Stage 6 — Cross-model replication v3 (6 model H11/H12 + 5 model H13 + M2 sub-variants 분화)
---

# Stage 6: Cross-model replication 결과 보고서 v2

## 1. 개요

**가설 H14 후보**: Stage 5 의 H11 (Extractor) / H12 (Reducer) / H13 (Search Tool) 의 *direction* 이 cross-family / cross-size 모델에서 generalize.

| 항목 | 내용 |
|------|------|
| **모델 (6 정규)** | Gemma 4 E4B (Stage 5) / gemma3:4b / gemma3:12b / rnj-1:8b / gpt-oss:20b / **ministral-3:8b** (NEW) |
| **Capability floor finding** | **ministral-3:3b** (H11 31.3% reject, H13 100% reject — paper §4.7.4 finding) |
| **실행일** | 2026-05-06 ~ 2026-05-09 (Ollama Cloud Pro $20/월) |
| **태스크셋** | main 15 task (H11/H12) + longctx 10 task (H13) |
| **trial** | 5 / (task, condition) — Stage 5 정합 |
| **외부 비용** | $20 (Ollama Cloud Pro 1 month, 3-concurrent-model) |

상세 분석: `docs/reference/stage6-cross-model-analysis-2026-05-08.md` (v2 갱신).

## 2. H11 (Extractor) — 6/7 양수, 1 outlier

| 모델 | family | size | Δ | Cohen's d | p (Wilcoxon) |
|---|---|---|---|---|---|
| Gemma 4 E4B (Stage 5) | Gemma 4 | 4B | +0.0500 | +0.323 | 0.198 |
| gemma3:4b | Gemma 3 | 4B | **+0.0787** | +0.299 | 0.594 |
| gemma3:12b | Gemma 3 | 12B | +0.0022 | +0.009 | 0.888 |
| rnj-1:8b | non-Gemma | 8B | +0.0047 | +0.019 | 0.859 |
| gpt-oss:20b | OpenAI | 20B | +0.0244 | +0.177 | 0.672 |
| **ministral-3:8b** | **Mistral 3** | **8B** | **−0.0433** ⚠ | **−0.292** | 0.311 |

→ **6/7 양수 direction match**, 1 outlier (ministral-3:8b). Magnitude small-model dependent.

## 3. H12 (Reducer) — Family-systematic pattern

| 모델 | size | Δ | Cohen's d | p (Wilcoxon) | family direction |
|---|---|---|---|---|---|
| Gemma 4 E4B (Stage 5) | 4B | −0.0711 | −0.323 | 0.180 | non-Gemma 음수 |
| gemma3:4b | 4B | **+0.0562** | +0.331 | 0.423 | Gemma 3 양수 |
| gemma3:12b (final) | 12B | **+0.0078** | +0.026 | 0.878 | Gemma 3 양수 |
| rnj-1:8b | 8B | **−0.0989** | **−0.617** | **0.036 ✅ SIG** | non-Gemma 음수 |
| gpt-oss:20b | 20B | −0.0100 | −0.052 | 0.735 | non-Gemma 음수 |
| **ministral-3:8b** | **8B** | **−0.0707** | **−0.287** | 0.327 | non-Gemma 음수 NEW |

→ **Gemma 3 family 2/2 양수**, **non-Gemma 4/4 음수**. *family-systematic pattern* = paper §4.6.2 의 *style mismatch caveat* 직접 evidence.

## 4. H13 cross-model — **Specific-model identification (size threshold 아님)**

| 모델 | search_tool 작동 | failure mode |
|---|---|---|
| Gemma 4 E4B (Stage 5) | partial (1-call premature) | **(M1) under-iteration** → Δ=−0.220 SIG |
| gemma3:4b | 0/50 calls | (M2-a) tool-calling absence |
| gemma3:12b | "Unknown" max-cycle | (M2-b) tool-calling not invoked |
| ministral-3:3b | 100% no-converge | (M2-c) tool-calls + final_answer 0 |
| ministral-3:8b | 100% no-converge (44/50) | (M2-c) tool-calls + final_answer 0 |
| **gemma4:31b** | **90% fail (45/50)** | **(M2-d) A-agent JSON schema mismatch** ⚠ NEW |

→ **5 small-and-mid dense 모델 모두 H13 fail. Gemma 4 E4B 만 (M1) 으로 측정 가능.** 동일 family 31B 도 fail (M2-d) — **size 가 아닌 *specific-model identification***.

**gemma4:31b baseline_chunked (no tool) 결과**: 50/50 trial, mean_acc **0.95**, errors 0/50 → 모델 자체는 26K-token document ABC chain 정상 처리. 실패는 *strictly* A-agent search-tool JSON schema contract 부합 실패.

### Mechanism — M1 + M2 (4 sub-variants)

- **(M1) under-iteration on multi-hop** — Gemma 4 E4B *한정*
- **(M2-a) tool-calling absence** — gemma3:4b
- **(M2-b) tool-calling not invoked** — gemma3:12b
- **(M2-c) final_answer 미생성** — ministral-3:3b/8b
- **(M2-d) A-agent JSON schema mismatch** ⚠ NEW — gemma4:31b

→ paper §4.7.4 mechanism narrative 5-mode 분화. §1.3 contribution 1 *narrowing*: "iteration effect (M1) 측정은 *Gemma 4 E4B 한정* — A-agent JSON-schema contract 가 *measurement-tool fit*. 다른 contract 는 다른 모델에서 M1 측정 가능할 수 있음".

## 5. 결론

### H14 verdict v3

⚠ **조건부 채택 (direction match 강함, family-systematic pattern, mechanism 5-mode 분화, single SIG, *measurement-tool fit caveat*)**:
- H11: 6/7 양수, 1 outlier (ministral-3:8b)
- H12: family-systematic (Gemma 3 양수 / non-Gemma 음수) — *style-mismatch caveat 직접 evidence*
- H13: 5 small-and-mid dense 모델 모두 fail. **(M1) measurable — Gemma 4 E4B 한정**. **(M2) 4 sub-variants 분화**. *Specific-model identification*, size threshold 아님.
- 통계 유의: rnj-1:8b H12 (p=0.036, |d|=0.617)
- A-agent JSON-schema contract 의 *measurement-tool fit* 명시 (paper §4.7.4 / §6 limitations)

### Stage 5 ↔ Stage 6 v2 통합 narrative

| 측면 | Stage 5 (single model) | Stage 6 v2 (cross-model) |
|---|---|---|
| Position-effect asymmetry | proposed | **direction confirmed** (6/7 H11, 4/6 H12) |
| H12 keyword scorer caveat | proposed (a vs b 분리 불가) | **family-systematic 직접 evidence** (Gemma 3 양수, non-Gemma 음수) |
| H13 mechanism | under-iteration 단일 | **두 갈래 분기**: M1 under-iteration (Gemma 4 E4B) / M2 capability floor (다른 small dense) |
| minimum operational size | 미식별 | **≈ Gemma 4 E4B (effective 4B)** — 그 아래 ABC + tool 사용 불가 |

## 6. 한계 (v2)

- n=15 task × 5 trial — Stage 5 정합 한계
- LLM-as-judge 보조 평가 미실시 (P1-3) — H12 (a/b) 완전 분리 불가
- cross-family scope — non-Gemma = rnj-1 / gpt-oss / Mistral 3 (3 family)
- H13 cross-model statistical 비교 불가 (M2 capability floor → search_tool 0% 작동)
- ministral-3:3b REJECT (H11 31.3%, H13 100%) — capability floor finding 으로 보존
- gemma4:31b H13 dry-run 미실시 — Gemma 4 family 안 size 효과 미검증
- Ollama Cloud Pro $20/월 결제 명시 (paper reproducibility)

## 7. 변경 이력

- 2026-05-08 v1: 5 model H11 (5/5 양수) + 3 model H12 (3/4 음수) + Gemma 3 family tool-calling 부재.
- 2026-05-09 v2: gemma3:12b H12 final + ministral-3:8b 추가 + ministral-3:3b capability floor finding + H13 family-agnostic small-dense fail. Narrative 갱신: 6/7 H11 + family-systematic H12 + Mechanism 분기 (M1/M2) + minimum operational size threshold.
- 2026-05-09 v3: gemma4:31b H13 추가 (Gemma 4 family same-family size-up). search_tool 90% fail — **(M2-d) A-agent JSON schema mismatch** sub-variant 발견. baseline_chunked 95% acc 정상 → 모델 capability 가 아닌 contract fit 문제. **H14 verdict 갱신**: H13 작동 = *Gemma 4 E4B 한정*, *size threshold* 가 아닌 *specific-model identification*. M2 4 sub-variants 분화. paper §1.3 *narrowing* + measurement-tool fit caveat 명시.
