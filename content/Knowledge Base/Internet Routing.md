---
title: Internet Routing
type: knowledge
domain: Networking
tags: [Networking, Internet, Routing, OJT]
aliases: [인터넷의 통신 구조, 인터넷 라우팅, Internet Path]
parent: Networking
status: growing
lang: ko
featured: false
created: 2026-11-16
updated: 2026-11-16
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 하나의 거대한 L2 네트워크가 아닌 이유|2. 하나의 거대한 L2 네트워크가 아닌 이유]]
- [[#3. PC에서 인터넷 서버까지의 전달 과정|3. PC에서 인터넷 서버까지의 전달 과정]]
- [[#4. 가정용 라우터가 모든 인터넷 경로를 몰라도 되는 이유|4. 가정용 라우터가 모든 인터넷 경로를 몰라도 되는 이유]]
- [[#5. 인터넷의 계층과 메시 구조|5. 인터넷의 계층과 메시 구조]]
- [[#6. 물리적으로 이어지는 경로|6. 물리적으로 이어지는 경로]]
- [[#7. 관련 문서|7. 관련 문서]]
- [[#8. 출처|8. 출처]]

## 1. 개요

Internet Routing(인터넷 라우팅)은 서로 독립적으로 운영되는 수많은 IP 네트워크가 경로 정보를 교환하고, 각 라우터가 다음 홉을 선택하여 패킷을 목적지까지 전달하는 체계이다.

인터넷은 하나의 회사가 소유한 거대한 망도 아니고, 최상위 라우터 한 대 아래에 모든 장비가 매달린 피라미드도 아니다. 통신 사업자, 기업, 대학, 클라우드와 콘텐츠 사업자가 각자의 네트워크를 운영하고 [[IXP Peering and Transit|Peering과 Transit]]으로 연결한다.

### 장비와 링크 중심의 기준 아키텍처

![가정·기업·데이터센터·Cloud VPC와 접속 ISP, IXP, Transit, 콘텐츠 AS, Cloud Provider를 실제 장비 링크로 연결한 기준 아키텍처](../assets/internet-reference-topology-v2.png)

기준 그림은 가정 LAN·기업 캠퍼스·데이터센터·Cloud VPC처럼 사용자가 직접 관리하는 네트워크와, 접속 ISP의 가입자 수용·집선·코어·Internet Edge, IXP·Transit·콘텐츠 AS·Cloud Provider의 연결을 한 화면에 배치한 개념 지도이다.

그림의 중앙은 인터넷을 지휘하는 상위 기관이나 실제 지리적 중심을 뜻하지 않는다. IXP·Transit·콘텐츠 사업자·Cloud Provider처럼 **서로 다른 AS의 연결이 많이 모이는 논리적 상호접속 영역**을 시각적으로 중앙에 놓은 것이다. 현실의 인터넷에는 이와 같은 상호접속 지점이 세계 여러 지역에 분산되어 있다.

선은 Packet Tracer의 Logical 화면처럼 **실제 장비 포트끼리 연결된 링크**를 나타낸다. 색상 영역은 소속과 관리 범위만 구분하며, 영역 상자 자체가 통신 대상이라는 뜻은 아니다. 특정 요청의 경로는 기준 그림 위에 고정하지 않고 다음과 같이 읽는다.

| 통신 예시 | 기준 그림에서 따라갈 장비 | 핵심 의미 |
|---|---|---|
| 가정 A → 콘텐츠 서비스 | 홈 라우터 → ISP A의 BNG·집선·코어·Internet Edge → IXP Peering 또는 Transit → Content Edge·Core → 서비스 서버 | 같은 목적지도 정책에 따라 Peering과 Transit 가운데 다른 경로를 사용할 수 있음 |
| 기업 캠퍼스 → Cloud 공개 서비스 | 고객 Edge → ISP A → IXP 또는 Transit → Cloud Provider Edge·Core·VPC 연결 → Internet Gateway → Public Load Balancer → Private App | 공개 진입점 이후에만 허용된 내부 계층으로 전달됨 |
| 가정 B → 온프레미스 공개 Web | 홈 라우터 → ISP B의 가입자 수용·내부 전달 → 데이터센터 고객 접속 → DC Edge → 방화벽 → Load Balancer → Web | 목적지가 같은 ISP 내부에 있으면 외부 IXP·Transit을 거치지 않을 수 있음 |

가정 사용자는 인터넷을 통해 온프레미스와 Cloud VPC의 **공개 진입점**에 접근할 수 있다. 그러나 공개 Web 뒤의 WAS·DB나 VPC의 Private App·DB에 인터넷에서 바로 연결된다는 뜻은 아니다. 외부 요청은 방화벽·Load Balancer·Internet Gateway 같은 경계를 통과하고, 내부 계층은 허용된 다음 통신만 이어받는다.

### 같은 ISP 안에서 코어를 거치는 이유

앞 표의 세 번째 통신 예시는 가정 LAN B와 온프레미스 데이터센터가 모두 ISP B의 회선을 사용하지만 서로 다른 접속 지점에 연결된 상황을 가정한다.

```text
가정 B
→ 가입자 접속/BNG
→ 집선 라우터
→ 코어 라우터
→ DC 연결 PE
→ DC Edge
→ 방화벽
→ Load Balancer
→ 공개 Web
```

BNG는 가정 가입자 회선을 수용하고, DC 연결 PE는 데이터센터 고객 회선을 수용한다. 두 장비가 서로 직접 연결되어 있지 않으므로 ISP의 집선망과 코어망이 두 접속 지점 사이를 운반한다. 쉽게 설명하면 동네 지점에서 접수한 화물이 사업자의 중앙 운송망을 거쳐 목적지 지역 지점으로 내려가는 것과 비슷하다.

이 경로는 같은 ISP B의 AS 안에서 완성되므로 다른 AS와 만나는 ISP Internet Edge·IXP·Transit을 거칠 필요가 없다. ISP 내부에서는 정적 경로, IGP, MPLS·Segment Routing과 BGP 등을 환경에 맞게 조합할 수 있지만, 그림은 특정 구현을 단정하지 않고 `가입자 접속 → ISP 코어 → 목적지 고객 접속`이라는 공통 원리만 나타낸다.

반드시 코어까지 올라가야 하는 것은 아니다. 가정 회선과 데이터센터 회선이 같은 BNG·PE에서 종료되고 장비의 정책이 직접 전달을 허용한다면 더 짧은 내부 경로를 사용할 수 있다. 반대로 목적지가 다른 ISP나 AS에 연결되어 있다면 ISP Internet Edge를 거쳐 Peering 또는 Transit 경로로 넘어가야 한다.

기준 아키텍처는 내가 관리하는 네트워크가 어디까지이며 ISP·IXP·Transit·다른 AS의 실제 장비가 어떻게 연결되는지를 설명한다. 특정 요청의 경로는 목적지와 정책에 따라 위 표와 같이 같은 링크 지도 위에서 따로 추적한다.

그림의 색상 상자는 단순한 장비 종류가 아니라 **운영 주체와 라우팅 정책의 경계**를 나타낸다. 접속 ISP, Transit 사업자, 다른 ISP, 콘텐츠 사업자와 Cloud Provider는 각자 독립된 AS가 될 수 있다. Google·Naver·SoftBank의 네트워크도 대표적인 AS 사례가 맞지만, AS를 단순히 ‘회사의 내부망’이라고만 부르면 범위가 지나치게 좁아진다. AS는 사내 PC만 있는 LAN이 아니라, 외부에 광고할 IP 프리픽스와 다른 AS를 향한 경로 정책까지 포함하는 라우팅 관리 영역이다.

- **회사와 AS의 관계**: 한 회사가 하나의 AS를 운영할 수도 있지만, 서비스·지역·인수 조직을 나누어 여러 AS를 운영할 수도 있다.
- **장소와 AS의 관계**: 하나의 AS가 여러 데이터센터와 국가에 걸쳐 있을 수 있으며, 한 데이터센터 안에도 여러 AS의 장비가 함께 존재할 수 있다.
- **접속 ISP와 Transit의 관계**: 가입자를 수용하는 접속 ISP도 AS이고, 광범위한 인터넷 도달성을 판매하는 Transit 사업자도 AS이다. 두 표현은 AS의 역할과 사업 관계를 설명한다.
- **IXP와 AS의 관계**: IXP의 L2 스위칭 패브릭은 여러 AS가 만나 Peering하는 교환 기반이다. IXP Route Server가 경로 교환용 ASN을 사용할 수는 있지만, 그렇다고 IXP 전체가 참가자 AS를 거느리는 상위 AS가 되는 것은 아니다.
- **일반 고객망과 AS의 관계**: 가정 LAN과 소규모 기업망은 보통 공인 ASN으로 BGP를 운영하지 않고 ISP의 기본 경로를 사용한다. 반면 다중 회선과 독립적인 경로 정책이 필요한 기업은 자체 AS를 운영할 수 있다.

**Network(네트워크)** 는 서로 통신하도록 연결된 장치와 경로의 집합을 넓게 가리킨다. 가정 LAN, 기업 VLAN, 데이터센터망과 Cloud VPC도 모두 네트워크이다. **Internet(인터넷)** 은 그중에서도 서로 다른 관리 주체의 네트워크들이 IP와 AS 간 라우팅으로 전 세계적으로 상호 연결된 체계이다. 따라서 모든 인터넷은 네트워크이지만, 모든 네트워크가 공용 인터넷인 것은 아니다.

그러므로 “ISP와 연결되는 순간 네트워크에 진입한다”는 표현은 정확하지 않다. PC는 ISP에 연결되기 전부터 가정 LAN이나 기업 LAN이라는 네트워크에 속해 있다. 더 정확하게는 **고객–ISP 접속 경계를 넘으면서 ISP가 운영하는 사업자 네트워크로 들어가고, ISP의 Internet Edge에서 다른 AS들과 연결되면서 공용 인터넷의 상호연결 경로를 이용한다**고 설명할 수 있다.

그림에는 서로 다른 두 경계를 붉은 점선으로 구분했다. 두 지점을 모두 막연하게 ‘인터넷 경계’라고 부르면 ISP가 인터넷의 바깥에 있는 것처럼 오해하기 쉽다.

1. **고객–ISP 접속 경계**: 가정·기업·데이터센터의 Customer Edge와 통신 사업자의 Provider Edge가 만나는 서비스·관리 책임의 접점이다. 사용자의 관점에서는 이 지점을 지나 ISP 망으로 들어가므로 흔히 “인터넷으로 나간다”고 표현한다.
2. **ISP Internet Edge·AS 경계**: 접속 ISP가 IXP, Transit 사업자, 콘텐츠 사업자와 BGP 경로와 트래픽을 교환하는 지점이다. ISP 관점에서 말하는 Internet Edge는 보통 이쪽에 가깝다.

접속 ISP의 가입자망·집선망·백본은 이미 ISP가 운영하는 하나의 AS이며 인터넷 연결을 구성하는 사업자 네트워크이다. 다만 ISP 내부의 모든 장비와 주소가 외부에서 직접 접근 가능한 ‘공개 구간’이라는 뜻은 아니다. 내부 링크에 사설 주소를 사용할 수도 있고, 가입자에게 CGNAT를 제공할 수도 있다. **인터넷을 구성하는 망인가**와 **외부에서 직접 접근 가능한가**는 별개의 질문이다.

- **가정·기업 관점**: 공유기·방화벽·Customer Edge와 ISP의 Provider Edge 사이가 접속 경계이다.
- **데이터센터 관점**: 외부 방화벽과 Edge Router가 내부 서비스망을 ISP 회선에 연결한다.
- **Cloud VPC 관점**: Internet Gateway가 VPC와 공용 연결의 논리적 접점이 된다. NAT Gateway는 Private Subnet의 외부 방향 주소 변환을 맡으며 인터넷 전체의 시작점은 아니다.
- **ISP 관점**: BNG·집선·백본을 거친 뒤 Internet Edge Router가 다른 AS, IXP와 Transit에 연결된다.
- **콘텐츠 사업자 관점**: 콘텐츠 Edge Router가 IXP·Transit·사설 Peering을 통해 다른 AS와 연결된다.

따라서 문서에서는 앞으로 막연한 ‘인터넷 경계’ 대신 **고객–ISP 접속 경계**, **ISP의 AS 경계**, **VPC Internet Gateway**처럼 어느 주체의 어떤 경계인지 함께 밝힌다.

기준 그림에서 기업 PC가 콘텐츠 AS까지 IXP 직접 Peering으로 전달되는 흐름을 단순화하면 다음과 같다.

```text
기업 PC
→ Access Switch와 L3 Core
→ 방화벽·NAT와 Edge Router
→ 접속 ISP의 집선·백본
→ 접속 ISP의 Internet Edge
→ IXP의 L2 Peering Fabric
→ Google 예시 콘텐츠 AS의 Edge Router
→ CDN Cache 또는 Origin
```

직접 Peering 경로를 사용할 수 없거나 정책상 선택하지 않으면 다음과 같은 Transit 경로가 대신 선택될 수 있다.

```text
접속 ISP의 Internet Edge
→ IP Transit AS의 Edge·Core
→ Google 예시 콘텐츠 AS의 Edge Router
→ CDN Cache 또는 Origin
```

이 경로에서도 사용자의 패킷이 Route Server를 통과하지는 않는다. Route Server는 BGP 경로 정보 교환을 돕고, 실제 데이터는 피어링 라우터와 IXP 스위칭 패브릭을 지나간다.

## 2. 하나의 거대한 L2 네트워크가 아닌 이유

[[Switch|L2 스위치]]는 같은 링크의 MAC 주소를 학습하며, 알 수 없는 유니캐스트와 브로드캐스트를 같은 VLAN 안에 전달한다. 전 세계 장치를 하나의 L2 네트워크로 묶으면 다음 문제가 생긴다.

- 모든 장치의 MAC 주소를 학습해야 하므로 테이블과 제어 범위가 지나치게 커진다.
- 브로드캐스트와 장애의 영향 범위가 전 세계로 확장될 수 있다.
- 주소만 보고 국가·사업자·지역 단위로 경로를 묶기 어렵다.
- 조직별 정책과 보안 경계를 독립적으로 운영하기 어렵다.

IP 주소는 프리픽스로 묶을 수 있다. 라우터는 `203.0.113.0/24`처럼 여러 주소를 하나의 경로로 다루므로 규모를 키우기 쉽다. L2는 현재 링크의 전달을, L3는 여러 네트워크 사이의 전달을 맡는 이유이다.

## 3. PC에서 인터넷 서버까지의 전달 과정

Qbo의 PC가 목적지 서버에 접속하는 과정을 단순화하면 다음과 같다.

1. DNS 조회로 서비스 이름에 대응하는 IP 주소를 얻는다.
2. PC는 목적지 IP가 자신의 서브넷 밖에 있음을 확인한다.
3. PC는 기본 게이트웨이의 MAC 주소를 ARP로 알아내고 첫 Ethernet 프레임을 보낸다.
4. 가정용 라우터는 기본 경로를 이용해 고객–ISP 접속 경계로 전달한다. 환경에 따라 사설 주소를 공인 주소로 바꾸는 NAT도 수행한다.
5. ISP의 Provider Edge·BNG가 가입자 트래픽을 받아 집선망과 백본으로 보낸다.
6. ISP 내부의 코어 라우터들이 해당 트래픽을 ISP의 Internet Edge까지 전달한다.
7. Internet Edge Router가 BGP로 선택된 직접 Peering, IXP 또는 Transit 경로로 다른 AS에 넘긴다.
8. 패킷은 목적지 AS의 Edge Router와 내부 라우팅을 거쳐 서버에 도달한다.

![라우터마다 바뀌는 MAC 주소와 유지되는 IP 주소](../assets/router-hop-frame.png)

NAT가 없는 단순한 예에서는 출발지·목적지 IP가 유지되고 MAC 주소가 링크마다 바뀐다. 실제 가정망에서는 NAT 때문에 사설 출발지 IP가 공인 IP로 변환될 수 있다.

## 4. 가정용 라우터가 모든 인터넷 경로를 몰라도 되는 이유

가정용 라우터는 보통 `0.0.0.0/0`, 즉 더 구체적인 경로가 없을 때 ISP로 보내는 기본 경로만 있으면 된다. 인터넷 전체 프리픽스를 저장할 필요가 없다.

반면 ISP의 인터넷 경계 라우터는 [[BGP]]를 통해 많은 프리픽스와 경로를 배울 수 있다. ISP 내부의 모든 장비가 동일한 전체 BGP 경로를 들고 있어야 하는 것도 아니다. 역할에 따라 기본 경로, 요약 경로, 내부 경로 또는 전체 경로를 나누어 가진다.

비유하면 PC와 가정용 라우터는 “일단 지역 물류센터로 보낸다”만 알면 된다. ISP의 경계 장비는 목적지 권역별 운송사를 선택하고, 각 중간 거점은 다음 거점만 선택한다. 실제 라우팅은 우편과 달리 계약, 혼잡, 장애와 정책까지 반영한다는 차이가 있다.

## 5. 인터넷의 계층과 메시 구조

가정에서 바라보면 `PC → 공유기 → ISP → 인터넷`처럼 계층적으로 보인다. 그러나 ISP와 콘텐츠 사업자 사이에는 직접 Peering, IXP, 복수 Transit, 사설 상호접속이 얽혀 있어 전체 모습은 메시 구조에 가깝다.

계층은 장비의 운영 역할을 설명하는 데 유용하다.

- **가입자·액세스 구간**: 가정과 회사를 ISP에 연결한다.
- **집선 구간**: 여러 가입자와 액세스 장비의 트래픽을 모은다.
- **백본·코어 구간**: 지역과 PoP 사이의 대용량 트래픽을 전달한다.
- **인터넷 경계**: 다른 AS와 BGP 경로를 교환하고 트래픽을 넘긴다.

그러나 이 계층이 전 세계 공통의 지휘 체계라는 뜻은 아니다. 각 AS가 자체 구조와 정책을 정한다.

## 6. 물리적으로 이어지는 경로

인터넷 통신은 결국 구리 케이블, 광섬유, 무선 링크, 해저 케이블과 장비 포트를 지난다. 먼 거리라고 해서 신호 약화를 해결하려고 Transit을 사용하는 것은 아니다. 장거리 전송에는 광전송 장비와 증폭기·재생기 등이 사용되고, Transit은 다른 AS의 수많은 목적지까지 경로와 패킷 운송을 제공하는 상업적·운영적 관계이다.

논리적 경로와 물리적 경로도 구분해야 한다. BGP에서 한 AS가 다음 AS로 보이더라도 실제 패킷은 그 사업자의 여러 PoP와 코어 라우터, 광전송망을 통과할 수 있다.

## 7. 관련 문서

- [[Switch|스위치]] · [[Router|라우터]] · [[BGP]]
- [[IXP Peering and Transit|IXP·피어링·트랜짓]] · [[Backbone Network|백본 네트워크]]
- [[Networking]] · [[CDN]]

## 8. 출처

- IETF, [RFC 1812 — Requirements for IP Version 4 Routers](https://www.rfc-editor.org/rfc/rfc1812.html). IPv4 라우터의 기본 전달 동작.
- IETF, [RFC 4632 — Classless Inter-domain Routing](https://www.rfc-editor.org/rfc/rfc4632.html). CIDR 프리픽스와 경로 집약의 배경.
- Internet Society, [How Does the Internet Work?](https://www.internetsociety.org/internet/how-it-works/). 독립 네트워크들이 상호 연결되는 인터넷의 기본 구조.

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Networking 분류|Networking]]
