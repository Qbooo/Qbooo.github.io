---
title: EMC
type: knowledge
domain: Storage
tags:
  - Dell
  - Storage
aliases:
  - "Dell EMC"
  - "EMC 스토리지"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-05
---

## 1. 개요

EMC는 엔터프라이즈 스토리지·데이터 보호 기술을 제공해 왔으며 2016년 Dell에 인수된 기업이다.

Dell은 2016년 9월 7일 EMC 인수 완료와 Dell EMC 브랜드 출범을 발표했다. 여기서는 스토리지 공급업체·제품군의 역사적 맥락을 다루며 단일 장비나 프로토콜 이름으로 사용하지 않는다.[^1]

## 2. EMC라는 이름과 스토리지 기술의 구분

EMC는 특정 저장 방식이 아니라 기업과 제품군의 역사에 속하는 이름이다. “EMC 스토리지”라는 말만으로는 디스크를 제공하는지, 파일을 공유하는지, 오브젝트 API를 제공하는지 알 수 없다. 실제 개념을 이해하려면 제조사 이름 다음에 제품과 제공 기능을 구분해야 한다.

예를 들어 PowerScale은 여러 노드로 파일 저장 공간을 제공하는 스케일아웃 NAS 제품군이며, ECS는 오브젝트 스토리지 플랫폼의 예이다. 이들은 모두 스토리지라는 큰 범주에 속하지만 클라이언트가 데이터를 다루는 방식은 다르다.[^2][^3]

Dell의 EMC 인수와 브랜드 변화는 제품이 어떤 공급업체 계보에 속하는지 설명한다. 반면 블록·파일·오브젝트, 컨트롤러·어레이·분산 노드는 데이터가 어떤 구조로 저장·제공되는지 설명한다. 기업의 이름과 기술의 이름을 구분하는 것이 이 문서의 핵심이다.[^1]

## 3. 스토리지를 이해하는 기준

스토리지 제품은 이름보다 서버가 데이터에 접근하는 방식으로 구분하는 것이 좋다.

| 구분 | 서버에서 보는 형태 | 대표적인 접근 방식 |
|---|---|---|
| [[Block Storage\|블록]] | 디스크와 같은 블록 장치 | iSCSI, Fibre Channel |
| [[File Storage\|파일]] | 파일과 디렉터리 공유 | NFS, SMB |
| [[Object Storage\|오브젝트]] | 키와 메타데이터를 가진 객체 | S3 호환 API 등 |

예를 들어 Dell PowerScale은 스케일아웃 NAS 제품군으로 소개되며, 현행 제품 설명에는 파일뿐 아니라 S3 기반 오브젝트 접근도 포함된다. NAS라는 분류를 파일 전용이라는 뜻으로 해석해서는 안 된다.[^2] Dell ECS는 소프트웨어 정의 오브젝트 스토리지 플랫폼의 예시이다. 이 설명은 ECS의 개념을 다루며 특정 모델의 현재 판매·지원 기간을 보장하지 않는다.[^3] EMC라는 이름만으로 NAS인지 SAN인지, 어떤 API를 지원하는지 결정할 수 없다.

## 4. 스토리지 제품의 내부 구성

엔터프라이즈 스토리지 어레이를 개념적으로 보면 호스트와 연결하는 front-end 포트, 요청을 처리하는 컨트롤러·CPU·캐시, 실제 데이터를 저장하는 back-end 드라이브와 데이터 보호 계층이 있다. 제품에 따라 전용 어레이·분산 노드·소프트웨어 정의 방식으로 구현되며 모든 Dell 제품이 같은 내부 구조를 갖는 것은 아니다. 예를 들어 PowerStore 문서는 호스트 쓰기의 캐시·비휘발성 보호·저장 매체로 이어지는 경로를 설명한다.[^d-powerstore]

블록 서비스에서는 호스트가 논리 디스크를 받아 그 위 파일시스템·DB를 관리하고, NAS에서는 파일 서비스가 파일시스템과 공유 접근을 제공하며, 오브젝트 서비스에서는 API가 키·메타데이터·객체 데이터를 다룬다. PowerScale·ECS 같은 제품 예시는 이 접근 방식의 차이를 이해하는 데 쓰는 것이지 한 회사의 모든 제품을 NAS나 SAN 하나로 묶기 위한 이름이 아니다.[^2][^3]

컨트롤러 이중화는 컨트롤러 장애 경로를, RAID·이레이저 코딩·미러링은 드라이브·노드 손실 범위를, 원격 복제는 다른 위치의 복구 가능성을, 스냅샷·백업은 데이터 상태의 보존·복구를 다룬다. 용어들이 같은 “보호”로 쓰여도 해결하는 실패와 필요한 운영 절차가 다르다. 실제 지원 범위는 비교할 제품의 구성과 문서를 기준으로 한다.

## 5. 관련 문서

- [[Block Storage]]
- [[File Storage]]
- [[Object Storage]]
- [[VPSA]]
- [[NGOS]]

- [[Storage]]

## 6. 출처

[^1]: [Dell — EMC 인수 완료 발표](https://www.dell.com/en-us/dt/corporate/newsroom/announcements/2016/09/20160907-01.htm)
[^2]: [Dell — PowerScale](https://www.dell.com/en-us/shop/storage-servers-and-networking-for-business/sf/powerscale)
[^3]: [Dell Technologies Info Hub — Introduction to ECS](https://infohub.delltechnologies.com/en-uk/l/scale-quickly-and-efficiently-with-dell-ecs-and-vertica-analytical-database/introduction-to-ecs/)

[^d-powerstore]: [Dell — PowerStore data efficiencies and write path](https://www.dell.com/en-us/infohub/articles/dell-powerstore-data-efficiencies)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Storage 분류|Storage]]
