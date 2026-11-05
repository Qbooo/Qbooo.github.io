---
title: Memory
type: knowledge
domain: Compute
tags: [Hardware, OJT]
aliases: [메모리, 주기억장치, RAM]
parent:
status: growing
lang: ko
featured: false
created: 2026-11-05
updated: 2026-11-05
---

## 1. 개요

메모리(Memory)는 실행 중인 프로그램의 명령어와 데이터를 CPU가 접근할 수 있도록 보관하는 작업 공간이다.

이 문서에서는 서버의 주기억장치인 DRAM 기반 RAM을 중심으로 설명한다. 문서를 저장하는 [[Disk|디스크]]가 작업이 끝나도 내용을 보관하는 장소라면, RAM은 현재 계산에 필요한 내용을 펼쳐 놓는 공간이다. 데이터베이스는 자주 읽는 데이터 페이지를 RAM에 보관하고, 웹 서버는 처리 중인 요청·연결 상태·응답 버퍼를 RAM에 둔다. 파일이 디스크에 존재하는 것과 프로그램이 그 내용을 즉시 계산할 수 있는 것은 다르다.

![DDR4 ECC RDIMM 실물 사진](https://upload.wikimedia.org/wikipedia/commons/d/d6/Two_8_GB_DDR4-2133_ECC_1.2_V_RDIMMs_%28straightened%29.jpg)

기판 위 DRAM 칩과 하단 접점이 보이는 DDR4-2133 ECC RDIMM 예시이다. 사진: Dsimic, 잘라 정렬한 판본: Kjerish, [원본 파일 페이지](https://commons.wikimedia.org/wiki/File:Two_8_GB_DDR4-2133_ECC_1.2_V_RDIMMs_(straightened).jpg), [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). 이 문서에서는 추가 편집 없이 사용했다.

## 2. RAM과 주소 공간의 구분

RAM 칩은 보통 DIMM이라는 기판 형태로 서버의 메모리 슬롯에 장착된다. DIMM은 모듈의 형태이고 DRAM은 그 위에서 데이터를 저장하는 반도체 기술이다. CPU 안의 캐시, GPU의 VRAM, 디스크 저장 공간은 위치와 역할이 다르다. 모두 데이터를 보관하지만 용량을 합쳐 하나의 RAM처럼 사용할 수 있는 것은 아니다.

프로그램이 다루는 주소도 RAM 칩의 위치와 바로 같지는 않다. 현대 운영체제는 프로세스에 가상 주소 공간을 제공하고, CPU의 주소 변환 장치와 페이지 테이블을 통해 실제 메모리 페이지에 연결한다. 이 계층은 프로그램끼리 주소를 격리하고 필요한 페이지만 RAM에 배치할 수 있게 한다. 가상 메모리는 '디스크를 RAM처럼 무한히 쓴다'는 기능만을 뜻하지 않는다.[^concepts]

예를 들어 프로그램이 큰 주소 영역을 예약했어도 그 모든 영역에 실제 RAM이 즉시 배정된 것은 아닐 수 있다. VSZ 같은 가상 주소 크기와 RSS 같은 현재 상주 메모리 크기가 다른 이유이다. 공유 페이지는 여러 프로세스가 함께 참조할 수 있으므로 프로세스 RSS를 단순 합산한 값도 시스템 전체 물리 사용량과 일치하지 않을 수 있다.[^proc]

## 3. CPU에서 DIMM까지의 경로

CPU가 데이터를 읽으면 먼저 캐시 계층에서 필요한 내용을 찾는다. RAM 접근이 필요하면 메모리 컨트롤러가 주소를 해석하여 메모리 채널을 통해 해당 DIMM의 DRAM에 요청한다. 채널은 CPU와 메모리 사이의 데이터 이동 경로이고, rank는 모듈에서 함께 선택되는 DRAM 칩 집합을 구분하는 개념이다. DIMM의 칩 개수나 양면 여부만으로 rank를 확정할 수는 없다.[^guide][^edac]

DRAM은 전하로 정보를 보관하므로 내용을 유지하기 위한 주기적인 재충전(refresh)가 필요하다. 일반적인 서버 RAM은 전원이 끊기면 작업 내용을 유지하지 못한다. CPU 가까운 캐시는 더 빠른 접근을 제공하지만 비용과 면적 때문에 RAM 전체를 대체하지 않는다. 이처럼 레지스터·캐시·DRAM·영속 저장장치의 계층은 속도·용량·비용의 차이를 조합한 구조이다.[^dram]

다중 소켓 서버에서는 메모리가 각 CPU 쪽에 나뉘어 연결될 수 있다. NUMA 구조에서는 실행 중인 CPU와 가까운 RAM에 접근하는 경우와 다른 CPU 쪽 RAM에 접근하는 경우의 비용이 다르다. 따라서 '총 RAM 512GB'가 같더라도 어느 CPU에 얼마나 연결되어 있고 작업이 어디에서 실행되는지가 성능에 영향을 준다.[^numa]

## 4. 용량과 대역폭이 다른 이유

용량은 한 번에 보관할 수 있는 작업 집합의 크기이고, 대역폭은 일정 시간에 이동할 수 있는 데이터의 양이다. 지연 시간은 개별 접근의 응답까지 걸리는 시간이다. 데이터가 RAM에 모두 들어가도 CPU가 요구하는 속도로 공급하지 못하면 메모리 대역폭에 묶일 수 있다. 반대로 작은 작업 집합에는 RAM 용량을 더 늘려도 효과가 거의 없을 수 있다.

DDR의 세대와 MT/s는 전송 기술과 전송률을 나타낸다. MT/s를 곧바로 프로그램 처리량으로 읽으면 안 된다. 실제 대역폭은 채널 수와 배치, 접근 패턴, 동시 요청, CPU의 지원 범위에 영향을 받는다. 한 채널에만 큰 DIMM을 넣은 구성과 여러 채널을 활용한 구성은 총 용량이 같아도 데이터 공급 능력이 다를 수 있다.[^population]

그렇다고 어떤 서버든 모든 슬롯을 채우는 것이 정답은 아니다. 채널당 DIMM 수와 rank 구성에 따라 지원 속도가 달라질 수 있고, CPU가 없는 소켓에 연결된 슬롯은 사용할 수 없는 모델도 있다. DDR4와 DDR5, UDIMM·RDIMM·LRDIMM의 지원 여부와 혼용 규칙은 서버별로 확인한다. Dell R650의 메모리 장착 규칙은 그 모델의 예시이며 다른 서버의 필수 배치 규칙으로 일반화하지 않는다.[^population][^guide]

## 5. 캐시와 부족한 메모리의 의미

Linux는 최근 읽은 파일 내용을 page cache에 보관한다. 같은 파일을 다시 읽을 때 디스크 접근을 줄이기 위해서이다. 쓰기 데이터도 page cache에 먼저 놓였다가 저장장치로 반영될 수 있다. 따라서 RAM 사용량이 높다고 모두 프로그램의 누수인 것은 아니다. 필요하면 회수할 수 있는 캐시와 프로그램이 계속 요구하는 작업 공간을 구분해야 한다. MemAvailable은 이런 회수 가능성 등을 고려해 새 작업에 사용할 수 있는 양을 추정한 값으로, 단순히 비어 있는 MemFree와 다르다.[^concepts][^proc]

페이지 폴트는 프로그램의 접근을 현재 페이지 매핑만으로 완료할 수 없어 운영체제의 처리가 필요한 사건이다. 모든 폴트가 디스크 읽기를 동반하지는 않는다. minor fault는 저장장치 읽기 없이 해결될 수 있고, major fault는 데이터 반입을 위한 I/O가 필요하다. swap은 일부 페이지를 저장장치로 내보내 RAM을 확보하지만 디스크의 지연 시간을 RAM 수준으로 바꾸지는 못한다.[^proc]

부족이 지속되면 페이지 회수와 재반입이 반복되어 실제 작업보다 데이터를 옮기는 데 시간을 쓰게 된다. 이때는 남은 용량뿐 아니라 reclaim·swap 활동과 memory pressure를 같이 보면 '할당된 양'과 '부족 때문에 멈춘 시간'을 구분할 수 있다. 컨테이너에서는 호스트에 여유 RAM이 있어도 cgroup의 제한 때문에 OOM이 발생할 수 있다. 전역 부족과 특정 작업의 제한 초과는 다른 조건이다.[^psi][^cgroup]

## 6. 오류 보호와 현대의 변화

ECC 메모리는 오류 검출·정정을 위한 정보를 함께 사용한다. 정정 가능한 오류와 정정 불가능한 오류는 다르며, ECC가 모든 종류의 데이터 손상을 막는 것은 아니다. CPU·메모리 컨트롤러·DIMM·펌웨어가 지원하는 보호 방식이 함께 맞아야 한다. DDR5의 on-die ECC는 DRAM 칩 내부 보호로, 메모리 경로 전체를 보호하는 시스템 수준 ECC와 동일하지 않다.[^ecc]

현대 서버의 메모리 요구는 프로그램 수뿐 아니라 데이터베이스 캐시, 많은 VM, 분석 작업, AI 데이터 준비 같은 작업 집합의 증가로 커진다. 동시에 CPU 코어가 늘면 코어들이 공유하는 메모리 공급 능력도 중요해진다. DDR 세대의 변화와 더 많은 채널은 이 요구에 대응하는 방식이며, 더 큰 용량 하나만으로 모든 병목이 해소되지는 않는다.

DIMM이 인식되지 않거나 오류가 누적될 때도 운영체제 수치만으로 모듈 불량을 확정하지 않는다. 서버의 하드웨어 오류 기록과 슬롯 위치·지원 구성을 함께 해석한다. 일반 DIMM을 실행 중 임의로 뽑는 방식은 메모리 확장 방법이 아니다. 메모리 hot-add·hot-remove는 하드웨어와 운영체제가 명시적으로 지원할 때만 가능한 별도 기능이다.[^service]

## 7. 관련 문서

- [[CPU]] · [[Server]] · [[Physical Server Configuration]]
- [[Disk]] · [[Database]] · [[GPU Farm]]

## 8. 출처

[^concepts]: Linux Kernel, [Memory concepts](https://docs.kernel.org/admin-guide/mm/concepts.html), 주소 변환·page cache·회수 설명.
[^proc]: Linux Kernel, [/proc Filesystem](https://docs.kernel.org/filesystems/proc.html), 프로세스 메모리와 시스템 메모리 통계.
[^numa]: Linux Kernel, [Memory concepts: Nodes](https://docs.kernel.org/admin-guide/mm/concepts.html#nodes).
[^guide]: Dell, [PowerEdge R650 Technical Guide](https://i.dell.com/sites/csdocuments/product_docs/en/poweredge-r650-technical-guide.pdf), 메모리 채널·rank·지원 구성의 제품 예시.
[^population]: Dell, [R650 memory installation guidelines](https://www.dell.com/support/manuals/en-us/poweredge-r650/per650_ism_pub/general-memory-module-installation-guidelines?guid=guid-6db30d04-e447-48c6-8748-d6389f356fdb&lang=en-us).
[^dram]: IBM, [Dynamic random-access memory](https://www.ibm.com/history/dram), DRAM의 전하 저장·재충전 원리와 등장 배경.
[^edac]: Linux Kernel, [EDAC: Main concepts](https://docs.kernel.org/driver-api/edac.html), DIMM·channel·chip-select row/rank 및 오류 보호의 정의.
[^psi]: Linux Kernel, [Pressure Stall Information](https://docs.kernel.org/accounting/psi.html).
[^cgroup]: Linux Kernel, [Control Group v2: Memory](https://docs.kernel.org/admin-guide/cgroup-v2.html#memory).
[^ecc]: Micron, [DDR5 DRAM](https://www.micron.com/products/memory/dram-components/ddr5-sdram), on-die ECC와 CRC 등 보호 범위 구분. Cisco, [Memory RAS Features](https://www.cisco.com/c/en/us/products/collateral/servers-unified-computing/ucs-m7-m8-memory-ras-overview-wp.html)의 System-level ECC와 On-die ECC 절 함께 참고.
[^service]: Dell, [PowerEdge R650 Installation and Service Manual](https://www.dell.com/support/manuals/en-us/poweredge-r650/per650_ism_pub/), 메모리 정비·전원 및 정전기 방지 절차 예시.

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
