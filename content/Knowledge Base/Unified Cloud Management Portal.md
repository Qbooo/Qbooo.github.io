---
title: Unified Cloud Management Portal
type: knowledge
domain: Cloud
tags:
  - Zadara
  - Management
aliases:
  - "통합 클라우드 관리 포털"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-05
---

## 1. 개요

Unified Cloud Management Portal은 클라우드 자원을 한곳에서 조회·생성·관리하는 웹 기반 관리 창구이다.

이 문서는 Zadara 환경에서 서비스 생성, 계정·권한 관리, 자원 상태 확인에 사용하는 통합 관리 화면을 다룬다. 이 표현은 별도 단일 제품명으로 확정한 것이 아니라 통합 관리 창구를 설명하는 학습 목록의 이름이다.[^1]

## 2. 통합 관리 화면이 필요한 이유

클라우드 자원은 VM·네트워크·볼륨·사용자처럼 서로 연결된 여러 요소로 이루어진다. 통합 포털은 각 요소를 따로 찾는 대신 공통 화면에서 필요한 자원을 만들고 상태를 확인하게 하는 접점이다. 관리 화면과 실제 자원은 분리되지만 같은 서비스 상태를 다룬다.[^1][^d-overview]

예를 들어 VM 목록에서 인스턴스를 선택해 연결된 볼륨과 네트워크를 확인하는 화면은 여러 자원의 관계를 사용자에게 보여 준다. CPU 사용률 같은 지표는 시간에 따른 측정값, 생성 완료 같은 이벤트는 발생 기록, 경보는 설정된 조건에 대한 알림이라는 차이가 있다.[^2]

포털·CLI·API는 사용자나 프로그램이 서비스를 다루는 서로 다른 인터페이스이다. 같은 관리 기능을 사용할 수 있는 부분도 있지만 화면의 모든 기능이 모든 API와 일대일로 대응한다고 가정해서는 안 된다. 통합이라는 말은 사용 창구의 통합을 뜻하며, 모든 실제 자원이 하나의 서버에서 실행된다는 뜻은 아니다.[^v-api]

## 3. 관리 화면과 실제 자원의 관계

포털은 관리 요청을 전달하는 인터페이스이다. 이는 자원에 관리 명령을 내리는 제어 경로와 실제 워크로드를 처리하는 데이터 경로를 구별하는 설명이다. 화면에 접속할 수 없다는 사실만으로 모든 VM이 중단되었다고 판단할 수는 없다. 반대로 특정 장애에서 서비스가 계속된다는 가용성 보장을 뜻하지도 않는다.

| 관리 영역 | 다루는 대상 |
|---|---|
| 자원 생성·변경 | VM, 네트워크, 볼륨 등 |
| 접근 통제 | 계정, 사용자, 역할과 정책 |
| 상태 확인 | 자원 사용량, 이벤트, 경보 |
| 서비스 관리 | 이용 가능한 서비스와 프로비저닝 상태 |

Zadara의 프로비저닝 포털과 개별 서비스의 관리 화면은 역할이 다를 수 있다. 하나의 화면에서 모든 서비스의 모든 기능을 제공한다고 일반화해서는 안 된다.[^1]

## 4. 화면·제어 서비스·실제 자원의 관계

사용자는 브라우저 화면을 보지만 클라우드 자원을 만드는 작업은 인증·권한·API·프로비저닝 기능과 실제 호스트·스토리지·네트워크가 함께 처리한다. 포털은 그 상태와 작업 창구를 제공한다. 자원 목록을 관리하는 인벤토리(inventory), 지표 등을 수집하는 텔레메트리(telemetry), VM이 실행되는 호스트와 데이터 경로는 서로 다른 기능이다. Zadara의 공개 개요도 서비스 프로비저닝·계정·프로젝트·가상 자원과 모니터링 기능을 나누어 설명한다.[^d-overview]

VM 생성의 일반적인 관리 흐름을 예로 들면 로그인 → 대상 Account·Project 선택 → 생성 요청 → 권한·할당량과 입력 조건 판정 → 실제 자원 배치·구성 → 상태 갱신으로 볼 수 있다. 이 일반 모델에서 스케줄링 기능은 실행할 호스트를 고르고, 가상화 계층은 VM을 실행하며, 네트워크·스토리지 기능은 NIC·연결·볼륨 같은 의존 자원을 준비한다. 이들은 API 요청에 대한 즉시 응답과 별도로 진행될 수 있다. 이는 클라우드 관리 원리를 설명하는 예시이며 Zadara 내부의 정확한 실행 순서나 모든 검증 항목이 공개되었다는 뜻은 아니다. 요청 접수와 VM 준비, VM 안의 애플리케이션 준비가 각각 다른 단계라는 점이 핵심이다.

```mermaid
flowchart LR
    A["브라우저 · 선택한 범위"] --> B["인증 · API · 관리 요청"]
    B --> C["컴퓨팅 · 네트워크 · 저장 자원 구성"]
    C --> D["상태 · 이벤트 · 지표"]
    D --> A
```

관리자 역할도 같은 화면의 보이는 메뉴만으로 정해지지 않는다. 제공자·테넌트 관리자·일반 사용자는 서로 다른 범위에 권한을 가질 수 있고, zCompute는 프로젝트별 사용자 역할·정책을 제공한다. 사용자에게 작업을 위임할 때에는 자원 범위와 가능한 행위를 함께 정해야 한다.[^d-project]

## 5. 계정 범위와 모니터링

동일한 포털에서도 [[Tenant|테넌트]]와 프로젝트, 사용자 권한에 따라 보이는 자원과 가능한 작업이 달라진다. 운영자는 현재 선택된 범위를 기준으로 상태를 해석해야 한다.

zCompute의 프로젝트 개요 화면은 VM·볼륨 개수와 상태, CPU·메모리·네트워크 사용량 등을 보여 준다.[^2] 프로젝트 전체의 합계와 개별 VM의 지표는 측정 범위가 다르다. 포털의 정상 표시만으로 애플리케이션 응답까지 정상이라고 단정할 수는 없다.

## 6. API와 자동화

웹 화면의 반복 작업은 [[Zadara Symphony API|Zadara API]], [[AWS CLI]], [[Terraform]] 같은 수단으로 자동화할 수 있다. 인터페이스가 달라도 실제 작업은 대상 서비스가 제공하는 권한과 기능 범위 안에서 이루어진다. API 지원 범위와 화면 기능의 일대일 대응은 이 문서에서 확인하지 않았다. 자동화는 개별 서비스의 문서화된 API를 기준으로 구성해야 한다.[^v-api]

## 7. 관련 문서

- [[Tenant]]
- [[Multi-tenancy]]
- [[Zadara Symphony API]]
- [[AWS CLI]]
- [[Terraform]]

- [[Server]]

- [[Networking]]

## 8. 출처

[^1]: [Zadara — Cloud Technical Overview](https://www.zadara.com/wp-content/uploads/Zadara-Cloud-Technical-Overview.pdf)
[^2]: [Zadara — Compute Project Overview](https://guides.zadara.com/cs-compute-guide/latest/compute-project-overview.html)

[^v-api]: [Zadara — AWS API supported parameter tables](https://guides.zadara.com/cs-compute-guide/latest/aws-ec2-api.html)

[^d-overview]: [Zadara — Cloud Services Overview](https://guides.zadara.com/cs-compute-guide/latest/cloud-services-overview.html)

[^d-project]: [Zadara — Projects and roles](https://guides.zadara.com/cs-compute-guide/latest/iam-projects.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
