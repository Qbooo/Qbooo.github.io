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
updated: 2026-11-04
---

## 1. 개요

GPU Farm(GPU 팜)은 여러 GPU 서버를 모아 AI·과학 계산·렌더링 같은 연산을 처리하는 인프라를 가리키는 표현이다.

GPU 클러스터와 비슷하게 쓰이지만, GPU 팜 자체가 특정 제조사의 공식 제품명이나 고정된 토폴로지를 뜻하지는 않는다.

## 2. 서버를 모으는 것 이상의 구성

각 서버의 GPU와 CPU·메모리, 서버 사이 네트워크, 데이터를 공급하는 스토리지, 작업을 배치하는 소프트웨어가 함께 필요하다. [[NVIDIA Reference Architecture]]는 이러한 구성 요소를 통합해서 보는 설계 기준을 제공한다.[^1]

독립적인 추론 작업을 여러 서버에 나눠 실행하는 방식과, 하나의 학습 작업을 여러 서버가 함께 수행하는 방식은 통신 요구가 다르다. 독립 작업의 병렬 배치와 GPU 사이에 데이터를 반복 교환하는 작업은 요구가 다르다. 예를 들어 NVIDIA NCCL은 AllReduce·AllGather 같은 집합 통신을 GPU 사이와 노드 사이에서 수행하는 라이브러리이다. 이처럼 통신과 동기화가 필요한 작업은 연결 구조와 대역폭의 영향을 받는다.[^v-nccl]

## 3. 용량과 성능의 관점

| 지표 | 의미 |
|---|---|
| GPU 수와 메모리 | 수용할 모델·작업 규모 |
| 연산 이용률 | 측정 시점의 GPU 활동 수준 |
| 스토리지 처리량 | 데이터 공급·체크포인트 저장 능력 |
| 통신 대역폭·지연 | 분산 작업의 교환 비용 |
| 작업 완료 시간·실패율 | 사용자가 경험하는 처리 품질 |

이용률이 높다는 사실만으로 작업이 효율적이라고 단정할 수 없다. GPU 상태와 진단은 DCGM 같은 도구로 관측할 수 있고, 애플리케이션 지표와 함께 해석한다. DCGM 진단 통과만으로 서비스의 정답·응답 시간·데이터 처리가 모두 정상이라고 판정할 수는 없다.[^2]

여러 GPU의 메모리 합계가 한 모델에서 자동으로 사용할 수 있는 단일 연속 메모리가 되는 것은 아니다. 다중 GPU를 활용하려면 애플리케이션·프레임워크가 작업과 데이터를 나누고 필요한 통신을 수행해야 한다.[^v-cuda]

## 4. 서비스와 운영 모델

GPU 팜을 여러 조직에 제공하면 [[Multi-tenancy]]와 자원 할당량 관리가 필요하다. 외부 사용자에게 GPU 자원을 서비스로 제공하는 모델이 [[GPUaaS]]이며, 데이터부터 AI 서비스까지 운영 흐름을 포괄하는 표현이 [[AI Factory]]이다. [[DGX]]나 [[HGX]] 기반 시스템은 GPU 팜을 구성하는 선택지가 될 수 있다.

## 5. 서버·네트워크·스케줄러의 실물 구조

GPU 팜의 기본 실행 단위는 GPU가 들어 있는 노드다. 노드 안에는 GPU 연산·HBM, CPU·시스템 메모리, 로컬 NVMe·NIC와 PCIe 연결이 있고, 여러 노드는 스위치와 외부 데이터 저장소로 연결된다. 스케줄러는 작업 요구의 GPU 수·메모리·위치·정책을 보고 실행 자원을 배정한다. Node와 rack·cluster는 서로 다른 확장·장애 단위다.[^d-cuda]

작업의 데이터와 통신 대상이 가까운 위치에 있으면 locality의 이점이 있고, 다른 랙·노드로 나뉘면 해당 연결의 비용이 추가된다. 동기화하는 작업에서는 가장 늦은 작업자(straggler)를 다른 작업자가 기다릴 수 있으므로 평균 GPU 이용률보다 작업자별 진행 시간·통신 대기를 비교하는 것이 중요하다.[^v-nccl]

독립적인 단일 GPU 작업은 노드별로 나누어 실행할 수 있지만 데이터 병렬 학습은 각 작업자가 처리한 결과를 collective 통신으로 교환할 수 있고, 모델 병렬은 모델의 일부를 여러 GPU에 나누면서 중간 데이터를 전달할 수 있다. NCCL 같은 라이브러리가 이런 통신을 제공한다. 체크포인트 저장은 다시 데이터 저장 경로를 사용하므로 연산망과 저장 경로를 함께 이해해야 한다.[^v-nccl]

PCIe는 호스트와 장치 등을 연결하는 내부 버스이고 NVLink는 GPU 간 고속 연결 계열이며, 노드 간에는 Ethernet·InfiniBand와 RDMA 같은 전송 구성이 쓰일 수 있다. RDMA는 특정 스위치 상표가 아니라 지원되는 장치·네트워크·소프트웨어로 구현하는 메모리 접근 방식이다. 어떤 조합이 가능한지는 실제 플랫폼과 통신 라이브러리 지원 조건을 따른다.[^v-nccl]

## 6. 작업 배치와 병목을 함께 보기

스케줄러가 많은 작은 작업을 한쪽에 모으면 다른 자원은 남아도 특정 GPU 메모리가 부족할 수 있다. 반대로 작업을 너무 넓게 퍼뜨리면 노드 간 통신이 늘어날 수 있다. GPU 수량·메모리·작업 위치·우선순위·할당량의 선택은 집계 이용률과 작업 완료 시간·서비스 품질에 서로 다른 영향을 준다.

Bin packing은 제한된 노드·GPU에 작업 요구를 조합해 넣는 배치 관점이다. 작은 작업을 모아 남는 장치를 줄이는 선택과, 통신하는 작업을 가까이 두는 선택이 서로 충돌할 수 있다. 예를 들어 빈 메모리가 조금씩 여러 GPU에 흩어져 있는 상태는 큰 단일 GPU 할당 요구를 바로 충족하는 상태가 아니다.

한 GPU 안에서도 프레임워크의 캐싱 allocator가 예약한 메모리와 실제 텐서가 사용하는 메모리, 큰 요청에 재사용할 수 있는 블록은 구분된다. PyTorch는 allocated·reserved 지표와 메모리 관리 설정을 설명한다. 단편화는 일부 OOM의 가능한 원인이지 모든 OOM의 결론은 아니며, 필요한 메모리 자체가 장치 용량보다 큰 경우와 나누어 보아야 한다.[^d-allocator]

OOM은 모델 가중치뿐 아니라 배치·활성값·작업 공간·KV cache·다른 프로세스의 메모리를 포함해 해석한다. GPU idle은 입력 데이터·CPU 처리·작업 대기·동기화를, 통신 hang은 일부 작업자의 실패·연결·collective 순서·네트워크 상태를, 체크포인트 지연은 동시 쓰기와 저장 경로를 구분하는 가설이 된다. 메모리 합계를 단일 모델의 즉시 사용 가능 공간으로 보는 오류도 다중 GPU 실행 구조를 이해하면 피할 수 있다.[^v-cuda]

전력·온도 한도에 따른 클록 변화도 성능을 바꿀 수 있다. GPU 메모리·이용률·전력·온도·링크 상태와 호스트·네트워크·스토리지 지표를 같은 작업의 시간축에서 비교한다. DCGM·nvidia-smi 같은 도구는 장치 상태의 근거를 주지만 최종 작업 정답과 응답 품질은 애플리케이션 지표로 확인해야 한다.[^d-smi]





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

[^d-smi]: [NVIDIA — System Management Interface](https://docs.nvidia.com/deploy/nvidia-smi/index.html)

[^d-allocator]: [PyTorch — CUDA memory management](https://docs.pytorch.org/docs/2.15/notes/cuda.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
