---
title: Terraform Plan과 Apply
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
- [[#계획의 읽는 순서]]
- [[#적용과 검증]]
- [[#관련 문서]]

## 개요

Terraform Plan은 원하는 구성으로 맞추기 위해 어떤 변경이 필요한지 계산한 결과이고, Apply는 변경 계획을 실행하는 단계다. 포맷 검사, 구성 검증, 계획, 실제 서비스 테스트는 서로 다른 실패를 찾는다. [HashiCorp Plan](https://developer.hashicorp.com/terraform/cli/commands/plan) · [HashiCorp Apply](https://developer.hashicorp.com/terraform/cli/commands/apply)

## 계획의 읽는 순서

먼저 리소스별 생성·제자리 변경·교체·삭제를 확인하고, 다음으로 변경 내역에서 적용 후 남을 값과 대상 범위를 확인한다. 예를 들어 같은 보안 그룹의 SSH 송신 규칙에 DB CIDR을 하나 추가하면, Plan에서 이전 중첩 블록 전체가 빠지고 새 블록 전체가 추가되는 형태로 보일 수 있다. 리소스가 `~ update in-place`라면 그룹 자체는 유지하는 계획이다. 중첩 목록의 기호만으로 Provider의 세부 API 호출 순서를 판단할 수는 없다. [HashiCorp Plan 실습](https://developer.hashicorp.com/terraform/tutorials/cli/plan)

## 적용과 검증

`terraform plan -out=change.tfplan`으로 저장하고 `terraform apply change.tfplan`을 실행하면 검토한 계획을 입력으로 사용한다. 저장된 계획에는 민감한 값이 들어갈 수 있다. 적용이 중간에 실패하면 성공한 리소스가 남을 수 있으므로 오류와 State를 확인하고 계획을 다시 계산한다. 적용 뒤 SSH, DNS, HTTP, DB 쿼리 같은 실제 통신을 확인하면 서비스 동작까지 검증할 수 있다. [HashiCorp Plan](https://developer.hashicorp.com/terraform/cli/commands/plan) · [HashiCorp Apply](https://developer.hashicorp.com/terraform/cli/commands/apply)

## 관련 문서

- [[Terraform State와 Backend]]
- [[구성 드리프트]]
- [[Terraform Core]]

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[IaC 분류]]
