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
updated: 2026-11-04
---

## 1. 개요

NVIDIA HGX는 서버 제조사가 고성능 AI·HPC 서버를 구성할 수 있도록 GPU와 고속 연결 기술 등을 통합하는 플랫폼 계열이다.

하나의 완제품 서버 모델명보다는 파트너 서버 설계의 기반으로 이해하는 것이 적절하다.[^1]

## 2. 서버 내부와 서버 사이 연결

HGX는 여러 GPU를 고속으로 연결하는 플랫폼 구성을 제공한다. CPU·시스템 메모리·네트워크·전원·냉각까지 포함한 실제 구성은 플랫폼 세대와 서버 제품에 따라 달라진다. CPU가 항상 플랫폼 밖에서 나중에 추가된다고 일반화할 수는 없다. 현재 HGX 소개는 Vera CPU를 포함하는 조합과 x86 CPU baseboard 조합을 구분해 설명한다.[^1]

GPU 사이의 scale-up 연결과 여러 시스템을 확장하는 네트워크 역할을 구분해 보아야 한다. NVLink가 사용되는 범위를 서버 내부로만 한정할 수는 없다. NVIDIA는 NVLink Switch를 통해 노드·랙 규모로 확장하는 구성도 설명한다.[^v-nvlink] 노드 내부 통신이 빠르더라도 서버 사이 네트워크나 데이터 공급이 느리면 전체 작업의 병목이 될 수 있다.[^1][^2]

## 3. DGX와의 구분

[[DGX]]는 NVIDIA의 통합 AI 시스템·플랫폼이고, HGX는 파트너 서버 구현의 기반이 되는 플랫폼이다. HGX 기반 서버의 실제 GPU·CPU·메모리 사양과 보증·지원 범위는 해당 서버 제조사의 제품을 기준으로 확인한다.

HGX가 항상 DGX보다 낮은 등급이거나 동일한 사양이라는 해석은 맞지 않는다. 비교에는 구체적인 세대·모델·지원 범위가 필요하다.

## 4. AI 인프라에서의 위치

HGX 기반 서버는 [[GPU Farm]]과 [[AI Factory]]의 컴퓨팅 노드가 될 수 있다. HGX라는 이름만으로 노드 간 패브릭과 학습 데이터 스토리지의 전체 설계까지 완성되는 것은 아니다. [[NVIDIA Reference Architecture]]는 서버를 네트워크·스토리지·소프트웨어와 결합하는 설계 기준을 제공한다. 사용자에게 자원을 서비스로 제공하는 [[GPUaaS]]는 이런 하드웨어 위에서 구현할 수 있는 운영 모델이다.

## 5. GPU 플랫폼과 서버 제조사의 구성 경계

HGX를 이해할 때는 GPU·HBM·NVLink/NVSwitch 같은 가속기 연결 플랫폼과, 이를 결합한 OEM 서버의 CPU·메모리·NIC·전원·냉각·관리 구성을 구별한다. SXM 형태 GPU와 baseboard를 사용하는 세대의 구성도 있지만 모든 세대·제품을 같은 보드 형태로 고정할 수는 없다. 현재 제품 소개처럼 CPU 조합을 포함해 확장되는 범위도 있으므로 실제 HGX 세대와 OEM 서버 사양을 함께 읽는다.[^1]

GPU끼리의 scale-up 통신과 노드 간 scale-out 통신은 다른 연결 범위를 다룬다. 호스트 CPU와 GPU 사이에는 PCIe 등 시스템 연결이 관여하고, 여러 GPU·노드의 큰 모델 작업에는 NVLink·외부 패브릭과 통신 소프트웨어가 관여한다. NVLink Switch가 노드·랙 단위 연결로 확장할 수 있다는 점 때문에 NVLink를 무조건 서버 내부 링크로만 정의하면 안 된다.[^v-nvlink]

HGX는 파트너 서버의 기반 플랫폼, OEM 서버는 실제 장비와 펌웨어·관리·지원의 제품, DGX는 NVIDIA의 통합 시스템 계열, RA는 그 시스템들을 엮는 설계 기준이라는 범주 차이가 있다. 부품 수준의 호환과 서버 제품의 검증, 클러스터 전체 검증·지원 범위는 같은 사실이 아니다.

## 6. 같은 GPU 수량을 넘어선 판단

8 GPU라는 표기만 같아도 GPU 세대·HBM 용량·메모리 대역폭·지원 정밀도·GPU 간 연결·전력·냉각이 다르면 모델 수용과 처리 성능이 달라진다. 단일 장치에 모델이 들어가는지와 여러 장치로 나눠 실행할 수 있는지, 노드 간 통신을 얼마나 요구하는지를 먼저 구분해야 한다. 노드 내부 성능을 활용하더라도 데이터 공급이나 외부 통신이 느리면 전체 작업이 대기할 수 있다.

파트너 서버에서 GPU가 인식되지 않으면 실제 장치·전원·PCIe·펌웨어·드라이버·VM 할당의 계층을 구별하고, 특정 GPU 링크가 저하되면 내부 연결·지원 토폴로지·링크 상태를 본다. 노드 내부 작업은 정상인데 분산 작업만 느리면 외부 NIC·스위치·전송 설정과 통신 라이브러리를 비교할 수 있다. 온도·전력 제한에 따른 클록 변화는 장치 지표와 함께 해석한다.[^d-smi]

이런 운영 해석은 플랫폼 이름을 원인으로 쓰기보다 문제가 발생한 계층과 증거를 특정하게 한다. 서버 제조사의 지원 범위와 NVIDIA 소프트웨어·플랫폼 지원을 연결할 때도 장비 모델·드라이버·펌웨어·재현 조건이 중요한 이유다.

## 7. 관련 문서

- [[DGX]]
- [[GPU Farm]]
- [[GPUaaS]]
- [[AI Factory]]
- [[NVIDIA Reference Architecture]]

- [[Server]]

## 8. 출처

[^1]: [NVIDIA — HGX Platform](https://www.nvidia.com/en-us/data-center/hgx/)
[^2]: [NVIDIA — Reference Architectures](https://docs.nvidia.com/reference-architectures/index.html)

[^v-nvlink]: [NVIDIA — NVLink and NVLink Switch](https://www.nvidia.com/en-us/data-center/nvlink/)

[^d-smi]: [NVIDIA — System Management Interface](https://docs.nvidia.com/deploy/nvidia-smi/index.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
