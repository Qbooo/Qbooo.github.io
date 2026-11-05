---
title: Agentless Migration
type: knowledge
domain: Cloud
tags: []
aliases:
  - 에이전트리스 마이그레이션
  - 에이전트 기반 마이그레이션
  - Agent-based Migration
  - 에이전트 방식 마이그레이션
  - Agentless
parent:
status: growing
lang: ko
featured: false
created: 2026-11-05
updated: 2026-11-05
---

## 1. 개요

Agentless Migration은 원본 서버의 운영체제 안에 복제용 소프트웨어를 설치하지 않고 서버를 옮기는 방식이다. 반대편의 에이전트 방식(Agent-based Migration)은 옮길 서버마다 소프트웨어를 설치해 그 서버의 데이터를 읽고 전송하게 한다.

이 문서에서 agent는 복제 대상 서버의 OS 안에 설치되어 디스크 데이터를 읽고 전송하는 소프트웨어를 뜻하며, 생성형 AI의 에이전트와는 관련이 없다. 이 정의는 Azure Migrate의 Mobility service 설명을 바탕으로 한 작성자의 정리이다.[^1]

건물을 점검하는 두 방법에 견주면, 에이전트 방식은 건물에 직원을 상주시켜 상황을 보고받는 것이고 에이전트리스 방식은 관리사무소가 건물 밖에서 정해진 시간마다 사진을 찍어 가져가는 것에 가깝다. 이 비유는 작성자의 정리이다. 한계가 있다. 에이전트리스라고 해서 소프트웨어가 필요 없는 것은 아니며, 사진을 찍는 장비에 해당하는 스냅샷과 별도의 어플라이언스나 커넥터가 필요하다. 없는 것은 게스트 OS 안의 설치뿐이다.

## 2. 용어 정리

- 에이전트 방식: 서버마다 복제용 소프트웨어를 설치하는 방식이다.
- 에이전트리스 방식: 게스트 OS 안에 복제용 소프트웨어를 설치하지 않는 방식이다. 하이퍼바이저나 클라우드 쪽 기능을 이용하는 경우가 많다.
- replication appliance와 process server: Azure Migrate에서 온프레미스에 두는 복제 중계 장비와, 그 안에서 서버가 보낸 데이터를 받아 압축하고 암호화해 전송하는 구성 요소이다.[^1]
- 커넥터, 어플라이언스: 에이전트리스 방식에서 vCenter 등에 접근해 복제를 수행하는 구성 요소이며 제품마다 이름이 다르다.

## 3. 에이전트 방식의 작동 원리

에이전트 방식에서는 원본 서버의 OS 안에서 에이전트가 디스크 데이터를 읽는다. Azure Migrate와 AWS 문서가 설명하는 에이전트 방식은 파일이 아니라 디스크 블록 단위로 복제한다. 모든 제품이 그렇다는 뜻은 아니다.

Azure Migrate의 agent 기반 방식은 공개 문서 기준으로 다음과 같다.[^1]

- 복제를 켜면 초기 복제가 시작되고, Mobility service가 서버 디스크의 데이터를 읽어 process server로 보낸다.
- 초기 복제 뒤에는 블록 수준의 거의 연속적인 변경분 복제가 이어진다.
- Mobility service는 OS의 스토리지 서브시스템과 통합되어 디스크 쓰기를 가로채며, 문서는 이 방식이 증분 복제에서 복제 대상 서버의 디스크 I/O를 일으키지 않는다고 설명한다. 반면 초기 복제에서는 서버 디스크를 읽으므로 읽기 부하가 따른다.

AWS 규범 가이드는 원본 서버에 에이전트를 설치하고 압축과 암호화가 적용된 연속 블록 수준 복제를 사용하라고 안내하며, 데이터는 AWS 쪽 staging area의 replication server와 EBS 볼륨으로 모인다.[^2] 에이전트가 디스크를 읽는 구체적 방식은 확인하지 못했다.

## 4. 하이퍼바이저 기반 에이전트리스의 작동 원리

가상 서버는 하이퍼바이저가 가상 디스크를 관리하므로 게스트 OS 바깥에서 디스크 변경을 알 수 있다. 이 점을 이용하는 것이 하이퍼바이저 기반 에이전트리스이다.

### 4.1. VMware: 스냅샷과 변경 블록 추적

VMware의 CBT(Changed Block Tracking)는 가상 머신 단위로 켜는 기능이며, 백업 도구는 스냅샷을 만들고 직전 변경 ID 이후에 바뀐 디스크 영역을 질의해 받아 간다.[^7]

- Azure Migrate의 VMware 에이전트리스 복제는 VMware 스냅샷과 CBT를 사용한다. 한 주기는 디스크별 스냅샷 생성, 업로드, 스냅샷 해제, VMware 디스크 통합의 순서로 진행되며, 이 일은 온프레미스 appliance의 구성 요소가 맡는다.[^3] Azure 문서에는 appliance 하나가 디스크 52개를 동시에 복제하고 ESXi 호스트당 동시 복제 디스크는 8개라는 제한도 있다.[^3]
- Google Cloud의 Migrate to Virtual Machines는 소스 VM의 스냅샷과 CBT로 변경분만 가져오며, 온프레미스 vSphere 데이터 센터에 Migrate Connector를 설치하고 vCenter 사용자 권한을 넘겨야 한다.[^5][^6]
- AWS Application Migration Service에도 vCenter 기반 에이전트리스 복제가 있다. 원본 서버에 AWS Replication Agent를 설치하는 대신 vCenter 환경에 MGN vCenter Client를 설치하고 스냅샷 기반으로 복제한다.[^8] 복제 대상 VM에 기존 VMware 스냅샷이 없어야 하고, 복제 중 스냅샷 때문에 디스크 성능이 느려질 수 있다고 문서는 안내한다.[^9]

### 4.2. Hyper-V: 호스트에 구성 요소를 두는 방식

Hyper-V는 구조가 다르다. Azure Migrate는 Hyper-V 호스트나 클러스터 노드에 Azure Site Recovery provider와 Recovery Services agent를 설치하고, Hyper-V VM 안에는 아무것도 설치하지 않는다. 변경분은 CBT가 아니라 Hyper-V Replica가 만드는 로그 파일로 추적한다.[^4] 즉 에이전트리스는 게스트 OS 안에 설치하지 않는다는 뜻이지, 어디에도 설치하지 않는다는 뜻이 아니다.

## 5. 사례: SSH 접속 기반 제품의 표현

RackWare는 공개 페이지에서 자사를 에이전트리스로 소개한다. 마이그레이션 페이지는 원본이나 대상 시스템에 영구적인 소프트웨어 설치가 필요 없다고 쓰고,[^10] 재해 복구 페이지는 보호 대상 시스템에 영구 에이전트가 필요 없다고 쓴다.[^11] 이 페이지들은 서버에 접속하는 방법을 설명하지 않는다.

접속 방법은 IBM Cloud의 RackWare 이전 튜토리얼에서 확인된다. 이 튜토리얼에서 RMM(RackWare의 관리 서버)은 암호 없이 원본 서버에 SSH로 접속해야 하고, Linux에서는 전용 계정과 sudoers 설정, RMM의 공개 키를 준비한다. 반면 Windows는 RMM이 제공하는 작은 MSI를 원본 VM에 설치하도록 안내하며, 이 MSI는 SSHD 서비스를 위한 것이다.[^12]

따라서 RackWare 공개 페이지는 영구 설치가 필요 없다는 뜻으로 agentless를 쓰고, IBM 튜토리얼은 Windows 원본에 MSI 설치와 SSH 접속을 안내한다. 이 표기만으로 원본에 준비할 것이 없다고 단정할 수 없으며, 이는 두 자료에 한정한 정리이다. RackWare가 서버 안에서 읽는 단위(파일 또는 블록)는 공개 자료로 확정하지 못했다.

## 6. 비교

| 항목 | 에이전트 방식 | 하이퍼바이저 기반 에이전트리스 |
|---|---|---|
| 설치 | 서버마다 에이전트 설치[^1] | 게스트 안에는 불필요. 대신 appliance, 커넥터, 호스트 구성 요소가 필요[^3][^4][^6] |
| 지원 대상 | 물리 서버, 다른 클라우드의 서버, 하이퍼바이저 API를 쓸 수 없는 서버. Azure는 이 경우 에이전트 방식을 안내함[^13] | VMware와 Hyper-V. VMware는 스냅샷과 CBT, Hyper-V는 호스트의 Replica 로그가 변경 추적 수단[^3][^4] |
| 성능 영향 | 증분 복제에서 디스크 I/O를 일으키지 않는다고 Azure가 설명하나 초기 복제에서는 디스크를 읽음[^1] | 주기마다 스냅샷 생성과 해제, 디스크 통합이 따름[^3]. AWS는 스냅샷 때문에 디스크 성능이 느려질 수 있다고 안내[^9] |
| 접근과 방화벽 | 원본 서버가 process server(Azure는 HTTPS 9443)나 replication server(AWS는 TCP 1500)로 통신하는 경로가 필요[^1][^14] | 어플라이언스나 커넥터가 vCenter와 ESXi에 접근하는 경로와 vCenter 권한이 필요. Google은 TCP 443과 902를 안내[^6] |

무결성 검증과 복구 시점의 일관성은 다른 개념이다. Azure의 VMware 에이전트리스는 비트맵과 체크섬을 비교해 불일치하면 해당 주기를 실패로 처리하고 다음 주기에서 바로잡는다.[^3] 이는 전송 데이터의 무결성 검증이다. 일관성은 별도 개념으로, Azure Site Recovery 문서는 크래시 일관성을 서버의 전원 코드를 뽑은 시점의 디스크 상태에 해당하는 것으로, 애플리케이션 일관성을 여기에 메모리 데이터와 진행 중인 트랜잭션까지 담은 것으로 설명한다.[^15] 각 이전 도구가 어느 수준을 보장하는지는 제품 문서에서 확인해야 한다.

## 7. 서버리스와의 구분

Agentless와 Serverless는 이름이 비슷해 혼동되기 쉽지만 서로 다른 축이다. 서버리스에도 서버는 있고 그 서버를 제공자가 관리한다.[^16][^17] 사용자는 서버를 프로비저닝하거나 관리하지 않고 코드만 실행한다. AWS Lambda 문서는 서버 유지 보수, 용량 확보, 확장, 패치 같은 기반 인프라를 Lambda가 관리한다고 밝힌다.[^16]

반면 에이전트 유무는 서버를 옮길 때 원본 OS 안에 복제용 소프트웨어를 두는가의 문제이다. 에이전트리스 마이그레이션도 서버를 대상으로 하며 서버를 없애 주는 모델이 아니다. 이 단락의 구분은 위 문서들에 근거한 작성자의 정리이다.

## 8. 선택할 때의 판단 기준

아래는 공개 자료를 읽고 작성자가 정리한 기준이다.

- 원본이 하이퍼바이저 위의 가상 서버이고 스냅샷과 CBT를 쓸 수 있으며 vCenter 접근이 허용되면 에이전트리스를 먼저 검토한다. Azure 문서도 이 조건에서 에이전트리스를 권장한다.[^13]
- 원본이 물리 서버이거나 다른 클라우드의 서버, 하이퍼바이저 API를 쓸 수 없는 가상 서버이면 하이퍼바이저 기반 에이전트리스는 쓸 수 없고, Azure에서는 에이전트 방식을 쓴다.[^13]
- 서버 안에 소프트웨어를 설치할 수 없는 정책이 있으면, 설치가 필요 없는지뿐 아니라 대안이 요구하는 접근(SSH 계정, 포트, 하이퍼바이저 권한)이 허용되는지도 확인한다.
- 스냅샷을 쓰는 방식은 같은 VM에 기존 스냅샷이나 백업이 걸려 있으면 막힐 수 있다.[^9]
- 어느 쪽이든 마지막 전환은 [[Cutover]]가 필요하며, 에이전트 유무가 중단 없는 전환을 뜻하지는 않는다.

## 9. 관련 문서

- [[Replication Levels]]

- [[Server Migration Types]]

- [[Delta Sync]]

- [[Cutover]]

- [[Rackware|RackWare]]

## 10. 출처

[^1]: [Microsoft Learn, Agent-based migration architecture (Azure Migrate)](https://learn.microsoft.com/en-us/azure/migrate/agent-based-migration-architecture). 벤더 자료이다.
[^2]: [AWS Prescriptive Guidance, Migrate an on-premises VM to Amazon EC2 by using AWS Application Migration Service](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/migrate-an-on-premises-vm-to-amazon-ec2-by-using-aws-application-migration-service.html). 벤더 자료이다.
[^3]: [Microsoft Learn, Azure Migrate VMware agentless migration concepts](https://learn.microsoft.com/en-us/azure/migrate/concepts-vmware-agentless-migration). 벤더 자료이다.
[^4]: [Microsoft Learn, Hyper-V migration architecture (Azure Migrate)](https://learn.microsoft.com/en-us/azure/migrate/hyper-v-migration-architecture). 벤더 자료이다.
[^5]: [Google Cloud, Migrate to Virtual Machines lifecycle](https://docs.cloud.google.com/migrate/virtual-machines/docs/5.0/discover/lifecycle). 벤더 자료이다.
[^6]: [Google Cloud, Create a VMware source (Migrate to Virtual Machines)](https://docs.cloud.google.com/migrate/virtual-machines/docs/5.0/how-to/create-a-vmware-source). 벤더 자료이다.
[^7]: [Broadcom, Workflow of incremental backup based on Changed Block Tracking](https://knowledge.broadcom.com/external/article/344418/workflow-of-incremental-backup-based-on.html). 벤더 자료이다.
[^8]: [AWS, Agentless replication overview (Application Migration Service)](https://docs.aws.amazon.com/ja_jp/mgn/latest/ug/installing-vcenter-overview-mgn.html). 일본어 경로의 페이지이며 본문은 영어이다. 벤더 자료이다.
[^9]: [AWS, Installing the Application Migration Service vCenter Client](https://docs.aws.amazon.com/mgn/latest/ug/installing-vcenter-appliance-mgn.html). 벤더 자료이다.
[^10]: [RackWare, Multi-Cloud Migration](https://www.rackwareinc.com/rackware-platform/migration). 벤더 소개 자료이다.
[^11]: [RackWare, Disaster Recovery and Backup](https://www.rackwareinc.com/rackware-platform/disaster-recovery). 벤더 소개 자료이다.
[^12]: [IBM Cloud, Migrating VMware VCF Classic to IBM Cloud VPC by using RackWare RMM](https://cloud.ibm.com/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-rmm-tutorial). IBM Cloud 대상의 구체적인 예시이다.
[^13]: [Microsoft Learn, Azure Migrate server migration overview](https://learn.microsoft.com/en-us/azure/migrate/server-migrate-overview). 벤더 자료이다.
[^14]: [AWS, Network requirements (Application Migration Service)](https://docs.aws.amazon.com/mgn/latest/ug/Network-Settings-Video.html). 벤더 자료이다.
[^15]: [Microsoft Learn, Azure Site Recovery FAQ](https://learn.microsoft.com/en-us/azure/site-recovery/site-recovery-faq). 벤더 자료이다.
[^16]: [AWS Lambda, What is AWS Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html). 벤더 자료이다.
[^17]: [Google Cloud, What is serverless computing](https://cloud.google.com/discover/what-is-serverless-computing). 벤더 자료이다.

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
