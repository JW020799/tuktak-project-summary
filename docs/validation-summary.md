# TUKTAK AI Validation Summary

TUKTAK 프로젝트에서 AI 견적과 리스크 리포트 흐름을 안정적으로 서비스에 연결하기 위해 진행한 검증 내용을 정리한 문서입니다.

---

## 1. Risk Report RAG

리스크 리포트는 AI 견적서 이후 추가 비용, 안전, 계약 분쟁, 현장 변수 등을 사용자에게 안내하는 기능입니다.  
근거가 중요한 기능이므로 LLM이 임의로 내용을 생성하지 않도록 ChromaDB 기반 RAG 검색 구조를 적용했습니다.

| Item | Result |
| --- | --- |
| RAG Document Count | 71 documents |
| Embedding Model | `jhgan/ko-sroberta-multitask` |
| Embedding Dimension | 768 |
| Vector DB | ChromaDB PersistentClient |
| Collection | `risk_documents_ko_sroberta_v2` |
| Top-K | 3 |
| Search Logic | `main_category + risk_category` metadata filter + cosine similarity threshold |
| Failure Rule | UNKNOWN 3개 이상이면 `FAILED` 처리 |

### Risk Categories

| Risk Category | Threshold | Weight | Meaning |
| --- | ---: | ---: | --- |
| PRICE | 0.30 | 10% | 가격/견적 리스크 |
| EXTRA_COST | 0.55 | 20% | 추가 공사 및 추가 비용 리스크 |
| SAFETY | 0.60 | 30% | 작업 중 안전 및 사용자 안전 리스크 |
| CONTRACT | 0.60 | 25% | 계약, 하자보수, AS, 분쟁 리스크 |
| FIELD | 0.65 | 15% | 현장 방문 전 확인하기 어려운 변수 |

### Representative Test Result

| Category | Status | Score | Grade | Evidence | UNKNOWN |
| --- | --- | ---: | --- | --- | --- |
| 타일/바닥 | COMPLETED | 75 | HIGH | 5/5 | 없음 |
| 창호/문 | COMPLETED | 67 | MEDIUM | 5/5 | 없음 |
| 전기/조명 | COMPLETED | 68 | MEDIUM | 5/5 | 없음 |
| 보일러/난방 | COMPLETED | 69 | MEDIUM | 5/5 | 없음 |
| 도배/벽면 | COMPLETED | 71 | HIGH | 5/5 | 없음 |
| 배관/누수 | COMPLETED | 69 | MEDIUM | 5/5 | 없음 |
| 욕실 | COMPLETED | 46 | MEDIUM | 4/5 | FIELD |
| 주방 | COMPLETED | 72 | HIGH | 5/5 | 없음 |
| 가전 | FAILED | null | INSUFFICIENT_EVIDENCE | 2/5 | EXTRA_COST, SAFETY, FIELD |
| 가구/설치 | COMPLETED | 40 | MEDIUM | 3/5 | SAFETY, FIELD |

### Key Decision

- 근거 문서가 충분한 케이스만 `COMPLETED`로 리스크 점수와 등급을 제공합니다.
- 근거가 부족한 케이스는 낮은 위험도로 단정하지 않고 `FAILED` 또는 보완 필요 상태로 처리합니다.
- PRICE 문서는 직접 가격 기준표가 부족해 운영 전 카테고리별 평균 수리비 자료 보강이 필요합니다.

---

## 2. Text Validity Rule Engine

텍스트 유효성 검사는 욕설, 스팸, 무의미 입력, 수리 무관 문장을 1차로 걸러내기 위한 단계입니다.  
명확한 입력은 Rule Engine에서 처리하고, 의미 판단이 애매한 문장만 NLP 단계로 전달하는 구조로 설계했습니다.

| Item | Design |
| --- | --- |
| Selected Tool | Rule-Engine |
| Preprocessing | 공백 정리, 영문 소문자화, 반복 문자 축약, 특수문자 정규화 |
| Output Label | VALID / INVALID / REVIEW_REQUIRED |
| Rule Scope | 형식 규칙, 욕설/스팸, 수리 객체, 증상, 요청 표현 |
| Review Policy | VALID과 INVALID 사이 점수 구간은 NLP 전달 |

### Evaluation Priority

| Metric | Meaning | Priority |
| --- | --- | --- |
| 정상 요청 오차단율 | 정상 수리 문장을 INVALID로 판정한 비율 | Highest |
| 비정상 요청 통과율 | 욕설, 스팸, 수리 무관 문장을 VALID로 통과시킨 비율 | High |
| Precision / Recall / F1 | 유형별 탐지 성능 | High |
| REVIEW_REQUIRED 비율 | NLP로 전달되는 애매한 문장 비율 | Medium |
| 처리 시간 / 메모리 | 실시간 API 적용 가능 여부 | Supporting |

### Key Decision

- 정상 요청을 잘못 차단하지 않는 것을 최우선으로 둡니다.
- 명확한 욕설, 스팸, 형식 오류는 Rule Engine에서 처리합니다.
- 의미 판단이 애매한 문장은 NLP 단계로 넘겨 과도한 오차단을 줄입니다.

---

## 3. Image Quality Check

사진 품질 검사는 사용자가 업로드한 사진이 AI 견적에 활용 가능한지 판단하기 위한 단계입니다.  
초점 불량, 저조도, 과다노출, 저해상도 사진을 탐지하는 모델을 비교했습니다.

| Model | Decision | Reason |
| --- | --- | --- |
| OpenCV baseline | 1순위 후보 | 순수 실사 기준 커스텀 분류기와 통계적 동률, 무료, 고속, 설명 가능 |
| Custom Classifier v1 | 1순위 후보 | 순수 실사 기준 OpenCV와 통계적 동률, 실사 데이터 축적 시 개선 가능 |
| BRISQUE | 제외 | OpenCV/커스텀 분류기 대비 낮은 성능 |
| NIMA | 제외 | 성능, 속도, 모델 크기 모두 불리 |

### Key Result

| Comparison | Result |
| --- | --- |
| Custom v1 vs OpenCV on real data | 통계적 동률 |
| Custom v1 vs BRISQUE/NIMA | 커스텀 우위 |
| OpenCV vs BRISQUE/NIMA | OpenCV 우위 |
| Short-term Recommendation | OpenCV baseline |
| Long-term Recommendation | 실사 데이터 축적 후 Custom Classifier 재학습 |

### Key Decision

- 단기 배포에는 OpenCV baseline이 적합합니다.
- OpenCV는 인프라 비용이 낮고, 사용자에게 초점/노출 등 판정 사유를 설명하기 쉽습니다.
- 중장기적으로는 실사 데이터가 쌓이면 커스텀 분류기 재학습을 검토합니다.

---

## Overall Insight

TUKTAK의 AI 기능은 단순히 모델 결과를 바로 사용하는 방식이 아니라, 입력 검증, 이미지 품질 검사, 벡터 검색, 근거 문서 검색, 실패 조건 처리를 함께 설계했습니다.

```text
사진/텍스트 입력
-> 품질 및 유효성 검사
-> 구조화 및 임베딩
-> 유사사례 또는 RAG 문서 검색
-> 근거가 충분하면 결과 생성
-> 근거가 부족하면 보완 필요 또는 FAILED 처리
```

이 구조는 잘못된 AI 결과가 사용자에게 그대로 노출되는 것을 줄이고, 서비스 단계에서 설명 가능한 판단 기준을 제공하기 위한 설계입니다.
