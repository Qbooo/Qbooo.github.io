---
title: VM과 네트워크 장비의 모니터링 방식
type: question
domain: Compute
tags:
  - 질문
  - Monitoring
aliases:
  - 모니터링 에이전트는 모든 VM과 라우터에도 설치해야 하는가
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

모니터링 에이전트는 모든 VM과 라우터에도 설치해야 하는가?

## 해답

에이전트로 호스트 내부 지표를 수집하려면 지표를 보려는 호스트마다 해당 에이전트가 필요하다. 다만 모든 장비에 같은 에이전트를 설치해야 하는 것은 아니다. 물리 라우터나 방화벽처럼 일반 프로그램을 설치하기 어려운 장비는 SNMP처럼 장비가 제공하는 관리 인터페이스를 사용할 수 있다.[^1]

- Zabbix agent는 대상 호스트에서 CPU·메모리·디스크 같은 운영 정보를 직접 수집한다.[^2]
- Prometheus는 대상 시스템이 직접 노출하지 않는 지표를 exporter로 변환해 수집할 수 있다.[^3]
- 따라서 서버에는 host agent나 exporter를, 네트워크 장비에는 SNMP exporter를 사용하는 식으로 대상에 맞는 수집 방식을 조합할 수 있다.[^1][^3]

## 프로젝트 판단

아래 판단은 소규모 연습 프로젝트(가상의 소규모 고객, 직원 30명 안팎의 유통업체를 가정한 DR 이전 과제)에서 요구사항과 규모를 고려해 내린 것이며, 모든 환경에 맞는 정답은 아니다.

- 맥락: 라우터를 장비가 아니라 VM으로 구성했기 때문에 에이전트를 설치할 수 있다.
- 판단: 라우터, 모니터링 서버, DR 쪽 VM까지 모든 VM에 에이전트를 설치하기로 했다. 에이전트를 템플릿 VM에 미리 넣어 두어 새 VM에서 누락되지 않게 한다(요구사항 5-1).
- 한계: 물리 라우터를 쓰는 환경이라면 SNMP 같은 에이전트리스 수집을 병행해야 한다.

## 관련 문서

- [[SNMP]]
- [[Virtual Machine]]

## 출처

[^1]: SNMP 관리 구조와 관리 대상 장비의 agent 역할은 표준 아키텍처로 정의되어 있다. [RFC Editor, RFC 3411](https://www.rfc-editor.org/rfc/rfc3411)
[^2]: Zabbix agent는 대상 호스트에서 운영 정보를 수집해 서버로 전달한다. [Zabbix 공식 문서, Agent](https://www.zabbix.com/documentation/current/en/manual/concepts/agent)
[^3]: Prometheus는 시스템별 exporter와 SNMP exporter 등 여러 통합 방식을 제공한다. [Prometheus 공식 문서, Exporters and integrations](https://prometheus.io/docs/instrumenting/exporters/)

## 둘러보기

[[Questions/index|질문과 해답]] · [[Knowledge Base/index|Knowledge Base]] · [[Projects/index|Projects]]
