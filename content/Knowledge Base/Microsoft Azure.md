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
updated: 2026-11-05
---

## 1. 개요

Microsoft Azure는 Microsoft가 제공하는 클라우드 플랫폼이다.

컴퓨팅·네트워크·스토리지·데이터베이스·컨테이너·AI 등의 서비스를 조합해 애플리케이션을 구축하고 운영한다. 목록의 “애저 서비스”를 이 문서에서 다룬다.[^1]

## 2. 물리 인프라를 클라우드 서비스로 사용하는 방식

직접 서버를 마련하면 장비·OS·네트워크·스토리지를 구성하고 그 위에 애플리케이션을 설치한다. Azure에서는 이 기반의 일부 또는 더 넓은 범위를 서비스로 이용할 수 있다. VM은 가상 컴퓨터를, 관리형 DB는 데이터베이스 기능을, Blob Storage는 오브젝트 저장 기능을 제공하는 식이다.[^1][^v-storage]

예를 들어 같은 웹 애플리케이션도 VM에 직접 실행 환경을 설치하거나 App Service 같은 플랫폼에 배포할 수 있다. 어느 쪽이든 프로그램은 실제 연산·메모리·네트워크·저장 자원을 사용하지만, 사용자가 직접 관리하는 범위가 달라진다.[^v-shared]

테넌트·구독·리소스 그룹은 이런 서비스를 누구의 권한과 비용, 관리 범위에 묶을지 정하는 논리 단위이다. 리전과 가용 영역은 배치 위치와 장애 범위를 설명한다. Azure를 이해한다는 것은 서비스 이름을 외우는 것뿐 아니라 물리 자원, 서비스 기능, 관리 단위가 어떤 관계인지 구분하는 것이다.

## 3. 주요 서비스 영역

| 영역 | 서비스 예시 | 역할 |
|---|---|---|
| 컴퓨팅 | Azure Virtual Machines | VM 실행 |
| 네트워크 | Azure Virtual Network, Load Balancer | 가상 네트워크와 트래픽 분산 |
| 스토리지 | Managed Disks, Azure Files, Blob Storage | 블록·파일·오브젝트 저장 |
| 컨테이너 | Azure Kubernetes Service(AKS) | 관리형 Kubernetes |
| 데이터베이스 | Azure SQL Database | 관리형 관계형 DB |
| 운영 | Azure Monitor | 지표·로그 관측 |

같은 분야라도 서비스별 지원 기능과 관리 범위가 다르다.[^1]

## 4. 데이터센터 자원과 관리 단위의 관계

Azure VM은 물리 호스트의 가상화 위에서 실행되고, 리전·가용 영역은 서비스 배치와 장애 범위를 이해하는 기준이다. VNet·subnet·가상 NIC는 네트워크의 논리 구성이고 Managed Disk는 VM의 영속 저장 자원이다. Entra 테넌트·구독·리소스 그룹은 신원·관리·청구·수명 주기를 조직하는 단위로, 물리 랙이나 네트워크 케이블과 일대일로 대응하지 않는다.[^d-vm][^d-vnet]

가상의 웹 서비스 경로를 생각하면 인터넷 요청 → Public IP·Load Balancer → VM NIC와 네트워크 보안 규칙 → OS의 수신 서비스 → 애플리케이션 → DB·스토리지로 연결할 수 있다. 실제 경로에는 서비스 종류에 따라 게이트웨이·프록시·Private Endpoint 등이 더해질 수 있다. 이는 구성 예시이며 모든 Azure 서비스가 같은 경로를 사용한다는 뜻은 아니다.

Entra 인증은 누구인지, Azure RBAC는 관리 작업을 어느 범위에서 할 수 있는지, NSG·라우팅·OS 방화벽은 패킷이 어떻게 전달·허용되는지, 애플리케이션 권한은 서비스 데이터에 무엇을 할 수 있는지를 다룬다. 구독에서 VM 생성 권한이 있다고 그 VM의 모든 애플리케이션에 로그인할 수 있는 것은 아니다. 관리 권한과 데이터 접근 범위를 별도로 확인해야 한다.

## 5. 관리 단위와 접근 권한

Azure 자원은 관리 그룹, 구독(Subscription), 리소스 그룹(Resource Group), 개별 리소스로 조직할 수 있다. 구독은 청구와 관리의 중요한 경계이고 리소스 그룹은 관련 자원을 묶는다.[^2]

Microsoft Entra 테넌트는 사용자·애플리케이션의 신원을 관리하는 디렉터리 단위이다.[^4] 각 구독은 하나의 Entra 디렉터리를 신뢰하며, 한 디렉터리는 여러 구독에서 사용될 수 있다.[^v-directory] [[Tenant|테넌트]]를 VM이나 VNet과 동일하게 생각하면 안 된다. 다른 클라우드의 Account·Project와 비교할 때도 이름보다 권한·청구·자원 범위를 기준으로 본다.

## 6. 서비스 모델과 책임

IaaS는 VM처럼 인프라 자원을 제공하고, PaaS는 애플리케이션 실행·데이터 처리 플랫폼의 일부 운영을 제공자가 맡는다. 관리형 서비스를 사용해도 사용자의 데이터와 접근 설정에 대한 책임이 모두 사라지는 것은 아니다.[^3] Microsoft의 공유 책임 표는 IaaS의 운영체제 관리와 PaaS의 운영체제 관리를 구분하면서, 데이터·신원·사용자가 통제하는 설정에 대한 책임을 별도로 설명한다. 이 표는 운영·거버넌스 지침이며 개별 계약의 법적 책임을 대체하지 않는다.[^v-shared]

[[VPSA]]와 비교할 때는 Managed Disks·Azure Files 같은 접근 목적을 구분하고, [[Object Storage]]는 Blob Storage와 비교할 수 있다. Azure Files는 SMB·NFS 파일 공유, Blob은 오브젝트 저장, Managed Disks는 VM용 디스크라는 서로 다른 접근 목적을 가진다. 목적이 비슷해도 VPSA·S3와 API나 인증·이동 절차까지 같다는 의미는 아니다.[^v-storage]

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

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
