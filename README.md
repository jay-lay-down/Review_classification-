🚀 **Project: Zero-Shot VOC Insight Engine**  
RAG 기반 퓨샷(Few-shot) 분류기를 활용한 신제품 콜드 스타트(Cold Start) 해결 솔루션

---

## 1. 개요 (Overview)
기존 텍스트 분류 모델의 한계(데이터 라벨링 비용, 재학습 시간, 신제품 출시 시 분석 불가)를 극복하기 위한 **RAG(Retrieval-Augmented Generation) 기반 자동 분류 시스템**입니다.  
과거에 축적된 수만 건의 VOC 데이터 자산(Data Dam)을 활용하여, **학습(Fine-tuning) 없이도** 신제품 리뷰를 즉시, 그리고 도메인 맥락에 맞게 정확히 분류합니다.

---

## 2. 핵심 아키텍처 (Core Architecture)
이 솔루션은 LLM에게 단순 분류를 시키는 것이 아니라, **"과거의 유사 사례를 참고하여 판단하게 하는" 메커니즘**을 따릅니다.

### 2.1 Data Dam & Vectorization (기억 저장소 구축)
- 기존 제품들의 VOC(리뷰, 문의)와 라벨(불만 유형, 감성 등)을 텍스트 임베딩 모델을 통해 벡터(Vector)로 변환하여 DB에 저장합니다.

### 2.2 Semantic Search (유사 사례 검색)
- 새로운 VOC(Query)가 들어오면, 벡터 공간에서 의미적으로 가장 가까운 과거의 사례(Reference) Top-k개를 추출합니다.

### 2.3 In-Context Learning (맥락 기반 추론)
- LLM에게 입력 데이터 + 검색된 과거 유사 사례(라벨 포함)를 함께 프롬프트로 제공합니다.
- 예:  
  *"참고 자료를 보니 이런 뉘앙스는 '배송 지연'이었어. 이번 건도 비슷해 보이는데 맞지?"*  
  와 같이 추론하게 합니다.

---

## 3. 비즈니스 가치 (Why This Matters?)

### ① 콜드 스타트(Cold Start) 완벽 해결
- **문제:** 신제품(예: 제로 콜라)이 출시되면 학습 데이터가 없어 기존 AI는 분석이 어려움  
- **해결:** 기존 제품(예: 사이다)의 '탄산 빠짐' 불만 패턴을 즉시 참조하여, **출시 첫날부터 분석 리포트 생성 가능**

### ② 도메인 지식 자동 이식 (Domain Adaptation)
범용 모델은 알 수 없는 산업 특화 뉘앙스를 별도 학습 없이 파악합니다.

- 샴푸 리뷰: **"거품이 안 나요"** → 부정(세정력 이슈)  
- 식기세척기 세제: **"거품이 안 나요"** → 긍정(헹굼력 우수)

RAG가 해당 카테고리의 과거 데이터를 참조하므로 문맥을 정확히 이해합니다.

### ③ 압도적 가성비 (Cost Efficiency)
- **No Fine-tuning:** 고가의 GPU로 모델을 재학습할 필요가 없습니다.  
- **Low Maintenance:** 데이터가 추가되면 벡터 DB에 넣기만 하면 끝. 모델 업데이트 주기가 필요 없습니다.

---

## 4. 기술적 과제 및 해결책 (Technical Deep Dive)

### ⚠️ 위험 요소: 맥락의 충돌 (Context Conflict)
서로 다른 카테고리의 데이터가 혼재될 때 발생하는 할루시네이션(Hallucination) 문제입니다.

- Example: **"톡 쏘네요"**라는 리뷰에 대해  
  DB가 '사이다(긍정)' 데이터를 가져오면  
  '화장품(피부 자극-부정)' 분석이 오작동할 수 있음

### ✅ 해결책: 메타데이터 필터링 (Metadata Filtering)
검색 단계에서 범위를 제한하는 **Hybrid Search 전략**을 사용합니다.

- Query 시 `Category == Target_Product_Category` 필터를 적용  
- 화장품 리뷰를 분석할 땐 식품 데이터를 배제하고, **화장품 데이터 안에서만** 유사 사례를 찾도록 강제함

---

## 5.기술 스택 (Tech Stack)
- **LLM:** Google Gemini-1.5-Flash  
  (빠른 속도, 저렴한 비용, 긴 컨텍스트 윈도우로 다수의 Few-shot 예제 입력 가능)
- **Embedding Model:** text-embedding-004  
  (다국어 지원 및 높은 성능)
- **Vector DB:**
  - **Prototyping:** ChromaDB (로컬, 가벼움)
  - **Production:** Pinecone or Elasticsearch (메타데이터 필터링 성능 우수)
- **Application:** Python + Streamlit (빠른 대시보드 구현)
- **Langchain
