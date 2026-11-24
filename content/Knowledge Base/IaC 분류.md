---
title: IaC 분류
type: map
domain: IaC
created: 2026-11-23
updated: 2026-11-24
---

## 목차

- [[#분야 소개|분야 소개]]
- [[#문서 목록|문서 목록]]
- [[#예정 문서|예정 문서]]

## 분야 소개

Infrastructure as Code(IaC)의 원리와 이를 구현하는 도구, 인프라 구성·변경·상태 관리에 필요한 개념을 정리합니다. 특정 클라우드나 제품의 사용 절차보다 반복 가능하고 검토 가능한 인프라 관리가 어떻게 이루어지는지를 중심으로 연결합니다.

## 문서 목록

### IaC의 원리

- [[Infrastructure as Code|Infrastructure as Code (IaC)]]
  - [[선언형 구성과 원하는 상태]] — 코드가 표현하는 목표와 리소스 연결
  - [[구성 드리프트]] — 코드, State, 실제 설정 사이의 차이

### Terraform

- [[Terraform]]
  - **구성과 실행**
    - [[Terraform Core]] — 구성 해석과 의존성 그래프
    - [[Terraform Provider]] — 버전, 스키마, 서비스 API 호환성
    - [[Terraform Resource와 Data Source]] — 객체 관리, 외부 조회, 참조와 입력값
  - **변경과 상태 관리**
    - [[Terraform Plan과 Apply]] — 변경 범위 검토와 적용
    - [[Terraform State와 Backend]] — 객체 연결 정보, 저장과 잠금
    - [[Terraform Import]] — 기존 객체를 Terraform 관리 대상으로 편입

### VM 초기화와 자동화 도구

- **VM 초기화**
  - [[cloud-init]] — 사용자 데이터로 게스트 운영체제의 초기 계정과 SSH 설정 적용
- **API·CLI·스크립트**
  - [[Zadara Symphony API]] — zCompute 관리 API와 AWS 호환 API의 관계
  - [[AWS CLI]] — AWS 호환 API 호출
  - [[Shell Script|셸 스크립트]] — 명령을 조합한 반복 작업 자동화

## 예정 문서

- Terraform Module
- 구성 관리 도구

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
