---
title: CPU
type: knowledge
domain: Compute
tags: [Hardware, OJT]
aliases: [중앙 처리 장치, Central Processing Unit]
parent:
status: growing
lang: ko
featured: false
created: 2026-11-05
updated: 2026-11-05
---

## 1. 개요

CPU(Central Processing Unit, 중앙 처리 장치)는 프로그램의 명령어를 해석하고 실행하여 컴퓨터의 계산과 제어를 수행하는 처리 장치이다.

웹 서버가 요청의 URL을 해석하고, 데이터베이스가 조건에 맞는 행을 비교하며, 운영체제가 다음에 실행할 프로세스를 정하는 일에는 CPU의 명령 실행이 필요하다. CPU가 모든 데이터를 직접 저장하거나 모든 입출력 신호를 직접 처리하는 것은 아니다. [[Memory|메모리]]에서 실행할 코드와 데이터를 가져오고 [[NIC]]·[[Disk]] 같은 장치 및 운영체제와 협력한다.

![Intel Xeon E5-1620 CPU 패키지 앞면과 뒷면](https://thumb.wikimedia.org/wikipedia/commons/thumb/3/3f/Intel_Xeon_E5-1620%2C_front_and_back.jpg/960px-Intel_Xeon_E5-1620%2C_front_and_back.jpg)

실제 CPU 패키지 사진이다. 금속 덮개와 접점이 보이며 내부의 코어나 트랜지스터 자체를 찍은 사진은 아니다. 구형 Xeon E5-1620의 예시로, 현대 CPU 전체의 접점·구조를 대표하지 않는다. 사진: Jud McCranie, [원본](https://commons.wikimedia.org/wiki/File:Intel_Xeon_E5-1620,_front_and_back.jpg), [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Commons에 게시된 사진을 추가 편집 없이 사용했다.

## 2. 패키지 안의 코어와 실행 자원

서버 메인보드의 소켓은 CPU 패키지를 장착하는 자리이다. 패키지에는 하나 이상의 반도체 다이와 여러 코어가 들어갈 수 있다. 코어는 명령 실행에 필요한 연산 장치와 제어 자원을 갖춘 실행 단위이다. 운영체제가 표시하는 'CPU 0, CPU 1'은 반드시 물리 패키지 두 개를 뜻하지 않는다. Linux는 패키지·코어·스레드의 관계를 별도 topology 정보로 제공한다.[^topology]

SMT(Simultaneous Multithreading)를 지원하는 코어는 여러 하드웨어 스레드의 실행 상태를 유지하면서 코어 자원을 활용한다. Intel의 Hyper-Threading은 그 구현 사례이다. 하드웨어 스레드 두 개가 보여도 독립된 물리 코어 두 개가 생기는 것은 아니며, 실행 장치와 캐시 등 일부 자원을 공유한다. 소프트웨어 스레드는 프로그램의 실행 흐름이고, 운영체제 스케줄러는 이를 사용 가능한 논리 CPU에 배치한다. 따라서 스레드를 많이 만든다고 연산 능력이 같은 비율로 늘지 않는다.[^ht]

VM의 vCPU는 게스트 운영체제가 보는 가상 실행 단위이다. 하이퍼바이저가 물리 CPU에서 실행 시간을 배정하므로 vCPU 개수와 전용 물리 코어 개수는 같은 개념이 아니다. 같은 호스트의 VM들이 실행 시간을 나누어 쓰는 구성에서는 호스트의 경쟁 상태도 VM 성능에 영향을 준다. 이 관계는 [[Server]]의 물리 자원과 가상 자원 구분으로 이어진다.

## 3. 명령과 데이터의 이동

단순화하면 CPU는 명령어를 가져오고(fetch), 의미를 해석하며(decode), 연산을 실행하고 결과를 반영한다. 현대 CPU는 명령을 여러 단계로 나누고 겹쳐 처리하며, 분기 예측이나 비순차 실행으로 실행 자원이 비는 시간을 줄인다. 프로그램에서 보이는 명령어 집합 ISA와 CPU 내부 구현은 구분해야 한다. 같은 ISA를 실행하는 CPU라도 명령을 처리하는 내부 구조와 실제 성능은 다를 수 있다.[^architecture]

레지스터는 실행 중인 값과 주소 등을 보관하는 매우 가까운 저장 공간이고, 캐시는 자주 참조하는 명령과 데이터를 CPU 가까이에 보관한다. L1·L2·L3라는 이름은 캐시 계층을 나타내지만 크기와 코어 간 공유 범위는 제품마다 다르다. 캐시에 필요한 데이터가 있으면 RAM까지 가지 않아도 된다. 없으면 하위 캐시나 메모리에서 가져와야 하므로 CPU는 데이터를 기다릴 수 있다. 데이터베이스의 큰 작업 집합이나 주소가 흩어진 자료구조에서는 이 대기가 중요한 병목이 될 수 있다.[^architecture]

다중 소켓 서버에서는 각 CPU에 연결된 메모리와 다른 CPU 쪽 메모리의 접근 비용이 다를 수 있다. NUMA는 이런 비균일 메모리 접근 구조를 설명한다. 실행 스레드는 한 CPU 쪽에 있는데 데이터가 멀리 배치되면 CPU 간 연결을 거쳐 접근한다. NIC나 NVMe 장치의 PCIe 연결 위치도 데이터 이동의 지역성과 관계가 있다. CPU·RAM·장치를 개별 사양의 합으로만 보면 놓치는 부분이다.[^numa]

## 4. CPU 성능을 결정하는 요소

GHz는 초당 클록 주기의 수이다. 명령 하나가 반드시 한 주기에 끝나는 것은 아니며, 한 주기에 여러 명령이 처리되거나 한 명령이 여러 주기를 기다릴 수도 있다. IPC는 주기당 처리한 명령 수를 나타내는 관점이지만 프로그램과 데이터 접근 패턴에 따라 달라진다. 서로 다른 세대의 CPU를 GHz 하나로 비교할 수 없는 이유이다.[^architecture]

병렬화할 수 있는 요청이 많으면 코어 수가 처리량에 도움을 준다. 반대로 하나의 직렬 실행 흐름이 전체 요청의 완료를 결정한다면 코어를 추가해도 그 부분은 빨라지지 않는다. 예를 들어 논리 CPU가 32개인 서버에서 한 스레드만 계속 실행되면 전체 평균은 낮아 보여도 해당 요청은 CPU 성능에 묶일 수 있다. 메모리 대역폭이나 잠금 경쟁이 병목이면 코어 증설이 오히려 경쟁을 늘릴 수도 있다.

Turbo는 여유 전력·전류·온도 등 조건에 따라 동작 주파수를 높이는 기능이다. 제품에 기재된 최대 Turbo 수치를 모든 코어가 모든 부하에서 계속 유지한다는 뜻으로 읽으면 안 된다. TDP나 Processor Base Power 역시 현재 순간 소비전력계의 값이 아니라 제품별 정의에 따른 열·전력 설계 기준이다. 전원과 냉각을 설계할 때는 CPU 수치뿐 아니라 서버 구성 전체의 제조사 사양을 함께 봐야 한다.[^turbo][^tdp]

## 5. CPU의 발전과 병렬 처리

마이크로프로세서는 CPU 기능을 집적회로에 구현하면서 컴퓨터의 크기와 구성 방식을 바꾸었다. Intel 4004는 초기 상용 마이크로프로세서의 대표 사례이다. 이후 발전은 단순한 주파수 상승만으로 진행되지 않았다. 더 많은 트랜지스터를 활용해 캐시와 여러 코어를 넣고, 메모리·입출력 제어 기능을 가까이 통합하는 방향도 중요해졌다.[^history]

클록과 전압을 높이면 전력과 발열 부담도 커진다. 그래서 현대 서버는 코어 수, 코어당 성능, 메모리 대역폭, 전력 효율을 함께 고려한다. SoC는 CPU 외 기능도 같은 칩이나 패키지에 통합하는 방식이며, GPU 같은 가속기는 특정 병렬 연산을 맡는다. 가속기가 있어도 CPU가 요청 제어·데이터 준비·장치 관리에 관여하므로 전체 시스템의 균형이 필요하다.[^frequency] [[AI Infrastructure Architect]]와 [[GPU Farm]]도 이 균형을 다룬다.

## 6. CPU 사용률의 의미

Linux의 user 시간은 사용자 프로그램 실행, system 시간은 커널 실행에 쓰인 시간을 구분한다. iowait는 입출력 대기가 있는 동안의 유휴 시간 회계와 관련하며 CPU가 디스크의 기계적 작업을 수행했다는 뜻이 아니다. 문서 자체도 다중 CPU 환경 등에서 iowait 해석에 한계가 있음을 설명한다. 가상 환경의 steal은 다른 실행을 위해 가상 CPU의 실행 기회가 지연된 시간과 관계가 있다.[^proc]

따라서 평균 사용률 하나보다 코어별 사용률, 실행을 기다리는 작업, 주파수와 온도, 메모리·I/O 대기 및 실제 응답 시간을 연결해 본다. CPU 자원 경합이 작업에 미친 시간은 Linux PSI 같은 관측 수단으로도 구분할 수 있다. 이는 특정 명령의 실행 지침보다 '계산이 부족한가, 데이터를 기다리는가, 실행 차례를 기다리는가'를 분리하기 위한 관점이다.[^psi]

CPU를 장착할 수 있는지는 소켓 모양만으로 결정되지 않는다. 지원 프로세서 목록, 보드·펌웨어, 전력과 방열 구성까지 맞아야 한다. 사진의 패키지를 다른 서버에 그대로 적용할 수 있다고 추론해서는 안 된다. 실제 정비는 해당 서버의 전원 분리·정전기 방지·방열판 작업 절차를 따른다.[^service]

## 7. 관련 문서

- [[Server]] · [[Physical Server Configuration]] · [[Memory]]
- [[NIC]] · [[Disk]] · [[AI Infrastructure Architect]] · [[GPU Farm]]

## 8. 출처

[^topology]: Linux Kernel, [CPU topology](https://docs.kernel.org/admin-guide/cputopology.html).
[^ht]: Intel, [Hyper-Threading Technology](https://www.intel.com/content/www/us/en/gaming/resources/hyper-threading.html).
[^architecture]: Intel, [Software Developer Manual, Volume 1: Basic Architecture](https://cdrdv2.intel.com/v1/dl/getContent/671436), Chapter 2, §2.2의 Intel 아키텍처·명령 처리와 superscalar·비순차 실행 설명. [매뉴얼 목록](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html).
[^numa]: Linux Kernel, [Memory concepts: Nodes](https://docs.kernel.org/admin-guide/mm/concepts.html#nodes).
[^turbo]: Intel, [Turbo Boost Technology overview](https://www.intel.com/content/www/us/en/support/articles/000007359/processors/intel-core-processors.html).
[^tdp]: Intel, [Thermal Design Power](https://www.intel.com/content/www/us/en/support/articles/000055611/processors.html).
[^history]: Intel, [Announcing a New Era of Integrated Electronics](https://www.intel.com/content/www/us/en/history/virtual-vault/articles/the-intel-4004.html).
[^frequency]: Linux Kernel, [CPU Performance Scaling](https://docs.kernel.org/admin-guide/pm/cpufreq.html).
[^proc]: Linux Kernel, [/proc Filesystem](https://docs.kernel.org/filesystems/proc.html), /proc/stat 필드와 iowait 주의사항.
[^psi]: Linux Kernel, [Pressure Stall Information](https://docs.kernel.org/accounting/psi.html).
[^service]: Dell, [PowerEdge R650 Installation and Service Manual](https://www.dell.com/support/manuals/en-us/poweredge-r650/per650_ism_pub/), 특정 서버의 지원 구성·정비 절차 예시.

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
