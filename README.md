# TUKTAK Project Summary

집수리 AI 견적 및 시공자 매칭 플랫폼

TUKTAK은 사용자가 수리 사진과 설명을 입력하면 AI 견적서, 리스크 리포트, 시공자 매칭으로 이어지는 서비스입니다.  
기존 집수리 시장의 불투명한 견적, 추가 비용 리스크, 시공자 선택의 어려움을 줄이는 것을 목표로 했습니다.

## Service Flow

```text
사용자 사진/설명 입력
-> AI 견적서 생성
-> 리스크 리포트 생성
-> 여러 시공자의 견적 제안
-> 사용자 비교 선택
-> 매칭/채팅/리뷰 관리
```

## Repository

- Backend: https://github.com/TUKTAKxAI/TUKTAK_Backend
- AI Service: https://github.com/TUKTAKxAI/TUKTAK_AI_Service
- Frontend: https://github.com/TUKTAKxAI/TUKTAK_Frontend

## My Role

AI 백엔드 및 RAG/유사도 검색 설계 구현을 중심으로 참여했습니다.

- Risk Report 도메인 모델 및 API 구조 구현
- Main Backend와 AI Service 연동 구조 설계
- ChromaDB 기반 RAG 리스크 리포트 검색 흐름 설계 및 구현
- 이미지 유사사례 검색 Threshold 설계
- Nomic Embed Vision 기반 이미지 임베딩 검색 구조 검토
- RAG 문서 메타데이터 기반 검색 테스트
- 시연 영상 제작

## AI Estimate Flow

AI 견적서는 사용자의 사진과 텍스트 설명을 기반으로 수리 요청을 구조화하고, 유사사례와 기준 데이터를 활용해 예상 견적을 생성합니다.

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

## Risk Report RAG Flow

리스크 리포트는 AI 견적서 결과를 기반으로 관련 문서를 검색하고, 추가 비용 가능성, 안전 주의사항, 계약 분쟁 가능성, 현장 변수를 사용자에게 안내하는 기능입니다.

```text
AI 견적서 결과
-> 카테고리/작업/증상 기반 Query 생성
-> 문서 임베딩 검색
-> ChromaDB에서 관련 근거 문서 조회
-> LLM이 검색된 근거를 참고해 리스크 리포트 작성
```

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

## Vector Search Design

리스크 리포트는 계약, 추가 비용, 안전, 현장 변수처럼 근거가 중요한 내용을 다루기 때문에 LLM이 임의로 문서를 생성하지 않도록 설계했습니다.

- 문서를 ChromaDB에 저장
- 문서별 메타데이터 관리
- 임베딩 모델로 Query와 문서 간 유사도 계산
- Threshold를 기준으로 관련 문서만 통과
- 관련 문서가 부족하면 UNKNOWN 또는 보완 필요 상태로 처리

## Image Similarity Evaluation

이미지 유사사례 검색은 잘못된 사례가 통과되는 것을 줄이는 방향으로 설계했습니다.  
정확도를 무리하게 높이는 것보다, 수리와 무관하거나 다른 증상인 이미지를 걸러내는 보수적인 Threshold 설정을 우선했습니다.

Tested Approach

- DINOv2
- Nomic Embed Vision
- CLIP
- ConvNeXt
- Cosine Similarity
- Category-based Threshold
- Ensemble Search

## Tech Stack

Backend  
FastAPI, SQLAlchemy, MySQL

AI Service  
LangGraph, LangChain, ChromaDB, Embedding Model, LLM API

AI Evaluation  
Nomic Embed Vision, DINOv2, CLIP, cosine similarity, threshold tuning

Infra  
Docker, AWS deployment architecture

Collaboration  
GitHub Branch/PR, API specification, team-based module ownership

## What I Learned

- AI 모델 결과를 서비스 API와 연결하는 구조
- RAG 검색에서 문서 품질과 메타데이터의 중요성
- 임베딩 검색은 정확도보다 오탐 방지가 중요한 경우가 있다는 점
- 팀 프로젝트에서 AI Service와 Main Backend를 분리해 설계하는 방식
- 서비스 시연을 위한 사용자/시공자 흐름 구성
