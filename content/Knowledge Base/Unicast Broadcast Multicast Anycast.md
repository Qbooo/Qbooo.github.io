---
title: Unicast, Broadcast, Multicast, Anycast
type: knowledge
domain: Networking
tags: [Networking, IPv4, IPv6, OJT]
aliases: [유니캐스트, 브로드캐스트, 멀티캐스트, 애니캐스트, Unicast, Broadcast, Multicast, Anycast, 통신 방식 4종, IGMP]
parent: Networking
status: growing
lang: ko
featured: false
created: 2026-11-16
updated: 2026-11-17
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 네 가지 방식의 비교|2. 네 가지 방식의 비교]]
- [[#3. Unicast|3. Unicast]]
- [[#4. Broadcast의 범위|4. Broadcast의 범위]]
- [[#5. Multicast의 그룹 전달|5. Multicast의 그룹 전달]]
- [[#6. Anycast의 가까운 노드 선택|6. Anycast의 가까운 노드 선택]]
- [[#7. 흔한 오해|7. 흔한 오해]]
- [[#8. 관련 문서|8. 관련 문서]]
- [[#9. 출처|9. 출처]]

## 1. 개요

Unicast, Broadcast, Multicast, Anycast는 목적지 주소 하나가 몇 개의 수신자를 가리키는지에 따라 패킷 전달 방식을 나누는 이름이다. 각각 하나의 수신자, 같은 네트워크의 모든 수신자, 특정 그룹의 수신자, 같은 주소를 가진 여러 노드 가운데 가장 가까운 하나를 대상으로 한다.

네 방식은 데이터를 담는 형식이 다른 것이 아니라 목적지 주소를 해석하는 방식이 다르다. IPv6에는 Broadcast 주소가 없고 그 기능을 Multicast 주소가 대신한다.[^rfc4291]

## 2. 네 가지 방식의 비교

| 방식 | 수신자 | 전달 범위 | 대표 사례 |
|---|---|---|---|
| Unicast | 주소 하나가 가리키는 인터페이스 1개 | 라우팅이 허용하는 전체 경로 | 웹 접속, SSH |
| Broadcast | 같은 L2 네트워크의 모든 호스트 | 라우터에서 멈춤 | ARP 요청 |
| Multicast | 그룹에 가입한 호스트 | 가입과 라우팅이 이어지는 범위 | IPTV 같은 한 번에 여러 곳으로 보내는 전송 |
| Anycast | 같은 주소를 가진 노드 중 가장 가까운 1개 | 라우팅이 선택한 경로 | DNS 서버 |

## 3. Unicast

Unicast는 하나의 송신자가 하나의 인터페이스로 보내는 1:1 통신이다. IPv6 규격도 Unicast 주소를 하나의 인터페이스를 식별하는 주소로 정의하며, 그 주소로 보낸 패킷은 해당 인터페이스로 전달된다.[^rfc4291] 일상적인 웹 접속과 SSH 접속이 대부분 Unicast이다. 나머지 세 방식은 이 Unicast 주소 체계 위에서 수신자의 범위만 다르게 정한 것으로 이해할 수 있다.

## 4. Broadcast의 범위

Broadcast는 같은 네트워크의 모든 호스트에게 보내는 방식이다. 이더넷에서는 목적지 MAC 주소를 브로드캐스트 주소로 지정하며, 스위치는 같은 VLAN의 모든 포트로 프레임을 전달한다. 대표 사례가 ARP 요청이다.[^rfc826] 스위치의 동작은 [[Switch|스위치]]에서 설명한다.

IPv4의 Broadcast 주소는 두 가지이다.

- **Limited Broadcast**: `255.255.255.255`이며 로컬 하드웨어 네트워크용이다. 이 주소의 패킷은 전달되어서는 안 된다고 규정되어 있다.[^rfc919]
- **Directed Broadcast**: 호스트 번호가 모두 1인 주소이며 특정 네트워크의 모든 호스트를 가리킨다. 예를 들어 `192.168.10.0/28`의 `192.168.10.15`가 이에 해당한다.[^rfc919]

Broadcast의 범위는 스위치 포트 전체가 아니라 같은 L2 broadcast domain 전체이다. 스위치가 여러 대여도 같은 VLAN이면 모두 받고, 라우터를 만나면 멈춘다. Limited Broadcast는 규격상 전달되지 않는다. Directed Broadcast는 원래 목적지 네트워크에 붙은 게이트웨이까지 정상적으로 라우팅된 뒤 그 네트워크에 뿌려지는 방식이었으나,[^rfc919] 증폭 공격(Smurf)에 악용되자 RFC 2644가 라우터의 기본값을 바꾸었다. 현재는 라우터가 Directed Broadcast를 받고 전달하는 기능이 기본적으로 꺼져 있어야 한다.[^rfc2644]

따라서 서브넷을 나누면 Broadcast의 범위도 함께 나뉘고, 그 경계가 라우터이다. 인터넷 전체가 하나의 L2 네트워크라면 Broadcast와 장애의 영향이 전 세계로 퍼지므로 라우터로 나누는 것이다. 이 점은 [[Internet Routing|인터넷의 통신 구조]]와 [[IP Address and Subnet|IP 주소와 서브넷]]의 `/28` 예에서도 확인할 수 있다.

## 5. Multicast의 그룹 전달

Multicast는 하나의 송신자가 호스트 그룹(host group)이라는 수신자 집합에 한 번만 보내는 방식이다. 호스트 그룹은 하나의 IP 목적지 주소를 공유하는 0개 이상의 호스트 집합이다. 전달은 Unicast와 마찬가지로 최선형(best-effort)이라 도착, 무결성, 순서가 보장되지 않는다.[^rfc1112]

- **그룹 가입**: 가입과 탈퇴는 언제든 가능하고 한 호스트가 여러 그룹에 속할 수 있다. 송신자는 그룹의 구성원이 아니어도 된다.[^rfc1112]
- **주소 범위**: IPv4 그룹 주소는 Class D이며 `224.0.0.0`부터 `239.255.255.255`까지이다.[^rfc1112]
- **IGMP**: 호스트가 자신의 그룹 가입 상태를 이웃한 멀티캐스트 라우터에 알리는 프로토콜이다. 라우터가 질의를 보내면 호스트가 보고로 응답한다.[^rfc1112]

Broadcast와 달리 가입하지 않은 호스트는 받지 않는다. Broadcast가 같은 네트워크의 모두에게 가는 방식이라면 Multicast는 원하는 호스트만 받는 방식이다.

같은 영상을 100명에게 보내는 서버를 비교하면 차이가 분명하다. 영상 한 개가 5Mbps라고 가정한 예이다.

| 비교 항목 | Unicast 100개 | Multicast |
|---|---|---|
| 서버가 내보내는 양 | 5Mbps × 100 = 500Mbps | 5Mbps |
| 공유하는 중간 링크 | 같은 데이터가 100개 복사본으로 통과 | 복사본 1개만 통과 |
| 복제의 주체 | 서버 | 경로가 갈라지는 지점의 네트워크 장비 |
| 수신자 관리 | 수신자별 연결 | 서버는 수신자를 알 필요 없음 |

복제를 서버가 아니라 네트워크 장비가 한다는 점이 핵심이며, 시청자가 늘어도 서버가 내보내는 양은 그대로이다. 수치는 설명을 위한 가정이고, 실제 구성은 장비의 멀티캐스트 라우팅 지원 여부에 따라 달라진다.

## 6. Anycast의 가까운 노드 선택

Anycast는 같은 서비스 주소를 서로 다른 여러 위치에서 제공하고, 패킷이 그중 한 곳으로 가도록 하는 방식이다.[^rfc4786] IPv6 규격도 Anycast 주소를 둘 이상의 인터페이스에 할당된 주소로 정의하며, 패킷은 라우팅 프로토콜의 거리 기준으로 가장 가까운 인터페이스로 전달된다. Anycast 주소는 Unicast 주소 공간에서 가져오므로 형식만으로는 Unicast 주소와 구분되지 않는다.[^rfc4291]

즉 Anycast는 새로운 종류의 패킷이 아니라 같은 Unicast 주소를 여러 곳에서 광고하는 운영 방식이다. 어느 노드로 갈지는 라우팅 시스템이 라우팅 토폴로지와 요청이 네트워크에 들어오는 위치를 기준으로 정한다.[^rfc4786] 여기서 가깝다는 것은 지리적 거리가 아니라 라우팅 기준의 거리이다. 한 노드가 장애를 일으키면 서비스 상태와 경로 광고를 연동해 그 노드의 경로를 거두고, 트래픽이 다른 노드로 옮겨 가도록 구성하는 것이 권장된다.[^rfc4786] 노드 간 부하 분산은 대체로 고르지 않다.[^rfc4786]

실제 사례로 여러 루트 DNS 서버 운영자가 Anycast로 서버를 넓게 분산해 두었다.[^rfc4786] Google Public DNS의 `8.8.8.8`도 Anycast 라우팅으로 질의를 가장 가까운 서버로 보낸다고 공식 FAQ에 설명되어 있다.[^googledns] 한국에서 `8.8.8.8`로 ping을 보내면 왕복 시간이 수십 ms 수준으로 측정되는데, 이는 대륙 반대편 서버의 응답으로는 물리적으로 어려운 값이다. 다만 이 관찰만으로 서버의 정확한 위치를 특정할 수는 없다.

**같은 IP가 여러 곳에 있어도 문제가 없는 이유**: 같은 L2 네트워크에 같은 IP를 가진 호스트가 둘 있으면 ARP 질의에 둘 다 응답해 통신이 엉킨다. Anycast의 노드는 서로 다른 위치의 별개 네트워크에 있고 각자 지역 라우터가 같은 주소를 광고한다. 각 라우터는 그중 하나의 경로를 골라 쓰므로 한 사용자의 패킷은 한 노드로만 간다. 같은 L2 안에 주인이 둘이라 문제인 것과 달리, 네트워크마다 주인이 하나이므로 문제가 되지 않는다.

Anycast에는 적합하지 않은 경우도 있다. 라우팅 결정은 하나의 트랜잭션보다 길게 유지되어야 하며, 오래 유지되는 연결은 경로가 바뀌면 다른 노드로 가서 끊길 수 있다. 특히 TCP는 같은 세션의 패킷이 서로 다른 노드에 도착하면 연결 설정이 깨질 수 있다.[^rfc4786] 또한 서비스마다 경로 하나를 광고하면 전역 인터넷에서는 확장성이 낮아, 소수의 핵심 서비스에 적합하다.[^rfc4786]

[[CDN]]과는 구분한다. CDN은 콘텐츠를 사용자에게 가까운 거점에서 전달하는 서비스이고, Anycast는 같은 주소로 가까운 노드에 연결하는 라우팅 방식이다.

## 7. 흔한 오해

- **Broadcast가 스위치의 모든 포트로만 간다는 생각**: 범위는 스위치 하나가 아니라 같은 L2 broadcast domain 전체이며, 스위치가 여러 대여도 같은 VLAN이면 포함된다.
- **Broadcast가 서브넷 경계를 자유롭게 넘는다는 생각**: Limited Broadcast는 라우터가 전달하지 않는다. Directed Broadcast는 목적지 서브넷까지 라우팅될 수 있는 형식이지만, 증폭 공격 방지를 위해 현대 라우터에서는 전달 기능을 기본적으로 비활성화한다.
- **Multicast가 Broadcast의 다른 이름이라는 생각**: Multicast는 그룹에 가입한 호스트만 받는다.
- **Anycast가 새로운 주소 종류나 패킷 형식이라는 생각**: Unicast 주소 공간의 주소를 여러 곳에서 광고하는 방식이다.
- **Anycast가 항상 지리적으로 가장 가까운 서버로 간다는 생각**: 가까움의 기준은 라우팅이다. 보통 가까운 노드가 선택되지만 지리적 거리와 항상 일치하지는 않는다.
- **CDN과 Anycast가 같다는 생각**: 서비스와 라우팅 방식이라는 서로 다른 층위의 개념이다.

## 8. 관련 문서

- [[Networking|네트워크]] · [[Encapsulation|캡슐화]] · [[IP Address and Subnet|IP 주소와 서브넷]]
- [[Switch|스위치]] · [[Router|라우터]] · [[BGP]] · [[ARP]]
- [[Internet Routing|인터넷의 통신 구조]] · [[CDN]]

## 9. 출처

[^rfc4291]: IETF, [RFC 4291, IP Version 6 Addressing Architecture](https://www.rfc-editor.org/rfc/rfc4291.html). 2절의 IPv6 Broadcast 주소 부재, 2.5절 Unicast, 2.6절 Anycast, 2.7절 Multicast.
[^rfc826]: IETF, [RFC 826, An Ethernet Address Resolution Protocol](https://www.rfc-editor.org/rfc/rfc826.html). 같은 로컬 네트워크에서 프로토콜 주소에 대응하는 하드웨어 주소를 찾는 요청의 브로드캐스트.
[^rfc919]: IETF, [RFC 919, Broadcasting Internet Datagrams](https://www.rfc-editor.org/rfc/rfc919.html). 4절 Broadcast Classes의 Directed Broadcast 라우팅, 7절의 Limited Broadcast와 Directed Broadcast 주소.
[^rfc2644]: IETF, [RFC 2644, Changing the Default for Directed Broadcasts in Routers](https://www.rfc-editor.org/rfc/rfc2644.html). Directed Broadcast의 수신과 전달을 기본적으로 끄도록 한 RFC 1812 개정과 그 배경(Smurf 공격).
[^rfc1112]: IETF, [RFC 1112, Host Extensions for IP Multicasting](https://www.rfc-editor.org/rfc/rfc1112.html). 2절 Introduction의 호스트 그룹과 가입 방식, 4절 Host Group Addresses의 Class D 주소 범위, 부록 I의 IGMP.
[^rfc4786]: IETF, [RFC 4786, Operation of Anycast Services](https://www.rfc-editor.org/rfc/rfc4786.html). 2절 Terminology의 Anycast 정의, 3.1절 General Description의 노드 선택과 부하 분산, 4.1절 Protocol Suitability, 4.4.1절 Signalling Service Availability, 4.4.3절 Equal-Cost Paths, 4.4.8절 Aggregation Risks.
[^googledns]: Google, [Google Public DNS FAQ](https://developers.google.com/speed/public-dns/faq). Technical 항목의 Anycast 라우팅으로 가장 가까운 서버에 질의를 보낸다는 설명.

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Networking 분류|Networking]]
