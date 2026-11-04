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
updated: 2026-11-04
---

## 1. 개요

EMC는 엔터프라이즈 스토리지·데이터 보호 기술을 제공해 왔으며 2016년 Dell에 인수된 기업이다.

Dell은 2016년 9월 7일 EMC 인수 완료와 Dell EMC 브랜드 출범을 발표했다. 여기서는 스토리지 공급업체·제품군의 역사적 맥락을 다루며 단일 장비나 프로토콜 이름으로 사용하지 않는다.[^1]

## 2. 스토리지를 이해하는 기준

스토리지 제품은 이름보다 서버가 데이터에 접근하는 방식으로 구분하는 것이 좋다.

| 구분 | 서버에서 보는 형태 | 대표적인 접근 방식 |
|---|---|---|
| [[Block Storage\|블록]] | 디스크와 같은 블록 장치 | iSCSI, Fibre Channel |
| [[File Storage\|파일]] | 파일과 디렉터리 공유 | NFS, SMB |
| [[Object Storage\|오브젝트]] | 키와 메타데이터를 가진 객체 | S3 호환 API 등 |

예를 들어 Dell PowerScale은 스케일아웃 NAS 제품군으로 소개되며, 현행 제품 설명에는 파일뿐 아니라 S3 기반 오브젝트 접근도 포함된다. NAS라는 분류를 파일 전용이라는 뜻으로 해석해서는 안 된다.[^2] Dell ECS는 소프트웨어 정의 오브젝트 스토리지 플랫폼의 예시이다. 이 설명은 ECS의 개념을 다루며 특정 모델의 현재 판매·지원 기간을 보장하지 않는다.[^3] EMC라는 이름만으로 NAS인지 SAN인지, 어떤 API를 지원하는지 결정할 수 없다.

## 3. Zadara와 비교할 때

[[VPSA]]와 비교할 때는 블록·파일 서비스, [[NGOS]]와 비교할 때는 오브젝트 서비스를 대상으로 잡는다. 제조사 이름과 특정 서비스 하나를 직접 대응시키면 범위가 맞지 않는다.

동일한 프로토콜을 제공해도 용량 확장, 컨트롤러 이중화, 스냅샷·복제, 관리 책임과 계약 조건은 다를 수 있다. 구매형 장비인지 관리형 서비스인지도 실제 제공 조건을 기준으로 구분해야 한다.

## 4. 제품 이름 뒤에 있는 스토리지 구성

엔터프라이즈 스토리지 어레이를 개념적으로 보면 호스트와 연결하는 front-end 포트, 요청을 처리하는 컨트롤러·CPU·캐시, 실제 데이터를 저장하는 back-end 드라이브와 데이터 보호 계층이 있다. 제품에 따라 전용 어레이·분산 노드·소프트웨어 정의 방식으로 구현되며 모든 Dell 제품이 같은 내부 구조를 갖는 것은 아니다. 예를 들어 PowerStore 문서는 호스트 쓰기의 캐시·비휘발성 보호·저장 매체로 이어지는 경로를 설명한다.[^d-powerstore]

블록 서비스에서는 호스트가 논리 디스크를 받아 그 위 파일시스템·DB를 관리하고, NAS에서는 파일 서비스가 파일시스템과 공유 접근을 제공하며, 오브젝트 서비스에서는 API가 키·메타데이터·객체 데이터를 다룬다. PowerScale·ECS 같은 제품 예시는 이 접근 방식의 차이를 이해하는 데 쓰는 것이지 한 회사의 모든 제품을 NAS나 SAN 하나로 묶기 위한 이름이 아니다.[^2][^3]

컨트롤러 이중화는 컨트롤러 장애 경로를, RAID·이레이저 코딩·미러링은 드라이브·노드 손실 범위를, 원격 복제는 다른 위치의 복구 가능성을, 스냅샷·백업은 데이터 상태의 보존·복구를 다룬다. 용어들이 같은 “보호”로 쓰여도 해결하는 실패와 필요한 운영 절차가 다르다. 실제 지원 범위는 비교할 제품의 구성과 문서를 기준으로 한다.

## 5. 요구사항과 증상을 구성 요소에 연결하기

제품 비교에는 용량 증가율·IOPS·처리량·지연·작업 크기·동시 접근과 호스트 운영체제·프로토콜·가용성·지원 조건이 필요하다. 작은 DB 동기 쓰기와 대형 파일 공유의 요구가 다르므로 브랜드 이름이나 총 용량만으로 적합성을 정하기 어렵다. 관리형 서비스와 구매형 시스템도 누가 패치·장애 대응·증설을 맡는지의 경계가 다르다.

호스트에 디스크가 안 보이는 문제는 연결·매핑·프로토콜을, SMB 파일 접근 거부는 신원·공유·파일 권한을, 풀 공간 부족은 논리 볼륨과 물리 소비·보존 이력을, 지연 증가는 호스트·네트워크·컨트롤러·매체와 작업 변화의 관계를 본다. 같은 스토리지 제조사의 제품에서도 원인의 계층이 다르므로 하나의 관리 화면 상태만으로 전체 서비스 정상 여부를 확정할 수는 없다.

이런 구성 요소 중심의 읽기 방법으로 [[VPSA]]나 다른 서비스와 비교하면 “EMC와 클라우드 중 어느 쪽이 빠른가” 같은 넓은 질문을 실제 비교 가능한 프로토콜·작업·성능 목표·운영 책임의 질문으로 바꿀 수 있다.

## 6. 관련 문서

- [[Block Storage]]
- [[File Storage]]
- [[Object Storage]]
- [[VPSA]]
- [[NGOS]]

- [[Storage]]

## 7. 출처

[^1]: [Dell — EMC 인수 완료 발표](https://www.dell.com/en-us/dt/corporate/newsroom/announcements/2016/09/20160907-01.htm)
[^2]: [Dell — PowerScale](https://www.dell.com/en-us/shop/storage-servers-and-networking-for-business/sf/powerscale)
[^3]: [Dell Technologies Info Hub — Introduction to ECS](https://infohub.delltechnologies.com/en-uk/l/scale-quickly-and-efficiently-with-dell-ecs-and-vertica-analytical-database/introduction-to-ecs/)

[^d-powerstore]: [Dell — PowerStore data efficiencies and write path](https://www.dell.com/en-us/infohub/articles/dell-powerstore-data-efficiencies)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Storage 분류|Storage]]
