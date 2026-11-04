---
title: SNMP
type: knowledge
domain: Networking
tags: []
aliases: ["Simple Network Management Protocol"]
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-04
---

## 1. 개요

SNMP(Simple Network Management Protocol)는 네트워크 장비와 서버 등의 관리 정보를 조회하거나 변경하고, 이벤트 알림을 주고받는 프로토콜이다.

스위치 전용 기능이 아니라 여러 종류의 장비를 관리하는 데 사용한다.[^1]

예를 들어 관리 시스템이 스위치의 인터페이스 상태를 주기적으로 읽거나, 장비에서 발생한 이벤트 알림을 수신하는 데 활용할 수 있다. 실제로 조회할 수 있는 항목은 장비의 구현과 MIB에 따라 달라진다.

## 2. 장비의 상태가 관리 시스템까지 전달되는 구조

스위치나 서버에서 Agent는 운영체제·드라이버·장비 기능이 가진 정보를 관리 객체의 값으로 노출한다. Manager는 별도 관리 서버의 프로세스로 동작하며 네트워크를 통해 필요한 객체 인스턴스를 조회한다. MIB 모듈은 그 값의 이름·형식·단위·의미를 정의하는 설명이지 상태 값을 직접 저장하는 외부 데이터베이스라는 뜻은 아니다. 같은 OID 계열에서도 인터페이스 인덱스 같은 인스턴스 식별을 통해 여러 NIC나 포트를 구분한다.[^1][^v-smi]

GET 요청이 UDP로 전송되면 Agent는 권한과 객체 지원 여부를 확인해 응답한다. 관리 시스템은 정해진 시간 안에 응답이 없을 때 timeout으로 처리할 수 있다. timeout은 장비 다운의 확정 증거가 아니다. 패킷 손실·방화벽·잘못된 보안 설정·응답 지연도 같은 겉모습을 만든다. Trap은 이벤트를 먼저 알리는 방식이며 프로토콜 차원의 응답을 받지 않는다. Inform의 확인 응답도 영구 기록·운영자의 처리 완료를 보장하는 의미는 아니다.[^3]

폴링은 주기 사이에 발생했다 사라진 순간 상태를 놓칠 수 있고, 알림은 네트워크나 수집기 장애 때 유실될 수 있다. 두 경로를 함께 사용하면 주기적인 현재 상태와 이벤트 시점을 연결할 수 있다. 전체 관리 시스템에는 수집 주기·보존·시간 정렬과 알림 규칙도 필요하다.

## 3. 구성 요소

| 요소 | 역할 |
|---|---|
| Manager | 관리 대상에 정보를 요청하거나 알림을 받는 관리 시스템 |
| Agent | 관리 대상에서 정보를 제공하거나 알림을 보내는 구성 요소 |
| MIB | 관리할 정보의 이름·형식·의미를 정의하는 체계 |
| OID | 관리 항목을 식별하는 계층형 식별자 |

Manager와 Agent는 이해를 돕는 전통적인 명칭이다. SNMP 표준은 요청 생성·응답·알림 송수신 등을 수행하는 애플리케이션 역할로 구조를 설명한다.[^1] 관리 정보의 객체 이름·형식·의미는 MIB 모듈에서 정의하며 OID로 식별한다. 이 정의 체계는 SMIv2를 설명하는 RFC 2578을 참고한다.[^v-smi]

## 4. 동작 방식

### 3.1. 조회와 변경

- GET: 지정한 관리 항목을 조회한다.
- GETNEXT: 사전식(lexicographic) 순서에서 접근 가능한 다음 OID의 항목을 조회한다.
- GETBULK: 여러 항목을 효율적으로 조회한다. SNMPv1에는 없다.
- SET: 쓰기 권한이 있는 관리 항목의 값을 변경한다.

주기적으로 조회하는 방식을 폴링(polling)이라고 한다. 여러 시점의 카운터 값을 수집하면 변화량을 계산해 트래픽 등의 추이를 분석할 수 있다.

### 3.2. 이벤트 알림

Trap은 관리 대상이 이벤트를 알리는 메시지로, 프로토콜 차원의 수신 확인을 요구하지 않는다. Inform은 수신 측의 응답을 요구한다. 알림과 주기적인 조회는 서로 보완적으로 사용할 수 있다.[^3]

## 5. 누적 카운터를 트래픽 속도로 바꾸기

인터페이스의 누적 바이트 카운터는 “현재 초당 전송량”이 아니다. 두 번 측정한 값의 차이를 실제 시간 차이로 나누어 속도로 해석한다. 예를 들어 10초 사이에 카운터가 100,000,000바이트 증가했다면 평균 약 10 MB/s이고 비트 단위로는 약 80 Mbit/s다. 단위와 실제 측정 간격을 맞추지 않으면 그래프의 의미가 달라진다.

Counter32는 표현 범위를 넘으면 순환할 수 있고 장비·인터페이스 재시작은 카운터의 연속성을 끊을 수 있다. 단순히 이전 값을 빼면 음수나 큰 급증으로 잘못 해석할 수 있다. 고속 인터페이스에서 Counter64와 discontinuity 정보를 사용하는 이유다. RFC 2863은 인터페이스 카운터 크기와 불연속 시점을 별도로 정의한다.[^d-interface]

오류 진단에서도 “응답 없음”, “객체 없음”, “값이 이상함”을 나눈다. 응답이 없으면 전송·보안·Agent 상태, noSuchObject류 결과는 지원 MIB와 정확한 OID·인스턴스, 갑작스러운 속도 급증은 단위·수집 간격·인덱스 변경·카운터 초기화를 비교할 수 있다. SNMPv3 인증·암호화가 성공해도 VACM이 해당 객체 조회를 허용하지 않을 수 있고, 반대로 객체가 읽힌다고 변경 권한까지 있다는 뜻은 아니다. 알림이 없다는 사실만으로 사건이 없었다고 결론 내리지 않는 것도 이 구조에서 나온다.



## 6. 통신과 보안

일반적인 UDP 매핑에서 요청을 받는 Agent는 UDP 161, 알림을 받는 시스템은 UDP 162를 사용한다. 이는 대표적인 매핑이며 SNMP가 모든 환경에서 UDP만 사용하는 것은 아니다.[^2]

SNMPv3의 USM은 사용자 기반 인증과 암호화 기능을 제공한다. 다만 SNMPv3라고 해서 항상 암호화되는 것은 아니다. USM의 noAuthNoPriv는 인증·암호화를 하지 않고, authNoPriv는 인증만, authPriv는 인증과 암호화를 제공한다. 허용할 관리 정보와 작업의 범위는 VACM 같은 접근 제어 모델이 별도로 다룬다.[^v-vacm] 알고리즘과 지원 범위도 장비별로 확인해야 한다.[^4]

SNMPv1·v2c의 community 기반 접근과 v3 USM을 구분한다. Community 문자열을 아는 것만으로 메시지 암호화가 제공되는 것은 아니므로 신뢰할 수 없는 네트워크에서 보호된 인증·기밀성을 제공하는 방식과 같게 보면 안 된다.[^d-community] 일반적인 UDP 161 요청·162 알림 경로에 방화벽이 허용한 방향과, Agent의 보안·VACM 정책이 허용하는 객체 범위가 함께 맞아야 실제 조회가 성립한다.


## 7. 관련 문서

- [[Networking]]

- [[Server]]

## 8. 출처

[^1]: [RFC 3411 — SNMP Management Frameworks](https://www.rfc-editor.org/rfc/rfc3411.html)
[^2]: [RFC 3417 — Transport Mappings for SNMP](https://www.rfc-editor.org/rfc/rfc3417.html)
[^3]: [RFC 3416 — Protocol Operations for SNMP](https://www.rfc-editor.org/rfc/rfc3416.html)
[^4]: [RFC 3414 — User-based Security Model for SNMPv3](https://www.rfc-editor.org/rfc/rfc3414.html). 초기 표준의 알고리즘 예시를 현재의 권장 설정으로 간주하지 않고, 구현체의 최신 자료도 확인한다.

[^v-smi]: [RFC 2578 — SMIv2](https://www.rfc-editor.org/rfc/rfc2578.html)

[^v-vacm]: [RFC 3415 — VACM](https://www.rfc-editor.org/rfc/rfc3415.html)

[^d-interface]: [RFC 2863 — Interfaces Group MIB](https://www.rfc-editor.org/rfc/rfc2863.html)

[^d-community]: [RFC 3584 — SNMP version coexistence and community-based messages](https://www.rfc-editor.org/rfc/rfc3584.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Networking 분류|Networking]]
