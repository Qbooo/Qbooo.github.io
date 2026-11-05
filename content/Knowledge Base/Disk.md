---
title: Disk
type: knowledge
domain: Storage
tags: [Hardware, OJT]
aliases: [디스크, 저장 드라이브, HDD, SSD]
parent:
status: growing
lang: ko
featured: false
created: 2026-11-05
updated: 2026-11-05
---

## 1. 개요

디스크(Disk)는 운영체제와 프로그램이 사용할 데이터를 전원이 꺼진 뒤에도 보관하는 저장 장치이다.

이 문서에서는 서버의 HDD와 SSD를 함께 설명한다. SSD에는 회전하는 원판이 없지만 운영체제나 실무 표현에서는 둘을 묶어 디스크라고 부르기도 한다. RAM이 현재 실행을 위한 작업 공간이라면 디스크는 운영체제 파일, 데이터베이스, 로그와 사용자 파일을 지속적으로 보관한다. 클라우드의 가상 볼륨 뒤에도 실제 저장 매체와 이를 제어하는 시스템이 존재한다. 장치 자체와 서비스로 제공하는 저장 공간의 차이는 [[Storage]]와 [[Block Storage]]에서 이어진다.

![덮개를 연 HDD의 플래터와 헤드 암](https://upload.wikimedia.org/wikipedia/commons/6/6c/Harddisk_opened.png)

HDD 내부의 원형 플래터와 그 위를 이동하는 헤드 암이 보이는 실제 사진이다. SSD에는 이 회전 구조가 없다. 구조 설명을 위한 개봉 사진이며 운영 장치를 열어 사용한다는 뜻은 아니다. 사진: Erwan velu, [원본](https://commons.wikimedia.org/wiki/File:Harddisk_opened.png), [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/). 추가 편집 없이 사용했다.

## 2. HDD와 SSD의 저장 방식

HDD는 회전하는 플래터의 자성 상태로 정보를 기록하고 헤드를 이동시켜 읽고 쓴다. 원하는 위치로 이동하는 탐색 시간과 해당 부분이 헤드 아래에 올 때까지의 회전 대기가 필요하다. 인접한 데이터를 이어 읽는 작업과 여기저기 흩어진 작은 데이터를 읽는 작업의 성능이 크게 달라질 수 있는 이유이다.[^hdd]

NAND 플래시 기반 SSD는 기계적 이동 없이 반도체 셀에 정보를 보관한다. 그러나 운영체제에서 보이는 주소와 NAND의 실제 위치가 곧바로 같지는 않다. 컨트롤러의 FTL(Flash Translation Layer)이 논리 주소를 물리 위치로 연결한다. NAND는 보통 page 단위로 읽고 프로그램하며 더 큰 block 단위로 지우므로, 이미 쓴 내용을 수정할 때 새 위치에 기록하고 이전 내용을 무효화하는 관리가 필요하다.[^ssd]

무효 데이터와 유효 데이터가 섞인 블록을 정리하는 garbage collection, 쓰기 소모를 분산하는 wear leveling, 여유 영역을 확보하는 over-provisioning은 이 특성에 대응한다. 사용자 프로그램이 1GB를 썼어도 내부 데이터 이동 때문에 NAND에 기록한 양은 더 클 수 있다. SSD도 여유 공간·쓰기 패턴·내부 정리 상태에 따라 지속 성능이 달라지며 항상 일정한 응답 시간을 보장하는 것은 아니다.[^ssd]

## 3. 연결 규격과 장치 모양

HDD와 SSD는 저장 매체의 구분이다. SATA·SAS·NVMe는 장치와 통신하는 기술·명령 체계의 구분이고, 2.5인치·3.5인치·M.2·U.2 같은 말은 주로 물리 형태와 연결 구조를 설명한다. 예를 들어 SSD는 SATA 제품일 수도 있고 PCIe로 연결되는 NVMe 제품일 수도 있다. M.2라는 모양만 보고 NVMe 지원을 확정할 수 없다.[^interface]

SATA는 저장장치 연결에 널리 쓰였고 SAS는 엔터프라이즈 환경에서 사용되는 직렬 SCSI 계열이다. NVMe는 비휘발성 메모리 저장장치를 위해 설계한 명령 인터페이스이며 다수의 큐와 병렬 처리를 활용한다. 로컬 PCIe NVMe SSD에서는 CPU의 PCIe 경로로 연결되지만, NVMe over Fabrics처럼 네트워크를 통한 구성도 있다. 따라서 NVMe를 특정 모양의 작은 SSD나 단순한 속도 등급으로 이해하면 범위를 놓친다.[^nvme]

서버의 드라이브 베이(drive bay)에 들어간다고 모두 호환되는 것도 아니다. 백플레인(backplane)의 지원 규격, 컨트롤러, 배선, 펌웨어가 장치와 맞아야 한다. 같은 서버 전면에도 SAS/SATA용 구간과 NVMe용 구간이 나뉘는 구성이 있을 수 있다. 이 물리 연결은 [[Physical Server Configuration]]과 연결된다.

## 4. 파일 저장 요청이 매체에 도착하기까지

프로그램이 파일에 쓰면 일반적인 버퍼 기반 입출력(buffered I/O)에서는 먼저 운영체제의 페이지 캐시(page cache)에 데이터가 놓일 수 있다. 파일시스템은 파일의 위치와 메타데이터를 관리하고 논리 블록 요청을 만든다. 블록 계층은 요청을 큐에 넣거나 인접 요청을 병합하고, 드라이버는 장치·컨트롤러에 명령을 전달한다. Linux blk-mq는 CPU 쪽 소프트웨어 큐와 장치에 전달하는 하드웨어 큐를 구분해 병렬성을 활용한다.[^cache][^queue]

장치가 명령을 받았다는 것, 캐시에 기록했다는 것, 전원 상실 뒤에도 복구 가능한 상태가 되었다는 것은 서로 다른 완료 지점이다. fsync 같은 동기화 요청은 변경 내용을 저장장치에 반영하도록 요구하지만 실제 지속성에는 파일시스템·장치의 flush 처리와 전원 보호 특성도 관여한다. 데이터베이스가 WAL을 먼저 기록하는 이유도 갑작스러운 중단 뒤 일관된 상태로 복구하기 위해서이다. 파일 복사 완료 표시만으로 모든 응용프로그램 상태의 일관성을 증명할 수는 없다.[^fsync][^wal]

읽기는 반대 방향으로 진행하지만 캐시에 데이터가 있으면 물리 장치까지 가지 않는다. 애플리케이션에서 측정한 읽기 속도와 장치 자체의 속도가 다른 이유 중 하나이다. 네트워크 [[File Storage]]나 [[Object Storage]]는 여기에 서버·네트워크·API 계층이 더해진다.

## 5. 용량과 성능의 의미

1 GB는 1,000,000,000바이트이고, 1 GiB는 1,073,741,824바이트이다. 같은 바이트 수를 다른 단위로 표시하면 숫자가 달라진다. 실제 사용 가능 용량은 제조사가 표시한 장치 용량에서 RAID 보호, 예비 영역, 파일시스템 메타데이터 등의 영향을 받는다. SSD 내부의 over-provisioning과 파일시스템에서 보이는 빈 공간도 다른 계층이다.[^capacity]

성능에서는 IOPS가 초당 완료한 요청 수, throughput이 초당 데이터 양, latency가 요청 완료까지의 시간을 나타낸다. 작은 임의 읽기가 많은 데이터베이스와 큰 파일을 연속으로 읽는 작업은 같은 장치에서 다른 한계에 도달한다. 요청 크기가 작으면 대역폭을 다 쓰기 전에 요청 처리 능력에 묶일 수 있다. 큐 깊이(queue depth)를 늘리면 장치의 병렬성을 활용할 수 있지만, 이미 포화된 장치에서는 대기와 응답 시간만 늘릴 수 있다.[^queue]

SSD의 TBW는 정해진 조건에서 허용되는 누적 쓰기량을 설명하는 수치이고 DWPD는 보증 기간 동안 하루에 사용자 용량을 몇 번 쓸 수 있는지로 환산한 지표이다. 테스트 workload와 보증 조건을 함께 읽어야 한다. SMART나 NVMe health 정보는 마모·오류 관측에 도움을 주지만 정상 표시가 미래의 무고장을 보장하지는 않는다.[^endurance]

## 6. 영속 저장 기술의 변화와 보호

HDD의 기계적 지연을 줄이는 방향과 플래시의 병렬성을 활용하는 방향은 운영체제의 I/O 구조에도 영향을 주었다. Linux blk-mq 문서는 단일 큐 중심의 구조가 빠른 다중 큐 장치에서 소프트웨어 병목을 만들 수 있었던 배경을 설명한다. NVMe는 이와 함께 현대 SSD의 병렬성을 활용하는 중요한 인터페이스가 되었다. 그러나 대용량·비용·쓰기 특성이 다른 작업에서는 HDD도 사용되므로 SSD가 모든 HDD를 완전히 대체한 단선적인 변화는 아니다.[^queue][^nvme]

TRIM/discard는 사용하지 않는 논리 영역을 하위 장치에 알리는 기능이다. 파일 삭제가 즉시 모든 물리 셀의 삭제나 스토리지 풀의 공간 반환으로 이어진다는 뜻은 아니다. 하위 계층의 지원, thin provisioning, 스냅샷이 보존하는 이전 데이터에 따라 공간 회수 시점이 달라진다.[^ssd][^thin]

RAID는 여러 장치에 데이터를 배치해 용량·성능·일부 장치 장애 대응을 구성하는 방법이다. 장치 장애 후 rebuild에는 추가 읽기·쓰기와 시간이 필요하다. 같은 서버·랙의 전원 장애나 잘못된 삭제까지 자동으로 해결하지는 않는다. 스냅샷 역시 특정 시점의 상태를 보존하는 기능으로, 별도 장애 영역에 보관하는 백업과 목적이 다르다. 장치가 보이지 않는 문제, 파일시스템이 마운트되지 않는 문제, 사용 가능한 inode나 저장 공간이 소진된 문제는 이 계층 구분으로 이해할 수 있다.[^raid]

hot-plug 지원은 커넥터·컨트롤러·OS 등 구성의 조건이며, 아무 드라이브나 동작 중 제거해도 된다는 뜻은 아니다. RAID 멤버의 역할과 영구 식별자·슬롯을 구분하는 이유도 물리 부품 하나의 제거가 논리 볼륨 전체에 영향을 줄 수 있기 때문이다.

## 7. 관련 문서

- [[Storage]] · [[Block Storage]] · [[File Storage]] · [[Object Storage]]
- [[Memory]] · [[Database]] · [[VPSA]] · [[Physical Server Configuration]]

## 8. 출처

[^hdd]: Samsung Semiconductor, [HDD](https://semiconductor.samsung.com/support/tools-resources/dictionary/semiconductor-glossary-hard-disk-drive-hdd/), 플래터·헤드·스핀들 모터의 역할.
[^ssd]: Kingston, [SSD Power Loss Protection](https://www.kingston.com/unitedkingdom/en/blog/servers-and-data-centers/ssd-power-loss-protection), FTL의 논리·물리 매핑과 전원 상실 보호. Samsung Semiconductor, [SSD FAQ](https://semiconductor.samsung.com/consumer-storage/support/faqs/internalssd-product-information/)의 page/block·garbage collection·TRIM 설명. Kingston [SSD FAQ](https://www.kingston.com/en/ssd/ssd-faq)의 wear leveling 설명 함께 참고.
[^interface]: Kingston, [SATA and M.2 SSD FAQ](https://www.kingston.com/en/ssd/ssd-faq).
[^nvme]: NVM Express, [NVM Express overview](https://nvmexpress.org/resource/nvm-express-overview/).
[^cache]: Linux Kernel, [Memory concepts: Page cache](https://docs.kernel.org/admin-guide/mm/concepts.html#page-cache).
[^queue]: Linux Kernel, [Multi-Queue Block I/O](https://docs.kernel.org/block/blk-mq.html).
[^fsync]: Linux man-pages, [fsync(2)](https://man7.org/linux/man-pages/man2/fsync.2.html).
[^wal]: PostgreSQL, [Write-Ahead Logging](https://www.postgresql.org/docs/current/wal-intro.html).
[^capacity]: Seagate, [Storage capacity measurement standards](https://www.seagate.com/support/kb/storage-capacity-measurement-standards-194563en/).
[^endurance]: Kingston, [Drive Writes Per Day](https://www.kingston.com/en/ssd/dwpd). 상태 관측의 한계는 Seagate, [S.M.A.R.T. error](https://www.seagate.com/support/kb/my-system-reported-a-smart-error-on-the-drive-184619en/) 함께 참고.
[^thin]: Linux Kernel, [Thin provisioning](https://docs.kernel.org/admin-guide/device-mapper/thin-provisioning.html), thin volume·snapshot·discard 계층의 예시.
[^raid]: Linux Kernel, [RAID](https://docs.kernel.org/admin-guide/md.html), 소프트웨어 RAID 구성과 재구축의 예시.

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Storage 분류|Storage]]
