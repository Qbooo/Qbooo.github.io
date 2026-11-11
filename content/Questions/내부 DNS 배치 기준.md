---
title: 내부 DNS 배치 기준
type: question
domain: Networking
tags:
  - 질문
aliases:
  - 내부 DNS는 어디에 두어야 하는가
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

내부 DNS는 어디에 두어야 하는가?

## 해답

소규모 네트워크에서는 DNS를 별도 서버로 두지 않고 라우터나 방화벽 장비가 겸하는 경우가 흔하다. dnsmasq가 대표적인 구현으로, 작은 네트워크용 DNS와 DHCP를 제공하도록 만들어졌으며 자원이 제한된 라우터와 방화벽에서도 돌아갈 만큼 가볍다.[^1]

- 로컬 이름은 hosts 파일이나 설정 파일로 정의하고, 나머지 질의는 상위 DNS로 전달하며 일부를 캐시한다.[^1]
- OpenWrt 같은 라우터용 배포판이 dnsmasq를 포함한다.[^2]

## 프로젝트 판단

아래 판단은 소규모 연습 프로젝트(가상의 소규모 고객, 직원 30명 안팎의 유통업체를 가정한 DR 이전 과제)에서 요구사항과 규모를 고려해 내린 것이며, 모든 환경에 맞는 정답은 아니다.

- 맥락: 내부 서버 이름을 IP 대신 이름으로 부르고 싶다는 요구(5-5)가 있었고, DNS 서버를 따로 두면 서브넷과 방화벽 규칙이 하나 더 늘어난다.
- 판단: 라우터 VM이 DNS 기능을 함께 맡도록 했다. 라우터는 이미 단일 장애점이라 DNS를 얹어도 추가 부담이 작다고 보았다. DR 라우터도 같은 설정으로 구성한다.
- 한계: 라우터가 멈추면 이름 해석도 함께 멈춘다. 서비스 규모가 커지면 DNS를 분리하고 이중화할 대상이다.

## 관련 문서

- [[Networking]]
- [[Server]]

## 출처

[^1]: dnsmasq는 작은 네트워크용 DNS, DHCP를 제공하고 자원이 제한된 라우터와 방화벽에 맞게 가볍다. [Dnsmasq 공식 문서](https://thekelleys.org.uk/dnsmasq/doc.html)
[^2]: OpenWrt, DD-WRT 등 라우터 배포판이 dnsmasq를 포함한다. [Linux Magazine, Small scale DNS and DHCP with Dnsmasq](https://www.linux-magazine.com/Issues/2009/101/Dnsmasq/(offset)/3)

## 둘러보기

[[Questions/index|질문과 해답]] · [[Knowledge Base/index|Knowledge Base]] · [[Projects/index|Projects]]
