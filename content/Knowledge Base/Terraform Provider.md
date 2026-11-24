---
title: Terraform Provider
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
- [[#요구사항과 연결 설정]]
- [[#호환성 검증]]
- [[#관련 문서]]

## 개요

Terraform Provider는 Terraform과 클라우드 또는 외부 서비스 API를 연결하는 플러그인이다. Provider에는 사용할 수 있는 리소스 유형, 인자와 속성의 스키마, 원격 객체를 읽고 변경하는 동작이 들어 있다. [HashiCorp Provider 요구사항](https://developer.hashicorp.com/terraform/language/providers/requirements)

## 요구사항과 연결 설정

`required_providers`는 플러그인 소스와 버전 제약을 선언한다. `terraform init`이 선택한 버전과 체크섬은 `.terraform.lock.hcl`에 기록된다. `provider` 블록은 설치한 플러그인이 사용할 리전, 인증 방법과 API 주소 같은 연결 설정을 지정한다. CLI 명령 그룹과 Terraform Provider 설정 필드의 이름은 각 도구의 스키마에서 확인한다. [HashiCorp 의존성 잠금 파일](https://developer.hashicorp.com/terraform/language/files/dependency-lock)

## 호환성 검증

AWS 호환 API에 CLI 조회가 성공하면 해당 조회 작업의 응답을 확인한 것이다. Terraform의 생성·변경까지 확인하려면 선택한 Provider 버전의 스키마, 대상 플랫폼의 작업별 API 지원, Plan과 Apply 결과를 함께 봐야 한다. 예를 들어 검증이 통과한 VM 인자라도 플랫폼 API가 지원하지 않으면 Apply에서 거절될 수 있다. [HashiCorp 리소스 구성](https://developer.hashicorp.com/terraform/language/resources/configure) · [Zadara AWS 호환 API](https://guides.zadara.com/cs-compute-guide/latest/aws-ec2-api.html)

## 관련 문서

- [[Terraform Core]]
- [[AWS CLI]]
- [[Zadara Symphony API]]

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[IaC 분류]]
