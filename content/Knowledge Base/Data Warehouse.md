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
updated: 2026-11-04
---

## 1. 개요

Data Warehouse(데이터 웨어하우스)는 여러 출처의 데이터를 분석하기 좋은 형태로 통합하고 관리하여 보고서와 의사결정에 활용하는 분석 시스템이다.

업무 시스템의 데이터를 정제하고 공통된 정의로 조회할 수 있게 하는 데 중점을 둔다.[^2]

## 2. 분석 데이터가 만들어지는 과정

원천 시스템의 데이터를 수집한 뒤 변환·적재하고, 분석용 테이블이나 뷰로 제공한다. ETL은 추출한 데이터를 변환한 뒤 대상에 적재하고, ELT는 대상에 적재한 뒤 그곳의 처리 기능으로 변환한다. 차이는 단순한 약어 순서뿐 아니라 변환이 수행되는 위치에도 있다.[^v-etl]

예를 들어 주문 데이터와 고객 정보를 공통 기준으로 정리하면 부서마다 다른 방식으로 계산하던 매출 지표를 일관되게 조회할 수 있다. 이 예시는 일반적인 활용 설명이다. 대용량 집계와 조인을 중심으로 하는 분석은 개별 주문의 빠른 입력·수정을 처리하는 트랜잭션 업무와 요구가 다르다. 이를 OLAP 중심의 분석과 OLTP 중심의 거래 처리라는 관점으로 구분할 수 있다.[^v-olap]

## 3. 데이터 레이크와 비교

| 기준 | [[Data Lake]] | Data Warehouse |
|---|---|---|
| 중심 목적 | 다양한 데이터 보관과 탐색·재활용 | 정제된 데이터의 일관된 분석 |
| 데이터 형태 | 정형·반정형·비정형 포함 | 분석 모델에 맞춘 구조화된 데이터 중심 |
| 활용 | AI 데이터 준비, 탐색 분석 등 | BI, 정기 보고, 공통 지표 분석 등 |

이는 일반적인 경향이다. 제품에 따라 비정형 데이터 분석 등 지원 범위가 넓어질 수 있다.[^1]

## 4. 디스크 종류와 별개의 개념

웨어하우스는 HDD나 SSD 같은 저장 매체의 이름이 아니다. [[Block Storage]]·[[File Storage]]·[[Object Storage]]는 저장·접근 방식이고, 웨어하우스는 그 위에서 데이터를 처리·관리하는 분석 시스템이다.

따라서 구성도에 웨어하우스용 저장 공간이 표시되어 있어도 해당 스토리지가 SQL 엔진과 데이터 모델까지 제공한다는 뜻은 아니다. 성능은 저장 매체뿐 아니라 쿼리 엔진, 데이터 배치와 동시 작업 수에 영향을 받는다. 예를 들어 Amazon Redshift는 워크로드 관리에서 쿼리 동시성과 메모리 할당을 별도로 관리한다. 이는 특정 웨어하우스 제품의 예시이지 모든 제품의 동일한 구현을 뜻하지 않는다.[^v-wlm]

## 5. 업무 데이터에서 SQL 분석까지

웨어하우스의 구성 예는 원천 OLTP 시스템 → ETL·ELT 수집과 변환 → staging → 분석 테이블 → SQL 엔진 → BI·보고서다. Staging은 가져온 데이터를 정리하는 중간 영역이고 분석 모델은 사용자에게 어떤 단위·의미로 데이터를 제공할지 정한다. 물리 디스크는 이 시스템이 데이터를 보관하는 기반일 뿐 SQL 엔진·변환·모델을 모두 뜻하지 않는다.

분석 모델에서는 사실(fact)을 수량·금액 같은 사건의 측정값, 차원(dimension)을 고객·상품·날짜 같은 분석 축으로 나눌 수 있다. 핵심은 fact 한 행이 주문 한 건인지 주문 항목 한 건인지 같은 집계 단위(grain)를 정하는 것이다. 주문 총액과 항목별 금액을 잘못 조인하면 행이 늘어나 같은 매출을 여러 번 더할 수 있다. 이런 가상 예시는 저장장치가 빨라도 보고서가 틀릴 수 있는 이유를 보여 준다. Star schema는 fact와 dimension을 연결하는 한 모델이며 모든 웨어하우스의 필수 구조는 아니다.[^d-distribution][^d-grain]

Redshift 같은 열 지향·분산 분석 엔진에서는 필요한 열과 블록을 읽고 필터·조인·집계를 실행한다. 열별 압축은 저장·읽기 양을 줄일 수 있고, 여러 작업 단위의 병렬 처리는 큰 분석을 나누어 수행하게 한다. 조인에 필요한 행이 서로 다른 위치에 있으면 재분배 통신이 필요할 수 있다. 이는 특정 제품을 통해 설명한 원리이며 모든 웨어하우스가 같은 내부 구조라는 뜻은 아니다.[^d-columnar][^d-distribution]

## 6. 느린 쿼리와 잘못된 보고서를 구분하기

성능 설계에서는 필터에 맞는 데이터 배치·정렬, 조인에 맞는 분산, 압축과 메모리·동시 실행을 함께 본다. 특정 키에 행이 몰리면 일부 작업만 오래 걸릴 수 있고, 많은 쿼리가 동시에 실행되면 실행 자체보다 큐에서 기다리는 시간이 커질 수 있다. Redshift의 테이블 설계 지침과 WLM은 이런 저장 배치와 실행 자원 관리를 각각 다룬다.[^d-design][^v-wlm]

쿼리 응답 시간은 대기 시간과 실행 시간을 분리하고 실행 계획·읽은 데이터 양·재분배·임시 공간·동시 작업을 연결한다. 쿼리가 느린 문제와 보고서 숫자가 다른 문제의 증거도 다르다. 숫자 불일치는 집계 단위·조인 중복·시간대·원천과 적재 시점·지표 정의를 먼저 비교할 수 있다. 적재 지연은 원천 수집·변환·적재 완료와 BI의 갱신 시점을 나누어 본다.

데이터를 더 자주 적재하면 최신성은 좋아질 수 있지만 수집·변환·쿼리와의 경합과 운영 비용이 늘 수 있다. 무조건 실시간을 목표로 잡기보다 업무가 허용하는 데이터 지연과 보고서 응답 시간을 정하고 그 조건에서 모델·연산·저장 비용을 선택한다. 분석 시스템 운영은 디스크를 크게 만드는 일보다 데이터 의미와 실행 경로를 함께 유지하는 일에 가깝다.



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

[^d-design]: [AWS Redshift — Table design best practices](https://docs.aws.amazon.com/redshift/latest/dg/c_designing-tables-best-practices.html)

[^d-grain]: [Microsoft — Star schema: fact tables and granularity](https://learn.microsoft.com/en-us/power-bi/guidance/star-schema)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Database 분류|Database]]
