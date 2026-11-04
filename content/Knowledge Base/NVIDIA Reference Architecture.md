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
updated: 2026-11-04
---

## 1. 개요

NVIDIA Reference Architecture는 AI 인프라의 컴퓨팅·네트워크·스토리지·소프트웨어를 함께 구성하기 위한 참조 설계이다.

목록의 “엔비디아 표준 아키텍트”는 정확한 원어가 확인되지 않아 이 문서에서는 NVIDIA 참조 아키텍처로 해석했다. 이 별칭이 공식 용어나 원래 의도와 일치한다고 확정한 것은 아니다. 사람의 직무인 [[AI Infrastructure Architect|AI 인프라 아키텍트]]와 구분한다.[^1]

## 2. 참조 설계가 필요한 이유

여러 GPU 서버를 연결할 때는 GPU 수량 외에도 서버별 네트워크 연결, 스위치 구조, 스토리지 대역폭, 관리망과 소프트웨어 구성이 맞아야 한다. 참조 아키텍처는 이러한 구성 요소와 연결 관계를 검토할 수 있는 기준을 제공한다.

NVIDIA는 엔터프라이즈 AI 팩토리용 지침과 파트너 설계를 공개한다. 하나의 고정된 장비 목록이 모든 조직에 적용되는 국제 표준이라는 뜻은 아니다.[^1][^2] 참조 설계를 읽는 것, NVIDIA-Certified 구성 요소를 사용하는 것, 파트너의 전체 설계가 NVIDIA endorsement를 받는 것은 서로 다른 범위이다. 공식 안내는 파트너 설계·설계 검토·통과 설계 공개의 절차를 구분한다.[^v-endorse]

## 3. 설계를 읽는 관점

| 영역 | 파악할 내용 |
|---|---|
| 컴퓨팅 | GPU·CPU·메모리와 서버 단위 구성 |
| 네트워크 | 노드 간 통신, 데이터 접근, 관리 트래픽 |
| 스토리지 | 데이터셋·체크포인트의 용량과 처리량 |
| 소프트웨어 | 드라이버·실행 환경·배포·관측 계층 |
| 시설 | 전력, 냉각과 랙 배치 조건 |

특정 설계의 지원 GPU 세대·서버·규모는 해당 문서의 버전과 검증 범위를 기준으로 확인한다. 일부 구성 변경이 전체 설계의 검증 결과까지 유지한다는 보장은 없다. 또한 현재 Enterprise RA 개요는 single-tenant·온프레미스 배치에 중점을 둔다고 명시한다. 이를 모든 멀티테넌트 GPU 클라우드의 인증 표준으로 확대하면 안 된다.[^v-tenets]

## 4. DGX·HGX와의 관계

[[HGX]]는 파트너 서버의 기반이 되는 GPU 플랫폼이고, [[DGX]]는 NVIDIA의 통합 AI 시스템·플랫폼이다. 참조 아키텍처는 이런 시스템들을 네트워크·스토리지·소프트웨어와 연결해 [[AI Factory]]를 구성하는 설계 관점이다. 제품 이름, 설계 지침, 엔지니어의 직무를 서로 구분해야 한다.

## 5. 노드에서 랙·확장 단위까지 읽는 방법

참조 설계는 서버 한 대의 GPU 수뿐 아니라 노드·랙·확장 단위와 그 사이 연결을 다룬다. Compute 노드는 CPU·GPU·메모리·NIC를 제공하고, 스위치 패브릭은 노드 간 통신을, 저장·제어 노드는 데이터 공급과 클러스터 운영을 담당한다. 랙에는 전력·냉각·포트·케이블이라는 물리 조건이 함께 적용된다. 개별 장치가 빠르다는 사실만으로 이 연결 전체가 균형을 이룬 것은 아니다.[^v-tenets]

East-west는 GPU 작업자 간 통신, north-south는 외부·스토리지·사용자와의 연결 맥락, 관리 트래픽은 구성·관측·운영의 맥락에서 읽을 수 있다. 실제 구분은 해당 설계 문서의 정의를 따르며 물리망을 통합하거나 전용으로 나누는 선택이 존재한다. 서로 다른 작업이 같은 링크를 공유하면 경합이 생길 수 있고, 분리하면 포트·장비·운영 복잡성이 늘 수 있다.

NVIDIA의 C-G-N-B 표기는 CPU 소켓·GPU·네트워크 어댑터·GPU당 east-west 대역폭이라는 구성 패턴을 설명한다. 이는 전체 애플리케이션 성능을 하나의 숫자로 보장하는 표기가 아니다. Certified 서버 노드를 사용한 사실과 그 노드·스위치·스토리지의 전체 배치가 DRB 검토를 거쳐 endorsement를 받은 사실도 구분해야 한다.[^v-tenets][^v-endorse]

## 6. 검증된 설계를 실제 요구에 적용하는 판단

클러스터를 늘리면 연산량뿐 아니라 collective 통신·데이터 공급·체크포인트·관리 자원의 요구도 커진다. 스위치 토폴로지와 상위 링크의 oversubscription은 동시에 통신할 때 사용할 수 있는 집계 대역폭에 영향을 준다. 장애 도메인을 어떻게 나누는지도 스위치·랙 단위 실패의 영향 범위를 바꾼다. 그래서 같은 GPU 수라도 연결과 운영 구성이 다르면 실제 결과가 달라진다.

참조 구성과 같은 장비를 배치했는데 작업이 느리다면 모델·입력 크기·정밀도·배치·병렬화 방식이 검증 조건과 같은지, 드라이버·펌웨어·네트워크 설정·스토리지 공급이 일치하는지를 비교한다. 원래 검증된 조건에서 벗어난 부분이 무엇인지가 설계를 평가하는 근거다. RA는 이 비교의 출발점을 제공하며 모든 워크로드의 수용량을 대신 산정하지는 않는다.

GPU 서버를 한 대 더 추가하는 선택이 실제 처리량을 높이는지, 네트워크·스토리지·작업 배치의 병목을 먼저 해결해야 하는지는 workload benchmark와 계층별 지표로 판단한다. 기존 문서의 single-tenant·온프레미스 범위와 버전별 검증 조건도 유지해, 다른 운영 모델에 적용할 때 필요한 추가 설계를 분명히 한다.

## 7. 관련 문서

- [[AI Infrastructure Architect]]
- [[AI Factory]]
- [[GPU Farm]]
- [[DGX]]
- [[HGX]]

- [[Server]]

- [[Networking]]

## 8. 출처

[^1]: [NVIDIA — Reference Architectures](https://docs.nvidia.com/reference-architectures/index.html)
[^2]: [NVIDIA — Enterprise Reference Architectures](https://www.nvidia.com/en-us/technologies/enterprise-reference-architecture/)

[^v-endorse]: [NVIDIA — Introducing Reference Architectures: endorsement](https://docs.nvidia.com/enterprise-reference-architectures/white-paper/latest/introducing-nvidia-reference-architectures.html)

[^v-tenets]: [NVIDIA — Design Tenets: deployment scope](https://docs.nvidia.com/enterprise-reference-architectures/white-paper/latest/design-tenets.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
