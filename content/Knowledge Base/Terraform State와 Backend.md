---
title: Terraform State와 Backend
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
- [[#코드와 State의 역할]]
- [[#저장과 잠금]]
- [[#관련 문서]]

## 개요

Terraform State는 구성의 리소스 주소를 실제 원격 객체와 연결해 기록하는 데이터다. Backend는 그 State를 어디에 저장하고 어떻게 다룰지 정한다. State가 있으면 Terraform은 다음 실행에서 관리 중인 VPC나 VM을 식별할 수 있다. [HashiCorp State의 역할](https://developer.hashicorp.com/terraform/language/state/purpose) · [HashiCorp Backend](https://developer.hashicorp.com/terraform/language/backend)

## 코드와 State의 역할

코드는 앞으로 원하는 구성을 표현하고, State는 Terraform이 이미 관리하는 객체의 ID와 기록된 속성을 담는다. State에 기록된 값은 마지막 조회 이후 실제와 달라질 수 있다. 복사한 코드로 다시 배포하는 것은 새 객체를 만들 수 있는 구성의 재사용이고, 기존 객체를 계속 관리하려면 해당 State를 유지하거나 Import로 연결한다. [HashiCorp State의 역할](https://developer.hashicorp.com/terraform/language/state/purpose)

## 저장과 잠금

기본 로컬 Backend는 State를 로컬 파일로 저장한다. 공유 Backend는 팀이 같은 State를 사용하게 할 수 있으며, 잠금은 지원하는 Backend에서 동시 변경을 조정한다. State에는 인증 자료나 민감한 속성이 포함될 수 있어 접근을 제한한다. `.terraform.lock.hcl`은 Provider 선택 정보를 담는 별도의 파일이다. [HashiCorp Backend](https://developer.hashicorp.com/terraform/language/backend) · [HashiCorp State 잠금](https://developer.hashicorp.com/terraform/language/state/locking)

## 관련 문서

- [[Terraform Plan과 Apply]]
- [[Terraform Import]]
- [[구성 드리프트]]

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[IaC 분류]]
