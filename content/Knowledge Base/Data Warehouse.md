---
title: Data Warehouse
type: knowledge
domain: Database
tags:
  - Data
  - Analytics
aliases:
  - "DataWarehouse"
  - "데이터 웨어하우스"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-11
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 업무 데이터와 분석 데이터의 차이|2. 업무 데이터와 분석 데이터의 차이]]
- [[#3. 분석 데이터 생성 과정|3. 분석 데이터 생성 과정]]
- [[#4. 업무 데이터의 SQL 분석 경로|4. 업무 데이터의 SQL 분석 경로]]
- [[#5. 데이터 레이크와 비교|5. 데이터 레이크와 비교]]
- [[#6. 저장 매체와 데이터 웨어하우스의 차이|6. 저장 매체와 데이터 웨어하우스의 차이]]
- [[#7. 관련 문서|7. 관련 문서]]
- [[#8. 출처|8. 출처]]

## 1. 개요

Data Warehouse(데이터 웨어하우스)는 여러 출처의 데이터를 분석하기 좋은 형태로 통합하고 관리하여 보고서와 의사결정에 활용하는 분석 시스템이다.

업무 시스템의 데이터를 정제하고 공통된 정의로 조회할 수 있게 하는 데 중점을 둔다.[^2]

## 2. 업무 데이터와 분석 데이터의 차이

주문 시스템은 주문 한 건을 빠르고 정확하게 등록·변경하는 데 중점을 둔다. 반면 분석 시스템은 여러 기간과 조직의 데이터를 모아 매출 추이, 고객군별 구매, 상품별 성과처럼 넓은 범위를 조회한다. 데이터 웨어하우스는 이런 분석을 위해 데이터를 공통 기준으로 통합하고 정리한다.[^2][^v-olap]

예를 들어 매장별 시스템이 서로 다른 상품 코드와 시간 기준을 사용하면 데이터를 합치기만 해서는 정확한 비교가 어렵다. 상품·고객·날짜 같은 공통 기준과 매출 계산 규칙을 정해야 같은 질문에 같은 의미의 결과를 낼 수 있다. 웨어하우스의 본질은 큰 디스크보다 일관된 분석이 가능한 데이터 모델에 있다.

열 지향 저장, 압축, 분산 처리와 작업 관리는 이런 조회를 효율적으로 실행하는 기술이다. 분석 대상 열만 읽거나 여러 노드가 작업을 나누도록 해 대규모 집계에 대응한다. 구체적인 저장·실행 방식은 제품마다 다르므로 웨어하우스 전체를 특정 DBMS의 내부 구조와 동일시하지 않는다.[^d-columnar][^d-distribution]

## 3. 분석 데이터 생성 과정

원천 시스템의 데이터를 수집한 뒤 변환·적재하고, 분석용 테이블이나 뷰로 제공한다. ETL은 추출한 데이터를 변환한 뒤 대상에 적재하고, ELT는 대상에 적재한 뒤 그곳의 처리 기능으로 변환한다. 차이는 단순한 약어 순서뿐 아니라 변환이 수행되는 위치에도 있다.[^v-etl]

예를 들어 주문 데이터와 고객 정보를 공통 기준으로 정리하면 부서마다 다른 방식으로 계산하던 매출 지표를 일관되게 조회할 수 있다. 이 예시는 일반적인 활용 설명이다. 대용량 집계와 조인을 중심으로 하는 분석은 개별 주문의 빠른 입력·수정을 처리하는 트랜잭션 업무와 요구가 다르다. 이를 OLAP 중심의 분석과 OLTP 중심의 거래 처리라는 관점으로 구분할 수 있다.[^v-olap]

## 4. 업무 데이터의 SQL 분석 경로

웨어하우스의 구성 예는 원천 OLTP 시스템 → ETL·ELT 수집과 변환 → 중간 적재 영역 → 분석 테이블 → SQL 엔진 → BI·보고서다. 중간 적재 영역(staging)은 가져온 데이터를 정리하는 곳이고 분석 모델은 사용자에게 어떤 단위·의미로 데이터를 제공할지 정한다. 물리 디스크는 이 시스템이 데이터를 보관하는 기반일 뿐 SQL 엔진·변환·모델을 모두 뜻하지 않는다.

분석 모델에서는 수량·금액 같은 사건의 측정값을 팩트(fact) 테이블에, 고객·상품·날짜 같은 분석 기준을 차원(dimension) 테이블에 담을 수 있다. 핵심은 팩트 테이블의 한 행이 주문 한 건인지 주문 항목 한 건인지와 같은 집계 단위(grain)의 명확한 정의에 있다. 주문 총액과 항목별 금액을 잘못 조인하면 행이 늘어나 같은 매출을 여러 번 더할 수 있다. 이런 가상 예시는 저장장치가 빨라도 보고서가 틀릴 수 있는 이유를 보여 준다. 스타 스키마(star schema)는 팩트 테이블과 차원 테이블을 연결하는 모델이며 모든 웨어하우스의 필수 구조는 아니다.[^d-distribution][^d-grain]

Redshift 같은 열 지향·분산 분석 엔진에서는 필요한 열과 블록을 읽고 필터·조인·집계를 실행한다. 열별 압축은 저장·읽기 양을 줄일 수 있고, 여러 작업 단위의 병렬 처리는 큰 분석을 나누어 수행하게 한다. 조인에 필요한 행이 서로 다른 위치에 있으면 재분배 통신이 필요할 수 있다. 이는 특정 제품을 통해 설명한 원리이며 모든 웨어하우스가 같은 내부 구조라는 뜻은 아니다.[^d-columnar][^d-distribution]

## 5. 데이터 레이크와 비교

| 기준 | [[Data Lake]] | Data Warehouse |
|---|---|---|
| 중심 목적 | 다양한 데이터 보관과 탐색·재활용 | 정제된 데이터의 일관된 분석 |
| 데이터 형태 | 정형·반정형·비정형 포함 | 분석 모델에 맞춘 구조화된 데이터 중심 |
| 활용 | AI 데이터 준비, 탐색 분석 등 | BI, 정기 보고, 공통 지표 분석 등 |

이는 일반적인 경향이다. 제품에 따라 비정형 데이터 분석 등 지원 범위가 넓어질 수 있다.[^1]

## 6. 저장 매체와 데이터 웨어하우스의 차이

웨어하우스는 HDD나 SSD 같은 저장 매체의 이름이 아니다. [[Block Storage]]·[[File Storage]]·[[Object Storage]]는 저장·접근 방식이고, 웨어하우스는 그 위에서 데이터를 처리·관리하는 분석 시스템이다.

따라서 구성도에 웨어하우스용 저장 공간이 표시되어 있어도 해당 스토리지가 SQL 엔진과 데이터 모델까지 제공한다는 뜻은 아니다. 성능은 저장 매체뿐 아니라 쿼리 엔진, 데이터 배치와 동시 작업 수에 영향을 받는다. 예를 들어 Amazon Redshift는 워크로드 관리에서 쿼리 동시성과 메모리 할당을 별도로 관리한다. 이는 특정 웨어하우스 제품의 예시이지 모든 제품의 동일한 구현을 뜻하지 않는다.[^v-wlm]

## 7. 관련 문서

- [[Data Lake]]
- [[Block Storage]]
- [[File Storage]]
- [[Object Storage]]

- [[Database]]

## 8. 출처

[^1]: [AWS — Data Lake, Data Warehouse, Data Mart 비교](https://aws.amazon.com/compare/the-difference-between-a-data-warehouse-data-lake-and-data-mart/)
[^2]: [AWS — What is a Data Warehouse?](https://aws.amazon.com/what-is/data-warehouse/)

[^v-etl]: [AWS — ETL vs ELT](https://aws.amazon.com/compare/the-difference-between-etl-and-elt/)

[^v-olap]: [AWS — OLAP vs OLTP](https://aws.amazon.com/compare/the-difference-between-olap-and-oltp/)

[^v-wlm]: [AWS — Redshift workload management](https://docs.aws.amazon.com/redshift/latest/mgmt/workload-mgmt-config.html)

[^d-columnar]: [AWS Redshift — Columnar storage](https://docs.aws.amazon.com/redshift/latest/dg/c_columnar_storage_disk_mem_mgmnt.html)

[^d-distribution]: [AWS Redshift — Distribution styles and star schema](https://docs.aws.amazon.com/redshift/latest/dg/t_designating_distribution_styles.html)

[^d-grain]: [Microsoft — Star schema: fact tables and granularity](https://learn.microsoft.com/en-us/power-bi/guidance/star-schema)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Database 분류|Database]]
