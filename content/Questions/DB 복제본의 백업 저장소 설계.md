---
title: DB 복제본의 백업 저장소 설계
type: question
domain: Storage
tags:
  - 질문
  - Backup
aliases:
  - DR 스토리지가 DB 복제본의 백업도 함께 맡을 수 있는가
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

DR 스토리지가 DB 복제본의 백업도 함께 맡을 수 있는가?

## 해답

역할이 섞이지 않게 구분하면 된다.

- 스냅샷: 특정 시점의 상태를 기록하는 빠른 되돌리기 수단이다. 원본과 같은 장애·권한 경계를 공유하면 원본과 함께 영향을 받을 수 있으므로 별도 보존 정책과 복구 지점이 필요하다.[^1]
- 백업: 되돌릴 시점, 보관 기간, 운영 환경과의 분리, 검증된 복원 절차를 갖춘 복사본이다.[^2]
- 복제본에서 백업 파일을 뜨는 방식: 복제본 서버에서 백업 파일을 만들어 별도 시스템에 보관하는 구성은 설계상 가능하다. 다만 이번 조사에서 이 패턴을 직접 설명하는 출처를 확인하지 못했다. 사용하는 DBMS 문서로 확인이 필요하다.

## 프로젝트 판단

아래 판단은 소규모 연습 프로젝트(가상의 소규모 고객, 직원 30명 안팎의 유통업체를 가정한 DR 이전 과제)에서 요구사항과 규모를 고려해 내린 것이며, 모든 환경에 맞는 정답은 아니다.

- 맥락: DR 쪽 스토리지 박스가 복제 수신과 백업을 함께 맡는 그림이 이상하다는 의문에서 시작되었다.
- 판단: 역할을 분리했다. DB 복제본은 복제를 받는 서버, Backup 박스는 백업 파일과 이미지 사본을 보관하는 별도 VM으로 도식에 따로 그렸다.
- 확인 필요: 복제본에서 백업 파일을 만드는 절차는 DB 제품 선정 후 해당 제품 문서로 확정한다.

## 관련 문서

- [[Storage]]
- [[Disk]]

## 출처

[^1]: 연속 복제는 데이터 손상이나 파괴까지 막지 못할 수 있으므로 버전 관리·시점 복구와 복제 데이터의 백업이 필요하다. [AWS Well-Architected Framework, REL13-BP02](https://docs.aws.amazon.com/wellarchitected/latest/framework/rel_planning_for_recovery_disaster_recovery.html)
[^2]: 백업 대상 식별, 자동화, 보안과 주기적인 복구 시험에 관한 원칙. [AWS Well-Architected Framework, Back up data](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/back-up-data.html)

## 둘러보기

[[Questions/index|질문과 해답]] · [[Knowledge Base/index|Knowledge Base]] · [[Projects/index|Projects]]
