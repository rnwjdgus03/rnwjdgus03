# KOSIS 뉴스 수치 검증 PoC

> 모든 뉴스를 자동 판정하는 서비스가 아니라, 공식 통계로 검증 가능한 범위를 안전하게 자동화한 PoC입니다.

[Repository](https://github.com/rnwjdgus03/NLP_05-Team-Project-3) · [Presentation](../assets/kosis-news-verification-final.pptx) · [Profile](../README.md)

## Problem

뉴스에 등장하는 숫자를 검증하려면 비슷한 통계표를 찾는 것만으로는 부족합니다. 공식값 조회에는 `tbl_id`, `ITEM`, `OBJ`, 기간, 주기와 단위가 모두 맞는 정확한 통계 좌표가 필요합니다.

## Architecture

```text
Article URL
→ HCX-007 Claim / Measurement Extraction
→ READY / ENRICH / REJECT Gate
→ Stage A: Lexical + BGE-M3 + Reranker Table Retrieval
→ Stage B: PostgreSQL Exact Coordinate Retrieval
→ Stage C: Coordinate Reranking
→ KOSIS Open API Preflight & Value Lookup
→ MATCH / MISMATCH_REVIEW_REQUIRED / UNRESOLVED
```

핵심 설계는 **BGE 의미 검색과 PostgreSQL 정확 좌표 조회를 분리한 것**입니다. 검색 모델은 후보를 넓게 찾고, 관계형 데이터베이스는 실제 존재하는 좌표만 승인합니다.

## What I Built

- HCX-007 Structured Outputs로 기사 문장에서 수치, 단위, 기간, 비교 기준과 대상을 measurement 단위로 구조화했습니다.
- Lexical Search, BGE-M3와 Reranker로 뉴스 표현과 KOSIS 통계표명 사이의 표현 차이를 보완했습니다.
- PostgreSQL에 KOSIS 메타데이터를 정규화해 실제 존재하는 ITEM / OBJ / period 조합만 조회했습니다.
- PostgreSQL preflight를 통과한 좌표만 KOSIS Open API에 요청했습니다.
- 근거가 부족한 경우 강제로 판정하지 않고 `UNRESOLVED`로 보류했습니다.
- 연도별로 달라지는 표 ID와 표명을 `table_family`, `valid_from`, `valid_to`로 연결했습니다.

## Results

| Evaluation | Result |
|---|---:|
| 개발 좌표 gold 300건 ITEM Top-5 | 77.3% |
| 개발 좌표 gold 300건 전체 좌표 Top-5 | 74.3% |
| 실제 기사 개발 E2E | READY 60건 중 MATCH 6건, UNRESOLVED 54건 |
| URL50 | 측정값 263건 → KOSIS-ready 96건 → 공식 근거 자동 확정 11건 |
| 전체 처리시간 | 193.25초 → 120.70초 |

> Top-k는 팩트체크 정확도가 아니라, 정답 좌표가 상위 k개 후보 안에 포함된 비율입니다.

## Troubleshooting

### 1. LLM에게 좌표 선택을 맡기면 위험했다

초기에는 LLM이 뉴스 문장을 보고 KOSIS 표와 좌표를 직접 선택하도록 실험했습니다. 그러나 실제 후보에 없는 `org_id`, `tbl_id`, `ITEM`, `OBJ` 코드를 그럴듯하게 생성할 수 있었습니다.

LLM은 최종 좌표 승인자가 아니라 구조화 추출기로 제한했습니다.

```text
HCX-007: 문장 구조화
BGE-M3: 관련 통계표 후보 검색
PostgreSQL: 실제 존재하는 좌표 승인
KOSIS API: 공식값 조회
```

### 2. 초기에는 Lexical이 BGE-M3보다 높았다

초기 24건 gold 평가에서는 Lexical Recall@5가 62.5%, BGE-M3 Hybrid Recall@5가 58.3%였습니다.

| Evaluation | Method | Recall@1 | Recall@5 | Recall@10 | Recall@20 |
|---|---|---:|---:|---:|---:|
| 자동 gold 200건 | Lexical-only | 36.5% | 61.0% | - | 84.5% |
| 자동 gold 200건 | Lexical + BGE-M3 + Reranker | 36.5% | 67.5% | 80.5% | 91.0% |

BGE-M3의 효과는 1위 정답률보다 넓은 후보군 안에 정답표를 회수하는 데서 나타났습니다. Lexical은 정확한 용어 후보 생성기로 유지하고 BGE-M3와 Reranker는 표현 차이를 보완하는 역할로 사용했습니다.

### 3. ChromaDB + SQLite에서 PostgreSQL로 전환했다

ChromaDB는 의미가 비슷하지만 실제로 존재하지 않는 ITEM / OBJ 조합도 후보로 만들 수 있었습니다. 공식 통계 검증에는 유사한 좌표가 아니라 정확히 존재하는 좌표가 필요했습니다.

| Table | Responsibility |
|---|---|
| `kosis_tables` | 기관 코드, 통계표 ID, 표명, 조사명, 주기 |
| `kosis_items` | ITEM 코드, 항목명, 단위, 값 유형 |
| `axes` / `axis_values` | OBJ 축, 분류 코드, 분류명, 상위 분류 |
| `periodicities` | 조회 가능한 주기와 기간 조건 |
| `coordinate_candidates` | ITEM / OBJ / period 후보 조합 |

PostgreSQL 전환으로 좌표 존재 여부를 API 호출 전에 검사하고 잘못된 요청과 불필요한 호출을 줄였습니다.

### 4. Reranker 이전의 후보 회수가 중요했다

Reranker는 이미 들어온 후보의 순서만 바꾸며 누락된 정답표를 새로 만들 수 없습니다.

```text
Lexical Top-50 Primary Pool
→ BGE-M3 Top-20 Signal
→ BGE Reranker
→ Final Table Candidates
```

먼저 candidate recall을 확보한 다음 의미 기반 점수로 재정렬하도록 변경했습니다.

### 5. 같은 통계도 연도에 따라 표가 달라졌다

동일한 통계 계열도 시기에 따라 표명과 표 ID가 변경됐습니다.

```text
2007–2012: 로봇 단품 및 부품 수입현황
2013–2018: 로봇 단품 및 부품 국가별 수입현황
2019–2024: 로봇산업 수입현황
```

하나의 `tbl_id`만 검색하지 않고 관련 표를 `table_family`로 묶어 유효 기간을 함께 확인했습니다.

### 6. 값이 다르다고 바로 불일치로 판단할 수 없었다

반올림, 0에 가까운 값, 단위 불명확, 잠정치와 개정치 문제를 고려해 판정 구간을 나눴습니다.

| Relative Error | Verdict |
|---:|---|
| 1.5% 이하 | MATCH |
| 1.5% 초과 4% 이하 | REVIEW_REQUIRED |
| 4% 초과 | MISMATCH_REVIEW_REQUIRED |

단위 환산 근거나 공식 좌표가 불확실하면 값을 비교하지 않고 `UNRESOLVED`로 처리했습니다.

### 7. UNRESOLVED는 실패가 아니라 안전장치다

실제 기사 개발 E2E에서는 READY 60건 중 MATCH 6건, UNRESOLVED 54건이었습니다. 좌표, 기간, 단위와 공식값 중 하나라도 불확실하면 자동 오판보다 보류가 안전하다고 판단했습니다.

### 8. 개발셋 성능과 일반화 성능은 달랐다

개발셋 재대입에서는 ITEM Top-5 90.3%, 좌표 Top-5 90.2%였지만 blind100에서는 각각 58.0%, 54.0%였습니다.

반복된 표와 키워드에 대한 과적합을 확인했으며, 이후에는 기사뿐 아니라 `tbl_id`와 `table_family`를 기준으로 개발·검증·테스트를 분리해야 한다는 결론을 얻었습니다.

## What I Learned

- 뉴스 수치 검증의 핵심은 숫자 추출보다 공식 통계 좌표 정합성에 있습니다.
- LLM은 최종 판정자보다 구조화 추출기로 사용할 때 더 안전했습니다.
- Dense Retrieval은 후보 확장에 유용하지만 좌표 존재성 검증에는 관계형 DB가 더 적합했습니다.
- Reranker 성능보다 reranker에 들어가기 전 후보 recall이 먼저 중요했습니다.
- 자동화 서비스에서는 모르는 것을 모른다고 말하는 정책이 정확도만큼 중요합니다.
- 개발셋과 신규 blind 결과를 분리해야 실제 일반화 성능을 판단할 수 있습니다.
