---
title: Networking
type: knowledge
domain: Networking
tags: [Infrastructure, OJT]
aliases: [네트워킹, 네트워크, Network]
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-04
---

## 1. 개요

Networking(네트워킹)은 장치와 네트워크 사이에서 규약에 따라 데이터를 전달하고 통신 경로와 접근을 관리하는 기술과 활동이다.

[[Server]]가 프로그램을 실행하고 [[Storage]]가 데이터를 기록하더라도, 필요한 대상에 요청이 도달하지 않으면 서비스는 완성되지 않는다. 네트워크는 케이블·광모듈·NIC·스위치·라우터 같은 장비와, 주소·경로·프로토콜·정책을 함께 다룬다. 클라우드의 VPC와 가상 NIC도 최종적으로는 호스트와 물리 네트워크의 동작 위에 구현된다.

## 2. 실제 장비와 각자의 책임

| 요소 | 주된 역할 | 역할을 혼동하면 생기는 문제 |
|---|---|---|
| NIC | 호스트의 네트워크 인터페이스와 프레임 송수신 | IP 설정이 맞아도 링크 단절·드롭을 놓칠 수 있음 |
| L2 스위치 | 같은 VLAN의 Ethernet 프레임을 MAC 주소 정보로 전달 | 다른 IP 네트워크로의 라우팅이 자동이라고 생각함 |
| 라우터·L3 스위치 | 목적지 IP와 경로표에 따라 다음 홉으로 전달 | 경로와 방화벽 허용을 같은 것으로 생각함 |
| 방화벽 | 주소·프로토콜·포트·연결 상태 등 정책으로 통신 제어 | 케이블이 연결되면 모든 통신이 가능하다고 생각함 |
| 로드밸런서 | 여러 서비스 대상에 연결·요청 분산 | 라우팅만으로 애플리케이션의 건강까지 확인된다고 생각함 |
| DNS 리졸버·권한 서버 | 이름에 대한 레코드 조회·응답 | IP로 접속되면 DNS까지 정상이라고 생각함 |

여러 역할이 하나의 장비에 들어갈 수도 있다. L3 스위치가 라우팅을 하거나 가상 방화벽이 VM으로 실행될 수 있으므로 “어느 상자인가”와 “어떤 기능을 수행하는가”를 구분한다.[^cisco]

## 3. 계층으로 나눠야 하는 이유

네트워크 계층 구분은 장애를 위치와 기능으로 나누는 설명 도구이다. OSI 7계층과 TCP/IP 모델은 분류 방식이 달라 정확히 같은 칸을 일대일 대응시키기보다 해당 기능이 무엇인지 이해하는 것이 중요하다.

| 실무에서 보는 구간 | 핵심 질문 | 대표 정보 |
|---|---|---|
| 물리·링크 | 신호가 통하고 같은 링크에서 전달되는가? | 케이블·링크·MAC·VLAN |
| 네트워크 | 목적지 네트워크에 갈 경로가 있는가? | IP·서브넷·라우팅·다음 홉 |
| 전송 | 상대 프로그램의 통신 지점에 연결되는가? | TCP·UDP·포트·연결 상태 |
| 애플리케이션 | 요청의 의미와 응답이 올바른가? | HTTP·DNS·DB 프로토콜·인증 |

IP는 목적지까지 패킷을 전달하는 기반이고 TCP는 그 위에서 순서 있는 바이트 스트림과 재전송·흐름 제어 등을 제공한다. TCP 연결 성공이 애플리케이션의 업무 성공을 의미하지는 않는다. UDP에는 TCP와 같은 연결·전달 보장이 내장되어 있지 않지만 상위 프로토콜이 필요한 제어를 구현할 수 있다.[^tcp]

## 4. IP·서브넷·MAC·포트는 무엇을 구별하는가

IPv4 주소는 32비트이며 `10.20.30.0/24` 같은 CIDR 표기는 앞의 24비트가 프리픽스임을 뜻한다. 단순한 예에서 `10.20.30.10/24`와 `10.20.30.20/24`는 같은 프리픽스에 있지만, 주소만 같게 맞춘다고 서로 다른 VLAN·격리된 가상 네트워크가 자동으로 연결되지는 않는다. CIDR는 주소를 유연하게 묶고 경로를 집약하는 배경에서 도입되었다.[^cidr]

MAC 주소는 Ethernet 링크에서 다음 전달 대상을 식별하는 데 쓰이고, IP 주소는 네트워크 간 전달에 쓰인다. TCP·UDP 포트는 호스트 안에서 서비스 통신을 구별한다. 예를 들어 `10.20.30.10:443`은 주소와 서비스 포트를 함께 표현한다. 같은 숫자의 TCP 포트와 UDP 포트는 서로 다른 전송 프로토콜의 통신 지점이다.

IPv4 호스트가 Ethernet으로 다른 서브넷에 보낼 때는 보통 경로표가 선택한 게이트웨이의 MAC 주소를 ARP로 알아낸다. 원격 목적지 서버의 MAC을 로컬 망에서 직접 찾아 전 인터넷을 통과시키는 방식이 아니다. IPv6의 이웃 탐색은 ARP와 별도 프로토콜이다.[^arp]

사설 IPv4 대역 `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`은 조직 내부에서 재사용할 수 있다. 사설 주소라는 이유만으로 안전한 것은 아니며, 외부와 통신하려면 별도의 연결·경로·주소 변환 등 환경에 맞는 구성이 필요하다. 조직 간 망 연결에서는 주소 중복이 경로 설계를 어렵게 할 수 있다.[^private]

예를 들어 `10.20.30.10/24`가 같은 링크의 `10.20.30.20`으로 보내면 직접 연결 경로를 선택하고 대상의 MAC 주소를 ARP로 알아낼 수 있다. `10.20.40.20`으로 보내면 일치하는 경로의 다음 홉을 선택하며 더 구체적인 경로가 없으면 기본 경로를 사용할 수 있다. 서브넷 마스크는 주소의 프리픽스 범위를 판단하는 데 쓰이고 기본 게이트웨이는 기본 경로의 다음 홉이지 모든 패킷이 무조건 지나는 장비가 아니다.

일반적인 Ethernet 라우팅에서 라우터는 들어온 프레임을 벗기고 IP 경로를 선택한 뒤 다음 링크의 MAC 주소를 사용해 새 프레임을 만든다. NAT·터널 같은 변환이 없는 기본 예에서는 IP 출발지·목적지는 유지되지만 링크별 MAC 주소는 달라진다. IPv4 TTL은 전달 과정에서 감소하고 소진된 패킷은 폐기되어 순환 경로를 무한히 돌지 않도록 한다.[^ip]

## 5. 웹 접속이 실제로 지나가는 경로

다음은 IPv4·Ethernet·TCP 기반 HTTPS 접속의 단순화한 예이다. HTTP/3처럼 UDP 기반 QUIC를 사용하는 경우에는 전송 연결의 동작이 달라진다.

```mermaid
sequenceDiagram
    participant U as 사용자 프로그램
    participant D as DNS 리졸버
    participant G as 로컬 게이트웨이·라우터
    participant W as 서비스 주소의 서버·LB
    U->>D: 이름의 주소 레코드 조회
    D-->>U: 주소 응답
    Note over U,G: 호스트 경로 선택·필요시 ARP
    U->>G: 목적지 IP로 가는 패킷
    G->>W: 경로와 정책에 따라 전달
    U->>W: TCP 연결·TLS 협상
    U->>W: HTTP 요청
    W-->>U: HTTP 응답
```

DNS는 이름에 대한 레코드를 제공하며 요청 본문 전체를 대신 전달하지 않는다. 캐시와 TTL 때문에 레코드 변경 직후 모든 클라이언트가 동시에 새 주소를 보지는 않을 수 있다.[^dns] TLS는 통신 구간의 기밀성과 무결성·서버 인증 등에 관여한다. 인증서가 유효한지, 접속 이름과 맞는지, 실제로 어느 장비에서 TLS가 종료되는지 구분한다.[^tls]

응답 방향도 경로와 정책을 통과해야 한다. 요청이 도착했다는 사실만으로 응답 경로까지 올바르다고 확정하지 않는다. NAT가 있으면 변환된 주소·포트와 연결 상태 때문에 로그에서 보이는 출발지가 원래 사용자 주소와 다를 수 있다.

## 6. VLAN·서브넷·VPC·보안 정책의 관계

VLAN은 L2 브로드캐스트 도메인을 논리적으로 나누는 수단이고, IP 서브넷은 주소와 라우팅 경계를 표현한다. 흔히 VLAN별 서브넷을 대응시키지만 두 개념의 정의가 같은 것은 아니다. VLAN 사이 통신은 필요한 L3 경로와 정책을 거친다.[^vlan]

클라우드의 VPC는 가상 네트워크 격리·주소·경로·접근 통제의 관리 단위이다. 서브넷에 VM을 넣고 Security Group을 허용했어도 경로표와 게이트웨이·대상 서비스 수신 설정이 빠지면 통신은 실패할 수 있다. 특정 클라우드의 “퍼블릭 서브넷”도 VM마다 무조건 공인 주소를 가진다는 뜻은 아니다.[^vpc]

[[3-Tier Architecture]]에서는 인터넷에서 WEB으로 필요한 요청만 받고, WEB에서 WAS, WAS에서 DB로 필요한 연결만 허용하는 구조를 설계할 수 있다. “계층을 분리했다”는 설명은 실제 네트워크 정책과 자격 증명 설정으로 구현되어야 한다.

연결 상태를 기억하는 stateful 정책과 각 방향의 패킷 규칙을 독립적으로 평가하는 stateless 정책은 반환 트래픽 처리 방식이 다르다. AWS Security Group은 stateful, Network ACL은 stateless인 구체적인 예이다. 다른 제품도 항상 같은 동작이라고 가정하지 않는다.[^policy] NAT는 주소·포트 변환이고 방화벽은 허용·차단 정책이므로 NAT만으로 보안을 보장할 수 없다.

L4 로드밸런서는 전송 계층 연결·주소·포트 관점으로, L7 로드밸런서는 HTTP Host·경로 같은 애플리케이션 정보를 활용하는 관점으로 구분한다. TLS 종료와 프로토콜 지원에 따라 볼 수 있는 정보가 달라진다. 분산과 헬스 체크는 별도 기능이다.

## 7. 장애 현상을 계층에 연결하기

| 관측 현상 | 확인 범위를 좁히는 관점 |
|---|---|
| 이름으로 실패하지만 IP 경로는 동작 | DNS 응답·캐시·접속 이름, TLS 이름 검증을 구분 |
| TCP 연결이 시간 초과 | 순방향·반환 경로, 드롭 정책, 대상 응답 상태 |
| 연결 거부 | 대상이 적극적으로 거절했는지, 포트 수신·방화벽 거부 확인 |
| HTTP 403·500 응답 | HTTP 응답은 왔지만 인증·업무·서버 내부 처리는 별도 확인 |
| 작은 요청은 되고 큰 전송만 정체 | MTU·경로 MTU 탐색·손실·버퍼 등 후보 조사 |
| VM 일부에서만 느림 | 같은 호스트·VLAN·경로·대상 분포와 NIC 지표 비교 |

`ping`은 주로 ICMP 도달성을 보는 도구이며 TCP 443·TLS·HTTP·DB 인증의 성공을 대신하지 않는다. RTT와 처리량도 다르다. 짧은 왕복 지연과 큰 파일을 충분히 빠르게 전달하는 능력은 서로 다른 지표이다. 재전송·손실·대역폭·동시성·서비스 처리 시간을 함께 해석한다.

이 표는 원인을 확정하는 진단표가 아니라 관측 사실로 조사 범위를 나누는 예이다. 변경 전후의 시각, 출발지·목적지, 프로토콜·포트, 어느 단계에서 실패했는지를 일관되게 기록하면 서버·네트워크·애플리케이션 담당자가 같은 현상을 놓고 대화할 수 있다.

| 성능 관점 | 뜻과 한계 |
|---|---|
| Bandwidth | 링크·서비스의 전송 용량. 앱이 실제로 얻은 속도와 다름 |
| Throughput | 관측 구간에 실제 전송한 양. 헤더·재전송·앱 처리에 영향받음 |
| Latency·RTT | 한 방향 지연과 왕복 지연. 측정 지점·방법 구분 |
| Jitter | 지연의 변동. 실시간 음성·영상 등에서 중요 |
| Loss·Retransmission | 손실과 재전송. TCP에서는 유효 처리량·대기에 영향 |

패킷 손실이 반드시 케이블 불량이라는 뜻은 아니다. 혼잡·큐 드롭·정책·수신 자원 문제도 후보이다. 링크 사용률이 낮아도 특정 경로의 손실이나 작은 요청의 왕복 대기 때문에 업무가 느릴 수 있다.

`ip route`는 호스트의 경로 선택 정보를, `ss`는 소켓의 수신·연결 상태를, `tcpdump` 같은 캡처 도구는 관측 인터페이스의 패킷을 확인하는 데 사용한다. `traceroute`의 중간 홉 무응답은 라우터가 진단 응답을 제한한 결과일 수도 있어 서비스 경로 단절과 곧바로 동일시하지 않는다. 한쪽 캡처에 요청만 보이면 대상 응답과 반환 경로를 구분해 조사한다. 암호화된 패킷 캡처만으로 HTTP 본문까지 항상 읽을 수 있는 것도 아니다.[^tools]

## 8. 관련 문서

- [[Server]], [[Storage]], [[3-Tier Architecture]]
- [[Web Server]], [[Web Application Server]], [[Database]]
- [[CDN]], [[SNMP]], [[iSCSI]]

## 9. 출처

[^cisco]: [Cisco — Networking basics](https://www.cisco.com/site/us/en/learn/topics/small-business/networking-basics.html)
[^tcp]: [IETF — RFC 9293: TCP](https://www.rfc-editor.org/rfc/rfc9293.html)
[^cidr]: [IETF — RFC 4632: CIDR](https://www.rfc-editor.org/rfc/rfc4632.html)
[^arp]: [IETF — RFC 826: ARP](https://www.rfc-editor.org/rfc/rfc826.html)
[^private]: [IETF — RFC 1918: Private Internets](https://www.rfc-editor.org/rfc/rfc1918.html)
[^dns]: [IETF — RFC 1034: DNS Concepts](https://www.rfc-editor.org/rfc/rfc1034.html)
[^tls]: [IETF — RFC 8446: TLS 1.3](https://www.rfc-editor.org/rfc/rfc8446.html)
[^vlan]: [Cisco — VLAN configuration concepts](https://www.cisco.com/c/en/us/td/docs/switches/lan/c9000/lyr2-fwd/vlan/vlan-configuration-guide/configure-vlan.html)
[^vpc]: [AWS — VPC route tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)

[^ip]: [IETF — RFC 791: IPv4 and TTL](https://www.rfc-editor.org/rfc/rfc791.html)
[^policy]: [AWS — Network ACLs와 Security Group의 stateless/stateful 차이](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
[^tools]: [iproute2 — ss manual](https://man7.org/linux/man-pages/man8/ss.8.html) · [tcpdump manual](https://www.tcpdump.org/manpages/tcpdump.1.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Networking 분류|Networking]]
