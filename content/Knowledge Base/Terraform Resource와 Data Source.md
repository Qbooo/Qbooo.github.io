---
title: Terraform Resource와 Data Source
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
- [[#리소스 주소와 참조]]
- [[#입력값과 파일]]
- [[#관련 문서]]

## 개요

Resource는 Terraform이 생명주기를 관리하는 객체의 선언이고, Data Source는 Provider에서 외부 데이터를 조회하는 선언이다. 콘솔에서 만든 이미지를 Data Source로 찾아 새 VM의 입력값으로 사용할 수 있다. 그 이미지에 대한 관리 권한은 별도의 Resource와 Import를 통해 맡긴다. [HashiCorp 리소스 구성](https://developer.hashicorp.com/terraform/language/resources/configure) · [HashiCorp Data Source](https://developer.hashicorp.com/terraform/language/data-sources)

## 리소스 주소와 참조

`resource` 뒤의 첫 문자열은 유형, 두 번째 문자열은 Terraform 안에서 쓰는 이름이다. 예를 들어 `aws_subnet.db.id`는 `aws_subnet` 유형의 `db` 리소스가 가진 ID를 참조한다. 이 표현식 전체를 따옴표로 감싸면 리소스 참조 대신 그 글자 그대로의 문자열이 된다. Terraform 주소와 콘솔의 Name 태그는 각각 관리 주소와 표시 이름이다. [HashiCorp 참조 표현식](https://developer.hashicorp.com/terraform/language/expressions/references)

## 입력값과 파일

`variable` 블록에는 이름과 타입, 필요하면 기본값을 선언한다. `terraform.tfvars`에는 `admin_cidr = "203.0.113.10/32"`처럼 값을 할당한다. 리소스 안에서는 `var.admin_cidr`로 참조한다. 여기의 IP는 문서용 예시다. [HashiCorp 입력 변수](https://developer.hashicorp.com/terraform/language/values/variables)

## 관련 문서

- [[Terraform Core]]
- [[Terraform Import]]
- [[cloud-init]]

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[IaC 분류]]
