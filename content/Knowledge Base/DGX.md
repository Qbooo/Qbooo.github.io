---
title: DGX
type: knowledge
domain: Compute
tags:
  - NVIDIA
  - GPU
aliases:
  - "NVIDIA DGX"
  - "엔비디아 DGX"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-05
---

## 1. 개요

NVIDIA DGX는 NVIDIA가 하드웨어·소프트웨어·지원 체계를 통합해 제공하는 AI 시스템 및 플랫폼 계열이다.

GPU 카드 한 장의 모델명이 아니며, 개별 시스템부터 확장된 인프라까지 제품 범위가 있다.[^1]

## 2. 통합 시스템을 사용하는 이유

GPU를 활용하려면 GPU 자체 외에도 CPU·메모리·장치 연결·네트워크·전원·냉각과 실행 소프트웨어가 맞아야 한다. DGX는 이러한 요소를 통합한 시스템·플랫폼으로 제공해 사용자가 부품 하나씩을 조합하는 것과 다른 도입 방식을 제공한다.[^1]

예를 들어 DGX H100/H200는 GPU 여러 개가 고속으로 통신하도록 구성된 서버의 구체적인 사례이고, 여러 DGX 시스템을 네트워크·스토리지와 결합한 참조 구성도 있다. 서버 한 대의 제품 구성과 클러스터 전체의 구조는 구분해야 한다.[^v-model][^d-basepod]

통합 제품이라는 사실이 모든 모델에 같은 GPU나 지원 범위가 있다는 뜻은 아니다. DGX라는 계열 이름은 제품의 성격을, H100/H200 같은 구체적인 모델은 실제 구성을 이해하는 기준을 제공한다. [[HGX]]와의 차이도 성능 등급보다 완제품·통합 시스템과 파트너 서버용 플랫폼의 제공 범위에서 이해하는 편이 정확하다.

## 3. DGX H100/H200의 내부 구성

구체적인 예로 DGX H100/H200는 해당 모델의 GPU 8개와 NVSwitch·CPU·시스템 메모리·NVMe·네트워크 어댑터·관리 기능을 결합한 시스템이다. GPU HBM은 연산 데이터의 장치 메모리이고 시스템 RAM은 CPU의 작업 메모리이며, 로컬 NVMe는 영속 저장·작업 데이터 영역이라는 서로 다른 역할을 가진다. BMC는 운영체제와 별도로 장비 전원·상태 등의 관리 경로를 제공한다. 각 수량·사양은 해당 모델의 사용자 가이드 기준이다.[^v-model]

데이터셋을 읽으면 저장 경로에서 CPU 처리·GPU 메모리 전송을 거쳐 연산에 사용된다. GPU 간 데이터 교환은 NVLink·NVSwitch 같은 내부 연결을, 다른 서버와의 분산 작업은 NIC와 외부 네트워크를 사용한다. 관리 BMC로 접속되는 것과 운영체제의 네트워크·GPU 작업이 정상이라는 사실은 다르다.

단일 DGX 노드의 내부 연결과 BasePOD·SuperPOD의 외부 패브릭·스토리지·제어 구성은 범위가 다르다. BasePOD는 DGX 시스템을 기반으로 네트워크·소프트웨어·관리와 저장 구성을 연결해 기업용 AI 클러스터의 배포를 안내하는 참조 설계다. H100/H200 BasePOD 가이드는 네트워크·NFS·클러스터 관리·작업 관리자·검증을 별도 장으로 다룬다.[^d-basepod] SuperPOD는 여러 시스템·저장·네트워크·소프트웨어·관리를 결합한 플랫폼이므로 서버 한 대를 구매한 사실만으로 전체 클러스터 설계가 완성되지는 않는다.[^v-superpod]

## 4. HGX와 비교

| 기준 | DGX | [[HGX]] |
|---|---|---|
| 중심 개념 | NVIDIA 통합 AI 시스템·플랫폼 | 파트너 서버에 통합되는 GPU 플랫폼 |
| 서버 선택 | DGX 제품군과 구성 기준 | 서버 제조사의 HGX 기반 제품 기준 |
| 운영·지원 | DGX 제공 범위와 계약 기준 | 서버 업체와 소프트웨어 제공 범위 기준 |

어느 쪽이 항상 더 빠르거나 저렴하다고 정할 수는 없다. GPU 세대, 서버 구성과 소프트웨어·지원 범위를 같은 조건으로 비교해야 한다.[^1][^2]

## 5. 클러스터와 서비스의 관계

DGX 기반 시스템도 [[GPU Farm]]이나 [[AI Factory]]의 구성 요소가 될 수 있다. 개별 DGX 시스템과 DGX SuperPOD의 범위도 다르다. SuperPOD는 여러 컴퓨팅 시스템과 스토리지·네트워크·소프트웨어·관리 체계를 통합하는 확장 인프라 플랫폼으로 설명된다.[^v-superpod] 여러 시스템을 묶는 경우에는 해당 규모에 맞는 네트워크·스토리지·시설 설계가 필요하다. DGX를 소유하는 방식과 [[GPUaaS]]로 자원을 사용하는 방식은 구매·운영 모델의 차이이다.

## 6. 관련 문서

- [[HGX]]
- [[GPU Farm]]
- [[GPUaaS]]
- [[AI Factory]]
- [[NVIDIA Reference Architecture]]

- [[Server]]

## 7. 출처

[^1]: [NVIDIA — DGX Platform](https://www.nvidia.com/en-us/data-center/dgx-platform/)
[^2]: [NVIDIA — HGX Platform](https://www.nvidia.com/en-us/data-center/hgx/)

[^v-model]: [NVIDIA — DGX H100/H200 Systems introduction](https://docs.nvidia.com/dgx/dgxh100-user-guide/introduction-to-dgxh100.html)

[^v-superpod]: [NVIDIA — DGX SuperPOD](https://www.nvidia.com/en-us/data-center/dgx-superpod/)

[^d-basepod]: [NVIDIA — DGX BasePOD introduction](https://docs.nvidia.com/dgx-basepod/deployment-guide-dgx-basepod/latest/introduction.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
