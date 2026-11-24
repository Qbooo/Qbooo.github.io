---
title: Terraform Core
type: knowledge
domain: IaC
tags: [IaC, Terraform]
aliases: []
parent: IaC 분류
status: growing
lang: ko
featured: false
created: 2026-11-24
updated: 2026-11-24
---

## 목차

- [[#개요]]
- [[#의존성 그래프]]
- [[#값을 알 수 있는 시점]]
- [[#관련 문서]]

## 개요

Terraform Core는 구성을 해석하고 리소스 간 의존성을 계산해 변경 계획과 적용을 조율하는 Terraform의 실행 엔진이다. 서비스별 객체의 조회와 변경은 Provider가 구현한다. [HashiCorp 의존성 그래프](https://developer.hashicorp.com/terraform/internals/graph)

## 의존성 그래프

예를 들어 라우팅 테이블이 NAT 게이트웨이의 ID를 참조하고, 서브넷 연결이 라우팅 테이블 ID를 참조하면 Core는 이 관계를 그래프로 구성한다. 앞선 객체의 생성 결과를 알아야 다음 객체의 인자를 결정할 수 있다. 서로 의존하지 않는 작업은 그래프 범위에서 병렬 처리할 수 있다. [HashiCorp 리소스 구성](https://developer.hashicorp.com/terraform/language/resources/configure)

## 값을 알 수 있는 시점

새 VM의 ID처럼 원격 서비스가 생성 시 정하는 값은 Plan에서 `known after apply`로 표시될 수 있다. Core는 값이 필요한 후속 작업과 참조를 유지하고, Apply에서 Provider가 반환한 값을 사용한다. [HashiCorp Plan](https://developer.hashicorp.com/terraform/cli/commands/plan)

## 관련 문서

- [[Terraform Provider]]
- [[Terraform Resource와 Data Source]]
- [[Terraform Plan과 Apply]]

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[IaC 분류]]
