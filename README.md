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

## Projects

프로젝트가 추가되어도 한눈에 비교할 수 있도록 모든 프로젝트를 같은 형식으로 정리합니다.

| Project | Area | Status | Links |
|---|---|---|---|
| KOSIS 뉴스 수치 검증 PoC | NLP · Retrieval · Backend · Evaluation | Completed | [Overview](./projects/kosis-news-verification.md) · [Repository](https://github.com/rnwjdgus03/NLP_05-Team-Project-3) · [PPTX](./assets/kosis-news-verification-final.pptx) |
| FIRE CARE | Mobile · AI Vision · Backend · PostgreSQL | In Progress | [Overview](./projects/fire-care.md) · [Repository](https://github.com/rnwjdgus03/fire-care) · [PPTX](./assets/fire-care-overview.pptx) |

### 01. KOSIS 뉴스 수치 검증 PoC

> 뉴스 기사 속 수치 주장을 KOSIS 공식 통계 좌표와 연결해, 검증 가능한 범위만 안전하게 자동화한 PoC

**Team Project · NLP · Retrieval · Backend · Evaluation**

- 뉴스 기사 URL을 입력하면 기사에서 검증할 수 있는 수치 주장을 자동으로 찾아냅니다.
- 수치가 의미하는 대상·기간·단위를 분석하고, 이에 대응하는 KOSIS 공식 통계표와 세부 좌표를 탐색합니다.
- 기사에 제시된 값과 KOSIS 공식값을 비교하고, 사용자가 직접 확인할 수 있도록 통계표와 좌표를 근거로 제공합니다.
- 공식 통계만으로 확인할 수 없는 주장은 임의로 판정하지 않고 `UNRESOLVED`로 분류합니다.

| Key Result | Value |
|---|---:|
| 개발 좌표 gold 300건 ITEM Top-5 | 77.3% |
| 개발 좌표 gold 300건 전체 좌표 Top-5 | 74.3% |
| 실제 기사 개발 E2E | READY 60건 중 MATCH 6건 |
| 전체 처리시간 | 193.25초 → 120.70초 |

> Top-k는 팩트체크 정확도가 아니라 정답 좌표가 상위 k개 후보에 포함된 비율입니다.

[프로젝트 상세 내용과 트러블슈팅 보기 →](./projects/kosis-news-verification.md)

### 02. FIRE CARE

> 시설설비 점검자가 모바일에서 체크리스트 기반 점검을 수행하고, Gemini 사진 분석과 자동 보고서 생성을 통해 현장 점검을 보조하는 앱

**Capstone Project · Mobile · AI Vision · Backend · PostgreSQL**

- React Native/Expo로 설비 등록, 점검, 보고서 생성 흐름을 구현했습니다.
- 16개 설비 유형과 137개 점검 항목을 서버 DB 기준으로 관리했습니다.
- Gemini가 선택 설비와 사진 속 설비의 일치 여부를 먼저 확인하고, 불일치 시 저장을 차단하도록 설계했습니다.
- 어둡거나 흐린 사진은 재촬영을 안내하고, 사진으로 확인할 수 없는 항목은 작업자가 직접 확인하도록 분리했습니다.
- PostgreSQL 기반으로 사용자, 건물, 층, 설비, 점검 결과, 사진 경로를 저장해 여러 휴대폰에서 같은 데이터를 볼 수 있도록 설계했습니다.

| Key Focus | Implementation |
|---|---|
| 설비-사진 불일치 방지 | Gemini 설비 일치 확인 후 불일치 시 저장 차단 |
| 저품질 사진 대응 | 어두움·흐림·원거리·잘림 사진 재촬영 안내 |
| 다중 작업자 동기화 | PostgreSQL 서버 DB 기반 점검 데이터 공유 |
| 보고서 자동화 | 건물 전체 점검 결과와 인증사진을 묶어 이메일 보고서 생성 |

[프로젝트 저장소와 상세 README 보기 →](https://github.com/rnwjdgus03/fire-care)

<!-- 새 프로젝트는 아래 형식으로 추가합니다.

### 02. Project Name

> 한 줄 문제 정의

**Role · Team Size · Period · Area**

- 핵심 구현 1
- 핵심 구현 2
- 핵심 구현 3

| Key Result | Value |
|---|---:|
| Metric | Result |

[상세 내용 보기 →](./projects/project-name.md)
-->

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
- [FIRE CARE](https://github.com/rnwjdgus03/fire-care) — AI 기반 시설설비 점검 및 보고서 자동화 모바일 앱
