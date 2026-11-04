---
title: GPUaaS
type: knowledge
domain: Cloud
tags:
  - GPU
  - AI
aliases:
  - "GPU as a Service"
  - "GPU 서비스"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-04
---

## 1. 개요

GPUaaS(GPU as a Service)는 사용자가 GPU 장비를 직접 소유·운영하는 대신 서비스 형태로 GPU 연산 자원을 이용하는 제공 모델이다.

AI 학습·추론과 병렬 연산에 필요한 자원을 VM이나 컨테이너 등으로 제공할 수 있다. Zadara는 GPU 자원과 클라우드 컴퓨팅·스토리지를 연결하는 플랫폼을 제공한다.[^1]

## 2. 서비스와 하드웨어의 구분

[[GPU Farm|GPU 팜]]은 여러 GPU 서버를 모은 인프라를 가리키고, GPUaaS는 그 자원을 사용자에게 제공하는 방식이다. [[DGX]]와 [[HGX]]는 하드웨어·시스템 플랫폼의 구분이므로 GPUaaS와 같은 종류의 이름이 아니다.

사용자는 GPU 종류와 수량 외에 메모리 용량, CPU·RAM, 데이터 입출력, 노드 간 통신, 소프트웨어 환경을 함께 고려해야 한다. 분산 학습에서는 GPU 사이 통신 때문에 서버를 늘린 만큼 성능이 비례해서 늘지 않을 수 있다. 분산 작업의 요구에 맞춘 네트워크 설계가 필요한 이유이며, GPU 개수만으로 작업 완료 시간을 예측할 수는 없다.[^v-network]

## 3. 할당과 격리

GPU 전체를 VM에 전달하는 패스스루, 지원 장치의 GPU 자원을 분할하는 방식 등 할당 모델에 따라 격리와 성능 특성이 달라진다. NVIDIA MIG는 지원 GPU에서 연산·메모리 자원을 나누는 기능이며, 모든 GPU나 모든 서비스에서 사용할 수 있는 것은 아니다.[^2] MIG 자체가 VM을 만드는 하이퍼바이저는 아니다.

시간을 나누어 같은 GPU를 공유하는 time-slicing과 MIG도 구분한다. NVIDIA GPU Operator 문서는 time-slicing 복제본 사이에 MIG 같은 메모리·장애 격리가 없다고 설명한다. 서비스의 GPU 공유 방식과 격리 범위를 따로 확인해야 한다.[^v-time]

## 4. 운영 책임과 품질

서비스 제공자는 제공 범위에 따라 물리 장비와 인프라를 관리하고, 사용자는 모델·데이터·작업 설정 등을 관리한다. 정확한 경계는 서비스 계약에 따른다. [[Multi-tenancy|멀티테넌시]]에서는 접근 격리뿐 아니라 자원 할당량과 성능 간섭도 관리 대상이다.

GPU 이용률 하나만으로 작업 효율을 판단하지 않는다. 처리량, 응답 지연, 메모리 사용량, 실패율과 작업 완료 시간을 함께 본다. NVIDIA DCGM은 GPU 상태·지표·진단 정보를 제공하는 운영 도구이다. GPU 하드웨어 지표만으로 사용자 요청의 전체 응답 시간이나 애플리케이션 정상 동작까지 입증하는 것은 아니다.[^3]

## 5. 물리 GPU에서 사용자 작업까지의 계층

GPU 서버에는 GPU 연산 장치와 HBM 같은 장치 메모리, CPU·시스템 RAM, 디스크·NIC가 함께 있다. 드라이버는 운영체제에서 GPU를 제어하고 CUDA 같은 실행 환경은 애플리케이션이 연산을 요청하게 한다. VM·컨테이너·스케줄러는 그 자원을 어떤 사용자와 작업에 제공할지 결정하는 또 다른 계층이다. GPU를 할당받았다는 사실만으로 실행 이미지의 라이브러리·드라이버 호환성이 자동으로 맞는 것은 아니다.[^d-background]

학습의 개념 흐름은 데이터 읽기 → CPU의 디코딩·전처리 → GPU 메모리 전송 → GPU 연산 → 필요시 GPU 간 통신 → 체크포인트 저장이다. GPU가 쉬는 동안 CPU가 데이터를 준비하거나 저장소 응답을 기다릴 수 있으므로 낮은 GPU 이용률의 원인을 GPU 자체로만 찾으면 안 된다. 여러 GPU가 공동 학습하면 collective 통신·동기화가 추가되고 노드 간 네트워크도 처리 경로가 된다.[^v-network]

대형 언어 모델 추론에서는 입력 문맥을 처리하는 prefill과 토큰을 차례로 생성하는 decode의 특성이 다르다. 모델·배치·입력 길이에 따라 연산과 메모리 접근의 비중이 달라지므로 단순 GPU 사용률보다 실제 요청의 지연과 처리량으로 서비스 품질을 판단한다.[^d-inference]

## 6. 자원 분할과 사용자 경험의 연결

전용 패스스루는 VM에 GPU 장치를 전달하는 방식이고 MIG는 지원 GPU의 연산·메모리를 분할하는 방식이며 time-slicing은 실행 시간을 공유하는 방식이다. VM과 컨테이너도 격리의 계층이 다르다. 하나의 “GPU 한 개” 표기만 보고 전용 메모리·장애 격리·최소 성능이 같다고 비교하면 안 된다.[^2][^v-time]

운영 지표에서 HBM 사용량은 모델 가중치뿐 아니라 활성값·작업 공간·KV cache 같은 실행 데이터의 영향을 받는다. GPU 이용률은 활동 여부, tokens/s는 출력 처리량, TTFT는 첫 토큰까지 기다린 시간, 토큰 간 지연은 생성 과정의 사용자 경험을 설명한다. 한 지표가 좋아져도 다른 지표가 나빠질 수 있으며 배치를 키워 처리량을 높이는 선택이 모든 요청의 지연 목표에 맞지는 않는다.[^d-metrics]

GPU가 보이지 않는 문제는 스케줄러 할당·장치 노출·드라이버, CUDA OOM은 해당 장치에서 필요한 메모리와 가용량·단편화·동시 작업, 낮은 이용률은 CPU·입력 데이터·작업 크기, 분산 작업 정지는 통신·동기화·일부 작업자의 실패라는 서로 다른 가설을 만든다. GPU 지표와 호스트 CPU·네트워크·저장소·작업 로그를 같은 시간축으로 연결하면 “GPU가 부족하다”는 결론보다 구체적인 자원 판단을 할 수 있다. 서비스의 quota와 격리 정책은 한 고객의 작업이 다른 고객에게 주는 영향까지 함께 고려한다.

## 7. 관련 문서

- [[GPU Farm]]
- [[Multi-tenancy]]
- [[Zadara Sovereign AI Edge Cloud]]
- [[DGX]]
- [[HGX]]

- [[Server]]

- [[Networking]]

- [[Storage]]

## 8. 출처

[^1]: [Zadara — Sovereign AI](https://www.zadara.com/sovereign-ai/)
[^2]: [NVIDIA — MIG Introduction](https://docs.nvidia.com/datacenter/tesla/mig-user-guide/introduction.html)
[^3]: [NVIDIA — DCGM Feature Overview](https://docs.nvidia.com/datacenter/dcgm/latest/user-guide/feature-overview.html)

[^v-time]: [NVIDIA GPU Operator — Time-Slicing GPUs](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-sharing.html)

[^v-network]: [NVIDIA — Design Tenets: networking](https://docs.nvidia.com/enterprise-reference-architectures/white-paper/latest/design-tenets.html)

[^d-background]: [NVIDIA — GPU Performance Background](https://docs.nvidia.com/deeplearning/performance/dl-performance-gpu-background/index.html)

[^d-inference]: [NVIDIA Dynamo — Disaggregated Serving](https://docs.nvidia.com/dynamo/dev/knowledge-base/concepts/system-architecture/disaggregated-serving)

[^d-metrics]: [NVIDIA AIPerf — Metrics Reference](https://docs.nvidia.com/aiperf/reference/ai-perf-metrics-reference)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
