---
title: VPSA
type: knowledge
domain: Storage
tags:
  - Zadara
aliases:
  - Virtual Private Storage Array
  - VPSA Storage Array
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-04
---

## 1. 개요

VPSA(Virtual Private Storage Array)는 Zadara의 소프트웨어 정의 스토리지 서비스이다.

이 문서에서는 블록 볼륨과 파일 공유를 제공하는 VPSA Storage Array 및 VPSA Flash Array를 중심으로 설명한다.[^1]

서버가 필요한 저장 공간을 네트워크로 연결해 사용하는 구조이며, 스토리지 관리자는 용량과 접근 권한, 데이터 보호 기능 등을 관리한다. 이름이 비슷한 VPSA Object Storage([[ZIOS]])는 [[Object Storage|오브젝트 스토리지]] 제품이므로 구분해야 한다.

## 2. 물리 장비와 가상 컨트롤러의 관계

VPSA의 “가상”은 실제 저장장치가 없다는 뜻이 아니다. 물리 스토리지 노드의 드라이브를 사용하면서, 가상 컨트롤러가 프로토콜 서비스와 볼륨·데이터 보호 기능을 제공하는 구조다. 공식 가이드는 전용 드라이브와 가상 컨트롤러를 구분하고, Controllers 화면의 객체를 클러스터로 구성된 컨트롤러 쌍으로 설명한다. 활성 컨트롤러 장애나 전환 시에는 대기 컨트롤러가 역할을 이어받는 구조를 가진다.[^1][^d-controller]

호스트 입장에서 블록 볼륨 요청은 스토리지 네트워크와 컨트롤러를 거쳐 풀의 실제 저장 공간으로 전달된다. 파일 공유는 컨트롤러의 NFS·SMB 파일 서비스와 파일시스템 계층을 추가로 거친다. 같은 VPSA에 있더라도 블록 장치를 사용하는 서버와 NAS 공유를 사용하는 서버가 거치는 상위 인터페이스가 다르다.

```mermaid
flowchart LR
    A["호스트 · iSCSI 또는 NFS·SMB"] --> B["VPSA 가상 컨트롤러"]
    B --> C["볼륨 · NAS 공유"]
    C --> D["스토리지 풀"]
    D --> E["할당된 물리 드라이브"]
```

이는 객체 관계와 요청 흐름을 단순화한 그림으로, 특정 버전의 내부 배치도나 케이블 연결도가 아니다. VPSA Storage Array에서는 RAID 그룹이 풀을 구성하는 기반이 될 수 있고, Flash Array에서는 다른 풀 생성·데이터 배치 방식이 제공된다. “드라이브 → RAID 그룹 → 풀”을 모든 제품의 유일한 경로로 고정하지 않고 실제 풀 유형을 구분한다.[^3] 컨트롤러가 이중화되어도 호스트의 단일 네트워크 경로 등 다른 단일 장애 지점은 별도로 남을 수 있다.

## 3. 제공 방식

| 방식 | 접근 방법 | 서버에서 사용하는 형태 |
|---|---|---|
| [[Block Storage\|블록 스토리지]] | [[iSCSI]], iSER, Fibre Channel 등 | 블록 장치로 연결해 운영체제나 애플리케이션에서 사용 |
| [[File Storage\|파일 스토리지]] | NFS, SMB | 공유 디렉터리로 연결해 파일 단위로 사용 |

공식 가이드는 블록 볼륨과 NAS 공유를 구분한다. 지원 프로토콜과 버전, 필요한 네트워크 구성은 환경에 맞춰 확인해야 한다.[^2] 예를 들어 가이드는 iSER 호스트 연결을 Primary Virtual Network로 제한하며, FC 사용에는 해당 연결을 위한 설정과 구성이 필요하다.[^v-first]

## 4. 저장 공간 구성과 연결

스토리지 풀은 볼륨에 사용할 저장 공간을 관리하는 단위이다. 볼륨은 풀 위에 생성하며, 풀의 가상화 계층은 볼륨·스냅샷·클론이 사용하는 물리 공간을 관리한다.[^3] 씬 프로비저닝에서 호스트에 보이는 논리 용량과 실제 사용 가능한 물리 공간은 다르다. 볼륨을 크게 만들었다고 실제 데이터의 물리 공간까지 미리 확보되는 것은 아니므로 풀의 사용량과 여유 공간을 함께 관리한다.

호스트가 볼륨을 사용하려면 VPSA에 서버를 정의하고 해당 볼륨에 대한 연결과 권한을 설정해야 한다.[^4] 단순히 볼륨을 생성하는 것과 서버에서 실제로 읽고 쓸 수 있는 상태는 구분된다.

## 5. 풀·씬 프로비저닝·스냅샷이 용량에 미치는 영향

풀은 데이터를 실제로 저장하는 공간을 관리하고, 볼륨은 호스트에 논리 용량을 제공한다. 논리 볼륨의 총 크기를 물리 공간보다 크게 제공할 수 있는 씬 프로비저닝에서는 아직 쓰지 않은 범위를 즉시 같은 양의 물리 공간으로 소비하지 않는다. 따라서 provisioned 용량, 실제 사용량, 남은 물리 공간을 각각 읽어야 한다.[^3]

스냅샷과 클론이 원본과 변하지 않은 공간을 공유하면 처음에는 추가 사용량이 작을 수 있다. 이후 원본이나 클론이 변경되면 이전 내용과 새 내용을 유지할 공간이 필요해진다. 스냅샷을 많이 보존한 상태의 빈번한 갱신은 현재 파일 크기가 일정해도 풀 사용량을 늘릴 수 있다. 실제 데이터 절감·회수·보존 동작은 풀 유형과 제품 규약에 따른다.[^2][^3]

이 구조에서 호스트 파일시스템의 “남은 용량”과 풀의 “남은 용량”은 서로 다른 값이다. 호스트에는 여유가 보여도 여러 볼륨과 스냅샷이 공유하는 풀의 물리 공간이 부족하면 서비스에 영향을 줄 수 있다. 용량 관리는 볼륨의 보이는 크기만 늘리는 작업이 아니라 데이터 증가율과 스냅샷 보존량·회수 가능 공간을 함께 판단하는 일이다.

## 6. 데이터 관리와 제품 구분

VPSA 가이드는 씬 프로비저닝, 스냅샷과 클론, 원격 미러링 등의 데이터 관리 기능을 설명한다. Storage Array는 HDD·SSD를 사용하는 하이브리드 구성, Flash Array는 플래시 매체 활용에 최적화된 구성으로 소개한다.[^1] 스냅샷·클론·미러는 서로 목적과 장애 범위가 다르며, 원본과 같은 장애 도메인이나 관리 권한 아래 있는 복사본이 독립 백업을 자동으로 대체하는 것은 아니다. 공식 가이드도 Backup to Object Storage와 복원 기능을 별도로 다룬다.[^v-backup]

[[NGOS]]와 [[ZIOS]]는 [[Object Storage|오브젝트 스토리지]] 서비스이다. 같은 Zadara 스토리지 환경에 있어도 블록·파일 볼륨과 오브젝트는 접근 방식과 관리 단위가 다르다. S3 호환 API의 버킷에 접근하는 작업과 [[iSCSI]] 볼륨을 서버에 연결하는 작업을 같은 절차로 생각하면 안 된다.

## 7. 볼륨이 느릴 때 구성 객체를 따라가기

VPSA Performance Monitor는 드라이브부터 풀·볼륨·연결 서버에 이르는 요소를 나누어 볼 수 있다. 처리량·IOPS·응답 시간을 같은 시간대와 읽기·쓰기 구분으로 비교하면 어느 계층에서 지연이 늘었는지 좁힐 수 있다. 공식 성능 가이드는 평균 I/O 크기와 IOPS가 처리량에 연결된다고 설명한다.[^d-performance]

예를 들어 모든 볼륨이 동시에 느리면 공통 컨트롤러·풀·네트워크를 비교할 근거가 생기고, 한 서버만 느리면 해당 호스트와 경로를 우선 구분할 수 있다. 단일 볼륨에서만 지연이 증가하더라도 그 원인이 반드시 볼륨 설정은 아니다. 해당 볼륨의 동기 쓰기·작업 크기·경합과 같은 워크로드 변화도 함께 본다.

운영 판단에서는 컨트롤러 페일오버, 파일 서비스 재시작, 서버 연결 변경을 서로 다른 조작으로 구분한다. 컨트롤러 전환은 활성·대기 역할의 변경이고 NFS·SMB 서비스 재시작은 파일 서비스 프로세스의 변경이다. 이 차이를 알면 같은 “스토리지 재시작”으로 묶기보다 영향을 받는 클라이언트와 세션의 범위를 정확히 설명할 수 있다.[^d-controller]

볼륨 생성은 스토리지 쪽 객체를 만들고, Server 객체와의 연결은 해당 호스트에 접근 범위를 부여하며, 호스트 discovery·재검색은 운영체제가 장치를 인식하게 한다. 파일시스템 생성·마운트는 블록 볼륨 위에서 호스트가 수행하는 별도 계층이다. NAS 공유의 파일시스템·공유 경로는 스토리지 서비스 쪽에 있으므로 클라이언트는 공유에 연결한다. 따라서 “볼륨은 존재한다”는 사실만으로 호스트의 파일 쓰기가 가능한지 판단할 수 없다.[^2][^4]



## 8. AWS 스토리지 서비스와의 비교

VPSA는 블록과 파일을 함께 제공하므로 AWS 서비스 하나와 완전히 대응하지 않는다. 아래는 사용 목적과 접근 방식을 기준으로 한 비교이다.

| VPSA 기능 | 비교할 AWS 서비스 | 공통점과 차이 |
|---|---|---|
| 블록 볼륨 | Amazon EBS | 서버에서 블록 장치로 사용한다는 목적이 비슷하다. EBS는 EC2 인스턴스에 연결하는 서비스이며, VPSA의 iSCSI 설정 절차를 그대로 사용하는 것은 아니다. |
| NFS 파일 공유 | Amazon EFS | 네트워크 파일 공유 용도로 비교할 수 있다. EFS는 NFSv4 기반이며 AWS가 [[File Storage\|파일 스토리지]] 인프라를 관리한다. |
| SMB 파일 공유 | Amazon FSx for Windows File Server | SMB 공유 용도가 비슷하다. FSx for Windows는 Windows Server 기반 기능과 Active Directory 연동을 제공하는 별도 서비스이다. |
| 오브젝트 저장 | Amazon S3 | 이 문서의 블록·파일 VPSA가 아니라 [[NGOS]]·[[ZIOS]]와 비교하는 것이 적절하다. |

AWS의 EBS·EFS·FSx는 서로 다른 서비스이며, VPSA의 풀·볼륨 구성과 관리 단위를 일대일로 대응시키면 안 된다. FSx 역시 여러 제품으로 이루어진 서비스군이므로 여기서는 SMB 비교 대상으로 FSx for Windows File Server를 지정했다.[^aws-ebs][^aws-efs][^aws-fsx]

실제 설계에서는 프로토콜뿐 아니라 다중 호스트 접근, 접근 권한, 고가용성 범위, 스냅샷·복구 방식과 성능 요구를 함께 확인해야 한다. 기능이 비슷하다는 설명은 API·데이터 형식의 호환성이나 무변경 이전을 보장하지 않는다.
## 9. 관련 문서

- [[iSCSI]]: IP 네트워크로 블록 장치에 접근하는 프로토콜
- [[NGOS]]: NextGen Object Storage
- [[ZIOS]]: VPSA Object Storage

- [[EMC]]
- [[Microsoft Azure]]

- [[Storage]]

- [[Server]]

## 10. 출처

[^1]: [Zadara — Introduction, VPSA User Guide](https://guides.zadara.com/vpsa-guide/latest/introduction.html)
[^2]: [Zadara — Volumes](https://guides.zadara.com/vpsa-guide/latest/volumes.html)
[^3]: [Zadara — Pools](https://guides.zadara.com/vpsa-guide/latest/pools.html)
[^4]: [Zadara — Servers](https://guides.zadara.com/vpsa-guide/latest/servers.html)

[^aws-ebs]: [AWS — What is Amazon EBS?](https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs.html)
[^aws-efs]: [AWS — What is Amazon EFS?](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html)
[^aws-fsx]: [AWS — What is FSx for Windows File Server?](https://docs.aws.amazon.com/fsx/latest/WindowsGuide/what-is.html)

[^v-first]: [Zadara — VPSA First steps](https://guides.zadara.com/vpsa-guide/latest/first-steps.html)

[^v-backup]: [Zadara — VPSA Introduction: data protection guides](https://guides.zadara.com/vpsa-guide/latest/introduction.html)

[^d-controller]: [Zadara — Controllers](https://guides.zadara.com/vpsa-guide/latest/controllers.html)

[^d-performance]: [Zadara — Performance](https://guides.zadara.com/vpsa-guide/latest/performance.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Storage 분류|Storage]]
