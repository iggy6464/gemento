---
type: result
status: in_progress
updated_at: 2026-05-08
experiment: Stage 6 — Cross-model replication (partial)
---

# Stage 6: Cross-model replication 결과 보고서 (partial)

## 1. 개요

**가설 H14 후보**: Stage 5 의 H11 (Extractor pre-stage) / H12 (Reducer post-stage) / H13 (Search Tool) 의 *direction* 이 cross-family / cross-size 모델에서 generalize.

| 항목 | 내용 |
|------|------|
| **모델 (5)** | Gemma 4 E4B (Stage 5 baseline) / gemma3:4b / gemma3:12b / rnj-1:8b / gpt-oss:20b |
| **실행일** | 2026-05-06 ~ 2026-05-08 (Ollama Cloud free → Pro $20/월) |
| **태스크셋** | main 15 task (Stage 2C 정합) |
| **trial** | 5 / (task, condition) — Stage 5 정합 |
| **외부 API 비용** | $20 (Ollama Cloud Pro 1 month, 3 model concurrent) |
| **인프라** | Ollama Cloud client + multi-account slot (1/2/3, .env), preventive throttle, partial JSON resume |

소스:
- `experiments/cross_model/results/s6_rnj1_h11_h12.json` (rnj-1:8b 마감)
- `experiments/cross_model/results/s6_gpt_oss_h11_h12.json` (gpt-oss:20b 마감)
- `experiments/cross_model/results/partial_stage6_gemma3_12b_ollama_s6_gemma3_12b_h11_h12.json` (gemma3:12b H11 마감, H12 partial)
- (이전) `partial_stage6_gemma3_4b_ollama_s6_gemma3_4b_h11.json` + `s6_gemma3_4b_h12.json` (gemma3:4b 마감)

상세 분석: `docs/reference/stage6-cross-model-analysis-2026-05-08.md`.

## 2. 핵심 메트릭 — H11 (Extractor) 5/5 양수

| 모델 | family | size | Δ | Cohen's d | p (Wilcoxon) |
|---|---|---|---|---|---|
| Gemma 4 E4B (Stage 5) | Gemma | 4B | +0.0500 | +0.323 | 0.198 |
| gemma3:4b | Gemma | 4B | **+0.0787** | +0.299 | 0.594 |
| gemma3:12b | Gemma | 12B | +0.0022 | +0.009 | 0.888 |
| rnj-1:8b | non-Gemma | 8B | +0.0047 | +0.019 | 0.859 |
| gpt-oss:20b | OpenAI/reasoning | 20B | +0.0244 | +0.177 | 0.672 |

→ **5/5 양수 direction match**. Magnitude small-model dependent.

## 3. 핵심 메트릭 — H12 (Reducer) 3/4 음수

| 모델 | size | Δ | Cohen's d | p (Wilcoxon) |
|---|---|---|---|---|
| Gemma 4 E4B (Stage 5) | 4B | −0.0711 | −0.323 | 0.180 |
| gemma3:4b | 4B | **+0.0562** | +0.331 | 0.423 (outlier) |
| rnj-1:8b | 8B | **−0.0989** | **−0.617** | **0.036 ✅ SIG** |
| gpt-oss:20b | 20B | −0.0100 | −0.052 | 0.735 |
| gemma3:12b | 12B | (24/75 진행) | TBD | TBD |

→ **3/4 음수 direction**. gemma3:4b outlier (very weak baseline 0.45). rnj-1:8b 통계 유의.

## 4. H13 cross-model — Gemma family tool-calling 부재

| 모델 | search_chunks 호출 |
|---|---|
| gemma3:4b | 0/50 trial |
| gemma3:12b | "Unknown" + max cycles (dry) |

→ **Gemma 3 family tool-calling 부분 미지원**. paper §4.7 의 family-level finding.

## 5. 결론

### H14 verdict

⚠ **조건부 채택 (cross-model direction generalization 강함, magnitude 모델 의존, 단일 SIG)**:
- H11 direction: 5/5 양수 일관
- H12 direction: 3/4 음수 일관 (1 outlier 가 caveat 직접 evidence)
- rnj-1:8b H12 SIG (Stage 5 narrative 강화)

### Stage 5 ↔ Stage 6 통합 narrative

| 측면 | Stage 5 (single model) | Stage 6 (cross-model) |
|---|---|---|
| Position-effect asymmetry | proposed | **direction confirmed (5/5 H11, 3/4 H12)** |
| H12 keyword scorer caveat | proposed | **gemma3:4b outlier = 부분 evidence** |
| H13 mechanism | under-iteration | + family-level tool-calling 부재 |

## 6. 한계

- n=15 task × 5 trial — Stage 5 정합 한계
- gemma3:12b H12 partial (24/75 baseline) — v2 갱신 시 보강
- LLM-as-judge 보조 평가 미실시 (P1-3)
- cross-family scope 제한 (rnj-1 + gpt-oss 만, Mistral/DeepSeek 미커버)
- H13 cross-model = Gemma family fail; 다른 family (rnj-1, gpt-oss) 의 H13 미실시
- Ollama Cloud Pro $20/월 결제 명시 (paper reproducibility)
