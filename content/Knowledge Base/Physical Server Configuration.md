---
title: Physical Server Configuration
type: knowledge
domain: Compute
tags: [Hardware, OJT]
aliases: [물리 서버 구성, 서버 하드웨어 구성]
parent:
status: growing
lang: ko
featured: false
created: 2026-11-05
updated: 2026-11-05
---

## 1. 개요

물리 서버 구성(Physical Server Configuration)은 CPU·메모리·저장장치·네트워크 장치와 전원·냉각을 하나의 서버 및 랙 시스템으로 연결하는 구조이다.

예를 들어 웹 서비스를 제공하는 서버는 요청을 받아들이는 NIC, 프로그램을 실행하는 CPU, 실행 내용을 담는 RAM, 파일을 보관하는 드라이브만으로 끝나지 않는다. 이 부품을 연결하는 메인보드, 안정된 전력을 공급하는 PSU, 열을 빼내는 냉각 장치가 함께 있어야 한다. 랙에 여러 서버를 넣으면 스위치·PDU·시설 전원·냉각까지 같은 시스템의 조건이 된다. [[Server]]가 서비스 역할과 실행 환경을 설명한다면 이 문서는 그 실행을 가능하게 하는 실물의 연결을 설명한다.

![덮개를 연 Dell 서버 내부](https://thumb.wikimedia.org/wikipedia/commons/thumb/3/33/Inside_of_webserver.jpg/960px-Inside_of_webserver.jpg)

덮개를 연 Dell 서버의 실제 내부 사진이다. 팬·방열 구조, 메모리 모듈과 드라이브 연결이 함께 들어가는 모습을 보여준다. 2013년에 촬영된 서버의 예시이며 아래에서 인용하는 R650과 같은 모델이라고 단정하지 않는다. 사진: arichnad, [원본](https://commons.wikimedia.org/wiki/File:Inside_of_webserver.jpg), [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). Commons 축소본을 사용했고 별도 주석이나 자르기는 하지 않았다.

## 2. 섀시 안에서 부품이 연결되는 방식

섀시(chassis)는 부품을 고정하고 공기 흐름과 정비 공간을 제공하는 몸체이다. 메인보드에는 CPU 소켓, DIMM 슬롯, PCIe 연결과 각종 제어 회로가 있다. CPU의 방열판은 칩에서 발생한 열을 전달하고 팬이나 냉각 장치는 그 열을 외부로 이동시킨다. 방열판이 보인다는 것과 CPU 패키지가 직접 보인다는 것은 다른 설명이다.[^inside]

CPU에 연결된 메모리 컨트롤러와 채널은 DIMM으로 이어진다. PCIe 경로에는 NIC, GPU, NVMe 장치 또는 저장장치 컨트롤러가 연결될 수 있다. 얇은 랙 서버에서는 riser가 확장 카드를 다른 방향으로 배치하도록 연결한다. 슬롯의 물리 길이만으로 지원 세대·lane 수·전력·장치 호환성을 모두 알 수는 없다.[^technical]

전면 드라이브 베이(drive bay)는 드라이브를 장착하는 자리이다. 드라이브가 꽂히는 백플레인(backplane)은 전원과 데이터 연결을 모아 주며, 구성에 따라 SAS/SATA 컨트롤러나 CPU의 PCIe 경로 등으로 이어진다. RAID 컨트롤러가 여러 드라이브를 하나의 논리 장치로 제공하는 구성도 있고, OS가 개별 NVMe 장치를 직접 보는 구성도 있다. 모든 저장 요청이 반드시 동일한 RAID 카드 하나를 거친다는 뜻은 아니다.[^inside][^technical]

PSU(Power Supply Unit)는 입력 전원을 서버가 사용하는 전력으로 변환한다. 보드의 전압 조절 회로는 CPU 등 각 부품의 동작 조건에 맞게 전압을 공급한다. BMC(Baseboard Management Controller)는 호스트 CPU와 별도로 하드웨어 상태와 관리 기능을 담당하는 제어 장치이다. Dell의 iDRAC, HPE의 iLO는 이런 서버 관리 기능의 제품 사례이다.[^management]

## 3. 요청 하나가 서버 내부를 지나는 과정

웹 요청을 처리하는 단순한 예를 보면, 네트워크로 들어온 데이터는 NIC와 드라이버를 거쳐 RAM의 버퍼와 운영체제 네트워크 스택으로 전달된다. CPU가 서버 프로그램을 실행하며 요청을 해석하고, 필요한 파일이나 데이터가 RAM에 없으면 저장장치 읽기를 요청한다. 읽은 데이터로 응답을 만든 뒤 NIC를 통해 내보낸다. [[Web Server]], [[Web Application Server]], [[Database]]는 같은 하드웨어 위에서 서로 다른 역할을 수행할 수 있다.

이 경로에서 각 부품은 고립되어 있지 않다. 데이터가 CPU와 가까운 RAM에 있는지, NIC와 저장장치가 어느 CPU의 PCIe 경로에 연결되는지, 여러 작업이 같은 경로를 공유하는지에 따라 결과가 달라진다. 다중 소켓 NUMA 서버에서 CPU 수와 총 RAM 용량이 같은 두 구성이 항상 같은 성능을 내는 것은 아니다.[^numa]

전원이 공급된 뒤에도 곧바로 서비스가 실행되는 것은 아니다. 보통 펌웨어가 하드웨어를 초기화하고 POST를 수행한 다음 부트 장치를 선택한다. 부트로더와 OS 커널이 올라오고, 드라이버와 서비스가 준비되어야 실제 요청을 처리한다. BMC는 별도 관리 경로와 대기 전원을 이용할 수 있으므로 호스트 OS가 꺼져 있어도 관리 화면이 동작할 수 있다. 반대로 BMC 접속 성공만으로 OS나 서비스가 정상이라는 결론은 내릴 수 없다.[^boot][^management]

## 4. 랙이 묶어 주는 네트워크와 전원

| 구성 요소 | 배치 예시와 역할 |
|---|---|
| 랙 서버 | 레일에 고정하며 CPU·메모리·드라이브·NIC 등으로 프로그램 실행 |
| 네트워크 스위치 | 서버의 케이블을 모아 다른 서버·상위 네트워크와 연결 |
| PDU | 랙 안의 여러 장비에 전원 분배 |
| UPS·상위 전원 | 랙 내부형 또는 시설 공용 구성으로 전원 유지·공급 지원 |
| 케이블·냉각 구성 | 데이터·전원 연결과 장비의 열 배출 지원 |

모든 랙에 같은 장비가 하나씩 들어가는 것은 아니다. 스위치나 UPS를 여러 랙이 공유할 수도 있고, 아래 사진은 그중 한 배치 예시이다.

랙 서버는 일정한 높이 규격으로 장비를 쌓아 설치한다. U는 높이를 나타내는 단위이며 1U는 1.75인치이다. 같은 1U 장비라도 깊이·무게·레일·케이블 여유 공간은 다르므로 높이만으로 랙 호환성을 판단할 수 없다. tower는 독립된 세로형 섀시이고, blade는 여러 서버 모듈이 enclosure의 전원·냉각·연결 자원을 공유하는 형태이다. 어떤 형태든 실제 연결과 공유하는 장애 영역을 함께 봐야 한다.[^rack]

ToR(Top-of-Rack) 스위치는 랙의 서버 연결을 모아 상위 네트워크로 전달하는 역할이다. 이름과 달리 실제 위치는 설계에 따라 달라질 수 있다. 서버의 NIC에서 ToR로 가는 케이블, ToR의 uplink, 상위 네트워크가 연결되어야 랙 밖의 통신이 가능하다. 서비스 트래픽, 스토리지 트래픽, 관리 트래픽을 별도 포트·스위치로 나누거나 논리적으로 구분하는 방식은 요구되는 격리와 비용에 따라 달라진다.[^tor]

![서버 랙에 장착된 PDU와 UPS 예시](https://upload.wikimedia.org/wikipedia/commons/1/10/APC_10-outlet_rackmount_19-inch_PDU.jpg)

랙 PDU가 여러 장비에 전원을 분배하고 UPS에 연결된 실제 예시이다. 이 사진 자체가 독립된 A/B 이중 전원 구성을 증명하는 것은 아니다. 사진: Dale Mahalko(DMahalko), [원본](https://commons.wikimedia.org/wiki/File:APC_10-outlet_rackmount_19-inch_PDU.jpg), [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0/). 추가 편집 없이 사용했다.

PDU(Power Distribution Unit)는 랙 장비에 전원을 분배한다. UPS는 설계된 시간 동안 전원을 유지하거나 전력 품질을 보조하는 장치로, PDU와 역할이 다르다. 이중 PSU 서버에서 한 PSU는 A 경로, 다른 PSU는 B 경로에 연결하고 각 경로가 필요한 부하를 감당하도록 설계할 수 있다. PSU 두 개를 같은 PDU 하나에 꽂으면 PSU 자체 고장에는 도움이 될 수 있어도 그 PDU 장애에는 함께 영향을 받는다. A/B라는 이름보다 실제 상위 전원 경로의 독립성이 중요하다.[^power]

## 5. 전력과 냉각이 성능의 조건이 되는 이유

CPU·RAM·NIC·드라이브가 소비한 전력은 대부분 열로 바뀌므로 전원을 충분히 공급하는 것과 열을 충분히 제거하는 것이 함께 필요하다. 공랭식 랙 서버는 흔히 앞에서 공기를 받아 뒤로 배출하지만 모든 장비가 같은 방향인 것은 아니다. 팬 방향과 서버의 air shroud, 빈 슬롯을 막는 blank는 공기가 의도한 부품을 통과하게 하는 구성 요소이다. 케이블이 배기 경로를 막으면 팬이 있어도 냉각 조건이 달라진다.[^cooling]

랙의 U 공간이 남아 있어도 전력·냉각·중량 한계 때문에 서버를 더 넣을 수 없을 수 있다. PSU의 정격 용량을 더한 값은 서버가 항상 소비하는 전력과 다르고, CPU TDP의 합도 서버 전체의 최대 입력 전력을 대신하지 않는다. 구성별 입력 전력과 redundancy 조건, 시설 전원·PDU 용량을 함께 고려한다. 고밀도 GPU 서버와 액체 냉각의 확대는 이런 전력 밀도와 열 이동 문제를 더 직접적인 설계 요소로 만든다.[^technical][^cooling]

## 6. 관리 경로와 공유 장애 영역

BMC의 관리 IP와 OS의 서비스 IP는 용도가 다르다. 전용 관리 포트를 쓰는 구성도 있고 물리 포트를 공유하는 구성도 있다. 관리망은 콘솔·전원 제어·펌웨어·하드웨어 이벤트에 접근할 수 있으므로 접근 권한과 인증, 업데이트 관리가 중요하다. Redfish는 이런 관리 기능을 표준 API로 제공하기 위한 기술이며, 서비스 애플리케이션 API와 역할이 다르다.[^management]

PSU 이중화, RAID, NIC bond는 각각 특정 부품과 경로의 실패를 다룬다. 메인보드·섀시·랙·공유 스위치가 남는 공통 의존성일 수 있으므로 부품이 두 개라고 서비스 전체가 이중화된 것은 아니다. 여러 서버에 애플리케이션을 배치하고 데이터 복제·복구를 설계하는 일은 그 위의 [[3-Tier Architecture]] 및 서비스 설계로 이어진다. 백업은 삭제나 손상 뒤 데이터를 되돌리는 별도 목적을 갖는다.

전원 자체가 없는 상태, POST를 통과하지 못한 상태, OS가 부팅되지 않은 상태, 네트워크가 연결되지 않은 상태, 프로그램만 동작하지 않는 상태는 서로 다른 단계이다. LED·BMC 이벤트·OS 로그가 설명하는 범위도 다르다. 이 구분은 부품 교체 절차를 외우기보다 서버가 어떻게 기동하고 데이터를 전달하는지 이해하는 데서 나온다.[^boot]

서버 구성의 확장은 지원되는 CPU·DIMM 배치·드라이브·PCIe 카드·펌웨어 조합 안에서 이뤄진다. 무거운 장비의 랙 설치와 내부 정비는 해당 모델의 전원 분리·잔류 전력·정전기 방지 및 hot-plug 조건을 따른다. 가상화와 클라우드는 사용자에게 이 물리 구성을 추상화하지만 실제 하드웨어와 전력·냉각의 의존성을 없애지는 않는다.[^service]

## 7. 관련 문서

- [[Server]] · [[CPU]] · [[Memory]] · [[Disk]] · [[NIC]]
- [[Networking]] · [[Storage]] · [[3-Tier Architecture]] · [[GPU Farm]]

## 8. 출처

[^inside]: Dell, [PowerEdge R650: Inside the system](https://www.dell.com/support/manuals/en-us/poweredge-r650/per650_ism_pub/inside-the-system?guid=guid-66eea213-2bf5-4f02-a8c5-f73e17eb0b93&lang=en-us), 구성 요소의 특정 모델 예시.
[^technical]: Dell, [PowerEdge R650 Technical Guide](https://i.dell.com/sites/csdocuments/product_docs/en/poweredge-r650-technical-guide.pdf), 프로세서·메모리·스토리지·PCIe·전원 구성 예시.
[^numa]: Linux Kernel, [Memory concepts: Nodes](https://docs.kernel.org/admin-guide/mm/concepts.html#nodes).
[^boot]: Dell, [No Power, No POST, No Boot 구분](https://www.dell.com/support/kbdoc/en-ca/000152161/dell-poweredge-troubleshooting-a-server-that-does-not-start-no-power-no-post-no-boot-and-no-video).
[^management]: DMTF, [Redfish Host Interface Specification](https://www.dmtf.org/sites/default/files/standards/documents/DSP0270_1.3.1.pdf), 호스트와 관리 프로세서·접근 경로의 구분.
[^rack]: Dell, [Rail sizing and rack compatibility](https://www.dell.com/support/manuals/en-us/poweredge-r650/per650_ism_pub/rail-sizing-and-rack-compatibility-matrix?guid=guid-7de4259b-a679-44fe-998e-ca4e70162e55&lang=en-us), 특정 모델 랙·레일 호환성 자료.
[^tor]: Cisco, [N9300 Platform Switches Data Sheet](https://www.cisco.com/c/en/us/products/collateral/switches/nexus-9000-series-switches/datasheet-c78-736967.html), ToR·MoR 배치와 서버 접근 계층의 제품 예시.
[^power]: Dell, [Power Distribution Units Buyer's Guide](https://i.dell.com/sites/content/shared-content/data-sheets/en/Documents/del-pdu-buyers-guide.pdf), 별도 전원 공급원과 한 PDU가 전체 부하를 감당하는 이중화 설계 설명.
[^cooling]: Dell, [PowerEdge R650 Installation and Service Manual](https://www.dell.com/support/manuals/en-us/poweredge-r650/per650_ism_pub/), fan·air shroud·blank·케이블 경로와 냉각 구성.
[^service]: Dell, [R650 Installation and Service Manual PDF](https://dl.dell.com/content/manual27146348-dell-poweredge-r650-installation-and-service-manual.pdf?language=en-us), Before working inside your system 및 Safety instructions 절. 공식 Inside the system 페이지의 PDF 링크로도 접근할 수 있다.

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
