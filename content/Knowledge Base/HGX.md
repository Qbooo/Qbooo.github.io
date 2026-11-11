---
title: HGX
type: knowledge
domain: Compute
tags:
  - NVIDIA
  - GPU
aliases:
  - "NVIDIA HGX"
  - "엔비디아 HGX"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-05
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 플랫폼과 완제품 서버의 관계|2. 플랫폼과 완제품 서버의 관계]]
- [[#3. 서버 내부와 서버 사이 연결|3. 서버 내부와 서버 사이 연결]]
- [[#4. GPU 플랫폼과 서버 제조사의 구성 범위|4. GPU 플랫폼과 서버 제조사의 구성 범위]]
- [[#5. AI 인프라에서의 역할|5. AI 인프라에서의 역할]]
- [[#6. 관련 문서|6. 관련 문서]]
- [[#7. 출처|7. 출처]]

## 1. 개요

NVIDIA HGX는 서버 제조사가 고성능 AI·HPC 서버를 구성할 수 있도록 GPU와 고속 연결 기술 등을 통합하는 플랫폼 계열이다.

하나의 완제품 서버 모델명보다는 파트너 서버 설계의 기반으로 이해하는 것이 적절하다.[^1]

## 2. 플랫폼과 완제품 서버의 관계

여러 GPU를 같은 서버에서 효율적으로 사용하려면 GPU의 개수뿐 아니라 장치 사이 연결과 전체 서버 구성이 중요하다. HGX는 서버 제조사가 이런 AI·HPC 서버를 구현할 수 있도록 제공하는 플랫폼 계열이다.[^1]

서버 제조사는 해당 플랫폼을 바탕으로 CPU·시스템 메모리·NIC·스토리지·전원·냉각·관리 기능을 갖춘 제품을 구성한다. 다만 세대에 따라 플랫폼에 포함되는 CPU와 연결 범위가 달라질 수 있으므로, HGX를 언제나 GPU 보드 하나의 이름으로만 이해하면 안 된다.

HGX와 DGX의 차이는 “부품은 낮은 등급, 완제품은 높은 등급”이라는 순위가 아니다. HGX는 파트너 서버의 설계 기반, DGX는 NVIDIA의 통합 시스템·플랫폼 계열이라는 제공 범위의 차이이다. 실제 성능은 구체적인 GPU 세대·메모리·연결·서버 구성을 기준으로 비교한다.[^1][^2]

## 3. 서버 내부와 서버 사이 연결

HGX는 여러 GPU를 고속으로 연결하는 플랫폼 구성을 제공한다. CPU·시스템 메모리·네트워크·전원·냉각까지 포함한 실제 구성은 플랫폼 세대와 서버 제품에 따라 달라진다. CPU가 항상 플랫폼 밖에서 나중에 추가된다고 일반화할 수는 없다. 현재 HGX 소개는 Vera CPU를 포함하는 조합과 x86 CPU baseboard 조합을 구분해 설명한다.[^1]

GPU 사이의 scale-up 연결과 여러 시스템을 확장하는 네트워크 역할을 구분해 보아야 한다. NVLink가 사용되는 범위를 서버 내부로만 한정할 수는 없다. NVIDIA는 NVLink Switch를 통해 노드·랙 규모로 확장하는 구성도 설명한다.[^v-nvlink] 노드 내부 통신이 빠르더라도 서버 사이 네트워크나 데이터 공급이 느리면 전체 작업의 병목이 될 수 있다.[^1][^2]

## 4. GPU 플랫폼과 서버 제조사의 구성 범위

HGX를 이해할 때는 GPU·HBM·NVLink/NVSwitch 같은 가속기 연결 플랫폼과, 이를 결합한 OEM 서버의 CPU·메모리·NIC·전원·냉각·관리 구성을 구별한다. SXM 형태 GPU와 baseboard를 사용하는 세대의 구성도 있지만 모든 세대·제품을 같은 보드 형태로 고정할 수는 없다. 현재 제품 소개처럼 CPU 조합을 포함해 확장되는 범위도 있으므로 실제 HGX 세대와 OEM 서버 사양을 함께 읽는다.[^1]

GPU끼리의 scale-up 통신과 노드 간 scale-out 통신은 다른 연결 범위를 다룬다. 호스트 CPU와 GPU 사이에는 PCIe 등 시스템 연결이 관여하고, 여러 GPU·노드의 큰 모델 작업에는 NVLink·외부 패브릭과 통신 소프트웨어가 관여한다. NVLink Switch가 노드·랙 단위 연결로 확장할 수 있다는 점 때문에 NVLink를 무조건 서버 내부 링크로만 정의하면 안 된다.[^v-nvlink]

HGX는 파트너 서버의 기반 플랫폼, OEM 서버는 실제 장비와 펌웨어·관리·지원의 제품, DGX는 NVIDIA의 통합 시스템 계열, RA는 그 시스템들을 엮는 설계 기준이라는 범주 차이가 있다. 부품 수준의 호환과 서버 제품의 검증, 클러스터 전체 검증·지원 범위는 같은 사실이 아니다.

## 5. AI 인프라에서의 역할

HGX 기반 서버는 [[GPU Farm]]과 [[AI Factory]]의 컴퓨팅 노드가 될 수 있다. HGX라는 이름만으로 노드 간 패브릭과 학습 데이터 스토리지의 전체 설계까지 완성되는 것은 아니다. [[NVIDIA Reference Architecture]]는 서버를 네트워크·스토리지·소프트웨어와 결합하는 설계 기준을 제공한다. 사용자에게 자원을 서비스로 제공하는 [[GPUaaS]]는 이런 하드웨어 위에서 구현할 수 있는 운영 모델이다.

## 6. 관련 문서

- [[DGX]]
- [[GPU Farm]]
- [[GPUaaS]]
- [[AI Factory]]
- [[NVIDIA Reference Architecture]]

- [[Server]]

## 7. 출처

[^1]: [NVIDIA — HGX Platform](https://www.nvidia.com/en-us/data-center/hgx/)
[^2]: [NVIDIA — Reference Architectures](https://docs.nvidia.com/reference-architectures/index.html)

[^v-nvlink]: [NVIDIA — NVLink and NVLink Switch](https://www.nvidia.com/en-us/data-center/nvlink/)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
