---
title: Delta Sync
type: knowledge
domain: Cloud
tags: []
aliases:
  - 델타 동기화
  - 변경분 동기화
parent:
status: growing
lang: ko
featured: false
created: 2026-11-05
updated: 2026-11-05
---

## 1. 개요

Delta Sync는 원본과 대상의 데이터를 맞출 때 이미 같은 부분은 다시 보내지 않고 달라진 부분만 전송하는 동기화 방식이다.

처음에 기준이 되는 사본을 만든 뒤, 이후의 갱신만 반복해서 반영하는 흐름으로 쓴다. 서버 마이그레이션과 재해 복구에서 전송량과 작업 시간을 줄이려고 사용한다.

이 문서의 Delta Sync는 원본에서 대상으로 향하는 단방향 복제를 다룬다. 여러 기기가 서로 변경을 주고받는 양방향 파일 동기화 서비스와는 구조가 다르다.

## 2. 전체 복사와 변경분 동기화

전체 복사는 매번 원본의 모든 데이터를 보낸다. 데이터가 클수록 시간이 오래 걸리고, 복사하는 동안에도 원본이 계속 바뀌므로 끝난 시점의 대상은 이미 오래된 사본이 된다.

변경분 동기화는 지난 동기화 이후 달라진 부분만 보낸다. 서버의 데이터는 대부분 지난번과 같으므로 전송량이 줄고, 같은 작업을 여러 번 반복해도 부담이 작다. 다만 무엇이 같은지 비교하려면 기준이 되는 사본이 먼저 있어야 하므로, 처음 동기화는 대상에 없는 데이터를 모두 보내는 작업이 된다.

## 3. 달라진 부분을 알아내는 방법

변경분을 보내려면 먼저 무엇이 바뀌었는지 알아야 한다. 방법은 크게 세 갈래이다. 책의 새 개정판을 비교하는 일에 견주면 이해하기 쉽다.

| 방법 | 비유 | 설명 |
|---|---|---|
| 크기와 수정 시각 비교 | 표지의 쪽수와 인쇄일만 확인 | 파일의 크기와 마지막 수정 시각이 같으면 같은 파일로 보고 건너뜀 |
| 체크섬 비교 | 쪽마다 내용을 대조 | 내용에서 계산한 값이 같은지 비교하므로 정확하지만 읽는 비용이 듦 |
| 변경 목록 기록 | 편집자가 고친 쪽을 적어 둔 메모 | 디스크 쪽에서 바뀐 영역의 목록을 따로 기록해 두고, 다음 동기화 때 그 목록만 조회 |

rsync는 기본적으로 크기나 수정 시각이 달라진 파일을 대상으로 삼고, 옵션을 주면 크기가 같은 파일도 체크섬으로 비교한다.[^2] 변경 목록을 기록하는 예로는 VMware의 CBT(Changed Block Tracking)가 있으며, 백업 도구가 이전 변경 ID 이후에 바뀐 디스크 영역을 질의해 받아 가는 방식이다.[^5]

## 4. rsync 알고리즘의 원리

rsync는 변경분 전송의 대표적인 사례이다. 두 파일을 한 장비에 모으지 않고도 차이를 계산하며, 대역폭이 낮고 지연이 큰 연결에서 특히 유용하도록 설계되었다.[^1]

동작은 수신 측이 먼저 시작한다. 수신 측은 가지고 있는 기존 파일을 같은 크기의 블록으로 나누고, 블록마다 빠르게 계산하는 약한 체크섬(롤링 체크섬)과 정확한 강한 체크섬을 송신 측에 보낸다. 송신 측은 자기 파일을 한 바이트씩 밀어 가며 같은 체크섬을 가진 블록을 찾고, 일치하는 구간은 "이 블록은 이미 있다"고 알리고 일치하지 않는 구간만 실제 데이터로 보낸다.[^1] 수신자가 "내가 가진 책의 쪽별 지문 목록"을 보내면 송신자가 새 책에서 같은 지문을 찾아 "이 쪽은 있으니 이 구간만 보내겠다"고 답하는 것과 같다.

단, 원본과 대상이 모두 같은 장비의 로컬 경로이면 rsync의 기본값은 파일 전체 복사이며, 이때는 위 방식을 쓰지 않는다.[^2] 즉 변경분만 보낸다는 이점은 네트워크로 연결된 환경에서 주로 나타난다.

## 5. 전송 단위와 제품별 표현

비교하고 전송하는 단위에 따라 방식을 나누어 볼 수 있다.

| 단위 | 전송 대상 | 예 |
|---|---|---|
| 파일 | 바뀐 파일 전체 | 크기와 수정 시각이 달라진 파일을 통째로 전송 |
| 디스크 블록 | 변경 목록에 오른 디스크 영역 | CBT처럼 디스크 쪽에서 바뀐 영역을 추적 |
| 파일 내부 구간 | 파일 안에서 일치하지 않는 구간 | rsync의 블록 매칭 |

세 번째 줄의 블록은 rsync가 비교용으로 나누는 블록이고, 두 번째 줄의 블록은 디스크에서 변경을 추적하는 단위여서 서로 다른 개념이다.

RackWare는 자사 페이지에서 delta sync를 byte-level이라고 소개하지만 구현 방식은 설명하지 않는다.[^6][^7] IBM Cloud의 RackWare 이전 튜토리얼은 같은 동작을 마지막 동기화 이후 변경된 파일만 전송하는 것으로 표현한다.[^8] 두 표현은 단위가 다르며, 공개 자료만으로는 실제 동작이 어느 쪽인지 확정할 수 없다. 사용하는 버전의 제품 문서에서 확인해야 한다.

## 6. 동기화 중의 일관성

운영 중인 서버는 동기화하는 동안에도 쓰기가 계속된다. 파일마다 서로 다른 시점의 내용이 섞이면 대상의 데이터가 어긋날 수 있다. 이를 줄이려면 어느 한 시점의 상태를 고정해서 읽어야 한다.

이 역할을 하는 것이 스냅샷이다. 스냅샷은 한 시점의 상태를 고정해 읽게 해 주는 기능이고, 무엇이 바뀌었는지 기록하는 변경 추적과는 역할이 다르다. Windows의 VSS는 writer가 열린 트랜잭션을 마무리하고 캐시를 디스크에 비운 뒤 쓰기를 조율한다.[^3] Linux의 LVM 스냅샷은 COW(Copy On Write) 방식으로, 볼륨 그룹의 여유 공간에서 변경된 블록을 저장할 공간을 할당받는다.[^4]

정전 직후 상태를 찍은 사진과, 장부를 정리하고 펜을 내려놓은 뒤 찍은 사진은 다르다. 전자는 복원은 되지만 쓰던 도중의 내용이 남아 있을 수 있고, 후자는 애플리케이션이 데이터를 정리한 상태이다. 애플리케이션 수준의 일관성이 필요하면 스냅샷만으로 충분하지 않을 수 있어서, IBM 튜토리얼은 최종 동기화 전에 애플리케이션을 정지하라고 안내한다.[^8] 이 마지막 단계가 [[Cutover]]이다.

## 7. 복제 방식과 RPO, RTO

복제에는 정해진 주기로 변경분을 보내는 방식과, 변경을 거의 실시간으로 계속 보내는 방식이 있다. 주기가 짧거나 연속적일수록 대상 사본이 원본에 가까워진다.

RPO(Recovery Point Objective, 복구 시점 목표)는 장애가 났을 때 허용하는 최대 데이터 손실 시간이고, RTO(Recovery Time Objective, 복구 시간 목표)는 복구에 걸려도 되는 시간이다. 복제 주기가 짧을수록 RPO가 작아진다.

RackWare는 DR 자료에서 연속 복제와 근접 0에 가까운 RPO를 내세우고, 시각을 정하는 예약 실행과 체크포인트 및 재시도 로직을 통한 자동 재개도 함께 소개한다.[^7] 구체적인 수치와 동작 조건은 해당 페이지에 나와 있지 않다.

## 8. 마이그레이션과 재해 복구에서의 쓰임

마이그레이션에서는 최종 전환 전에 Delta Sync를 여러 번 반복해 원본과 대상의 차이를 줄인다. 마지막에 옮길 변경분이 작을수록 전환에 필요한 시간이 짧아질 수 있다.[^6][^8]

재해 복구에서는 복구용 사본을 원본에 가깝게 유지하는 데 같은 방식을 쓴다. 장애 후 원래 환경으로 되돌리는 Fallback에도 변경분만 반영하는 방식이 쓰이는지는 제품과 구성에 따라 다르므로 해당 제품 문서에서 확인해야 한다.

## 9. 한계와 유의점

- 처음 동기화는 기준 사본을 만드는 작업이라 데이터를 모두 보내야 한다.
- 변경량이 많으면 변경분도 커져서 이점이 줄어든다.
- 비교와 변경 추적에는 계산 자원이 들 수 있다. 벤더는 운영 중단 없는 방식임을 강조하지만, 실제 부하 정도는 공개 자료에서 확인되지 않는다.
- 단방향 복제에서는 원본에서 지운 파일이 대상에 남을 수 있다. rsync는 이를 위해 대상의 불필요한 파일을 지우는 옵션을 따로 둔다.[^2]
- 변경분만 보낸다는 사실이 무중단 전환을 뜻하지는 않는다. 마지막 동기화와 애플리케이션 정지, 네트워크 전환은 여전히 필요하다.

## 10. 관련 문서

- [[Cutover]]

- [[Rackware|RackWare]]

- [[Server Migration Types]]

- [[Replication Levels]]

- [[Agentless Migration]]

## 11. 출처

[^1]: [Tridgell, Mackerras — The rsync algorithm (Technical Report)](https://rsync.samba.org/tech_report/node2.html)
[^2]: [rsync(1) manual page](https://www.man7.org/linux/man-pages/man1/rsync.1.html)
[^3]: [Microsoft Learn — Volume Shadow Copy Service](https://learn.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service)
[^4]: [lvcreate(8) manual page](https://man7.org/linux/man-pages/man8/lvcreate.8.html)
[^5]: [Broadcom — Workflow of incremental backup based on Changed Block Tracking](https://knowledge.broadcom.com/external/article/344418/workflow-of-incremental-backup-based-on.html)
[^6]: [RackWare — Multi-Cloud Migration](https://www.rackwareinc.com/rackware-platform/migration). 벤더 소개 자료이다.
[^7]: [RackWare — Disaster Recovery and Backup](https://www.rackwareinc.com/rackware-platform/disaster-recovery). 벤더 소개 자료이다.
[^8]: [IBM Cloud — Migrating VMware VCF Classic to IBM Cloud VPC by using RackWare RMM](https://cloud.ibm.com/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-rmm-tutorial). IBM Cloud 대상의 구체적인 예시이다.

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
