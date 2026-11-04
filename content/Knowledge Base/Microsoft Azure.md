---
title: Microsoft Azure
type: knowledge
domain: Cloud
tags:
  - Microsoft
  - Cloud
aliases:
  - "애저 서비스"
  - "Azure"
  - "마이크로소프트 애저"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-04
---

## 1. 개요

Microsoft Azure는 Microsoft가 제공하는 클라우드 플랫폼이다.

컴퓨팅·네트워크·스토리지·데이터베이스·컨테이너·AI 등의 서비스를 조합해 애플리케이션을 구축하고 운영한다. 목록의 “애저 서비스”를 이 문서에서 다룬다.[^1]

## 2. 주요 서비스 영역

| 영역 | 서비스 예시 | 역할 |
|---|---|---|
| 컴퓨팅 | Azure Virtual Machines | VM 실행 |
| 네트워크 | Azure Virtual Network, Load Balancer | 가상 네트워크와 트래픽 분산 |
| 스토리지 | Managed Disks, Azure Files, Blob Storage | 블록·파일·오브젝트 저장 |
| 컨테이너 | Azure Kubernetes Service(AKS) | 관리형 Kubernetes |
| 데이터베이스 | Azure SQL Database | 관리형 관계형 DB |
| 운영 | Azure Monitor | 지표·로그 관측 |

같은 분야라도 서비스별 지원 기능과 관리 범위가 다르다.[^1]

## 3. 관리 단위와 접근 권한

Azure 자원은 관리 그룹, 구독(Subscription), 리소스 그룹(Resource Group), 개별 리소스로 조직할 수 있다. 구독은 청구와 관리의 중요한 경계이고 리소스 그룹은 관련 자원을 묶는다.[^2]

Microsoft Entra 테넌트는 신원·디렉터리의 맥락을 가진다.[^4] 각 구독은 하나의 Entra 디렉터리를 신뢰하며, 한 디렉터리는 여러 구독에서 사용될 수 있다.[^v-directory] [[Tenant|테넌트]]를 VM이나 VNet과 동일하게 생각하면 안 된다. 다른 클라우드의 Account·Project와 비교할 때도 이름보다 권한·청구·자원 범위를 기준으로 본다.

## 4. 서비스 모델과 책임

IaaS는 VM처럼 인프라 자원을 제공하고, PaaS는 애플리케이션 실행·데이터 처리 플랫폼의 일부 운영을 제공자가 맡는다. 관리형 서비스를 사용해도 사용자의 데이터와 접근 설정에 대한 책임이 모두 사라지는 것은 아니다.[^3] Microsoft의 공유 책임 표는 IaaS의 운영체제 관리와 PaaS의 운영체제 관리를 구분하면서, 데이터·신원·사용자가 통제하는 설정에 대한 책임을 별도로 설명한다. 이 표는 운영·거버넌스 지침이며 개별 계약의 법적 책임을 대체하지 않는다.[^v-shared]

[[VPSA]]와 비교할 때는 Managed Disks·Azure Files 같은 접근 목적을 구분하고, [[Object Storage]]는 Blob Storage와 비교할 수 있다. Azure Files는 SMB·NFS 파일 공유, Blob은 오브젝트 저장, Managed Disks는 VM용 디스크라는 서로 다른 접근 목적을 가진다. 목적이 비슷해도 VPSA·S3와 API나 인증·이동 절차까지 같다는 의미는 아니다.[^v-storage]

## 5. 데이터센터 자원과 관리 단위의 관계

Azure VM은 물리 호스트의 가상화 위에서 실행되고, 리전·가용 영역은 서비스 배치와 장애 범위를 이해하는 기준이다. VNet·subnet·가상 NIC는 네트워크의 논리 구성이고 Managed Disk는 VM의 영속 저장 자원이다. Entra 테넌트·구독·리소스 그룹은 신원·관리·청구·수명 주기를 조직하는 단위로, 물리 랙이나 네트워크 케이블과 일대일로 대응하지 않는다.[^d-vm][^d-vnet]

가상의 웹 서비스 경로를 생각하면 인터넷 요청 → Public IP·Load Balancer → VM NIC와 네트워크 보안 규칙 → OS의 수신 서비스 → 애플리케이션 → DB·스토리지로 연결할 수 있다. 실제 경로에는 서비스 종류에 따라 게이트웨이·프록시·Private Endpoint 등이 더해질 수 있다. 이는 구성 예시이며 모든 Azure 서비스가 같은 경로를 사용한다는 뜻은 아니다.

Entra 인증은 누구인지, Azure RBAC는 관리 작업을 어느 범위에서 할 수 있는지, NSG·라우팅·OS 방화벽은 패킷이 어떻게 전달·허용되는지, 애플리케이션 권한은 서비스 데이터에 무엇을 할 수 있는지를 다룬다. 구독에서 VM 생성 권한이 있다고 그 VM의 모든 애플리케이션에 로그인할 수 있는 것은 아니다. 관리 권한과 데이터 접근 경계를 별도로 읽어야 한다.

## 6. 운영 책임과 장애 범위의 선택

VM IaaS에서는 게스트 OS·미들웨어·앱 운영을 사용자가 더 많이 맡고, App Service·Azure SQL 같은 PaaS는 제공자가 관리하는 실행 계층이 넓어진다. AKS도 관리형 Kubernetes라는 이름만으로 워크로드·이미지·권한·네트워크·데이터 운영이 모두 사라지는 것은 아니다. 서비스별 책임 표와 실제 구성 범위를 함께 이해해야 한다.[^v-shared]

가용 영역 분산·중복 저장·스냅샷·백업은 서로 다른 장애와 복구 요구를 다룬다. VM 두 대를 다른 영역에 배치해도 공통 DB·네트워크·인증 의존성이 단일 장애 지점이면 전체 서비스 품질이 영향을 받을 수 있다. 저장 중복성은 사용자 오삭제의 과거 상태를 복구하는 정책과 다르므로 SLO·RPO·RTO와 비용을 연결해 선택한다.

VM이 running이지만 접속이 안 되면 Public IP·라우팅·NSG·게스트 방화벽·프로세스 수신을 나누고, AuthorizationFailed는 실행 신원·RBAC 범위, 디스크 부족은 VM 내부 파일시스템과 저장 용량, HTTP 앱 오류는 프로세스·설정·DB 같은 의존성으로 가설을 좁힌다. Activity Log는 자원 생성·변경 등의 관리 작업을 기록하며, 애플리케이션의 모든 데이터 요청 로그를 대신하지 않는다.[^d-activity] 지표는 자원 사용 추이를 보여 주고 Resource Health는 지원되는 자원의 현재·과거 상태를 서비스 신호로 평가한다. Available·Unavailable·Degraded·Unknown 같은 판정의 의미도 단순 애플리케이션 성공·실패와 구분한다.[^d-health] 이 신호들을 작업·장애 시각에 맞춰 연결하면 “Azure가 안 된다”는 표현을 구체적인 영향과 원인 범위로 바꿀 수 있다.[^d-monitor]



## 7. 관련 문서

- [[Tenant]]
- [[VPSA]]
- [[Block Storage]]
- [[File Storage]]
- [[Object Storage]]

- [[Server]]

- [[Networking]]

- [[3-Tier Architecture]]

## 8. 출처

[^1]: [Microsoft — Azure products](https://azure.microsoft.com/en-us/products/)
[^2]: [Microsoft Learn — Organize your Azure resources](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-setup-guide/organize-resources)
[^3]: [Microsoft Learn — Describe cloud service types](https://learn.microsoft.com/en-us/training/modules/describe-cloud-service-types/)
[^4]: [Microsoft Learn — What is Microsoft Entra?](https://learn.microsoft.com/en-us/entra/fundamentals/what-is-entra)

[^v-directory]: [Microsoft — Entra tenant and subscriptions](https://learn.microsoft.com/en-us/entra/fundamentals/how-subscriptions-associated-directory)

[^v-shared]: [Microsoft — Shared responsibility in the cloud](https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility)

[^v-storage]: [Microsoft — Azure Storage introduction](https://learn.microsoft.com/en-us/azure/storage/common/storage-introduction)

[^d-vm]: [Microsoft — Azure virtual machines](https://learn.microsoft.com/en-us/azure/virtual-machines/overview)

[^d-vnet]: [Microsoft — Virtual Network overview](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)

[^d-monitor]: [Microsoft — Azure Monitor overview](https://learn.microsoft.com/en-us/azure/azure-monitor/overview)

[^d-activity]: [Microsoft — Azure Monitor Activity Log](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/activity-log)

[^d-health]: [Microsoft — Resource Health overview](https://learn.microsoft.com/en-us/azure/service-health/resource-health-overview)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
