---
title: Terraform Import
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
- [[#두 가지 진입 방식]]
- [[#가져온 뒤 확인]]
- [[#관련 문서]]

## 개요

Terraform Import는 이미 만들어진 원격 객체를 Terraform 리소스 주소에 연결해 관리 대상으로 받아들이는 기능이다. 예를 들어 콘솔에서 생성한 VM의 ID를 Terraform 주소와 연결하면 이후 계획에서 해당 VM의 변경을 함께 검토할 수 있다. [HashiCorp Import](https://developer.hashicorp.com/terraform/cli/import)

## 두 가지 진입 방식

`terraform import 주소 ID`는 기존 객체를 State에 연결한다. 해당 주소에 대응하는 리소스 구성을 준비한다. 선언형 `import` 블록을 쓰면 가져오기 의도를 코드에 남기고 Plan에서 검토할 수 있다. 지원되는 경우 `plan -generate-config-out=generated.tf`로 초기 HCL을 생성할 수 있다. [HashiCorp 구성 생성](https://developer.hashicorp.com/terraform/language/import/generating-configuration)

## 가져온 뒤 확인

생성된 코드는 검토의 출발점이다. 가져온 VPC와 서브넷을 코드로 정리할 때는 기존 ID와 리소스 참조가 의도대로 연결되는지 본다. Import 후 일반 Plan에 원치 않는 갱신이나 교체가 나오면 구성의 인자를 실제 객체와 대조한다. 같은 원격 객체는 하나의 Terraform 주소에만 연결한다. [HashiCorp Import](https://developer.hashicorp.com/terraform/cli/import)

## 관련 문서

- [[Terraform State와 Backend]]
- [[Terraform Resource와 Data Source]]
- [[Terraform Plan과 Apply]]

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[IaC 분류]]
