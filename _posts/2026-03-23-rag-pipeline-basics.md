---
layout: post
title: "RAG 파이프라인 기초: 나만의 AI 지식 베이스 만들기"
date: 2026-03-23
categories: [AI엔지니어링]
tags: [RAG, 벡터DB, LLM, 임베딩]
description: "RAG(Retrieval-Augmented Generation) 파이프라인의 핵심 개념과 구축 방법을 알아봅니다."
---

LLM은 학습 데이터에 포함되지 않은 최신 정보나 도메인 특화 지식에 대해 정확한 답변을 하기 어렵습니다. RAG(Retrieval-Augmented Generation)는 이 문제를 해결하는 가장 실용적인 방법입니다.

## RAG란?

RAG는 **검색(Retrieval)** 과 **생성(Generation)** 을 결합한 방식입니다.

1. 사용자 질문이 들어오면
2. 관련 문서를 벡터 DB에서 검색하고
3. 검색된 문서를 컨텍스트로 LLM에 전달하여
4. 정확하고 근거 있는 답변을 생성합니다

## 핵심 구성 요소

### 문서 전처리

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    separators=["\n\n", "\n", ". ", " "]
)
chunks = splitter.split_documents(documents)
```

문서를 적절한 크기의 청크로 나누는 것이 중요합니다. 너무 크면 노이즈가 많아지고, 너무 작으면 맥락이 손실됩니다.

### 임베딩 & 벡터 저장

```python
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import Chroma

embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(chunks, embeddings)
```

### 검색 & 생성

```python
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})
relevant_docs = retriever.get_relevant_documents("질문")
```

## 성능 향상 팁

- **Hybrid Search**: 키워드 검색과 의미 검색을 결합
- **Re-ranking**: 검색된 문서의 관련성을 재평가
- **Chunk 최적화**: 도메인에 맞는 청크 크기와 오버랩 조정
- **메타데이터 필터링**: 날짜, 카테고리 등으로 검색 범위 제한

## 마무리

RAG는 LLM의 한계를 보완하는 강력한 패턴입니다. 사내 문서 검색, 고객 지원 챗봇, 연구 어시스턴트 등 다양한 곳에 활용할 수 있습니다. 다음 글에서는 실제 프로덕션 환경에서의 RAG 최적화 기법을 다뤄보겠습니다.
