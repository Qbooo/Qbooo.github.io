---
title: NIC
type: knowledge
domain: Networking
tags: [Hardware, OJT]
aliases: [네트워크 인터페이스 카드, Network Interface Card, 랜 카드]
parent:
status: growing
lang: ko
featured: false
created: 2026-11-05
updated: 2026-11-05
---

## 1. 개요

NIC(Network Interface Card 또는 Network Interface Controller)는 컴퓨터와 네트워크 전송 매체 사이에서 데이터의 송수신을 처리하는 하드웨어이다.

웹 서버의 응답은 프로그램이 만든 바이트 상태로 메모리에 존재하지만, 다른 컴퓨터에 도착하려면 케이블이나 광섬유를 통해 전달할 수 있는 신호가 되어야 한다. NIC는 운영체제·드라이버와 협력해 이 경계를 연결한다. 서버에 꽂는 확장 카드뿐 아니라 메인보드에 통합된 컨트롤러도 NIC에 해당한다. 이 문서는 서버에서 흔히 사용하는 Ethernet NIC를 중심으로 설명한다.

![RJ45 포트가 있는 PCI Ethernet NIC](https://upload.wikimedia.org/wikipedia/commons/7/7e/GB_Network_PCI_Card.jpg)

Belkin Gigabit Ethernet PCI 카드의 실제 사진이다. 금색 PCI 접점, 컨트롤러와 금속 브래킷의 RJ45 포트가 보인다. 현대 서버의 PCIe·OCP NIC나 광모듈 장착형 포트를 찍은 사진은 아니다. 사진: Harke, [원본 및 Public Domain 선언](https://commons.wikimedia.org/wiki/File:GB_Network_PCI_Card.jpg). 추가 편집 없이 사용했다.

## 2. 카드와 포트, 주소의 구분

한 NIC에는 여러 물리 포트가 있을 수 있고 각 포트는 케이블을 연결하는 접점이다. MAC 주소는 Ethernet 계층에서 인터페이스를 식별하는 주소이며, IP 주소는 네트워크 계층의 논리 주소이다. NIC 한 장에 IP 주소 하나만 붙는 것은 아니다. 하나의 인터페이스에 여러 IP를 설정하거나, VLAN·bond·bridge 같은 논리 인터페이스를 구성할 수 있다. VM의 vNIC 역시 게스트가 보는 가상 장치로, 물리 카드와 1:1로 대응하지 않는다.

일반적인 서버 NIC는 PCIe를 통해 호스트와 연결되고, 드라이버는 운영체제가 장치를 제어할 수 있도록 큐·버퍼·인터럽트 등을 설정한다. 케이블 쪽에서는 물리 계층 회로(PHY)와 광·전기 변환 부품이 실제 신호를 처리한다. RJ45 구리선, 광섬유와 광 송수신 모듈(transceiver), DAC 케이블은 전송 매체와 연결 방식이 다르다. SFP·QSFP 같은 명칭만으로 모든 포트의 지원 속도를 확정할 수 없으며 NIC·모듈·케이블·스위치의 지원 조합이 맞아야 한다.[^adapter]

## 3. 메모리의 바이트가 네트워크로 나가는 과정

TCP 통신을 예로 들면 애플리케이션이 소켓에 데이터를 쓰고, 운영체제의 TCP/IP 스택이 연결 상태·시퀀스·주소 등을 처리한다. 드라이버는 송신할 버퍼를 NIC가 찾을 수 있도록 버퍼의 위치·크기 등을 담은 디스크립터(descriptor)를 큐에 등록한다. DMA를 이용하는 일반적인 NIC는 호스트 메모리와 데이터를 교환하며, CPU가 모든 바이트를 직접 장치 레지스터로 복사하는 부담을 줄인다. NIC는 프레임을 전송 매체의 신호로 내보낸다.[^dma][^napi]

수신에서는 NIC가 도착한 데이터를 준비된 메모리 버퍼에 넣고 처리할 일이 있음을 알린다. Linux의 NAPI는 인터럽트와 polling을 조합해 수신 작업을 처리한다. 운영체제는 Ethernet·IP·TCP 등 해당 계층을 처리하고 데이터를 적절한 소켓으로 전달한다. 실제 구현에서는 zero-copy나 각종 offload 등으로 일부 경로가 달라질 수 있지만, '장치·메모리·CPU·프로토콜 스택이 함께 움직인다'는 점은 같다.[^napi]

이 때문에 NIC의 링크 속도를 올려도 CPU의 패킷 처리, PCIe 대역폭, 메모리 접근, 애플리케이션의 읽기 속도가 부족하면 전체 처리량은 그만큼 늘지 않는다. 작은 패킷이 많은 통신은 같은 바이트 수를 보내더라도 처리해야 할 패킷 수와 인터럽트·프로토콜 작업이 더 많다.

## 4. 큐와 오프로딩의 등장

빠른 네트워크에서는 한 CPU가 모든 수신 작업을 처리하는 것이 병목이 될 수 있다. multi-queue NIC와 RSS는 패킷의 주소·포트 등에 따른 흐름을 여러 수신 큐에 배치하고 여러 CPU가 작업을 나누게 한다. 보통 하나의 흐름은 순서를 유지하기 위해 특정 큐에 배치되므로, 단일 TCP 연결 하나가 모든 큐의 성능을 합산해서 쓰는 것으로 이해하면 안 된다.[^rss]

interrupt coalescing은 여러 이벤트를 묶어 CPU에 알리는 방식이다. 인터럽트 부담을 줄이는 대신 알림을 기다리는 시간이 늘 수 있어 처리량과 지연 시간 사이의 선택이 생긴다. 큐 수를 무작정 늘리는 것도 인터럽트와 메모리 관리 비용을 늘릴 수 있다.[^rss]

checksum offload와 TSO는 체크섬 계산이나 큰 TCP 데이터의 분할 같은 일을 장치에 맡기는 기능이다. GSO·GRO는 운영체제에서 분할·결합을 활용한다. 따라서 호스트에서 캡처한 패킷이 실제 선로의 프레임보다 크게 보이거나, NIC가 계산하기 전 체크섬이 잘못된 것처럼 보일 수 있다. 관측 위치를 무시하면 정상 offload 동작을 선로 오류로 오해할 수 있다.[^offload][^checksum]

## 5. 대역폭과 연결의 의미

10Gbps에서 b는 bit이고, 파일 전송에 흔한 GB/s의 B는 byte이다. 8bit가 1byte이므로 단순 환산한 이론 상한과 실제 애플리케이션 데이터 전송률을 구분한다. 프레임·IP·TCP 등의 헤더, 재전송, 수신 측 처리, 저장장치 쓰기와 암호화 비용 등이 실제 결과에 영향을 준다. full duplex는 양방향 동시 송수신을 뜻하며 한 방향 파일 전송이 두 배가 된다는 뜻이 아니다.

링크가 올라왔다는 것은 물리 링크 수준의 상태이다. 그 위의 VLAN·IP·라우팅·방화벽·TLS·애플리케이션 동작까지 보장하지 않는다. MTU는 인터페이스에서 처리하는 패킷 크기와 관련하고, 경로에는 더 작은 MTU의 구간이 있을 수 있다. jumbo frame을 활용하려면 관련 경로의 지원과 설정이 맞아야 한다. 작은 통신은 되는데 큰 전송이 멈추는 현상은 경로 MTU 탐색과 ICMP 처리 같은 조건으로도 설명될 수 있다.[^pmtu]

여러 NIC를 bond로 묶는 방식은 모드에 따라 대기 경로 또는 부하 분산을 제공한다. LACP 구성도 통상 흐름별 해시로 포트를 선택하므로 단일 흐름의 속도가 모든 포트 속도의 합이 되지는 않는다. 두 케이블이 같은 스위치나 같은 전원에 의존하면 그 공통 장애 영역도 남는다. 대역폭 집계와 경로 이중화는 관련되지만 서로 같은 목적은 아니다.[^bond]

## 6. 가상화와 현대 NIC의 역할

가상 스위치를 사용하는 VM에서는 게스트의 vNIC에서 호스트 네트워크 계층을 거쳐 물리 NIC로 나갈 수 있다. SR-IOV는 지원 PCIe 장치가 여러 Virtual Function을 제공하여 장치 기능을 나누는 방식이다. 이는 일반 vNIC나 물리 장치 전체 passthrough와 다르며 하드웨어·드라이버·하이퍼바이저와 격리 설정의 지원이 필요하다.[^sriov]

RDMA는 원격 메모리 접근을 위한 별도 기술로, 지원 NIC와 소프트웨어·네트워크 구성이 함께 필요하다. 어떤 Ethernet NIC든 RDMA를 자동 제공하는 것은 아니다. SmartNIC나 DPU는 네트워크 기능에 프로그래머블 처리나 추가 연산 자원을 결합해 인프라 처리를 맡길 수 있다. 이 역시 모든 NIC의 공통 구조는 아니다. CPU의 일을 줄이는 대신 장치 소프트웨어·보안·관측의 경계가 넓어진다.[^dpu]

NIC 오류를 이해할 때는 단순 속도보다 어느 단계에서 데이터가 지연되거나 손실되는지를 본다. 프레임 오류와 링크 불안정은 케이블·모듈·포트 조건과 연결되고, 수신 버퍼 부족이나 CPU 포화는 호스트 쪽 drop과 연결될 수 있다. TCP 재전송은 종단 간 현상으로 NIC 불량 하나만을 뜻하지 않는다. Linux 통계도 표준 인터페이스·프로토콜·드라이버별 항목을 구분하므로 수치의 정의와 증가 시점을 함께 읽는다.[^stats]

## 7. 관련 문서

- [[Networking]] · [[Server]] · [[Physical Server Configuration]]
- [[CPU]] · [[Memory]] · [[iSCSI]] · [[GPU Farm]]

## 8. 출처

[^adapter]: NVIDIA, [ConnectX-6 Dx Ethernet Adapter Cards User Manual](https://docs.nvidia.com/networking/display/nvidia-connectx-6-dx-ethernet-adapter-cards-user-manual.pdf), 모델별 포트·케이블·모듈·지원 구성 자료의 예시.
[^dma]: Linux Kernel, [Dynamic DMA mapping guide](https://docs.kernel.org/core-api/dma-api-howto.html).
[^napi]: Linux Kernel, [NAPI](https://docs.kernel.org/networking/napi.html).
[^rss]: Linux Kernel, [Scaling in the Linux Networking Stack](https://docs.kernel.org/networking/scaling.html).
[^offload]: Linux Kernel, [Segmentation Offloads](https://docs.kernel.org/networking/segmentation-offloads.html).
[^checksum]: Wireshark, [Checksums and offloading](https://www.wireshark.org/docs/wsug_html_chunked/ChAdvChecksums.html).
[^pmtu]: IETF, [RFC 1191: Path MTU Discovery](https://www.rfc-editor.org/rfc/rfc1191), IPv4의 경로 MTU 탐색 원리.
[^bond]: Linux Kernel, [Ethernet Bonding Driver HOWTO](https://docs.kernel.org/networking/bonding.html).
[^sriov]: Linux Kernel, [PCI Express I/O Virtualization](https://docs.kernel.org/PCI/pci-iov-howto.html).
[^dpu]: NVIDIA, [BlueField networking platform](https://www.nvidia.com/en-us/networking/products/data-processing-unit/), DPU 구현 사례.
[^stats]: Linux Kernel, [Interface statistics](https://docs.kernel.org/networking/statistics.html).

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Networking 분류|Networking]]
