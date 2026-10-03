# 구정현 | AI · NLP Developer

자연어를 구조화하고, 검색 결과를 공식 데이터와 연결해 **근거를 추적할 수 있는 AI 시스템**을 만드는 데 관심이 있습니다.

모델의 답변만 보여주는 것보다 어떤 문서와 데이터에서 답을 찾았는지, 불확실한 경우 왜 판단을 보류했는지 설명할 수 있는 시스템을 지향합니다.

---

## Profile

- **관심 분야**: NLP, Retrieval, RAG, LLM Application, Evidence-grounded AI
- **개발 방향**: 비정형 텍스트 구조화 → 후보 검색 → 정형 데이터 검증 → 근거 기반 응답
- **중요하게 보는 것**: 재현 가능한 평가, 데이터 정합성, 안전한 실패, 추적 가능한 근거

---

## Core Competencies

### AI / Deep Learning Fundamentals

- 인공지능과 딥러닝의 발전 과정, 지도학습과 범용 모델의 차이를 학습했습니다.
- PyTorch의 Tensor 연산, Dataset / DataLoader, 모델 구성, 자동 미분과 GPU 연산 흐름을 이해하고 실습했습니다.
- Forward Pass → Loss 계산 → Backward Pass → Weight Update로 이어지는 학습 과정과 Gradient Descent를 정리했습니다.
- RNN의 순차 처리 방식과 기울기 소실, 장기 의존성, 병렬화 한계를 학습했습니다.

### NLP & Language Models

- 문장·어절·형태소·subword 단위 토큰화와 정규화, padding 등 텍스트 전처리 과정을 학습했습니다.
- Hugging Face 데이터셋을 활용해 Text Classification을 비롯한 NLP Task의 데이터 구조를 실습했습니다.
- Seq2Seq와 Attention이 RNN의 정보 병목을 보완하는 과정을 학습했습니다.
- Transformer의 구조, Dot-Product Attention과 Multi-Head Attention을 학습했습니다.
- BERT의 양방향 문맥 표현과 Masked Language Modeling을 학습했습니다.
- GPT 계열의 Decoder-only 구조와 자기회귀 생성 방식을 학습했습니다.

### Retrieval & Grounded AI

- LLM의 Knowledge Cutoff, 도메인 지식 부족, 계산·추론 한계와 hallucination 문제를 학습했습니다.
- Parametric Knowledge에 외부 문서 검색을 결합하는 RAG의 배경과 동작 원리를 이해하고 프로젝트에 적용했습니다.
- Lexical Search, Dense Retrieval, Hybrid Retrieval, Reranking의 역할과 실패 지점을 평가했습니다.
- 검색 근거가 없을 때 답을 생성하지 않는 보수적인 응답·판정 정책에 관심이 있습니다.

### Multimodal

- CLIP의 Text Encoder / Image Encoder 구조와 contrastive learning을 학습했습니다.
- 이미지와 텍스트를 같은 임베딩 공간에 정렬하는 방식과 zero-shot classification의 원리를 이해했습니다.

---

## Projects

프로젝트마다 **문제 → 담당 역할 → 핵심 구현 → 성과 → 트러블슈팅** 순서로 기록합니다.

| No. | Project | Area | Status | Links |
|---:|---|---|---|---|
| 01 | KOSIS 뉴스 수치 검증 PoC | NLP · Retrieval · Backend · Evaluation | Completed | [Repository](https://github.com/rnwjdgus03/NLP_05-Team-Project-3) · [Presentation](./assets/kosis-news-verification-final.pptx) |

### 01. KOSIS 뉴스 수치 검증 PoC

> 뉴스 기사 속 수치 주장을 KOSIS 공식 통계 좌표와 연결해 검증 가능한 범위만 안전하게 자동화한 PoC

#### Problem

뉴스에 등장하는 숫자를 검증하려면 단순히 비슷한 통계표를 찾는 것만으로는 부족했습니다. 공식값을 조회하려면 `tbl_id`, `ITEM`, `OBJ`, 기간, 주기, 단위가 모두 맞는 **정확한 통계 좌표**가 필요했습니다.

#### My Work

- HCX-007 Structured Outputs로 기사 문장에서 수치, 단위, 기간, 비교 기준과 대상을 measurement 단위로 구조화했습니다.
- Lexical Search + BGE-M3 + Reranker로 관련 통계표 후보를 검색했습니다.
- PostgreSQL에 KOSIS 메타데이터를 정규화해 실제 존재하는 ITEM / OBJ / period 조합만 조회하도록 구성했습니다.
- PostgreSQL preflight를 통과한 좌표만 KOSIS Open API에 요청했습니다.
- 근거가 부족한 경우 강제로 판정하지 않고 `UNRESOLVED`로 보류하는 정책을 적용했습니다.
- 연도별로 달라지는 표 ID와 표명을 `table_family`, `valid_from`, `valid_to`로 연결했습니다.

#### Architecture

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

#### Results

| Evaluation | Result |
|---|---:|
| 개발 좌표 gold 300건 ITEM Top-5 | 77.3% |
| 개발 좌표 gold 300건 전체 좌표 Top-5 | 74.3% |
| 실제 기사 개발 E2E | READY 60건 중 MATCH 6건, UNRESOLVED 54건 |
| URL50 | 측정값 263건 → KOSIS-ready 96건 → 공식 근거 자동 확정 11건 |
| 전체 처리시간 | 193.25초 → 120.70초 |

> Top-k는 팩트체크 정확도가 아니라, 정답 좌표가 상위 k개 후보 안에 포함된 비율입니다.

<details>
<summary><strong>Troubleshooting 1 — Lexical과 BGE-M3의 역할을 다시 정의</strong></summary>

초기 24건 표 검색에서는 Lexical Recall@5가 62.5%, BGE-M3 Hybrid Recall@5가 58.3%였습니다. 작은 초기 평가만 보면 Lexical이 더 높았지만, 자동 gold 200건에서는 BGE Rerank 결합 방식이 Recall@20을 84.5%에서 91.0%로 높였습니다.

| Evaluation | Method | Recall@1 | Recall@5 | Recall@10 | Recall@20 |
|---|---|---:|---:|---:|---:|
| 자동 gold 200건 | Lexical-only | 36.5% | 61.0% | - | 84.5% |
| 자동 gold 200건 | Lexical + BGE-M3 + Reranker | 36.5% | 67.5% | 80.5% | 91.0% |

Lexical은 정확한 용어를 회수하는 후보 생성기로 유지하고, BGE-M3와 Reranker는 표현 차이를 보완하고 넓은 후보군을 재정렬하는 데 사용했습니다.

</details>

<details>
<summary><strong>Troubleshooting 2 — ChromaDB + SQLite에서 PostgreSQL로 전환</strong></summary>

ChromaDB에 좌표를 문서처럼 저장하면 의미가 비슷하지만 실제로 존재하지 않는 ITEM / OBJ 조합도 후보가 될 수 있었습니다. 공식 통계 검증에는 유사한 좌표가 아니라 정확히 존재하는 좌표가 필요했습니다.

PostgreSQL에 다음 구조로 메타데이터를 정규화했습니다.

| Table | Responsibility |
|---|---|
| `kosis_tables` | 기관 코드, 통계표 ID, 표명, 조사명, 주기 |
| `kosis_items` | ITEM 코드, 항목명, 단위, 값 유형 |
| `axes` / `axis_values` | OBJ 축, 분류 코드, 분류명, 상위 분류 |
| `periodicities` | 조회 가능한 주기와 기간 조건 |
| `coordinate_candidates` | ITEM / OBJ / period 후보 조합 |

이 전환으로 좌표 존재 여부를 API 호출 전에 확인하고, 잘못된 요청과 불필요한 호출을 줄일 수 있었습니다.

</details>

<details>
<summary><strong>Troubleshooting 3 — Reranker 이전 후보 회수 문제</strong></summary>

Reranker는 이미 들어온 후보의 순서만 바꿀 수 있고, 누락된 정답표를 새로 만들 수 없습니다. 초기에는 정답표가 후보 풀에 들어오지 않아 재정렬 효과가 제한됐습니다.

```text
Lexical Top-50 Primary Pool
→ BGE-M3 Top-20 Signal
→ BGE Reranker
→ Final Table Candidates
```

먼저 후보 recall을 확보한 뒤 의미 기반 점수로 재정렬하도록 변경했습니다.

</details>

<details>
<summary><strong>Troubleshooting 4 — 표 ID 변경과 시계열 연결</strong></summary>

동일한 통계 계열도 시기에 따라 표명과 표 ID가 달랐습니다. 하나의 `tbl_id`만 검색하면 과거 또는 최신 기간이 누락되므로 관련 표를 `table_family`로 묶고 유효 기간을 함께 검사했습니다.

```text
2007–2012: 로봇 단품 및 부품 수입현황
2013–2018: 로봇 단품 및 부품 국가별 수입현황
2019–2024: 로봇산업 수입현황
```

</details>

<details>
<summary><strong>Troubleshooting 5 — UNRESOLVED와 일반화 성능</strong></summary>

실제 기사 개발 E2E에서 READY 60건 중 54건이 UNRESOLVED였습니다. 이는 모든 기사를 강제로 판정하기보다 좌표·기간·단위·공식값 중 하나라도 불확실하면 오판을 막는 안전장치입니다.

개발셋 재대입에서는 ITEM Top-5 90.3%, 좌표 Top-5 90.2%였지만 blind100에서는 각각 58.0%, 54.0%였습니다. 반복된 표와 키워드에 대한 과적합을 확인했고, 이후 평가셋은 기사뿐 아니라 `tbl_id`와 `table_family` 기준으로 분리해야 한다는 결론을 얻었습니다.

</details>

---

## Technical Toolbox

### AI / NLP

Python · PyTorch · Hugging Face · HCX-007 · BGE-M3 · BGE Reranker · BM25 · Transformer · BERT · GPT · CLIP

### Retrieval / LLM Application

RAG · Lexical Search · Dense Retrieval · Hybrid Retrieval · Reranking · Structured Outputs · Prompt Design

### Backend / Data

FastAPI · PostgreSQL · SQLite · ChromaDB · REST API · KOSIS Open API · JSON Schema · CSV Pipeline

### Evaluation

Top-k Recall · Coordinate Accuracy · Gold Dataset · Blind Evaluation · Regression Test · Error Analysis

---

## What I Value

- 검색 모델의 점수와 실제 데이터의 존재 여부를 분리해 검증합니다.
- 개발셋 성능과 blind 성능을 구분해 일반화 가능성을 확인합니다.
- 성공 사례뿐 아니라 누락, 오탐, 과적합과 같은 실패 원인을 기록합니다.
- 근거가 없을 때는 그럴듯한 답보다 `UNRESOLVED`가 더 안전하다고 생각합니다.
- 새로운 프로젝트도 동일한 기준으로 문제, 역할, 구현, 결과와 시행착오를 기록할 예정입니다.
