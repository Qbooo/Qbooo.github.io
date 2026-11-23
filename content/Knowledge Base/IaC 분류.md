---
title: IaC 분류
type: map
domain: IaC
created: 2026-11-23
updated: 2026-11-23
---

## 목차

- [[#분야 소개|분야 소개]]
- [[#학습 지도|학습 지도]]
- [[#문서 목록|문서 목록]]
- [[#예정 문서|예정 문서]]

## 분야 소개

Infrastructure as Code(IaC)의 원리와 이를 구현하는 도구, 인프라 구성·변경·상태 관리에 필요한 개념을 정리합니다. 특정 클라우드나 제품의 사용 절차보다 반복 가능하고 검토 가능한 인프라 관리가 어떻게 이루어지는지를 중심으로 연결합니다.

## 학습 지도

```text
Infrastructure as Code
├─ IaC
│  ├─ 선언형 구성
│  ├─ 원하는 상태
│  └─ 구성 드리프트
├─ Terraform
│  ├─ Terraform Core
│  ├─ Provider
│  ├─ Resource와 Data Source
│  ├─ Module
│  ├─ Plan과 Apply
│  ├─ State와 Backend
│  └─ Import
└─ 관련 자동화 인터페이스
   ├─ API
   ├─ CLI
   └─ 구성 관리 도구
```

이 지도는 학습 순서와 문서 확장 방향을 함께 나타냅니다. 독립적으로 설명할 내용이 충분해지면 별도 문서로 만들고, 짧은 문법이나 명령 설명은 상위 개념 문서에 포함합니다.

## 문서 목록

- [[Infrastructure as Code|IaC]]: 인프라 구성을 실행 가능한 파일로 정의해 반복 적용·검토·변경 관리하는 방식
- [[Terraform]]: 선언형 구성, Provider와 API, Plan·Apply, State를 이용해 인프라 생명주기를 관리하는 IaC 도구
- [[AWS CLI]]: AWS 호환 API의 개별 작업을 명령줄에서 호출하는 인터페이스
- [[Zadara Symphony API]]: zCompute의 관리 API와 AWS 호환 API, 인증 방식과 자동화 도구의 연결
- [[Shell Script|셸 스크립트]]: 명령과 제어 흐름을 조합해 반복 작업을 자동화하는 방식

## 예정 문서

- 선언형 구성과 원하는 상태
- 구성 드리프트
- Terraform Core
- Terraform Provider
- Terraform Resource와 Data Source
- Terraform Module
- Terraform Plan과 Apply
- Terraform State와 Backend
- Terraform Import
- 구성 관리 도구

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
