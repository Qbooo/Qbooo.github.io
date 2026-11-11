---
title: Server Migration Types
type: knowledge
domain: Cloud
tags: []
aliases:
  - 서버 마이그레이션 유형
  - P2V
  - V2V
  - P2P
  - V2P
  - P2C
  - V2C
  - C2P
  - C2V
  - C2C
parent:
status: growing
lang: ko
featured: false
created: 2026-11-05
updated: 2026-11-05
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 약어의 의미|2. 약어의 의미]]
- [[#3. 유형과 이전 전략의 차이|3. 유형과 이전 전략의 차이]]
- [[#4. 물리가 포함된 이전에서 부팅이 실패하는 이유|4. 물리가 포함된 이전에서 부팅이 실패하는 이유]]
- [[#5. 가상 디스크 형식과 변환|5. 가상 디스크 형식과 변환]]
- [[#6. 이전을 수행하는 방식|6. 이전을 수행하는 방식]]
- [[#7. 약어를 읽을 때 주의할 점|7. 약어를 읽을 때 주의할 점]]
- [[#8. 관련 문서|8. 관련 문서]]
- [[#9. 출처|9. 출처]]

## 1. 개요

서버 마이그레이션 유형은 이전의 원본과 대상이 어떤 종류의 환경인지를 P2V, V2V처럼 약어로 구분해 부르는 표기이다.

앞 글자는 원본, 뒤 글자는 대상이며 가운데 2는 to를 뜻한다. P는 Physical(물리 서버), V는 Virtual(가상 머신), C는 Cloud(클라우드)이다. 예를 들어 PlateSpin Migrate 문서는 P2V를 Physical to Virtual이라고 풀어 쓴다.[^4] 택배에 견주면 약어는 보내는 주소와 받는 주소만 알려 줄 뿐, 짐을 어떻게 다루는지는 말해 주지 않는다.

## 2. 약어의 의미

| 약어 | 원본 → 대상 | 설명 |
|---|---|---|
| P2P | 물리 → 물리 | 물리 서버의 내용을 다른 물리 서버로 옮김 |
| P2V | 물리 → 가상 | 물리 서버의 운영체제, 애플리케이션, 데이터를 분리해 가상 머신으로 옮김[^1] |
| V2V | 가상 → 가상 | 한 가상 머신을 다른 가상화 플랫폼이나 호스트의 가상 머신으로 옮김 |
| V2P | 가상 → 물리 | 가상 머신의 내용을 물리 서버로 옮김 |
| P2C | 물리 → 클라우드 | 물리 서버를 클라우드로 옮김 |
| V2C | 가상 → 클라우드 | 가상 머신을 클라우드로 옮김 |
| C2P | 클라우드 → 물리 | 클라우드의 워크로드를 물리 서버로 되돌림 |
| C2V | 클라우드 → 가상 | 클라우드의 워크로드를 가상 머신으로 되돌림 |
| C2C | 클라우드 → 클라우드 | 한 클라우드에서 다른 클라우드로 옮김 |

PlateSpin Migrate 문서는 이 아홉 가지를 모두 나열한다. P2V, V2V, V2P, P2P를 한 묶음으로, P2C, V2C, C2P, C2V, C2C를 클라우드 묶음으로 나누어 보여 준다.[^4] 다만 이 문서는 약어를 따로 정의하지 않고 풀어 쓴 이름을 표에 나열할 뿐이므로, 약어의 의미는 풀어 쓴 이름에서 읽는다. V2P는 백업을 다른 장비에 복구하는 방식과 절차가 겹치는 제품도 있다.[^3]

## 3. 유형과 이전 전략의 차이

유형 약어는 어디에서 어디로 옮기는지를 말하고, 이전 전략은 옮긴 뒤 애플리케이션을 얼마나 바꾸는지를 말한다. 서로 다른 축이다.

AWS의 이전 전략 문서는 Rehost를 애플리케이션을 바꾸지 않고 그대로 옮기는 lift and shift로, Replatform을 옮기면서 일정 수준의 최적화를 더하는 방식으로, Refactor를 클라우드 고유 기능을 활용하도록 아키텍처를 고치는 방식으로 설명한다.[^9] 같은 택배 주소라도 포장을 그대로 보낼 수도 있고 풀어서 다시 쌀 수도 있듯이, 같은 P2C라도 Rehost일 수 있고 Replatform일 수 있다. 따라서 약어만 보고 이전 전략을 짐작해서는 안 된다. 전략의 이름과 구분은 [[Lift and Shift]]에서 다룬다.

## 4. 물리가 포함된 이전에서 부팅이 실패하는 이유

물리 서버의 운영체제는 설치될 때 그 장비의 디스크 컨트롤러와 네트워크 카드에 맞는 드라이버를 함께 받아 둔다. 이 상태를 가상 머신으로 그대로 옮기면 전혀 다른 가상 장치가 달려 있어서 운영체제가 자기 디스크를 찾지 못하고 멈출 수 있다. 이사 간 집에 예전 집 열쇠를 가져간 것과 같아서, 열쇠인 드라이버가 새 자물쇠인 가상 장치에 맞지 않으면 문이 열리지 않는다.

부팅 방식도 문제가 된다. 컴퓨터가 켜질 때 가장 먼저 동작하는 펌웨어에는 오래된 BIOS와 새로운 UEFI가 있으며, 운영체제는 설치될 때 둘 중 하나에 맞춰 준비된다. Hyper-V의 1세대 가상 머신은 BIOS, 2세대는 UEFI를 쓰고, 2세대는 Secure Boot가 기본으로 켜져 있으며, 만든 뒤에는 세대를 바꿀 수 없다.[^5] 부팅 가능한 2세대용 디스크를 1세대 가상 머신에 붙이면 부팅하지 못한다.[^5] AWS의 가져오기 기능도 UEFI와 Legacy BIOS 두 가지 부팅 모드를 지원하며, Windows 11은 UEFI를 요구한다.[^6]

그래서 이전 도구는 대상 환경의 부팅과 드라이버 요구에 맞게 원본을 변환하는 기능을 제공하기도 한다.[^2] 도구마다 지원 범위가 다르다. 예를 들어 AWS의 가져오기 기능은 P2V 변환으로 만들어진 가상 머신을 지원하지 않는다고 밝힌다.[^6]

## 5. 가상 디스크 형식과 변환

가상 디스크는 서버의 하드디스크를 파일 하나로 만든 것이며, 파일을 담는 규격이 플랫폼마다 다르다. VMware는 VMDK, Hyper-V는 VHD와 VHDX, QEMU는 QCOW2와 RAW 등을 사용한다.[^6][^8] Hyper-V 2세대 가상 머신의 부팅 디스크는 VHDX 형식이다.[^5]

대상 플랫폼이 읽지 못하는 형식이면 변환이 필요하다. AWS는 VMDK, VHD와 VHDX, RAW, OVA를 가져오고,[^6] Azure에 직접 올리는 디스크는 Azure용으로 준비한 고정 크기 VHD여야 한다.[^7] 형식 변환에는 `qemu-img convert` 같은 도구를 쓸 수 있으며, 이 도구는 VMDK, VHD, VHDX, QCOW2 등 여러 형식을 지원한다.[^8] 한국어 문서를 일본어 번역기에 넣으려면 번역기가 읽는 문서 형식으로 저장해야 하는 것과 비슷하다.

## 6. 이전을 수행하는 방식

P2V를 이전하는 방식은 수동, 반자동, 자동으로 나눈다.[^1] 이 분류는 P2V를 설명하는 문서에서 쓰인 것이며 다른 유형에 그대로 적용하지 않는다.

- 수동: 대상에 가상 머신을 만들고 원본의 운영체제, 애플리케이션, 데이터 파일을 직접 복사한다.
- 반자동: 전원이 켜진 Windows와 Linux 서버를 변환하도록 돕는 도구를 사용한다.
- 자동: 네트워크를 통해 서버를 옮기는 도구가 사용자 입력을 최소화해 수행한다.

이전 방식은 크게 둘로 나뉜다. 하나는 원본에서 대상으로 데이터를 직접 복제하는 방식이다. RackWare는 변경분만 전송하는 delta 방식 동기화로 전환 시간을 줄이고, 예약된 동기화와 전환을 지원한다고 소개한다.[^2] 다른 하나는 원본의 백업을 다른 장비에 복구해 옮기는 방식이다.[^3] 직접 복제에서 쓰는 [[Delta Sync]]와 최종 전환의 절차는 [[Cutover]]에서 다룬다.

## 7. 약어를 읽을 때 주의할 점

약어가 같아도 제품마다 묶음과 범위가 다를 수 있다. PlateSpin Migrate가 P2V, V2V, V2P, P2P를 한 묶음으로 두는 것처럼,[^4] 다른 제품도 같은 묶음과 범위로 쓴다고 가정하면 안 된다. 대상 서버를 이전 도구가 만들어 주는지 미리 준비된 서버로 이전하는지, 이전이 변환을 포함하는지는 약어만으로 알 수 없다. 사용하는 제품의 문서에서 해당 약어가 가리키는 범위를 확인해야 한다.

## 8. 관련 문서

- [[Rackware|RackWare]]

- [[Delta Sync]]

- [[Cutover]]

- [[Lift and Shift]]

- [[Agentless Migration]]

- [[Replication Levels]]

- [[Server]]

## 9. 출처

[^1]: [Wikipedia — Physical-to-Virtual](https://en.wikipedia.org/wiki/Physical-to-Virtual)
[^2]: [RackWare — Multi-Cloud Migration](https://www.rackwareinc.com/rackware-platform/migration). 벤더 소개 자료이다.
[^3]: [Acronis — Machine migration](https://www.acronis.com/en/support/documentation/AcronisCyberProtect_15/machine-migration.html)
[^4]: [PlateSpin Migrate 2019.8 — Understanding Workload Migration](https://cabs.microfocus.com/documentation/platespin/platespin-migrate-2019-8/migrate-user/mig-understanding.html)
[^5]: [Microsoft Learn — Should I create a generation 1 or 2 virtual machine in Hyper-V?](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/plan/should-i-create-a-generation-1-or-2-virtual-machine-in-hyper-v)
[^6]: [AWS — VM Import/Export requirements](https://docs.aws.amazon.com/vm-import/latest/userguide/prerequisites.html)
[^7]: [Microsoft Learn — Upload a VHD to Azure](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/upload-vhd)
[^8]: [QEMU — qemu-img](https://qemu-project.gitlab.io/qemu/tools/qemu-img.html)
[^9]: [AWS Prescriptive Guidance — About the migration strategies](https://docs.aws.amazon.com/prescriptive-guidance/latest/large-migration-guide/migration-strategies.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
