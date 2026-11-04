---
title: Tenant
type: knowledge
domain: Cloud
tags:
  - Cloud
  - IAM
aliases:
  - "테넌트"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-04
---

## 1. 개요

Tenant(테넌트)는 하나의 서비스 안에서 사용자·데이터·설정·자원에 대한 독립된 관리와 격리가 필요한 고객 또는 조직 단위이다.

테넌트는 사람 한 명이나 VM 한 대와 동일한 개념이 아니다.[^1]

## 2. 사용자와 자원 사이의 관리 단위

한 조직에는 여러 사용자가 있고, 사용자는 역할에 따라 자원에 접근한다. 테넌트는 이때 누구의 자원인지와 어느 범위에서 권한을 행사할 수 있는지를 구분하는 기준이 된다.

예를 들어 하나의 서비스에 고객 A와 고객 B가 있다면 두 조직의 사용자가 같은 로그인 화면을 사용하더라도 서로의 데이터에는 접근할 수 없어야 한다. 같은 조직 안의 개발·운영 환경은 프로젝트 등 추가 관리 단위로 나눌 수 있다. 이 예시는 논리적 관계를 설명한 것이며 제품마다 이름과 계층은 다르다.

## 3. Zadara의 Account와 Project

zCompute는 Account → Project → 자원의 관리 계층을 사용한다. Account에 속한 사용자는 할당된 Project에 접근하며, 역할·정책에 따라 볼 수 있거나 관리할 수 있는 자원이 달라진다.[^2] 프로젝트에 사용자를 할당할 때 Member·Tenant Admin 역할과 정책을 설정하는 구체적인 관리 기능이 제공된다.[^v-project]

Tenant는 서비스 격리의 일반 개념이고 Account와 Project는 제품이 제공하는 구체적인 관리 단위이다. 다른 클라우드의 tenant·account·subscription을 이름만 보고 일대일로 대응시키면 안 된다. 예를 들어 Azure의 Entra 테넌트는 신원 디렉터리이며 한 테넌트를 여러 구독이 신뢰할 수 있다. 구독은 테넌트와 별개의 관리 단위이다.[^v-entra]

## 4. 인증과 격리

![인증된 신원, 허용된 역할과 작업, 자원의 테넌트 범위를 서버에서 함께 검증하는 테넌트 격리 개념도](../assets/tenant-isolation.png)

BOwiki 자체 제작 개념도. 요청에 표시된 테넌트 이름만 믿는 것이 아니라 신원·작업 권한·자원 범위를 확인하는 논리적 경계를 설명한다. 모든 테넌트가 물리 전용 장비를 사용한다는 의미는 아니다.[^v-isolation]

인증은 사용자의 신원을 확인하고, 권한 부여는 허용된 작업을 결정한다. 테넌트 격리는 그 작업이 해당 테넌트의 자원에만 적용되도록 경계를 유지하는 것이다. 로그인에 성공한 사용자라도 다른 테넌트의 자원을 읽으면 격리에 실패한 것이다.[^1]

## 5. 한 조직에서 자원까지 이어지는 예시

가상의 서비스 제공자가 연구 조직 A와 B에 클라우드를 제공한다고 하자. A의 여러 구성원은 개발·운영 프로젝트에 서로 다른 역할로 참여하고, 각 프로젝트는 VM·네트워크·볼륨을 가진다. 사용자 한 명은 신원의 단위, 프로젝트는 작업을 묶는 단위, 정책은 허용되는 행위의 규칙, 테넌트는 고객 조직의 자원 경계를 나타낸다. 이는 제품 중립적인 예시이며 실제 zCompute에서는 Account·Project·사용자·역할·정책이 이런 관계를 구체화한다.[^2][^v-project]

요청 처리에서는 먼저 신원을 확인하고 그 사용자가 어떤 테넌트·프로젝트의 맥락에서 작업하는지 정한 다음, 대상 자원이 그 범위에 있는지와 요청한 행위가 허용되는지를 검사한다. URL에 적힌 자원 ID가 유효하다는 사실만으로 접근을 허용하면 다른 고객의 자원을 조회하는 문제가 생길 수 있다. AWS의 격리 설명도 인증·권한 부여와 테넌트 자원 범위의 검증을 분리한다.[^v-isolation]

```mermaid
flowchart LR
    A["요청의 신원 확인"] --> B["테넌트 · 프로젝트 맥락"]
    B --> C["자원 소유 범위와 작업 권한"]
    C --> D["허용된 자원에만 실행"]
```

이 그림은 검증 관점을 설명하며 특정 제품 내부의 정확한 실행 순서를 고정한 것은 아니다.

## 6. 권한·할당량·청구가 다른 경계인 이유

권한은 할 수 있는 작업을 정하고 할당량은 사용할 수 있는 자원의 양을 제한한다. VM 생성 권한이 있어도 프로젝트의 vCPU 한도에 도달하면 새 VM 생성이 제한될 수 있다. 청구는 사용량을 누구에게 귀속할지의 문제이고 데이터 격리는 누구의 정보를 볼 수 있는지의 문제다. 하나의 Account가 이 기능의 공통 기준으로 쓰이더라도 각 판정의 목적은 다르다.[^2]

사용자가 퇴사하면 로그인 계정뿐 아니라 발급된 토큰·자동화 계정·역할 위임의 수명도 고려해야 한다. 프로젝트 이동은 기존 자원 소유권과 사용자의 권한을 함께 바꾸는지 구분해야 하며, 공통 운영자가 여러 테넌트를 지원하는 경우에는 명시적으로 부여된 운영 범위와 고객별 일반 권한을 나누어야 한다. 모든 교차 테넌트 접근이 무조건 금지라는 단순 규칙보다 허용된 관리 경로와 감사 가능한 위임을 구체화하는 것이 중요하다.

로그인은 되지만 VM이 보이지 않는다면 선택한 Account·Project와 조회 권한을, 생성이 거부되면 권한과 할당량·자원 가용성을 각각 해석한다. 다른 고객 자료가 보이는 문제는 단순 화면 오류가 아니라 소유 범위 검증의 실패일 수 있다. 이런 분류는 고객·사용자·프로젝트·정책·자원이라는 서로 다른 개념을 분리해야 가능한 운영 판단이다.

## 7. 관련 문서

- [[Multi-tenancy]]
- [[Unified Cloud Management Portal]]
- [[Zadara Symphony API]]
- [[Microsoft Azure]]

- [[Server]]

## 8. 출처

[^1]: [AWS — SaaS Architecture Fundamentals: Tenant isolation](https://d1.awsstatic.com/onedam/marketing-channels/website/aws/en_US/events/approved/documents/saas-architecture-fundamentals-whitepaper.pdf)
[^2]: [Zadara — Accounts](https://guides.zadara.com/cs-compute-guide/latest/iam-accounts.html)

[^v-project]: [Zadara — Projects](https://guides.zadara.com/cs-compute-guide/latest/iam-projects.html)

[^v-entra]: [Microsoft — Subscription and Entra directory relationship](https://learn.microsoft.com/en-us/entra/fundamentals/how-subscriptions-associated-directory)

[^v-isolation]: [AWS — Tenant isolation](https://docs.aws.amazon.com/whitepapers/latest/saas-architecture-fundamentals/tenant-isolation.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
