---
title: Replication Levels
type: knowledge
domain: Cloud
tags: []
aliases:
  - 복제 계층
  - 복제 레벨
  - 파일 레벨 복제
  - 블록 레벨 복제
  - 애플리케이션 레벨 복제
  - 호스트 기반 복제
  - 하이퍼바이저 기반 복제
  - 스토리지 어레이 기반 복제
parent:
status: growing
lang: ko
featured: false
created: 2026-11-05
updated: 2026-11-05
---

## 1. 개요

Replication Levels(복제 계층)는 데이터를 복제할 때 애플리케이션, 파일, 블록 가운데 어느 층에서 읽어 복사하는지, 그리고 그 일을 어디서 수행하는지에 따라 복제 방식을 나누는 분류이다. 서버 마이그레이션과 재해 복구에서 도구를 고를 때 참고하는 기준 가운데 하나이다. 변경된 부분만 보내는 기법은 [[Delta Sync]]에서 다룬다.

도서관 장서를 다른 도서관에 복사하는 일에 견주면, 내용을 이해하며 필요한 장만 옮겨 적는 방법이 애플리케이션 레벨, 책을 한 권씩 복사하는 방법이 파일 레벨, 낱장을 번호 순서대로 그대로 촬영하는 방법이 블록 레벨에 해당한다. 낱장을 촬영하는 방식은 그 낱장이 어느 책에 속하는지 모르듯이, 블록 레벨 복제도 블록이 어느 파일의 일부인지 알지 못한다. 이 비유는 작성자의 정리이다. 한계가 있는데, 책은 복사하는 동안 바뀌지 않지만 운영 중인 서버의 데이터는 계속 바뀌므로 일관성 문제는 비유로 설명되지 않는다.

## 2. 용어 정리

- RPO와 RTO: RPO(Recovery Point Objective)는 재해 뒤에 잃어도 되는 데이터의 최대 시간이고, RTO(Recovery Time Objective)는 재해 복구를 마치기까지 허용하는 최대 시간이다.[^14]
- 동기와 비동기 복제: DRBD 문서에서 동기 복제는 모든 호스트에 쓰기가 끝난 뒤 완료를 알리고, 비동기 복제는 로컬 쓰기가 끝나면 완료를 알린다.[^5]
- 복제와 백업: 복제는 원본의 변경을 계속 따라가므로 데이터 손상이나 삭제도 그대로 옮겨질 수 있어, 특정 시점으로 되돌리는 백업과 역할이 다르다.[^15]
- failover와 failback: AWS Elastic Disaster Recovery 문서에서 failover는 운영 트래픽을 원본에서 복구 환경으로 돌리는 일이고, failback은 복구 환경에서 원래 인프라로 되돌리는 일이다.[^16] 다른 제품의 정의로 일반화하지 않는다.

## 3. 분류 기준이 출처마다 다르다

복제 계층을 하나로 정한 표준 분류는 확인하지 못했다. 수행 위치와 데이터를 보는 층은 서로 다른 기준이며, 출처마다 같은 용어를 다른 층에 쓴다. 아래 두 표는 각 출처의 표현을 옮긴 것이고 하나의 표준 분류가 아니다.

표 A. 수행 위치 기준(출처의 용어)

| 용어 | 출처의 설명 |
|---|---|
| array-based | NetApp SnapMirror는 볼륨 수준으로 데이터를 미러링한다고 설명한다.[^10] |
| host-based | StoneFly는 파일 시스템 또는 애플리케이션 수준에서 일어나는 복제로 설명한다.[^1] AWS Elastic Disaster Recovery는 서버에 설치한 에이전트가 볼륨의 블록을 읽는다.[^9] VMware는 ESXi 호스트에 내장된 vSphere Replication을 host-based라고 부른다.[^7] |
| hypervisor-based | StoneFly는 보통 VM 수준에서 동작한다고 설명한다.[^1] |

StoneFly는 network-based를 네트워크 계층에서 동작하는 방식이라고 정의하면서 같은 글에서 애플리케이션 또는 파일 시스템 수준에서 포착한다고도 적어 정의가 일관되지 않는다.[^1] 다른 출처에서 대응하는 정의를 확인하지 못해 이 문서에서는 다루지 않는다.

표 B. 데이터를 보는 층(작성자의 정리)

| 층 | 읽는 단위 | 예시 |
|---|---|---|
| 애플리케이션 또는 DB | DB가 이해하는 변경 단위 | PostgreSQL logical replication[^2] |
| 파일 | 파일과 디렉터리 | rsync, robocopy[^3][^17] |
| 블록 | 디스크 또는 볼륨의 블록 | DRBD, Storage Replica, AWS Elastic Disaster Recovery[^5][^6][^9] |

두 표를 이어 보면, 블록 레벨 복제는 호스트 OS 안에서, 하이퍼바이저에서, 스토리지 어레이에서 모두 일어날 수 있다. 이 연결은 작성자의 정리이다.

## 4. 층별 특징

### 4.1 애플리케이션 또는 데이터베이스 레벨

데이터베이스가 이해하는 변경 단위를 복사한다. PostgreSQL은 복제를 logical과 physical로 나누며, logical은 복제 식별자(대개 기본 키)를 기준으로 데이터 객체와 변경을 복제한다.[^2] 구독자는 발행자와 같은 순서로 적용하므로 하나의 구독 안의 발행에 대해 트랜잭션 일관성이 보장된다.[^2] PostgreSQL의 physical은 DB 엔진이 정확한 블록 주소를 따라 바이트 단위로 복제하는 방식으로, 이 문서의 디스크 블록 레벨과는 대상이 다르다(작성자의 정리). 복제 대상을 세밀하게 고를 수 있는 것이 장점이고, 해당 DB 제품의 기능에 묶이며 OS 전체를 옮길 수 없다는 것이 단점이다(작성자의 정리).

### 4.2 파일 레벨

파일 시스템이 보여 주는 파일과 디렉터리를 읽어 복사한다. 먼저 용어를 구분해야 한다. rsync가 말하는 블록은 비교용 체크섬을 계산하려고 파일 내용을 일정 크기로 나눈 단위이며,[^4] 디스크 위의 블록과는 다르다. rsync는 크기나 수정 시각이 달라진 파일을 대상으로 삼고 이미 있는 파일은 변경된 부분만 보내는 delta-transfer로 전송량을 줄인다.[^3] Robocopy는 파일과 디렉터리 트리를 복사하며 /MIR 옵션은 원본에 없는 대상 파일을 지운다.[^17] 대상을 파일 단위로 고를 수 있어 단순한 것이 장점이다. 복사 중에 바뀌는 파일의 시점이 어긋날 수 있고 부팅 영역 같은 파일 시스템 바깥의 구조는 옮기기 어렵다는 것이 단점이다(작성자의 정리).

### 4.3 블록 레벨, 호스트 OS 안에서 수행

파일 시스템 아래의 볼륨에서 변경된 블록을 읽어 보낸다. 호스트 OS 안에서 동작하는 구성요소가 일을 맡으며 형태는 제품마다 다르다. Storage Replica는 Windows Server의 기능이고,[^6] DRBD는 디스크, 파티션, 논리 볼륨 같은 블록 장치를 미러링하는 Linux 커널 모듈이며,[^5] AWS Elastic Disaster Recovery는 원본 서버에 설치하는 에이전트로 볼륨 전체를 블록 단위로 한 번 읽고 이후 쓰기를 가로채 변경분을 동기화한다.[^9]

파일 잠금이나 파일 종류에 얽매이지 않는 것이 장점이다. Storage Replica는 복제를 구성하면 대상 볼륨이 마운트 해제되어 사용자가 읽거나 쓸 수 없다고 설명한다. Windows Server 1709부터는 Test-Failover로 대상 볼륨의 읽기와 쓰기 가능한 스냅샷을 임시로 마운트해 백업이나 테스트에 쓸 수 있다.[^6] 다른 블록 복제 제품의 대상 접근 여부는 확인하지 않았다.

### 4.4 하이퍼바이저 기반

가상화 계층에서 VM의 가상 디스크 변경을 추적해 복제한다. vSphere Replication은 VM마다 설정하고 하부 스토리지와 무관하며, 변경 추적에 vSCSI 필터를 쓰고 RPO를 5분에서 24시간 사이로 정한다. 복제 구성요소는 vSphere에 내장되어 별도로 설치하지 않는다고 설명한다.[^7] Hyper-V Replica는 Hyper-V 호스트 사이에서 VM을 복제하며 RCT로 블록 수준의 변경을 추적하고 30초, 5분, 15분 간격으로 복제한다.[^8] VMware의 CBT는 이전 변경 ID 이후 바뀐 디스크 영역을 백업 도구가 질의하는 방식이다.[^20] VSS를 쓸 때 게스트 쪽에 필요한 구성요소는 확인하지 못했다. VM 단위로 다루므로 특정 하이퍼바이저 환경을 전제로 한다는 점이 단점이다.[^1]

### 4.5 스토리지 어레이 기반

어레이가 자신의 볼륨이나 LUN을 다른 어레이로 복제한다. NetApp SnapMirror는 볼륨 단위로, 처음에는 스냅샷과 그것이 참조하는 모든 블록을 보내고 이후에는 새 스냅샷만 보낸다.[^10] 같은 회사의 지식 문서에 따르면 볼륨 단위 SnapMirror는 NTFS ACL을 SID와 함께 보존하지만 CIFS 공유, NFS export, SAN 구성 같은 데이터 접근 설정은 복제하지 않는다.[^11] 어레이 복제는 제품마다 범위가 다르므로 사용하는 제품 문서에서 확인해야 한다.

## 5. 혼동하기 쉬운 두 개념

### 5.1 파일 레벨 복제와 파일 공유 프로토콜

파일 레벨 복제는 NFS나 SMB 같은 파일 공유 프로토콜과 다르다. NFS version 4는 분산 파일 시스템 프로토콜이고,[^18] SMB는 네트워크로 파일을 읽고 쓰게 해 주는 파일 공유 프로토콜이다.[^19] 둘은 클라이언트가 서버의 파일에 접근하는 규약이고, 파일 레벨 복제는 파일을 읽어 사본을 만드는 작업이다. 공유 경로를 마운트하면 rsync나 robocopy의 원본이나 대상으로 쓸 수 있어 함께 쓰이기도 한다(작성자의 정리).

### 5.2 블록 레벨 복제와 블록 스토리지

블록 스토리지는 데이터를 고정 크기의 블록으로 나누어 저장하는 방식이며, 운영체제가 블록마다 순번을 매겨 주소를 정하는 LBA 방식을 쓴다.[^12] 블록 레벨 복제는 블록을 복사하는 복제 방식이다. 하나는 저장소의 종류이고 다른 하나는 복제하는 층이므로 서로 다른 축이다. 블록 스토리지 위의 볼륨을 파일 레벨 도구로 복사할 수도 있다(작성자의 정리). 저장소는 [[Block Storage]]와 [[File Storage]]에서 다룬다.

## 6. 일관성

Azure Site Recovery 문서에서 크래시 일관성(crash-consistent)은 전원이 갑자기 끊긴 직후와 같은 디스크 상태이고, 애플리케이션 일관성(application-consistent)은 여기에 메모리의 데이터와 진행 중인 트랜잭션까지 담은 상태이다.[^13] 후자는 Windows에서 VSS 같은 프레임워크로 애플리케이션을 정지해 만든다고 한다.[^13] Storage Replica는 모든 복제 모드에서 크래시 일관성을 보장한다고 밝힌다.[^6] 아래 층일수록 애플리케이션의 내부 상태를 알 수 없어서 애플리케이션 일관성은 상위 계층의 협조에 기댄다는 점은 작성자의 정리이다.

## 7. 선택할 때의 판단 기준

아래는 작성자의 정리이며 특정 제품의 권고가 아니다.

- 범위: 서버 전체와 OS이면 블록 레벨이나 하이퍼바이저 기반, 특정 데이터나 DB이면 파일 레벨이나 DB 레벨이 후보이다.
- 환경: VM뿐이면 하이퍼바이저 기반, 물리 서버가 섞이면 호스트 OS 안에서 수행하는 방식이나 어레이 기반이 후보이다.
- 일관성: 트랜잭션이 중요하면 DB 자체 복제나 애플리케이션 일관성 지점을 먼저 본다.
- 목표: RPO와 RTO를 먼저 정한 뒤 동기, 비동기, 주기를 고른다.
- 대상 사본 접근: 복제 중에 대상을 읽어야 하면 제품의 대상 접근 제약을 확인한다.[^6]

## 8. 관련 문서

- [[Delta Sync]]

- [[Agentless Migration]]

- [[Server Migration Types]]

- [[Cutover]]

- [[Block Storage]]

- [[File Storage]]

## 9. 출처

[^1]: [StoneFly, Compare Array vs Host vs Hypervisor vs Network Based Replication](https://stonefly.com/blog/array-vs-host-vs-hypervisor-vs-network-based-replication/). 벤더 자료이다.
[^2]: [PostgreSQL, Logical Replication](https://www.postgresql.org/docs/current/logical-replication.html)
[^3]: [rsync(1) manual page](https://www.man7.org/linux/man-pages/man1/rsync.1.html)
[^4]: [Tridgell, Mackerras, The rsync algorithm (Technical Report)](https://rsync.samba.org/tech_report/node2.html)
[^5]: [LINBIT, The DRBD User's Guide (8.4)](https://linbit.com/drbd-user-guide/users-guide-drbd-8-4/). 개발사 자료이다.
[^6]: [Microsoft Learn, Storage Replica overview](https://learn.microsoft.com/en-us/windows-server/storage/storage-replica/storage-replica-overview). 벤더 자료이다.
[^7]: [VMware, vSphere Replication Technical Overview](https://www.vmware.com/docs/vsphere-replication-technical-overview). 벤더 자료이다.
[^8]: [Microsoft Learn, Set up Hyper-V Replica](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/manage/set-up-hyper-v-replica). 벤더 자료이다.
[^9]: [AWS, Elastic Disaster Recovery, Agent related FAQ](https://docs.aws.amazon.com/drs/latest/userguide/Agent-Related-FAQ.html). 벤더 자료이다.
[^10]: [NetApp, SnapMirror disaster recovery data transfer](https://docs.netapp.com/us-en/ontap/concepts/snapmirror-disaster-recovery-data-transfer-concept.html). 벤더 자료이다.
[^11]: [NetApp KB, What does volume level SnapMirror replicate](https://kb.netapp.com/onprem/ontap/dp/SnapMirror/What_does_volume_level_snapmirror_replicate). 벤더 자료이다.
[^12]: [AWS, Difference between block, file, and object storage](https://aws.amazon.com/compare/the-difference-between-block-file-object-storage/). 벤더 자료이다.
[^13]: [Microsoft Learn, Azure Site Recovery FAQ](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-faq). 벤더 자료이다.
[^14]: [AWS, What is disaster recovery](https://aws.amazon.com/what-is/disaster-recovery/). 벤더 자료이다.
[^15]: [AWS, Disaster recovery options in the cloud](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html). 벤더 자료이다.
[^16]: [AWS, Elastic Disaster Recovery, Failback](https://docs.aws.amazon.com/drs/latest/userguide/failback.html). 벤더 자료이다.
[^17]: [Microsoft Learn, robocopy](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy). 벤더 자료이다.
[^18]: [RFC 7530, Network File System (NFS) Version 4 Protocol](https://www.rfc-editor.org/rfc/rfc7530)
[^19]: [Microsoft Learn, SMB overview](https://learn.microsoft.com/en-us/windows-server/storage/file-server/file-server-smb-overview). 벤더 자료이다.
[^20]: [Broadcom, Workflow of incremental backup based on Changed Block Tracking](https://knowledge.broadcom.com/external/article/344418/workflow-of-incremental-backup-based-on.html). 벤더 자료이다.

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
