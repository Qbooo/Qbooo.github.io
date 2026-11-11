---
title: CI/CD 변경 사항의 DR 반영
type: question
domain: Cloud
tags:
  - 질문
  - CI/CD
aliases:
  - WEB이나 WAS가 CI·CD로 바뀌면 DR에는 어떻게 반영하는가
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

WEB이나 WAS가 CI/CD로 바뀌면 DR에는 어떻게 반영하는가?

## 해답

CI/CD 파이프라인이 같은 배포 산출물을 주 센터와 DR 센터 양쪽에 배포하도록 구성한다. 한 번 만든 산출물을 신뢰할 수 있는 저장소에 보관하고 각 환경으로 배포하는 방식은 환경별 재빌드에서 생길 수 있는 차이를 줄인다.[^1]

- 환경마다 다시 빌드하면 운영에 올라간 산출물이 시험한 산출물과 달라질 수 있다.[^1]
- 파이프라인은 소스·빌드·시험·환경별 배포 단계를 자동화하고, 산출물에는 버전과 출처를 추적할 정보를 남긴다.[^2]
- DR이 구버전에 머물면 장애 전환 때 예기치 않은 불일치가 생긴다. DR 배포를 같은 파이프라인 단계에 넣어 두면 이를 막을 수 있다.

## 프로젝트 판단

아래 판단은 소규모 연습 프로젝트(가상의 소규모 고객, 직원 30명 안팎의 유통업체를 가정한 DR 이전 과제)에서 요구사항과 규모를 고려해 내린 것이며, 모든 환경에 맞는 정답은 아니다.

- 맥락: DR은 warm standby이고, 서버 설정은 이미지와 템플릿으로 맞춘다(요구사항 5-1, 5-6).
- 판단: 외부 CI/CD 서비스가 같은 산출물을 주 센터와 DR 센터 양쪽에 배포하도록 했다. 도식에는 CI/CD 타일에서 양쪽 센터로 향하는 배포 화살표로 표현했다.
- 주의: 앱 배포와 DB 스키마 변경은 별개이다. DB 변경은 DB 복제로 DR에 전달되므로 배포 순서를 절차서에 적을 필요가 있다. 이 부분은 설계 판단이며 별도 출처로 확인하지는 않았다.

## 관련 문서

- [[Web Server]]
- [[Web Application Server]]

## 출처

[^1]: 여러 번 빌드하지 않고 한 번 만든 산출물을 신뢰할 수 있는 저장소에서 배포하는 원칙. [AWS DevOps Guidance, Anti-patterns for continuous delivery](https://docs.aws.amazon.com/wellarchitected/latest/devops-guidance/anti-patterns-for-continuous-delivery.html)
[^2]: CI/CD 파이프라인의 빌드·시험·배포 단계와 산출물 관리 원칙. [AWS Prescriptive Guidance, Continuous integration and continuous delivery](https://docs.aws.amazon.com/prescriptive-guidance/latest/aws-caf-platform-perspective/ci-cd.html)

## 둘러보기

[[Questions/index|질문과 해답]] · [[Knowledge Base/index|Knowledge Base]] · [[Projects/index|Projects]]
