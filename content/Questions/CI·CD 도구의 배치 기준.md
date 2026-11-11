---
title: CI/CD 도구의 배치 기준
type: question
domain: Cloud
tags:
  - 질문
  - CI/CD
aliases:
  - CI·CD 도구는 어디에 두어야 하는가
parent:
status: growing
lang: ko
created: 2026-11-11
updated: 2026-11-11
resolved: true
resolved_at: 2026-11-11
---
## 목차

- [[#질문|질문]]
- [[#해답|해답]]
- [[#프로젝트 판단|프로젝트 판단]]
- [[#관련 문서|관련 문서]]
- [[#출처|출처]]

## 질문

CI/CD 도구는 어디에 두어야 하는가?

## 해답

CI/CD 도구도 하나의 시스템이므로 두는 위치와 장애 대비를 따로 정해야 한다. 선택지는 크게 두 가지이다.

- 직접 운영하는 도구를 VM에 두는 방식: 통제권이 크지만 그 도구의 이중화, 백업, DR을 다시 설계해야 한다.
- 외부 서비스를 쓰는 방식: 운영 부담이 줄지만 외부 서비스에 의존하고 접속 자격증명을 관리해야 한다.

CI/CD 파이프라인은 소스 변경을 기준으로 빌드·시험·배포 단계를 자동화한다.[^1] 같은 코드를 환경마다 다시 빌드하기보다 한 번 만든 산출물을 신뢰할 수 있는 저장소에서 각 환경으로 배포하면 환경 차이로 인한 불일치를 줄일 수 있다.[^2] 이 원칙은 도구 위치와 무관하게 적용된다.

## 프로젝트 판단

아래 판단은 소규모 연습 프로젝트(가상의 소규모 고객, 직원 30명 안팎의 유통업체를 가정한 DR 이전 과제)에서 요구사항과 규모를 고려해 내린 것이며, 모든 환경에 맞는 정답은 아니다.

- 맥락: 설계를 하다 보니 "그 도구는 어디에 둘 것인가"가 계속 꼬리를 물었고, 소규모 6주 프로젝트에서는 어디선가 끊어야 했다.
- 판단: 외부 서비스를 사용하기로 했다. 내부에 두면 그 도구의 HA와 DR이 또 필요해지고, 외부면 주 센터가 멈춰도 DR 쪽으로 배포할 수 있다.
- 범위 밖: 외부 서비스 장애 시 대응, 자격증명 보관 방식은 문서에서 범위 밖으로 표기한다. 도구 선정은 5-6 요구사항(3주차 이후)에서 다룬다.

## 관련 문서

- [[Server]]

## 출처

[^1]: CI/CD 파이프라인은 소스 변경을 계기로 애플리케이션 빌드·자동 시험·환경 배포 단계를 실행한다. [AWS Prescriptive Guidance, Continuous integration and continuous delivery](https://docs.aws.amazon.com/prescriptive-guidance/latest/aws-caf-platform-perspective/ci-cd.html)
[^2]: 여러 번 빌드하는 방식을 피하고, 한 번 만든 산출물을 신뢰할 수 있는 저장소에서 배포하는 원칙을 설명한다. [AWS DevOps Guidance, Anti-patterns for continuous delivery](https://docs.aws.amazon.com/wellarchitected/latest/devops-guidance/anti-patterns-for-continuous-delivery.html)

## 둘러보기

[[Questions/index|질문과 해답]] · [[Knowledge Base/index|Knowledge Base]] · [[Projects/index|Projects]]
