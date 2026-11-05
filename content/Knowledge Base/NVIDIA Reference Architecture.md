---
title: NVIDIA Reference Architecture
type: knowledge
domain: Compute
tags:
  - NVIDIA
  - AI
aliases:
  - "엔비디아 표준 아키텍트"
  - "엔비디아 참조 아키텍처"
  - "NVIDIA Enterprise Reference Architecture"
  - "NVIDIA ERA"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-05
---

## 1. 개요

NVIDIA Reference Architecture는 AI 인프라의 컴퓨팅·네트워크·스토리지·소프트웨어를 함께 구성하기 위한 참조 설계이다.

목록의 “엔비디아 표준 아키텍트”는 정확한 원어가 확인되지 않아 이 문서에서는 NVIDIA 참조 아키텍처로 해석했다. 이 별칭이 공식 용어나 원래 의도와 일치한다고 확정한 것은 아니다. 사람의 직무인 [[AI Infrastructure Architect|AI 인프라 아키텍트]]와 구분한다.[^1]

## 2. 참조 설계와 실제 구축의 관계

AI 클러스터는 서버·스위치·스토리지를 각각 구매하는 것만으로 완성되지 않는다. 서버가 얼마나 통신하는지, 데이터를 어디서 읽는지, 랙에 필요한 전력과 냉각이 충분한지까지 연결해야 한다. 참조 아키텍처는 이러한 요소의 조합을 검토할 수 있는 설계의 출발점이다.[^1][^v-tenets]

예를 들어 같은 수의 GPU 서버를 사용해도 노드 사이 연결과 데이터 공급 구조가 다르면 분산 학습의 결과가 달라질 수 있다. 참조 설계는 어떤 조건에서 구성 요소를 조합하고 검토했는지 설명한다. 다른 장비나 규모로 바꾸면 원래 검증 범위를 그대로 적용할 수 있는지 다시 판단해야 한다.

따라서 참조 설계는 모든 조직에 적용되는 국제 표준이나 특정 제품의 성능 보증서와 다르다. NVIDIA-Certified 구성 요소 사용과 전체 파트너 설계의 검토·endorsement도 구분해야 한다. 설계를 참고한다는 것은 그림을 복사하는 것이 아니라 그 연결 관계와 적용 조건을 이해하는 것이다.[^v-endorse]

## 3. 참조 설계의 검토 항목

| 영역 | 파악할 내용 |
|---|---|
| 컴퓨팅 | GPU·CPU·메모리와 서버 단위 구성 |
| 네트워크 | 노드 간 통신, 데이터 접근, 관리 트래픽 |
| 스토리지 | 데이터셋·체크포인트의 용량과 처리량 |
| 소프트웨어 | 드라이버·실행 환경·배포·관측 계층 |
| 시설 | 전력, 냉각과 랙 배치 조건 |

특정 설계의 지원 GPU 세대·서버·규모는 해당 문서의 버전과 검증 범위를 기준으로 확인한다. 일부 구성 변경이 전체 설계의 검증 결과까지 유지한다는 보장은 없다. 또한 현재 Enterprise RA 개요는 single-tenant·온프레미스 배치에 중점을 둔다고 명시한다. 이를 모든 멀티테넌트 GPU 클라우드의 인증 표준으로 확대하면 안 된다.[^v-tenets]

## 4. 노드·랙·확장 단위의 구성

참조 설계는 서버 한 대의 GPU 수뿐 아니라 노드·랙·확장 단위와 그 사이 연결을 다룬다. Compute 노드는 CPU·GPU·메모리·NIC를 제공하고, 스위치 패브릭은 노드 간 통신을, 저장·제어 노드는 데이터 공급과 클러스터 운영을 담당한다. 랙에는 전력·냉각·포트·케이블이라는 물리 조건이 함께 적용된다. 개별 장치가 빠르다는 사실만으로 이 연결 전체가 균형을 이룬 것은 아니다.[^v-tenets]

East-west는 GPU 작업자 사이의 통신, north-south는 외부·스토리지·사용자와 주고받는 통신으로 이해할 수 있다. 관리 트래픽은 구성 변경·상태 확인·운영에 사용된다. 실제 구분은 해당 설계 문서의 정의를 따르며 물리망을 통합하거나 전용으로 나누는 선택이 존재한다. 서로 다른 작업이 같은 링크를 공유하면 경합이 생길 수 있고, 분리하면 포트·장비·운영 복잡성이 늘 수 있다.

NVIDIA의 C-G-N-B 표기는 CPU 소켓·GPU·네트워크 어댑터·GPU당 east-west 대역폭이라는 구성 패턴을 설명한다. 이는 전체 애플리케이션 성능을 하나의 숫자로 보장하는 표기가 아니다. Certified 서버 노드를 사용한 사실과 그 노드·스위치·스토리지의 전체 배치가 DRB 검토를 거쳐 endorsement를 받은 사실도 구분해야 한다.[^v-tenets][^v-endorse]

## 5. DGX·HGX와의 관계

[[HGX]]는 파트너 서버의 기반이 되는 GPU 플랫폼이고, [[DGX]]는 NVIDIA의 통합 AI 시스템·플랫폼이다. 참조 아키텍처는 이런 시스템들을 네트워크·스토리지·소프트웨어와 연결해 [[AI Factory]]를 구성하는 설계 관점이다. 제품 이름, 설계 지침, 엔지니어의 직무를 서로 구분해야 한다.

## 6. 관련 문서

- [[AI Infrastructure Architect]]
- [[AI Factory]]
- [[GPU Farm]]
- [[DGX]]
- [[HGX]]

- [[Server]]

- [[Networking]]

## 7. 출처

[^1]: [NVIDIA — Reference Architectures](https://docs.nvidia.com/reference-architectures/index.html)

[^v-endorse]: [NVIDIA — Introducing Reference Architectures: endorsement](https://docs.nvidia.com/enterprise-reference-architectures/white-paper/latest/introducing-nvidia-reference-architectures.html)

[^v-tenets]: [NVIDIA — Design Tenets: deployment scope](https://docs.nvidia.com/enterprise-reference-architectures/white-paper/latest/design-tenets.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
