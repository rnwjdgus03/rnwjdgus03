# Hi there, I'm rnwjdgus03 👋

AI와 NLP를 공부하며, LLM이 생성한 답변보다 **근거를 추적할 수 있는 AI 시스템**에 관심이 있습니다.

저는 뉴스 문장, 검색 후보, 공식 데이터, 평가 지표가 서로 어떻게 연결되는지 확인할 수 있는 **evidence-grounded AI pipeline**을 만드는 것을 중요하게 생각합니다.

---

## Now

- AI / NLP 학습 내용을 [TIL](https://github.com/rnwjdgus03/TIL)에 정리하고 있습니다.
- 공식 통계 기반 뉴스 수치 검증 PoC를 팀 프로젝트로 개발했습니다.
- HCX-007, BGE-M3, reranker, PostgreSQL, KOSIS Open API를 활용한 검증 파이프라인을 실험했습니다.
- 검색 모델이 만든 후보를 그대로 믿지 않고, 정형 메타데이터와 공식 API로 검증하는 구조에 관심이 있습니다.

---

## Featured Project

### KOSIS 뉴스 수치 검증 PoC

Official Statistics-grounded News Claim Verification PoC  
Team Project | NLP · Retrieval · Backend · Evaluation

뉴스 기사 URL에서 수치 주장을 추출하고, 해당 주장이 KOSIS 공식 통계로 검증 가능한지 판단하는 PoC를 개발했습니다.

이 프로젝트의 핵심은 뉴스 속 숫자를 바로 맞히는 것이 아니라, 그 숫자가 어떤 공식 통계표, 항목, 분류, 기간, 단위에 해당하는지 **정확한 통계 좌표**를 찾는 것이었습니다.

- HCX-007 Structured Outputs를 사용해 뉴스 문장에서 수치, 단위, 기간, 비교 기준, 대상을 measurement 단위로 구조화했습니다.
- Lexical search, BGE-M3, reranker를 활용해 뉴스 표현과 KOSIS 통계표명 사이의 표현 차이를 보완했습니다.
- PostgreSQL에 KOSIS 메타데이터를 정규화해 저장하고, 실제 존재하는 ITEM / OBJ / period 조합만 통과시키는 exact retrieval 구조를 설계했습니다.
- PostgreSQL preflight를 통과한 좌표에 대해서만 KOSIS Open API를 호출해 공식값을 조회했습니다.
- 불확실한 경우 억지로 MATCH / MISMATCH를 내지 않고 UNRESOLVED로 보류하는 보수적 정책을 적용했습니다.

#### Key Result

- 개발 좌표 gold 300건 기준 ITEM Top-5: 77.3%
- 개발 좌표 gold 300건 기준 전체 좌표 Top-5: 74.3%
- 실제 기사 개발 E2E: READY 60건 중 MATCH 6건, UNRESOLVED 54건

> Top-k는 fact-check accuracy가 아니라, 정답 좌표가 상위 k개 후보 안에 포함된 비율입니다.

---

## System Architecture

```text
Article URL
→ Claim / Measurement Extraction with HCX-007
→ READY / ENRICH / REJECT Gate
→ Lexical + BGE-M3 + Reranker Table Retrieval
→ PostgreSQL Exact Coordinate Retrieval
→ KOSIS Open API Official Value Lookup
→ MATCH / MISMATCH_REVIEW_REQUIRED / UNRESOLVED
```

---

## Troubleshooting

### 1. LLM에게 좌표 선택을 맡기면 위험했다

초기에는 LLM이 뉴스 문장을 보고 KOSIS 표와 좌표를 직접 선택하도록 실험했습니다.

하지만 LLM은 실제 후보 목록에 없는 `org_id`, `tbl_id`, `ITEM`, `OBJ` 코드를 그럴듯하게 생성할 수 있었습니다.

그래서 LLM의 역할을 최종 좌표 승인자가 아니라 **구조화 추출기**로 제한했습니다.

- HCX-007: 뉴스 문장을 measurement로 구조화
- BGE-M3: 관련 통계표 후보 검색
- PostgreSQL: 실제 존재하는 좌표 승인
- KOSIS API: 공식값 조회

---

### 2. 초기에는 Lexical이 BGE-M3보다 높았다

초기 24건 gold 기준 표 검색 실험에서는 lexical이 BGE-M3 hybrid보다 높은 결과를 보였습니다.

| 평가셋 | 방식 | Recall@1 | Recall@2 | Recall@3 | Recall@5 |
|---|---|---:|---:|---:|---:|
| READY 39건 중 표 gold 24건 | Lexical | 54.2% | 62.5% | 62.5% | 62.5% |
| READY 39건 중 표 gold 24건 | BGE-M3 hybrid | 50.0% | 58.3% | 58.3% | 58.3% |

이 결과를 통해 BGE-M3를 무조건적인 대체재로 보지 않았습니다.

대신 lexical은 정확한 용어 매칭에 강한 후보 생성기로 유지하고, BGE-M3와 reranker는 표현 차이를 보완하는 역할로 사용했습니다.

이후 자동 gold 200건 개발셋에서는 BGE rerank를 결합하면서 후보 회수율이 개선되었습니다.

| 평가셋 | 방식 | Recall@1 | Recall@5 | Recall@10 | Recall@20 |
|---|---|---:|---:|---:|---:|
| 자동 gold 200건 | Lexical-only | 36.5% | 61.0% | - | 84.5% |
| 자동 gold 200건 | Lexical + BGE-M3 + reranker | 36.5% | 67.5% | 80.5% | 91.0% |

BGE-M3의 효과는 1위 정답률을 바꾸는 것보다, 정답 통계표를 더 넓은 후보군 안에 회수하는 데서 나타났습니다.

---

### 3. ChromaDB + SQLite에서 PostgreSQL로 전환했다

처음에는 KOSIS 좌표를 문서화해 ChromaDB에 저장하고, SQLite / CSV로 메타데이터를 관리하는 방식을 실험했습니다.

하지만 공식 통계 검증에서는 “비슷한 좌표”가 아니라 “실제로 존재하는 정확한 좌표”가 필요했습니다.

예를 들어 같은 통계표 안에서도 다음 중 하나만 달라져도 공식값 비교가 성립하지 않습니다.

- ITEM: 월평균임금 vs 증감률
- OBJ: 정규직 vs 비정규직
- period: 월간 vs 연간
- unit: 명, 천명, %, %p

그래서 최종적으로 PostgreSQL에 KOSIS 메타데이터를 정규화해 저장했습니다.

| 개념 테이블 | 역할 |
|---|---|
| `kosis_tables` | 기관 코드, 통계표 ID, 통계표명, 조사명, 주기 |
| `kosis_items` | ITEM 코드, 항목명, 단위, 값 유형 |
| `kosis_objects` | OBJ 축 레벨, 분류 코드, 분류명, 상위 분류 |
| `available_periods` | 조회 가능한 기간과 주기 |
| `coordinate_view` | ITEM / OBJ / period 조합의 후보 좌표 |

> ChromaDB는 비슷한 좌표를 찾았고, PostgreSQL은 존재하는 좌표만 남겼습니다.

---

### 4. Reranker는 후보를 새로 찾지 못한다

Reranker는 후보를 새로 발굴하는 모델이 아니라, 이미 들어온 후보의 순서를 다시 정렬하는 모델입니다.

초기 BGE-only run에서는 정답표가 후보 풀에 잘 들어오지 않아 Recall@20이 낮았습니다.

이 상태에서는 reranker가 아무리 좋아도 정답을 위로 올릴 수 없습니다.

그래서 구조를 다음처럼 바꿨습니다.

```text
Lexical Top-50 primary pool
→ BGE-M3 Top-20 signal
→ BGE reranker
→ Final table candidates
```

즉, lexical로 후보 풀을 넓히고, BGE-M3와 reranker로 재정렬하는 방식으로 전환했습니다.

---

### 5. UNRESOLVED는 실패가 아니라 안전장치다

실제 기사 개발 E2E에서는 READY 60건 중 MATCH 6건, UNRESOLVED 54건이 나왔습니다.

이 PoC의 목표는 모든 뉴스를 강제로 판정하는 것이 아니라, 공식 통계로 검증 가능한 범위만 안전하게 자동화하는 것이었습니다.

좌표, 기간, 단위, 공식값 중 하나라도 불확실하면 자동 오판보다 보류가 안전하다고 판단했습니다.

---

## What I Learned

- 뉴스 수치 검증의 핵심은 숫자 추출보다 공식 통계 좌표 정합성에 있다.
- LLM은 최종 판정자보다 구조화 추출기로 사용할 때 더 안전했다.
- Dense retrieval은 후보 확장에 유용하지만, 공식 좌표 존재성 검증은 관계형 DB가 더 적합했다.
- Reranker 성능보다 reranker에 들어가기 전 후보 풀의 recall이 먼저 중요했다.
- 자동화 서비스에서는 “모르는 것을 모른다고 말하는 정책”이 정확도만큼 중요하다.

---

## Tech Stack

### AI / NLP

HCX-007 · Structured Outputs · BGE-M3 · BGE Reranker · BM25 · Dense Retrieval · RAG · Reranking

### Backend / Data

Python · FastAPI · PostgreSQL · SQLite · KOSIS Open API · JSON Schema · CSV Pipeline

### Evaluation

Top-k Recall · Coordinate Accuracy · READY / ENRICH / REJECT Gate · MATCH / UNRESOLVED Policy · Regression Tests

---

## Repositories

- [TIL](https://github.com/rnwjdgus03/TIL) — AI 공부 기록
- [NLP_05-Team-Project-3](https://github.com/rnwjdgus03/NLP_05-Team-Project-3) — KOSIS 뉴스 수치 검증 PoC
