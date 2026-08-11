# TUKTAK

**집수리 AI 견적 및 시공자 매칭 플랫폼**

TUKTAK은 사용자가 수리 사진과 설명을 입력하면 AI 견적서, 리스크 리포트, 역경매 매칭으로 이어지는 서비스입니다.  
집수리 시장의 불투명한 견적, 추가 비용 리스크, 시공자 선택의 어려움을 줄이는 것을 목표로 했습니다.

---

## Project Links

| Repository | Description |
| --- | --- |
| https://github.com/TUKTAKxAI/TUKTAK_Backend | Main Backend |
| https://github.com/TUKTAKxAI/TUKTAK_AI_Service | AI Service |
| https://github.com/TUKTAKxAI/TUKTAK_Frontend | Frontend |

---

## My Role

**AI 백엔드 / RAG / 이미지 유사도 검색 설계 및 구현**

- Risk Report 도메인 모델 및 API 구조 구현
- Main Backend와 AI Service 연동 구조 설계
- ChromaDB 기반 RAG 리스크 리포트 검색 흐름 설계 및 구현
- RAG 문서 메타데이터 기반 검색 테스트
- 이미지 유사사례 검색 Threshold 설계
- Nomic Embed Vision 기반 이미지 임베딩 검색 구조 검토
- 시연 영상 제작

---

## Service Flow

```text
사용자 사진/설명 입력
-> AI 견적서 생성
-> 리스크 리포트 생성
-> 여러 시공자의 견적 제안
-> 사용자 비교 선택
-> 매칭 / 채팅 / 리뷰 관리
```

TUKTAK은 단순히 시공자를 연결하는 서비스가 아니라, AI 견적서를 기준으로 수리 판단 기준을 먼저 제공하고 이후 매칭까지 연결하는 구조입니다.

---

## AI Estimate Flow

```text
사진 입력
-> 사진 품질 검사
-> 이미지 임베딩 생성
-> 카테고리/작업 조건 기반 유사사례 검색

텍스트 입력
-> 텍스트 유효성 검사
-> 수리 대상/증상/작업 구조화
-> 유사사례 검색 조건 생성

유사사례 존재
-> 유사사례 기반 견적 생성

유사사례 부족
-> LLM 기반 보완 판단
```

AI 견적서는 사진과 설명을 기반으로 수리 대상, 문제 상태, 예상 작업, 예상 비용, 예상 시간, 신뢰도 등을 생성하는 흐름으로 설계했습니다.

---

## Risk Report RAG Flow

리스크 리포트는 AI 견적서 결과를 기반으로 관련 근거 문서를 검색하고, 추가 비용 가능성, 안전 주의사항, 계약 분쟁 가능성, 현장 변수를 사용자에게 안내하는 기능입니다.

```text
AI 견적서 결과
-> 카테고리/작업/증상 기반 Query 생성
-> 문서 임베딩 검색
-> ChromaDB에서 관련 근거 문서 조회
-> LLM이 검색된 근거를 참고해 리스크 리포트 작성
```

RAG를 사용한 이유는 리스크 리포트가 계약, 추가 비용, 안전, 현장 변수처럼 근거가 중요한 내용을 다루기 때문입니다.  
LLM이 임의로 내용을 생성하지 않도록, 직접 수집한 문서와 메타데이터를 ChromaDB에 저장하고 검색된 문서를 기반으로 리포트를 생성하는 구조로 설계했습니다.

---

## RAG Document Types

- 소비자분쟁해결기준
- 표준계약서
- 한국소비자원 피해구제 사례
- KCS 표준시방서
- 품질보증기간 및 부품보유기간 자료
- 공사 소음/공동주택 관련 기준
- 추가 공사 비용 발생 조건
- 안전 주의사항
- 현장 변수 정리 자료

---

## Vector Search Design

| Design Point | Description |
| --- | --- |
| Vector DB | ChromaDB |
| Search Method | Embedding similarity search |
| Similarity 기준 | Cosine Similarity |
| Filtering | Category / Risk Type / Threshold |
| Conservative Rule | 관련 문서가 부족하면 UNKNOWN 또는 보완 필요 상태로 처리 |

RAG 검색은 잘못된 근거 문서가 통과되는 것보다, 관련성이 부족한 문서를 제외하는 것이 더 중요하다고 판단했습니다.  
따라서 Threshold를 보수적으로 설정하고, 근거 문서가 부족한 경우 낮은 위험도로 단정하지 않도록 설계했습니다.

---

## Image Similarity Evaluation

이미지 유사사례 검색은 잘못된 수리 사례가 통과되는 것을 줄이는 방향으로 설계했습니다.  
정답률을 무리하게 높이는 것보다, 수리와 무관하거나 다른 증상인 이미지를 걸러내는 보수적인 Threshold 설정을 우선했습니다.

**Tested Models / Methods**

- DINOv2
- Nomic Embed Vision
- CLIP
- ConvNeXt
- Cosine Similarity
- Category-based Threshold
- Ensemble Search

**Key Insight**

이미지 임베딩 검색은 카테고리마다 난이도가 달랐습니다.  
따라서 모든 카테고리에 하나의 Threshold를 적용하기보다, 이미지가 중요한 카테고리를 선별하고 카테고리별 Threshold를 적용하는 방향이 더 적합하다고 판단했습니다.

---

## Tech Stack

| Area | Stack |
| --- | --- |
| Backend | FastAPI, SQLAlchemy, MySQL |
| AI Service | LangGraph, LangChain, ChromaDB, Embedding Model, LLM API |
| AI Evaluation | Nomic Embed Vision, DINOv2, CLIP, Cosine Similarity, Threshold Tuning |
| Infra | Docker, AWS Deployment Architecture |
| Collaboration | GitHub Branch/PR, API Specification, Team Module Ownership |

---

## What I Learned

- AI 모델 결과를 서비스 API와 연결하는 구조
- RAG 검색에서 문서 품질과 메타데이터의 중요성
- 임베딩 검색은 정확도보다 오탐 방지가 중요한 경우가 있다는 점
- Main Backend와 AI Service를 분리해 설계하는 방식
- 팀 프로젝트에서 AI 기능을 API 명세와 서비스 흐름에 맞춰 정리하는 방법
