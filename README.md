# Hi there, I'm Jeonghyeon Gu 👋

AI·NLP를 공부하고 프로젝트로 구현하며, **근거를 추적할 수 있는 AI 서비스**를 만들고 있습니다.

자연어 처리, 검색, 데이터베이스와 공식 API를 연결해 모델의 판단 과정을 관찰하고 평가할 수 있는 시스템에 관심이 있습니다. 좋은 결과뿐 아니라 실패 원인과 불확실성까지 설명할 수 있는 개발을 지향합니다.

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/rnwjdgus03)
[![Email](https://img.shields.io/badge/Email-03C75A?style=flat-square&logo=naver&logoColor=white)](mailto:rnwjdgus03@naver.com)
![Profile Views](https://komarev.com/ghpvc/?username=rnwjdgus03&color=blueviolet&style=flat-square)

---

## Now

- [AI·NLP Learning Notes](https://github.com/rnwjdgus03/TIL)에 딥러닝 기초부터 Transformer, RAG, CLIP까지 학습 내용을 정리하고 있습니다.
- 공식 통계와 뉴스 수치 주장을 연결하는 evidence-grounded AI pipeline을 개발했습니다.
- 검색 모델의 후보를 정형 메타데이터와 공식 API로 검증하는 구조를 탐구하고 있습니다.
- Retrieval 성능뿐 아니라 좌표 정합성, blind evaluation, 안전한 실패 정책까지 함께 평가하고 있습니다.

---

## Featured AI Project

### KOSIS 뉴스 수치 검증 PoC

Official Statistics-grounded News Claim Verification PoC  
Team Project | NLP · Retrieval · Backend · Evaluation

[발표자료 PPTX](./assets/kosis-news-verification-final.pptx) · [Project Repository](https://github.com/rnwjdgus03/NLP_05-Team-Project-3)

뉴스 기사 URL에서 수치 주장을 추출하고, 해당 주장이 KOSIS 공식 통계로 검증 가능한지 판단하는 PoC를 개발했습니다.

이 프로젝트의 핵심은 뉴스 속 숫자를 바로 맞히는 것이 아니라, 그 숫자가 어떤 공식 통계표, 항목, 분류, 기간, 단위에 해당하는지 **정확한 통계 좌표**를 찾는 것이었습니다.

- HCX-007 Structured Outputs를 사용해 뉴스 문장에서 수치, 단위, 기간, 비교 기준, 대상을 measurement 단위로 구조화했습니다.
- Lexical search, BGE-M3, reranker를 활용해 뉴스 표현과 KOSIS 통계표명 사이의 표현 차이를 보완했습니다.
- PostgreSQL에 KOSIS 메타데이터를 정규화해 저장하고, 실제 존재하는 ITEM / OBJ / period 조합만 통과시키는 exact retrieval 구조를 설계했습니다.
- PostgreSQL preflight를 통과한 좌표에 대해서만 KOSIS Open API를 호출해 공식값을 조회했습니다.
- 불확실한 경우 억지로 MATCH / MISMATCH를 내지 않고 UNRESOLVED로 보류하는 보수적 정책을 적용했습니다.
- 표 ID가 연도별로 바뀌는 문제를 처리하기 위해 `table_family`, `valid_from`, `valid_to` 개념으로 같은 통계 계열을 연결했습니다.

#### Key Result

- 개발 좌표 gold 300건 기준 ITEM Top-5: 77.3%
- 개발 좌표 gold 300건 기준 전체 좌표 Top-5: 74.3%
- 실제 기사 개발 E2E: READY 60건 중 MATCH 6건, UNRESOLVED 54건
- URL50 실험: HCX 구조화 측정값 263건 중 KOSIS-ready 96건, 공식 근거 자동 확정 11건
- 처리시간 최적화: 전체 파이프라인 193.25초에서 120.70초로 단축

> Top-k는 fact-check accuracy가 아니라, 정답 좌표가 상위 k개 후보 안에 포함된 비율입니다.

---

## System Architecture

```text
Article URL
→ Claim / Measurement Extraction with HCX-007
→ READY / ENRICH / REJECT Gate
→ Stage A: Lexical + BGE-M3 + Reranker Table Retrieval
→ Stage B: PostgreSQL Exact Coordinate Retrieval
→ Stage C: Coordinate Reranking
→ KOSIS Open API Official Value Lookup with Preflight
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
| `axes` / `axis_values` | OBJ 축 레벨, 분류 코드, 분류명, 상위 분류 |
| `periodicities` | 조회 가능한 주기와 기간 조건 |
| `coordinate_candidates` | ITEM / OBJ / period 조합의 후보 좌표 |

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

### 5. 같은 통계도 연도에 따라 표 ID와 표명이 바뀌었다

KOSIS에서는 같은 지표라도 연도에 따라 표명과 표 ID가 바뀌는 경우가 있었습니다.

예를 들어 로봇 수입 관련 지표는 시기별로 다른 표명으로 제공되었습니다.

```text
2007–2012: 로봇 단품 및 부품 수입현황
2013–2018: 로봇 단품 및 부품 국가별 수입현황
2019–2024: 로봇산업 수입현황
```

하나의 표만 찾으면 시계열 전체를 덮을 수 없어서, 같은 계열의 표를 `table_family`로 묶고 각 표의 유효 기간을 함께 확인했습니다.

이 경험을 통해 표 검색은 단일 `tbl_id` 검색보다 “통계 계열 검색”에 가까워야 한다는 점을 배웠습니다.

---

### 6. 값이 다르다고 바로 불일치로 처리할 수 없었다

공식값과 기사 수치가 다를 때도 바로 오보로 판단하지 않았습니다.

반올림 표현, 0에 가까운 값, 단위 불명확, 잠정치와 개정치 문제 때문에 경계 구간을 나누었습니다.

| 상대오차 | 판정 |
|---:|---|
| 1.5% 이하 | MATCH |
| 1.5% 초과 4% 이하 | REVIEW_REQUIRED |
| 4% 초과 | MISMATCH_REVIEW_REQUIRED |

단위 환산 근거가 부족하거나 공식 좌표가 불확실하면 값을 비교하지 않고 UNRESOLVED로 처리했습니다.

---

### 7. UNRESOLVED는 실패가 아니라 안전장치다

실제 기사 개발 E2E에서는 READY 60건 중 MATCH 6건, UNRESOLVED 54건이 나왔습니다.

이 PoC의 목표는 모든 뉴스를 강제로 판정하는 것이 아니라, 공식 통계로 검증 가능한 범위만 안전하게 자동화하는 것이었습니다.

좌표, 기간, 단위, 공식값 중 하나라도 불확실하면 자동 오판보다 보류가 안전하다고 판단했습니다.

URL50 실험에서도 HCX가 263개의 측정값을 구조화했지만, KOSIS-ready gate를 통과한 것은 96건이었고 공식 근거까지 자동 확정한 것은 11건이었습니다.

이 감소는 단순 손실이 아니라, KOSIS 범위 밖 주장과 좌표 불확실성을 걸러내는 과정이었습니다.

---

### 8. 개발셋 성능을 일반화 성능으로 착각할 수 있었다

개발셋 재대입에서는 ITEM Top-5 90.3%, 좌표 Top-5 90.2%까지 올라갔지만, 신규 blind100에서는 ITEM Top-5 58.0%, 좌표 Top-5 54.0%로 낮아졌습니다.

원인은 개발셋에 반복 등장한 통계표와 키워드를 지나치게 잘 기억한 것이었습니다.

이후에는 기사 단위 분리만으로 충분하지 않다고 보고, `tbl_id`와 `table_family` 기준으로 개발·검증·테스트를 분리해야 한다는 결론을 냈습니다.

---

## What I Learned

- 뉴스 수치 검증의 핵심은 숫자 추출보다 공식 통계 좌표 정합성에 있다.
- LLM은 최종 판정자보다 구조화 추출기로 사용할 때 더 안전했다.
- Dense retrieval은 후보 확장에 유용하지만, 공식 좌표 존재성 검증은 관계형 DB가 더 적합했다.
- Reranker 성능보다 reranker에 들어가기 전 후보 풀의 recall이 먼저 중요했다.
- 자동화 서비스에서는 “모르는 것을 모른다고 말하는 정책”이 정확도만큼 중요하다.
- 개발셋 재평가 결과와 신규 blind 결과를 분리해서 보고해야 모델의 실제 일반화를 판단할 수 있다.
- 공식 통계 검증에서는 표 하나가 아니라 같은 계열의 표와 유효 기간까지 함께 봐야 한다.

---

## Learning & Research Interests

- Text preprocessing, tokenization, PyTorch model training
- RNN, Seq2Seq, Attention, Transformer, BERT, GPT
- Retrieval-augmented generation and evidence-grounded AI
- Lexical, dense, hybrid retrieval and reranking
- LLM evaluation, hallucination control and human-reviewable workflows
- CLIP, contrastive learning and multimodal representation

학습 노트는 [TIL Repository](https://github.com/rnwjdgus03/TIL)에서 주제별로 정리하고 있습니다.

---

## Tech Stack

### AI / NLP

![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=flat-square&logo=pytorch&logoColor=white)
![Hugging Face](https://img.shields.io/badge/Hugging_Face-FFD21E?style=flat-square&logo=huggingface&logoColor=black)
![Transformers](https://img.shields.io/badge/Transformers-FFCC4D?style=flat-square&logo=huggingface&logoColor=black)
![BERT](https://img.shields.io/badge/BERT-4B32C3?style=flat-square&logo=google&logoColor=white)
![GPT](https://img.shields.io/badge/GPT-412991?style=flat-square&logo=openai&logoColor=white)
![CLIP](https://img.shields.io/badge/CLIP-000000?style=flat-square&logo=openai&logoColor=white)
![HCX-007](https://img.shields.io/badge/HCX--007-03C75A?style=flat-square&logo=naver&logoColor=white)

### Retrieval / LLM Application

![RAG](https://img.shields.io/badge/RAG-6C63FF?style=flat-square&logo=semanticweb&logoColor=white)
![BGE-M3](https://img.shields.io/badge/BGE--M3-0052CC?style=flat-square&logo=buffer&logoColor=white)
![BM25](https://img.shields.io/badge/BM25-005571?style=flat-square&logo=elasticsearch&logoColor=white)
![Dense Retrieval](https://img.shields.io/badge/Dense_Retrieval-7B61FF?style=flat-square&logo=databricks&logoColor=white)
![Hybrid Retrieval](https://img.shields.io/badge/Hybrid_Retrieval-008080?style=flat-square&logo=searchengineland&logoColor=white)
![Reranking](https://img.shields.io/badge/Reranking-FF6F00?style=flat-square&logo=weightsandbiases&logoColor=white)

### Backend / Data

![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-003B57?style=flat-square&logo=sqlite&logoColor=white)
![ChromaDB](https://img.shields.io/badge/ChromaDB-FF6446?style=flat-square&logo=googlechrome&logoColor=white)
![REST API](https://img.shields.io/badge/REST_API-0096D6?style=flat-square&logo=fastapi&logoColor=white)
![JSON](https://img.shields.io/badge/JSON-000000?style=flat-square&logo=json&logoColor=white)
![Git](https://img.shields.io/badge/Git-F05032?style=flat-square&logo=git&logoColor=white)
![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)

### Evaluation

![Top-k Recall](https://img.shields.io/badge/Top--k_Recall-2E8B57?style=flat-square)
![Coordinate Accuracy](https://img.shields.io/badge/Coordinate_Accuracy-4682B4?style=flat-square)
![Blind Evaluation](https://img.shields.io/badge/Blind_Evaluation-8A2BE2?style=flat-square)
![Regression Tests](https://img.shields.io/badge/Regression_Tests-228B22?style=flat-square)

---

## Repositories

- [TIL](https://github.com/rnwjdgus03/TIL) — AI·NLP 개념을 학습 흐름에 따라 정리한 노트
- [NLP_05-Team-Project-3](https://github.com/rnwjdgus03/NLP_05-Team-Project-3) — KOSIS 뉴스 수치 검증 PoC
