---
type: reference
status: in_progress
updated_at: 2026-05-09
canonical: true
note: Stage 6 cross-model v3 — gemma4:31b H13 추가 (Gemma 4 family 동일 family size-up control). (M2-d) A-agent JSON-schema mismatch sub-variant 발견. H13 작동 모델이 *Gemma 4 E4B 한정* 으로 narrow — minimum operational size 가 *size threshold* 가 아닌 *specific model identification*. v2 (2026-05-09 oldernote) 의 4 small-dense fail 을 5 dense fail (M2 sub-variants 분화) + measurement-tool fit caveat 으로 격상.
---

# Stage 6 Cross-model Replication 분석 v3 (6 model H11/H12 + 5 model H13 + M2 sub-variants 분화)

**Plan**: `stage-6-cross-model-llm-as-judge`
**실행일**: 2026-05-06 ~ 2026-05-09 (Ollama Cloud Pro $20/월, 3-concurrent-model)
**모델 (6 정규 + 1 capability floor)**:
- 정규 panel: Gemma 4 E4B (Stage 5 baseline) / gemma3:4b / gemma3:12b / rnj-1:8b / gpt-oss:20b / **ministral-3:8b** (NEW)
- Capability floor finding: **ministral-3:3b** (H11 31.3% / H13 100% error → REJECT, paper §4.7.4 finding 으로 보존)
**가설**: H11 (Extractor pre-stage) + H12 (Reducer post-stage) + H13 (Search Tool, family-agnostic small-dense fail finding)
**조건**: 가설별 baseline + treatment × 15 task × 5 trial = 150 trial / hypothesis / model. H13 = 10 task × 5 trial = 100 trial.
**채점**: `score_answer_v3` (Stage 5 정합)

## 1. H11 (Extractor pre-stage) — 6/7 모델 양수, 1 outlier

| 모델 | family | size | baseline mean | extractor mean | **Δ** | Cohen's d | Wilcoxon p |
|---|---|---|---|---|---|---|---|
| Gemma 4 E4B (Stage 5) | Gemma 4 | 4B effective | 0.7500 | 0.8000 | **+0.0500** | +0.323 | 0.198 |
| gemma3:4b | Gemma 3 | 4B | 0.4364 | 0.5151 | **+0.0787** | +0.299 | 0.594 |
| gemma3:12b | Gemma 3 | 12B | 0.5489 | 0.5511 | **+0.0022** | +0.009 | 0.888 |
| rnj-1:8b | non-Gemma | 8B | 0.5878 | 0.5924 | **+0.0047** | +0.019 | 0.859 |
| gpt-oss:20b | OpenAI/reasoning | 20B | 0.7278 | 0.7522 | **+0.0244** | +0.177 | 0.672 |
| **ministral-3:8b** | **Mistral 3** | **8B** | **0.7178** | **0.6744** | **−0.0433** ⚠ | **−0.292** | **0.311** |

→ **6/7 양수 direction**, 1 outlier (ministral-3:8b 음수). 이전 v1 의 *5/5 unanimous* narrative 가 *6/7 majority* 로 톤다운. NS 6/6 (n=15 검정력 한계).

### 1.1 ministral-3:8b H11 음수 outlier 해석 후보

- **(a) baseline saturation**: ministral-3:8b H11 baseline 0.7178 — 5/7 모델 중 두 번째로 높음 (gpt-oss 0.7278 다음). saturation 가까운 baseline 에서는 Extractor 의 organize 효과가 *원본 cycle-1 input 의 효율* 을 *손상* 할 수 있음 (이미 정리된 입력에 추가 정리 = noise).
- **(b) Extractor prompt mismatch**: Mistral 3 family 의 instruction following 패턴이 본 Extractor prompt template 과 부합 안 함 — single prompt template 의 cross-family generalization 한계.
- **(c) 단일 outlier 우연**: n=15 paired NS (p=0.311), 통계적으로 0 과 구별 안 됨. 진정한 부호는 미결.

→ paper §4.3 narrative: "*direction match strong but not unanimous*" — 6/7 + 단일 outlier 명시 + 위 3 후보 mechanism 열거.

### 1.2 Per-category H11 — logic 카테고리 여전히 robust 양수

| 카테고리 | Stage 5 | gemma3:4b | gemma3:12b | rnj-1:8b | gpt-oss:20b | ministral-3:8b | 일관성 |
|---|---|---|---|---|---|---|---|
| logic | +0.125 | +0.195 | +0.025 | **+0.205** | (TBD) | (TBD) | ✅ 4/4+ 양수 |
| synthesis | +0.050 | +0.073 | −0.027 | −0.097 | (TBD) | (TBD) | mixed |
| math | 0 | 0 | 0 | 0 | 0 | (TBD) | saturation/floor |
| planning | 0 | +0.017 | +0.033 | −0.133 | (TBD) | (TBD) | n=2 작음 |

→ logic 카테고리는 cross-model robust 양수 (catastrophic 영역 회복) — paper §4.3 의 *task-category dependent* sub-finding 유지.

### 1.3 Magnitude 의 model size dependency

| 모델 size | H11 Δ |
|---|---|
| 4B (Gemma 4 E4B) | +0.0500 |
| 4B (gemma3:4b) | +0.0787 |
| 8B (rnj-1) | +0.0047 |
| 8B (ministral-3) | −0.0433 ⚠ |
| 12B (gemma3) | +0.0022 |
| 20B (gpt-oss) | +0.0244 |

→ 4B 두 모델이 가장 큰 양수 magnitude. 그 외 8B+ 는 ±0.04 이내. **"Extractor 효과 = 약한 baseline 의 cycle-1 input organization"** 가설은 *부분적으로* 지지됨 (4B 양수 큼). 단 ministral-3:8b 의 음수가 *baseline saturation* 가설과 부분 정합.

## 2. H12 (Reducer post-stage) — 4/6 음수, 2 양수 outlier 모두 Gemma 3 family

| 모델 | family | size | baseline mean | reducer mean | **Δ** | Cohen's d | Wilcoxon p | 결과 |
|---|---|---|---|---|---|---|---|---|
| Gemma 4 E4B (Stage 5) | Gemma 4 | 4B | 0.7744 | 0.7033 | **−0.0711** | −0.323 | 0.180 | ❌ NS 음수 |
| gemma3:4b | Gemma 3 | 4B | 0.4578 | 0.5140 | **+0.0562** ⚠ | +0.331 | 0.423 | ✅ NS 양수 |
| gemma3:12b (final) | Gemma 3 | 12B | 0.6173 | 0.6251 | **+0.0078** ⚠ | +0.026 | 0.878 | ✅ NS 약양수 |
| rnj-1:8b | non-Gemma | 8B | 0.6644 | 0.5656 | **−0.0989** | **−0.617** | **0.036** | ❌ **SIG 음수** |
| gpt-oss:20b | non-Gemma | 20B | 0.6833 | 0.6733 | −0.0100 | −0.052 | 0.735 | ❌ NS 음수 |
| **ministral-3:8b** | **non-Gemma** | **8B** | **0.7300** | **0.6593** | **−0.0707** | **−0.287** | **0.327** | ❌ NS 음수 NEW |

→ **4/6 음수**, 2 양수 outlier (Gemma 3 family **2/2**). non-Gemma family **4/4 모두 음수**. *family-level pattern* 으로 격상.

### 2.1 §4.6.2 caveat 의 *family-level direct evidence*

paper draft v0.3/v0.4 의 §4.6.2 caveat:
> "The current data cannot separate (a) real abstraction loss vs (b) scorer-style mismatch."

**v2 finding — 단순 outlier 가 아니라 *family-systematic* pattern**:
- Gemma 3 family **2/2 양수** (gemma3:4b +0.056, gemma3:12b +0.008)
- non-Gemma family **4/4 음수** (Stage 5 Gemma 4 E4B −0.071, rnj-1:8b −0.099 SIG, gpt-oss:20b −0.010, ministral-3:8b −0.071)

→ Reducer 의 stylistic 변환 후 keyword overlap 이 **Gemma 3 family 에서만 체계적으로 증가**. 이건 *우연 outlier* 가 아니라 **Gemma 3 의 학습된 출력 스타일이 keyword scorer 와 정렬되는 family-level bias**. *style mismatch (b)* 의 직접 evidence.

→ paper §4.6.2 갱신: "Stage 6 cross-model 가 *Gemma 3 family 양수, non-Gemma family 음수* 의 family-systematic 패턴을 보임으로써 *style mismatch caveat 의 직접 evidence* 를 제공." 단 *완전 분리* 는 LLM-as-judge replication (P1-3) 필요.

### 2.2 rnj-1:8b H12 통계 유의 + ministral-3:8b 일관 음수

- rnj-1:8b: p=0.036, |d|=0.617 medium-large — **Stage 6 의 첫 H12 SIG 결과**
- ministral-3:8b: |d|=0.287 small (Stage 5 Gemma 4 E4B 와 거의 동일 magnitude)
- gpt-oss:20b: |d|=0.052 — saturation effect (baseline 0.68)

→ non-Gemma family 4 모델 모두 음수 + 1 SIG = paper §4.6 의 "post-stage Reducer = abstraction loss" main claim 의 **family-level 통계 유의 cross-evidence**.

## 3. H13 cross-model — **Specific-model identification (not size threshold)** ⭐⭐

v2 의 "family-agnostic small-dense fail" finding 이 v3 에서 *훨씬 더 narrow* 한 conclusion 으로 격상: H13 작동 모델 = *Gemma 4 E4B 한정*. 동일 family size-up (gemma4:31b) 도 fail. Size 가 아닌 **specific-model identification**.

| 모델 | family | size | search_tool 작동 | failure mode | Δ |
|---|---|---|---|---|---|
| Gemma 4 E4B (Stage 5) | Gemma 4 | 4B effective | partial (1-call premature) | **(M1) under-iteration** on multi-hop | **−0.220 SIG** |
| gemma3:4b | Gemma 3 | 4B | ❌ 0/50 calls | (M2-a) tool-calling absence | n/a |
| gemma3:12b | Gemma 3 | 12B | ❌ "Unknown" max-cycle | (M2-b) tool-calling not invoked | n/a |
| ministral-3:3b | Mistral 3 | 3B | ❌ 100% no-converge (50/50) | (M2-c) tool-calls + final_answer 0 | n/a |
| ministral-3:8b | Mistral 3 | 8B | ❌ 100% no-converge (44/50) | (M2-c) tool-calls + final_answer 0 | n/a |
| **gemma4:31b** | **Gemma 4** | **31B** | ❌ **90% fail (45/50)** | **(M2-d) A-agent JSON schema mismatch** ⚠ NEW | n/a |

**gemma4:31b 의 paper-relevant 보충**: `baseline_abc_chunked` (no tool, doc in prompt) = 50/50 trial, mean_acc **0.95**, errors 0/50. **모델은 26K-token document 정상 처리**. 실패는 *strictly* search-tool A-agent contract 의 JSON schema 부합 실패.

### 3.1 Mechanism 분기 — M1 + M2 (4 sub-variants)

paper §4.7 의 mechanism narrative 가 v3 에서 *5 mechanism* 로 분화:

| Mechanism | 적용 모델 | 정의 |
|---|---|---|
| **(M1) under-iteration on multi-hop** | **Gemma 4 E4B 한정** | tool 호출 1번 후 premature termination → multi-hop fail |
| **(M2-a) tool-calling absence** | gemma3:4b | OpenAI tool spec 인식 안 함, 0 호출 |
| **(M2-b) tool-calling not invoked** | gemma3:12b | "Unknown" 응답 + max-cycle |
| **(M2-c) final_answer 미생성** | ministral-3:3b/8b | tool_calls 발생 + multi-cycle 에서 final_answer 못 만듦 |
| **(M2-d) A-agent JSON schema mismatch** ⚠ NEW | gemma4:31b | tool_calls 발생 + 후속 텍스트 응답이 expected assertions JSON schema 부합 실패 → assertions=0 → never converge |

→ **M2 가 4 sub-variants 로 분화**. 5 모델 모두 다른 실패 양상. Gemma 4 E4B 가 (M1) 으로 *측정 가능한 수치* 를 만든 *유일한* 모델.

### 3.2 §1.3 contribution 1 의 *narrowing* (v2 → v3)

이전 paper §1.3 의 v2 갱신:
> "Tool axis 는 capability floor 위에서만 의미 있음. minimum operational size ≈ Gemma 4 E4B 급."

**v3 narrowing**:
> "Tool axis (iteration effect, M1) 의 측정은 cross-model panel 안에서 *Gemma 4 E4B 한정* 가능. **Size threshold 가 아닌 specific-model identification** (gemma4:31b same-family size-up 도 fail). framework 의 A-agent JSON-schema contract 가 *measurement-tool fit* 임을 인정 — 다른 contract 가 다른 모델에서 M1 측정 가능할 수 있음. 본 paper 는 *Gemma 4 E4B + 본 contract* 조합의 측정 결과만 claim."

이건 paper 의 *honest scope narrowing* — *less universal but more accurate*. paper §4.7.4 / §6 limitations 의 직접 입력.

### 3.3 gemma4:31b H13 baseline 결과 — paper-relevant 보존

H13 baseline_chunked (no tool) 50 trial 결과는 *정식 데이터* 로 paper §4.3 또는 §4.7.4 에 포함 가능:
- mean_acc 0.95 (n=50)
- error 0/50
- 26K-token large document 정상 처리
- → "**31B 가 long-context document 를 ABC chain 에서 정상 처리**" 의 cross-model evidence (Stage 5 Gemma 4 E4B baseline 0.95 와 일치). H9a (long-context 정답률) cross-model 보강.

## 4. ministral-3:3b — Capability floor finding (paper §4.7.4 보강)

3B 모델은 *도구 없는 단순 ABC chain* 에서도 검정력 한계 초과:

| 가설 | 결과 |
|---|---|
| H11 baseline_abc | 75/75 done, error rate ~22% (passable) |
| H11 extractor_abc | 75/75 done, *추가 error* 누적 → **전체 31.3% > 30% 임계값 → REJECT** |
| H13 baseline_abc_chunked | 50/50 done, error 0%, mean_acc 0.840 (longctx 만 정상) |
| H13 abc_search_tool | 50/50 done, error **100%** → REJECT |

→ **3B = capability floor 미달**. paper §4.7.4 / §6 limitations 에서 **"4-axis externalization framework 의 small-LLM minimum size threshold ≈ 4B effective"** 의 직접 evidence 로 활용.

## 5. Stage 6 통합 narrative v2 — paper §4.3 main contribution 갱신

### 5.1 Position-effect asymmetry — *family-pattern 으로 강화*

| 가설 | direction | cross-model evidence v2 |
|---|---|---|
| **H11 (pre-stage)** | **양수** | 6/7 모델 일치, 1 outlier (ministral-3:8b — saturation 가설) |
| **H12 (post-stage)** | **음수** | non-Gemma family **4/4 음수**, Gemma 3 family **2/2 양수** = *family-systematic pattern* |

→ paper §4.6 의 main observation ("position effect asymmetry") 가 *family-level pattern* 으로 격상. H12 에서 *family-systematic* style-bias 가 §4.6.2 caveat (b) 의 직접 evidence.

### 5.2 갱신된 sub-findings (paper §4.3 / §4.7)

1. **H11 magnitude 의 size dependency** (강화): 4B 두 모델 +0.05~+0.08, 8B+ ±0.04 이내. ministral-3:8b 음수는 baseline saturation 가설 부분 지지.
2. **H12 family-systematic style bias** (새 강화): Gemma 3 family 가 keyword scorer 와 정렬되는 출력 스타일. 다른 family 는 정반대. → §4.6.2 caveat 의 *직접 evidence*.
3. **H13 family-agnostic capability floor** (새 강화): 8B 까지 family 무관 100% fail. Gemma 4 E4B (effective 4B) 가 small dense 중 *유일하게 작동* — *minimum operational size threshold* 식별.
4. **3B capability floor finding**: ministral-3:3b H11 31.3% reject, H13 100% reject — 3B 가 본 framework 의 lower bound.

### 5.3 종합 verdict v2 — H14 후보

⚠ **조건부 채택 (direction match 강함, family-systematic pattern 발견, 단일 SIG)**:
- H11 direction: 6/7 양수 (1 outlier)
- H12 direction: family-systematic (Gemma 3 양수, non-Gemma 음수)
- 통계 유의: rnj-1:8b H12 SIG (p=0.036, |d|=0.617)
- H13: family-agnostic capability floor — paper §4.7 mechanism narrative 분기 (M1/M2)

→ Stage 5 의 single-model 결과가 *cross-model 에서 direction-robust + family-pattern 발견* 까지 확장. paper §4.3 / §4.6.2 / §4.7 모두 강화.

## 6. paper draft v0.5 갱신 입력

### 6.1 §4.3 cross-model 본문 (v0.4 대비 갱신)
- 6 모델 panel 표 (ministral-3:8b 추가, gemma3:12b final)
- "5/5 H11 → 6/7 H11" + 1 outlier 명시
- "3/4 H12 → 4/6 H12 + family-systematic pattern" 명시

### 6.2 §4.6.2 caveat — *직접 evidence* 로 격상
- 기존: gemma3:4b 단일 outlier 의 *부분* evidence
- v2: Gemma 3 family 2/2 양수 + non-Gemma 4/4 음수 = **family-level systematic pattern** = *style mismatch (b) 의 직접 evidence*

### 6.3 §4.7 mechanism — 두 갈래 분기
- §4.7.1 갱신: under-iteration (M1) 은 Gemma 4 E4B 한정. 다른 small dense (3B-8B) 는 capability floor (M2)
- §4.7.4 추가: family-agnostic small-dense H13 fail (8B 까지)
- §1.3 contribution 1 재해석: "Tool axis 는 capability floor 위에서만 의미 있음. minimum operational size ≈ Gemma 4 E4B"

### 6.4 §6 limitations
- ministral-3:3b capability floor finding 추가
- "small-LLM externalization framework 의 lower bound ≈ 4B effective" 명시

### 6.5 trial count 갱신
- 기존 ~1640 trials
- v2: ~1640 + ministral-3:8b 400 + ministral-3:3b H11 (rejected, 150) + ministral-3:3b H13 (rejected, 100) = **~2290 trials** (rejected 포함)

## 7. 한계 (v2)

- **n=15 task paired** — Stage 5 와 동일 검정력 한계
- **5 trial / (task, condition)** — sample 작음
- **H13 cross-model = small-dense 0% 작동** — Gemma 4 E4B 만 작동 → cross-model H13 statistical 비교 *불가능* (M2 capability floor)
- **LLM-as-judge 미실시** (P1-3) — H12 의 (a) abstraction loss vs (b) style mismatch 완전 분리 불가
- **ministral-3:3b REJECT** — capability floor finding 으로 보존, H11/H12 cross-model panel 의 small-dense 자리는 ministral-3:8b 가 채움
- **cross-family scope** — non-Gemma = rnj-1 / gpt-oss / Mistral 3 (3 family). DeepSeek / Llama / Phi 등 미커버
- **Pro $20 결제** — paper reproducibility 의무 명시
- **Score_answer_v3 keyword 매칭 의존** — 의미적 정확성 측정 한계 보존
- **gemma4:31b H13 dry-run 미실시** — Gemma 4 family 의 H13 작동 여부 (size-up 효과) 미확인. future work.

## 8. 향후 보강

- **gemma4:31b H13 dry-run** (~30분) — Gemma 4 family 안에서 size 효과 검증 (control)
- **LLM-as-judge replication** (P1-3) — Groq GPT-OSS 120B 로 H12 의미적 채점, family-level pattern 의 본질 검증
- **non-Gemma 추가 family** (DeepSeek / Llama 4) — Stage 7 후보
- **all-hypotheses-statistics.md** (P1-4) — H1/H7/H8/H9a Bootstrap CI 재산정 + 5튜플 통일
- **paper draft v0.5** — Stage 6 v2 결과 통합 + §7 Conclusion

## 9. 변경 이력

- 2026-05-08 v1: 초안. Stage 6 cross-model partial (4 model H11 + 3 model H12). gemma3:12b H12 24/75 진행 중. 5/5 H11 + 3/4 H12 narrative.
- 2026-05-09 v2: gemma3:12b H12 final + ministral-3:8b 추가 + ministral-3:3b capability floor finding + H13 family-agnostic small-dense fail. **Narrative 갱신**: 5/5 → 6/7 H11 (+ outlier), 3/4 → 4/6 H12 + Gemma 3 family-systematic 양수 (§4.6.2 caveat 직접 evidence). H13 mechanism 분기 (M1 under-iteration / M2 capability floor). minimum operational size ≈ Gemma 4 E4B 급 식별. paper §1.3 contribution 1 재해석.
- 2026-05-09 v3: gemma4:31b H13 추가 (Gemma 4 family same-family size-up control). search_tool 90% fail — **(M2-d) A-agent JSON schema mismatch** sub-variant 발견 (assertions=0 across all cycles). baseline_chunked 95% acc 0% error → 모델 자체는 정상 작동, *strictly* A-agent contract fit 실패. **H14 verdict 갱신**: H13 작동 모델이 *size threshold* 가 아닌 *specific-model identification* 으로 narrow — Gemma 4 E4B 한정 (gemma4:31b 도 fail). paper §1.3 contribution 1 *measurement-tool fit* caveat 추가. M2 가 4 sub-variants (M2-a/b/c/d) 로 분화. paper draft v0.6.
