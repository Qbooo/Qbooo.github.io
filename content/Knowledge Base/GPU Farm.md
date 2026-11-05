---
title: GPU Farm
type: knowledge
domain: Compute
tags:
  - GPU
  - Infrastructure
aliases:
  - "GPU 팜"
  - "GPU 클러스터"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-05
---

## 1. 개요

GPU Farm(GPU 팜)은 여러 GPU 서버를 모아 AI·과학 계산·렌더링 같은 연산을 처리하는 인프라를 가리키는 표현이다.

GPU 클러스터와 비슷하게 쓰이지만, GPU 팜 자체가 특정 제조사의 공식 제품명이나 고정된 토폴로지를 뜻하지는 않는다.

## 2. GPU 팜의 주요 구성 요소

각 서버의 GPU와 CPU·메모리, 서버 사이 네트워크, 데이터를 공급하는 스토리지, 작업을 배치하는 소프트웨어가 함께 필요하다. [[NVIDIA Reference Architecture]]는 이러한 구성 요소를 통합해서 보는 설계 기준을 제공한다.[^1]

독립적인 추론 작업을 여러 서버에 나눠 실행하는 방식과, 하나의 학습 작업을 여러 서버가 함께 수행하는 방식은 통신 요구가 다르다. 예를 들어 NVIDIA NCCL은 AllReduce·AllGather 같은 집합 통신을 GPU 사이와 노드 사이에서 수행하는 라이브러리이다. 이처럼 통신과 동기화가 필요한 작업은 연결 구조와 대역폭의 영향을 받는다.[^v-nccl]

## 3. GPU 여러 개로 작업을 나누는 방식

서로 독립된 작업은 여러 GPU에 하나씩 배치해 동시에 실행할 수 있다. 예를 들어 서로 다른 이미지를 렌더링하거나 여러 사용자의 독립된 추론 요청을 처리하는 경우이다. 이때 전체 처리량을 높이는 것과 개별 작업 한 건을 빠르게 만드는 것은 다른 목표이다.

하나의 큰 작업을 여러 GPU가 함께 수행하려면 프로그램도 일을 나누어야 한다. 데이터 병렬 학습은 작업자들이 서로 다른 입력을 처리한 결과를 교환하고, 모델 병렬은 모델의 일부를 여러 장치에 나누어 중간 데이터를 전달한다. NCCL의 집합 통신은 이런 GPU 간 데이터 교환을 지원한다.[^v-nccl]

따라서 GPU 수가 두 배라고 모든 작업이 두 배 빨라지는 것은 아니다. 데이터를 읽는 시간, CPU 처리, 장치 사이 통신과 동기화도 필요하다. 여러 GPU의 메모리 합계 역시 자동으로 하나의 연속 메모리가 되지 않는다. GPU 팜은 장치의 집합이며, 그것을 하나의 작업에 활용하는 방법은 소프트웨어와 연결 구조가 결정한다.[^v-cuda]

## 4. 서버·네트워크·스케줄러의 구성

GPU 팜의 기본 실행 단위는 GPU가 들어 있는 노드다. 노드 안에는 GPU와 장치 메모리(HBM 등), CPU·시스템 메모리, 로컬 NVMe·NIC와 PCIe 연결이 있고, 여러 노드는 스위치와 외부 데이터 저장소로 연결된다. 스케줄러는 작업에 필요한 GPU 수·메모리·위치·정책을 보고 실행 자원을 배정한다. 노드·랙·클러스터는 확장 단위와 장애의 영향 범위가 다르다.[^d-cuda]

작업의 데이터와 통신 대상이 가까운 위치에 있으면 데이터 지역성(locality)의 이점이 있고, 다른 랙·노드로 나뉘면 해당 연결의 비용이 추가된다. 동기화하는 작업에서는 가장 늦은 작업자(straggler)를 다른 작업자가 기다릴 수 있으므로 평균 GPU 이용률보다 작업자별 진행 시간·통신 대기를 비교하는 것이 중요하다.[^v-nccl]

체크포인트 저장은 다시 데이터 저장 경로를 사용하므로 연산망과 저장 경로를 함께 이해해야 한다.[^v-nccl]

PCIe는 호스트와 장치 등을 연결하는 내부 버스이고 NVLink는 GPU 간 고속 연결 계열이며, 노드 간에는 Ethernet·InfiniBand와 RDMA 같은 전송 구성이 쓰일 수 있다. RDMA는 특정 스위치 상표가 아니라 지원되는 장치·네트워크·소프트웨어로 구현하는 메모리 접근 방식이다. 어떤 조합이 가능한지는 실제 플랫폼과 통신 라이브러리 지원 조건을 따른다.[^v-nccl]

## 5. 용량·성능 지표

| 지표 | 의미 |
|---|---|
| GPU 수와 메모리 | 수용할 모델·작업 규모 |
| 연산 이용률 | 측정 시점의 GPU 활동 수준 |
| 스토리지 처리량 | 데이터 공급·체크포인트 저장 능력 |
| 통신 대역폭·지연 | 분산 작업의 교환 비용 |
| 작업 완료 시간·실패율 | 사용자가 경험하는 처리 품질 |

이용률이 높다는 사실만으로 작업이 효율적이라고 단정할 수 없다. GPU 상태와 진단은 DCGM 같은 도구로 관측할 수 있고, 애플리케이션 지표와 함께 해석한다. DCGM 진단 통과만으로 서비스의 정답·응답 시간·데이터 처리가 모두 정상이라고 판정할 수는 없다.[^2]

## 6. 서비스와 운영 모델

GPU 팜을 여러 조직에 제공하면 [[Multi-tenancy]]와 자원 할당량 관리가 필요하다. 외부 사용자에게 GPU 자원을 서비스로 제공하는 모델이 [[GPUaaS]]이며, 데이터부터 AI 서비스까지 운영 흐름을 포괄하는 표현이 [[AI Factory]]이다. [[DGX]]나 [[HGX]] 기반 시스템은 GPU 팜을 구성하는 선택지가 될 수 있다.

## 7. 관련 문서

- [[GPUaaS]]
- [[AI Factory]]
- [[Multi-tenancy]]
- [[DGX]]
- [[HGX]]
- [[NVIDIA Reference Architecture]]

- [[Server]]

- [[Networking]]

- [[Storage]]

## 8. 출처

[^1]: [NVIDIA — Reference Architectures](https://docs.nvidia.com/reference-architectures/index.html)
[^2]: [NVIDIA — DCGM Feature Overview](https://docs.nvidia.com/datacenter/dcgm/latest/user-guide/feature-overview.html)

[^v-nccl]: [NVIDIA — NCCL Overview](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/overview.html)

[^v-cuda]: [NVIDIA CUDA — Programming Systems with Multiple GPUs](https://docs.nvidia.com/cuda/cuda-programming-guide/03-advanced/multi-gpu-systems.html)

[^d-cuda]: [NVIDIA CUDA — Multi-GPU systems](https://docs.nvidia.com/cuda/cuda-programming-guide/03-advanced/multi-gpu-systems.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
