---
title: Separation of Concerns
type: knowledge
domain: Cloud
tags: [Architecture, Container, Isolation]
aliases: [관심사 분리, 관심사의 분리, 한 컨테이너 한 역할, one concern per container]
parent:
status: seed
lang: ko
featured: false
created: 2026-11-05
updated: 2026-11-05
---

## 1. 개요

Separation of Concerns(관심사 분리)는 서로 다른 책임을 서로 다른 단위로 나누어 다루는 설계 원칙이다. Edsger Dijkstra가 처음 쓴 용어이며,[^srp] 인프라에서는 이 원칙을 프로세스, 컨테이너, Pod, VM, 네트워크 구간 같은 배치 단위에 적용해 변경과 장애가 미치는 범위를 줄이는 데 쓴다.

예를 들어 웹 애플리케이션을 웹 서버, 애플리케이션 서버, 데이터베이스로 나누어 각각 별도의 컨테이너나 VM에 두면, 관심사가 배치 단위로 분리된 구성이 된다. Docker 문서도 컨테이너 하나가 애플리케이션의 여러 측면을 맡지 않도록 서비스마다 컨테이너를 따로 두라고 권장한다.[^docker-multi]

분리는 늘 지켜야 하는 규칙이 아니라 이득과 비용을 따져 고르는 설계 선택이다. 아래에서는 분리로 얻는 것과 치르는 비용, 합쳐서 운용하는 경우를 차례로 정리한다.

## 2. 비슷한 용어와의 구분

| 용어 | 정의와 유래 | 이 문서와의 관계 |
|---|---|---|
| 관심사 분리 | 소프트웨어와 시스템 설계 전반에 쓰이는 원칙 | 이 문서의 주제 |
| 단일 책임 원칙(SRP) | 객체지향 설계 원칙. Robert C. Martin은 모듈이 변경될 이유가 하나뿐이어야 한다고 설명하며, 그 이유를 변경을 요청하는 사람(하나의 업무 기능을 대표하는 집단)으로 정의함[^srp] | 코드 모듈의 책임 구분. 인프라 배치에는 비유로 차용 |
| 한 컨테이너에 하나의 concern | Docker가 이미지 작성 지침에서 쓰는 표현[^docker-best] | 컨테이너 단위에 적용한 관심사 분리 |
| 벌크헤드(Bulkhead) 패턴 | 한 요소가 실패해도 나머지가 계속 동작하도록 요소를 풀로 격리하는 장애 격리 패턴[^bulkhead] | 분리의 장애 격리 측면 |

SRP는 본래 소프트웨어 모듈에 대한 원칙이어서, 서버나 컨테이너 배치를 "단일 책임 원칙"이라고 부르는 것은 비유에 가깝다. 인프라에서 분리를 정당화하는 근거는 장애 범위, 자원 한도, 확장 단위, 접근 제어 같은 구체적인 이점에 있다.

논리 계층(layer)과 물리 티어(tier)도 구별해야 한다. 논리 계층은 책임을 나누고 의존 방향을 정리하는 구분이고, 물리 티어는 별도 머신에서 실행되는 배치 단위이다. Azure Architecture Center의 N-tier 설명에 따르면 계층마다 티어를 따로 둘 수도 있지만 반드시 그래야 하는 것은 아니며, 여러 계층을 한 티어에 둘 수도 있다.[^n-tier] 이 문서에서 말하는 비용은 주로 물리 티어를 나눌 때 생기는 비용이다.

## 3. 분리 단위

아래 단위는 서로 바꿔 쓸 수 있는 선택지가 아니라 나누는 기준이 서로 다르다. 컨테이너와 VM은 프로세스를 실행하는 격리 단위, Pod는 함께 스케줄링되며 자원을 공유하는 단위, 네트워크 구간은 트래픽이 지나는 경계이다. 실제 구성에서는 이들이 겹친다. VM 위에서 Pod를 실행하고, 그 Pod들을 서브넷으로 나뉜 구간에 배치하는 식이다. 서비스, 컨테이너, 티어가 일대일로 대응하지도 않는다.

| 단위 | 나누는 대상 | 분리 방식 | 격리의 특징 |
|---|---|---|---|
| 컨테이너 | 실행 격리 | 컨테이너 하나에 서비스 하나 | 필요한 파일을 갖춘 격리된 프로세스. 여러 컨테이너가 같은 호스트 커널을 공유함[^docker-container] |
| Pod | 스케줄링과 공유 | Pod 하나에 컨테이너 하나가 일반적 | 같은 노드에 함께 배치되며 네트워크 namespace와 볼륨을 공유함[^k8s-pods] |
| VM | 실행 격리 | VM 하나에 역할 하나 | 자체 커널을 포함한 전체 운영체제를 실행하며 컨테이너보다 강한 보안 경계를 제공함. Microsoft 문서의 "호스트와 다른 VM으로부터의 완전한 격리"는 보안 경계에 대한 설명임[^ms-container-vm] |
| 네트워크 구간 | 트래픽 경계 | 티어마다 서브넷과 접근 규칙을 분리 | 서브넷을 보안 경계로 삼고 Azure의 네트워크 보안 그룹(NSG)과 라우팅 규칙을 적용함[^n-tier] |

컨테이너 격리는 Linux 커널 기능이 맡는다. namespace는 프로세스마다 자원이 독립된 것처럼 보이게 하고,[^namespaces] cgroup은 프로세스 그룹이 쓰는 자원을 제한하고 측정한다.[^cgroups] Kubernetes에서는 Pod 하나에 컨테이너 하나를 두는 방식이 가장 흔하고, 밀접하게 결합된 컨테이너 여러 개를 한 Pod에 묶는 것은 상대적으로 고급 사용 사례로 분류된다.[^k8s-pods]

## 4. 분리로 얻는 이점

아래 이점은 나누기만 해서 저절로 생기지 않는다. 장애 범위를 줄이려면 경계 안팎의 의존을 끊어야 하고, 자원을 제한하려면 컨테이너마다 limit을 설정해야 한다. 독립 배포는 단위마다 따로 배포할 수 있게 구성해야 하며, 접근 제어는 서브넷과 규칙을 실제로 적용해야 효과가 있다.

### 장애 범위의 축소

장애 격리는 구성 요소나 시스템에 생긴 장애의 영향을 정해 둔 경계 안으로 제한하는 것이며, 격리가 제대로 되어 있으면 경계 밖 구성 요소는 그 장애의 영향을 받지 않는다.[^aws-isolation] Azure Architecture Center는 벌크헤드 패턴을 선박의 격벽에 빗댄다. 선체 일부가 손상되어도 그 구획에만 물이 차 배가 가라앉지 않듯이, 한 요소의 문제가 시스템 전체로 번지는 것을 막는다.[^bulkhead] 같은 문서는 서비스를 벌크헤드로 나누는 방법으로 별도의 VM, 컨테이너, 프로세스에 배포하는 방식을 들며, 컨테이너가 자원 격리와 낮은 오버헤드 사이에서 균형이 좋다고 평가한다.

다만 컨테이너는 호스트 커널을 공유하므로 VM만큼 강한 보안 경계를 제공하지는 못한다. 더 강한 경계가 필요하면 VM이나 Hyper-V isolation 같은 추가 격리 수단을 검토한다.[^ms-container-vm] 격리 수준은 분리 단위마다 다르므로 장애 범위를 얼마나 좁힐 수 있는지도 단위 선택에 달려 있다.

### 자원 사용의 제한

자원 한도를 두지 않으면 컨테이너는 호스트 커널 스케줄러가 허용하는 만큼 자원을 쓸 수 있다.[^docker-limits] 역할마다 컨테이너를 나누면 역할별로 한도를 줄 수 있다. Kubernetes에서 CPU limit은 CPU 스로틀링으로 강제되지만, memory limit은 커널이 메모리 압박을 감지했을 때 OOM 종료로 강제되므로 limit을 넘긴 컨테이너가 곧바로 종료되지 않을 수 있다.[^k8s-resources] 그래도 역할마다 컨테이너를 나누고 limit을 걸어 두면, 한 서비스의 과다한 자원 사용이 다른 서비스의 자원까지 잠식할 가능성을 줄일 수 있다.

### 확장 단위의 분리

Docker 문서는 애플리케이션을 여러 컨테이너로 나누면 수평 확장과 컨테이너 재사용이 쉬워진다고 설명한다.[^docker-best] Azure Architecture Center는 N-tier 구조에서 티어가 확장성, 안정성, 보안의 경계가 된다고 보고, 이 세 가지에서 요구 사항이 다른 서비스는 티어를 나누는 방안을 검토하라고 권한다.[^n-tier] Azure 문서의 참조 구성에서는 웹 티어와 업무 티어를 무상태로 두어 어느 VM이 요청을 받아도 처리할 수 있게 하고, 티어마다 VM을 추가해 수평으로 확장한다.[^n-tier] 12-Factor도 프로세스를 무상태, share-nothing으로 두고 지속해야 하는 데이터는 데이터베이스 같은 상태 저장 백엔드 서비스에 두라고 하는데, 상태를 별도 서비스로 분리하는 설계와 같은 방향이다.[^12factor]

### 배포의 독립성

독립 배포는 티어를 나눈다고 저절로 보장되지 않는다. Azure Architecture Center는 N-tier의 단점으로, 모놀리식 설계가 기능의 독립 배포를 막는다는 점을 꼽는다.[^n-tier] 마이크로서비스 문서는 둘 이상의 서비스를 함께 배포해야 하는 상호 의존이 없어야 하고, 각 서비스를 다른 서비스의 재배포 없이 배포할 수 있어야 한다는 기준을 제시한다.[^ms-boundaries]

### 접근 제어의 세분화

Azure는 N-tier 구조에서 티어마다 자체 서브넷을 두어 티어별로 네트워크 보안 그룹과 라우팅 규칙을 쉽게 적용하도록 하며, 예를 들어 데이터 티어는 업무 티어의 요청만 받도록 제한한다.[^n-tier] 이 내용은 [[3-Tier Architecture]]의 "네트워크와 접근 경계"와 이어진다.

## 5. 분리의 비용

분리한 구성 요소는 서로 통신해야 하므로 비용이 따른다.
- 티어를 물리적으로 나누면 확장성과 복원력이 좋아지는 대신, 추가 네트워크 통신으로 지연이 늘어난다.[^n-tier]
- 여러 티어를 거치는 요청과 데이터는 테스트와 모니터링을 어렵게 한다.[^n-tier]
- 중간 티어가 단순 조회와 저장만 전달하면 의미 있는 가치 없이 지연과 복잡성만 더한다.[^n-tier]
- 벌크헤드로 나눌 때는 기술이 제공하는 격리 수준과 함께 비용, 성능, 관리 편의성 측면의 오버헤드를 따져야 한다.[^bulkhead]
- 분리한 구성 요소 사이의 네트워크 호출은 새로운 실패 지점이 된다. Azure Architecture Center에 따르면 서비스 간 호출은 노드 장애나 인스턴스 장애 등으로 실패할 수 있고, 실패한 요청이 쌓여 메모리, 스레드, 데이터베이스 연결 같은 자원을 붙잡으면 연쇄 장애로 번질 수 있다. 완화 방법으로는 Circuit Breaker 패턴이 제시된다.[^ms-interservice]

### 분리해도 남는 공유 의존성

컨테이너나 서비스를 나눠도 다음 자원을 함께 쓰면 그 지점에서는 영향이 전파된다.
- 같은 노드: 컨테이너와 Pod는 노드의 자원을 나누어 쓴다. Kubernetes 문서는 CPU limit이 같은 노드에서 한 워크로드가 다른 워크로드의 자원을 고갈시키는 noisy neighbor 문제를 막는 데 도움이 된다고 설명한다. 노드가 실패하면 그 노드의 Pod가 멈추고, Deployment 같은 컨트롤러가 관리하는 Pod는 컨트롤러가 대체 Pod를 만들어 스케줄러가 정상 노드에 배치한다.[^k8s-pods]
- 공유 자원 일반: Azure Architecture Center는 멀티테넌트 시스템에서 하나의 자원을 공유하면 noisy neighbor 문제의 위험이 따르고 이를 완전히 피할 수 없으며, 가능성을 낮추거나 영향을 줄이는 조치만 취할 수 있다고 설명한다.[^ms-noisy]
- 공유 데이터 저장소: 같은 물리 데이터베이스 서버를 공유하는 것은 안전할 수 있다. 그러나 서비스들이 같은 스키마나 테이블을 읽고 쓰면 스키마를 바꿀 때 모든 서비스를 함께 조율해야 하므로 서비스를 나눈 효과가 줄어든다.[^ms-data]

### 과잉 분리의 신호

분리 비용은 N-tier의 티어 사이 호출뿐 아니라 서비스 사이 호출에서도 나타난다. 아래 기준은 Azure Architecture Center가 마이크로서비스 경계를 정할 때 제시하는 것이어서 컨테이너, VM, 티어의 개수에 그대로 적용되지는 않지만, 얼마나 잘게 나눌지 판단하는 관점으로는 참고할 만하다.[^ms-boundaries]
- 서비스를 둘로 나눈 뒤 호출이 지나치게 잦아졌다면, 두 기능이 같은 서비스에 속한다는 징후일 수 있다.
- 둘 이상의 서비스를 함께 배포해야 하는 상호 의존이 없어야 한다.
- 경계는 데이터 일관성과 무결성 문제를 피하도록 정해야 한다. 일관성을 유지하려고 관련 기능을 한 서비스로 묶어야 하는 경우도 있으나, 강한 일관성이 항상 필요한 것은 아니며 분해의 이득이 관리 복잡성보다 큰 경우도 많다.

판단이 서지 않으면 더 굵은 단위에서 시작하라고 권한다. 서비스 하나를 둘로 쪼개는 일이 이미 여러 서비스로 나뉜 기능을 다시 정리하는 일보다 쉽기 때문이다.[^ms-boundaries]

## 6. 합쳐서 운용하는 경우

컨테이너 하나에 프로세스 하나만 두는 것은 좋은 경험칙이지만 엄격한 규칙은 아니다.[^docker-best] Docker 문서는 서비스가 여러 프로세스로 나뉠 수 있음을 인정하며, 한 컨테이너에 여러 프로세스가 필요할 때 쓰는 방법으로 래퍼 스크립트, bash job control, supervisord를 안내한다.[^docker-multi]

Kubernetes의 사이드카 컨테이너는 같은 Pod에서 주 애플리케이션 컨테이너와 함께 실행되는 보조 컨테이너로, 로깅, 모니터링, 보안, 데이터 동기화 같은 부가 기능을 맡는다. 독립된 수명 주기를 가지며, 주 컨테이너가 완전히 종료된 뒤에 종료된다.[^k8s-sidecar] 별도 컨테이너이므로 컨테이너 단위의 분리는 유지되어, 주 애플리케이션과 무관하게 갱신하거나 재시작할 수 있다. 다만 같은 Pod에 있으므로 네트워크 namespace와 볼륨은 공유한다.[^k8s-pods]

### 합치는 쪽을 검토하는 조건

- 자원 효율과 단순성이 더 중요한 경우. 벌크헤드 패턴 문서는 비효율적인 자원 사용을 감수할 수 없거나 추가된 복잡성이 필요하지 않은 경우를 이 패턴이 맞지 않는 조건으로 든다.[^bulkhead]
- 두 기능이 서로 자주 호출하는 경우. 나눈 뒤 호출이 지나치게 잦아졌다면 같은 서비스에 속한다는 징후일 수 있다.[^ms-boundaries]
- 함께 배포해야 하는 경우. 독립 배포가 되지 않는다면 경계를 다시 살펴본다.[^ms-boundaries]
- 중간 티어가 조회와 저장만 전달하는 경우. 지연과 복잡성만 더하므로 그 티어를 따로 둘 이유가 약하다.[^n-tier]

## 7. 3-Tier 구조에서의 적용

[[3-Tier Architecture]]에서 WEB, WAS, DB를 티어로 나누는 것은 이 원칙을 적용한 사례로 볼 수 있다. Azure Architecture Center는 티어가 확장성, 안정성, 보안의 경계가 되므로 이 세 영역에서 요구 사항이 다른 서비스는 별도 티어로 두는 방안을 검토하라고 설명한다.[^n-tier] 또 티어마다 자체 서브넷을 두고 데이터 티어는 업무 티어의 요청만 받도록 제한하는 구성을 제시한다.[^n-tier] 티어를 몇 개로 나눌지는 앞의 분리 비용과 합치는 조건을 기준으로 정한다.

## 8. 관련 문서

- [[3-Tier Architecture]]
- [[Web Server]], [[Web Application Server]], [[Database]]
- [[Server]], [[Networking]]
- [[Multi-tenancy]]

## 9. 출처

[^docker-multi]: [Docker Docs: Run multiple processes in a container](https://docs.docker.com/engine/containers/multi-service_container/)
[^docker-best]: [Docker Docs: Building best practices, Decouple applications](https://docs.docker.com/build/building/best-practices/)
[^docker-container]: [Docker Docs: What is a container?](https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-a-container/)
[^srp]: [Robert C. Martin: The Single Responsibility Principle (The Clean Code Blog, 2014)](https://blog.cleancoder.com/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html)
[^bulkhead]: [Microsoft Azure Architecture Center: Bulkhead pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/bulkhead)
[^namespaces]: [Linux man-pages: namespaces(7)](https://man7.org/linux/man-pages/man7/namespaces.7.html)
[^cgroups]: [Linux man-pages: cgroups(7)](https://man7.org/linux/man-pages/man7/cgroups.7.html)
[^k8s-pods]: [Kubernetes Docs: Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
[^aws-isolation]: [AWS Well-Architected Reliability Pillar: Use fault isolation to protect your workload](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/use-fault-isolation-to-protect-your-workload.html)
[^docker-limits]: [Docker Docs: Resource constraints](https://docs.docker.com/engine/containers/resource_constraints/)
[^k8s-resources]: [Kubernetes Docs: Resource Management for Pods and Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
[^n-tier]: [Microsoft Azure Architecture Center: N-tier architecture style](https://learn.microsoft.com/en-us/azure/architecture/guide/architecture-styles/n-tier)
[^12factor]: [The Twelve-Factor App: VI. Processes](https://12factor.net/processes)
[^k8s-sidecar]: [Kubernetes Docs: Sidecar Containers](https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers/)
[^ms-container-vm]: [Microsoft Learn: Windows and containers, Containers vs. virtual machines](https://learn.microsoft.com/en-us/virtualization/windowscontainers/about/containers-vs-vm)
[^ms-boundaries]: [Microsoft Azure Architecture Center: Identify microservice boundaries](https://learn.microsoft.com/en-us/azure/architecture/microservices/model/microservice-boundaries)
[^ms-noisy]: [Microsoft Azure Architecture Center: Noisy Neighbor antipattern](https://learn.microsoft.com/en-us/azure/architecture/antipatterns/noisy-neighbor/noisy-neighbor)
[^ms-data]: [Microsoft Azure Architecture Center: Data considerations for microservices](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/data-considerations)
[^ms-interservice]: [Microsoft Azure Architecture Center: Interservice communication in microservices](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/interservice-communication)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
