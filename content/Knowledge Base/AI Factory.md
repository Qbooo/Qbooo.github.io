---
title: AI Factory
type: knowledge
domain: Compute
tags:
  - AI
  - Infrastructure
aliases:
  - "AI 팩토리"
  - "AI 공장"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-04
---

## 1. 개요

AI Factory(AI 팩토리)는 데이터를 받아 학습·추론을 수행하고 AI 결과를 지속적으로 만들어 내는 인프라와 운영 체계를 공장에 빗댄 표현이다.

NVIDIA는 가속 컴퓨팅과 AI 소프트웨어를 통합해 대규모 AI 처리를 수행하는 운영 모델로 설명한다. 이 문서는 그 설명을 참고한 개념 정리이며 업계 전체에 적용되는 단일 표준 정의를 뜻하지 않는다.[^1]

## 2. 데이터에서 서비스까지

개념적으로 데이터 수집·정제 → 학습 또는 모델 준비 → 평가 → 추론 서비스 → 모니터링과 개선의 흐름으로 볼 수 있다. 모든 AI 서비스가 처음부터 대형 모델을 직접 학습하는 것은 아니며, 이미 학습된 모델을 조정하거나 추론 중심으로 운영할 수도 있다. 아래 흐름은 가능한 단계의 연결을 보여 주는 설명이며, 매 요청마다 수집부터 학습까지 순서대로 수행해야 한다는 뜻은 아니다. NVIDIA의 참조 설계도 학습뿐 아니라 RAG·미세조정·추론 등 여러 워크로드를 구분한다.[^v-workload]

| 구성 | 목적 |
|---|---|
| 데이터 계층 | 데이터셋·체크포인트·결과 보관 |
| 가속 컴퓨팅 | 학습·추론 연산 |
| 네트워크 | GPU·서버 간 통신과 데이터 이동 |
| 소프트웨어 | 작업 배포, 실행, 자원 배분 |
| 운영 체계 | 관측, 접근 통제, 장애와 변경 관리 |

이 표는 일반적인 역할 구분이다.[^2]

## 3. GPU 팜과의 차이

[[GPU Farm|GPU 팜]]은 GPU 서버 집합이라는 인프라 관점의 표현이다. 이 문서에서는 AI 팩토리라는 표현으로 데이터 공급, 모델 실행과 서비스 운영 흐름을 함께 설명한다. GPU 팜과의 이런 구별은 설명 관점이며 두 용어 사이의 공인된 등급·인증 기준은 아니다. GPU 수량이 많아도 데이터 준비나 네트워크가 병목이면 효율적인 AI 서비스가 되지 않는다.

[[DGX]]·[[HGX]]는 이를 구성할 수 있는 시스템 플랫폼이고, [[GPUaaS]]는 자원을 사용자에게 제공하는 서비스 방식이다.

## 4. 설계와 운영의 기준

[[NVIDIA Reference Architecture|참조 아키텍처]]는 구성 요소를 함께 설계하는 출발점이 된다. 실제 설계는 학습과 추론의 비중, 데이터 크기, 동시 요청과 응답 지연, 전력·냉각·운영 역량에 따라 달라진다. [[Harbor]]는 실행 이미지 관리, [[Data Lake]]는 원본 데이터 관리에 연결할 수 있는 선택 예시이다. 이 두 제품·구성이 모든 NVIDIA 설계의 필수 요소라는 뜻은 아니다.

## 5. 데이터·연산·서비스가 이어지는 시스템

물리 기반에는 GPU 서버·CPU·메모리·스토리지·고속 네트워크·랙 전력·냉각이 있고, 플랫폼에는 스케줄러·실행 이미지·모델 저장·관측·접근 통제가 있다. 데이터 수집과 정제, 학습·평가, 모델 배포와 추론 서비스는 이 기반을 서로 다른 방식으로 사용한다. GPU 팩토리를 단순히 GPU 서버가 많은 방으로만 이해하면 데이터와 운영 경로를 놓치게 된다.[^2]

한 가지 AI 생명주기 예로, 데이터셋을 정제해 학습 작업에 공급하고 작업은 중간 체크포인트를 저장한다. 평가된 모델은 버전·실행 이미지와 함께 추론 환경에 배포되고, 운영 중의 품질·지연·오류는 다음 개선의 입력이 된다. 학습을 직접 하지 않는 조직은 이미 학습된 모델을 준비·조정해 이 흐름의 일부만 운영할 수 있다. 파이프라인은 한 번의 직선 작업보다 반복되는 개선 체계로 보는 편이 적절하다.

체크포인트는 중간 학습 상태의 저장, 모델 아티팩트는 배포할 결과, 컨테이너 이미지는 실행 환경이라는 서로 다른 산출물이다. [[Harbor]]와 모델·데이터 저장소의 역할도 이 구분에 따라 나누어 설계할 수 있다. 모든 데이터를 이미지 안에 넣거나 스토리지 하나가 이 기능을 모두 자동 제공한다고 가정하지 않는다.

## 6. 학습·추론·RAG에 맞춘 운영

학습은 여러 GPU의 연산·동기화와 큰 데이터셋·체크포인트 입출력이 중요할 수 있다. 추론은 요청 도착 패턴·모델 메모리·배치·첫 응답과 연속 출력의 지연을 다룬다. 미세조정은 기존 모델을 특정 데이터에 맞게 바꾸는 과정이고, RAG는 추론 요청에 검색한 자료를 연결하는 구성이다. 같은 “AI”라도 데이터 흐름과 자원 요구가 다르므로 하나의 성능 수치로 비교할 수 없다.[^v-workload][^d-inference]

스케줄러는 어떤 작업을 어느 GPU·노드에서 실행할지 결정하고, 할당량과 우선순위는 여러 팀의 작업을 조정한다. 레지스트리·데이터·모델의 버전은 실행 재현성에, 관측과 로그는 문제의 원인·영향 범위에 연결된다. 전력과 냉각이 부족하면 준비한 연산 장치를 정상 성능으로 쓰지 못하므로 시설 조건도 실제 처리 용량의 일부다.

GPU는 많은데 응답이 느리면 작업 대기·데이터 공급·CPU 전처리·GPU 메모리·모델 배치·collective 통신과 서비스 계층을 구분한다. 학습 완료 시간·유효 처리량·TTFT·토큰 간 지연·오류율을 실제 작업 조건에서 보고 GPU 이용률·저장·네트워크·시설 지표를 연결한다. 장비 목록을 완성하는 것과 요구한 품질의 AI 서비스를 지속 운영하는 것은 다른 수준의 결과다.

## 7. 관련 문서

- [[GPU Farm]]
- [[GPUaaS]]
- [[NVIDIA Reference Architecture]]
- [[DGX]]
- [[HGX]]
- [[Harbor]]
- [[Data Lake]]

- [[Server]]

- [[Storage]]

## 8. 출처

[^1]: [NVIDIA — AI Factories](https://www.nvidia.com/en-us/solutions/ai-factories/)
[^2]: [NVIDIA — Powering AI Factories with Enterprise Reference Architectures](https://developer.nvidia.com/blog/powering-ai-factories-with-nvidia-enterprise-reference-architectures/)

[^v-workload]: [NVIDIA — Design Tenets: supported AI workloads](https://docs.nvidia.com/enterprise-reference-architectures/white-paper/latest/design-tenets.html)

[^d-inference]: [NVIDIA Dynamo — Prefill and decode serving](https://docs.nvidia.com/dynamo/dev/knowledge-base/concepts/system-architecture/disaggregated-serving)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
