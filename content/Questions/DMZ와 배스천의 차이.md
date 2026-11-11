---
title: DMZ와 배스천의 차이
type: question
domain: Security
tags:
  - 질문
aliases:
  - DMZ와 배스천은 어떻게 다른가
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

DMZ와 배스천은 어떻게 다른가?

## 해답

DMZ는 네트워크 구역이고, 배스천은 그 경계에 두는 서버 한 대이다. 두 개념은 층위가 다르며 함께 쓰이는 경우가 많다.

- DMZ: 신뢰하는 내부망과 신뢰하기 어려운 외부망 사이에 두는 경계 네트워크 구간이다.[^1] DMZ와 내부망 사이를 오가는 트래픽도 방화벽을 거치므로 방화벽 정책을 적용할 수 있다.[^1] 방화벽 한 대에 인터페이스 세 개를 두는 구성으로도 만든다.[^2]
- 배스천(bastion host): 관리자가 내부 서버에 접속하기 전에 반드시 먼저 거치는, 강하게 보안 설정한 중간 서버이다.[^3] 점프 서버와 일상적으로 같은 뜻으로 쓰인다.[^3]
- 관계: 배스천은 보통 DMZ 같은 경계 구역에 둔다. DMZ에는 웹 서버처럼 외부에서 접근해야 하는 서버도 함께 놓을 수 있으므로, DMZ는 구역의 이름이고 배스천은 그 구역 안의 한 가지 역할이다.
- 용어 차이: 배스천이 원래는 외부에 노출되는 강화 서버 전반을 가리켰고 점프 서버는 관리용 중계 지점을 가리켰다는 설명이 있다.[^3] 출처마다 경계가 조금씩 달라서 문서에서는 역할을 풀어 쓰는 편이 안전하다.
- 한계: 배스천은 공개 접점이자 단일 장애점이 될 수 있다.[^4] 열어 두는 포트를 최소화하고 접근을 통제해야 한다.

## 프로젝트 판단

아래 판단은 소규모 연습 프로젝트(가상의 소규모 고객, 직원 30명 안팎의 유통업체를 가정한 DR 이전 과제)에서 요구사항과 규모를 고려해 내린 것이며, 모든 환경에 맞는 정답은 아니다.

- 맥락: 요구사항에 개인 계정 접속(4-3)과 외부 노출 없는 점검(4-6)이 있고, 관리자 접속 경로를 하나로 모으고 싶었다.
- 판단: 배스천을 DMZ 서브넷에 한 대 두고 SSH만 허용했다. 관리 접속이 한 곳을 지나면 접근 통제와 접속 기록을 한 곳에서 볼 수 있다.
- 감수한 부분: 배스천 이중화는 범위 밖으로 두었다. 라우터가 이미 단일 장애점이라 배스천만 이중화하는 효과가 작다고 보았다.

## 관련 문서

- [[Networking]]
- [[SSH]]
- [[Server]]

## 출처

[^1]: NIST는 DMZ를 경계 네트워크 구간으로 정의하고, 보호 측과의 트래픽도 방화벽을 지난다고 설명한다. [NIST 용어집 Demilitarized Zone (SP 800-41 Rev.1 인용)](https://csrc.nist.gov/glossary/term/demilitarized_zone)
[^2]: 방화벽 한 대에 세 인터페이스를 두는 screened subnet 구성으로 DMZ를 만들 수 있다. [TechTarget, Screened subnet](https://www.techtarget.com/cybersecurity/definition/screened-subnet?amp=1)
[^3]: 배스천은 관리용 게이트웨이이며 점프 서버와 거의 같은 뜻으로 쓰이고, 역사적으로는 노출된 강화 서버 전반을 가리켰다. [Monofor, Bastion Host 용어집](https://monofor.com/glossary/bastion-host)
[^4]: 배스천은 공개 대상이자 단일 장애점이 될 수 있다. [WZ-IT, SSH Bastion and Jump Host](https://wz-it.com/en/knowledge/remote-access/ssh-bastion-jump-host/)

## 둘러보기

[[Questions/index|질문과 해답]] · [[Knowledge Base/index|Knowledge Base]] · [[Projects/index|Projects]]
