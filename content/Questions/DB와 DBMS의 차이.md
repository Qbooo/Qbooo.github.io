---
title: DB와 DBMS의 차이
type: question
domain: Database
tags:
  - 질문
aliases:
  - DB와 DBMS는 같은 말인가
parent:
status: growing
lang: ko
created: 2026-11-11
updated: 2026-11-11
resolved: true
resolved_at: 2026-11-11
---
## 목차

- [[#질문|질문]]
- [[#해답|해답]]
- [[#프로젝트 판단|프로젝트 판단]]
- [[#관련 문서|관련 문서]]
- [[#출처|출처]]

## 질문

DB와 DBMS는 같은 말인가?

## 해답

엄밀하게 구분할 수 있지만, 실제 문맥에서는 `database`가 데이터 집합과 DBMS를 모두 가리키기도 한다.[^1]

- 데이터베이스(DB): DBMS가 관리하는 데이터의 집합이다.
- DBMS: 물리 데이터와 사용자 사이에서 데이터 접근을 관리하는 소프트웨어 계층이다. MySQL과 PostgreSQL 같은 제품이 여기에 해당한다.[^1]
- 혼용: Microsoft ODBC 용어집도 `database`를 DBMS 안의 데이터 모음이면서 DBMS 자체를 가리킬 수 있는 말로 함께 설명한다.[^1]

"DB 서버를 복제한다"고 할 때의 DB는 보통 DBMS가 동작하는 서버를 뜻한다.

## 프로젝트 판단

아래 판단은 소규모 연습 프로젝트(가상의 소규모 고객, 직원 30명 안팎의 유통업체를 가정한 DR 이전 과제)에서 요구사항과 규모를 고려해 내린 것이며, 모든 환경에 맞는 정답은 아니다.

- 맥락: 도식의 "DB" 박스는 DBMS가 동작하는 VM이고, 그 안에 데이터베이스(데이터)가 있다.
- 판단: 설계서에서는 "DB 서버(DBMS)"처럼 표기해 복제와 백업 대상이 소프트웨어가 아니라 데이터임을 헷갈리지 않게 한다.

## 관련 문서

- [[Database]]

## 출처

[^1]: database와 DBMS의 정의 및 실제 용어 혼용. [Microsoft Learn, ODBC Glossary](https://learn.microsoft.com/en-us/sql/odbc/odbc-glossary)

## 둘러보기

[[Questions/index|질문과 해답]] · [[Knowledge Base/index|Knowledge Base]] · [[Projects/index|Projects]]
