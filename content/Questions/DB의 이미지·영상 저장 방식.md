---
title: DB의 이미지·영상 저장 방식
type: question
domain: Database
tags:
  - 질문
aliases:
  - DB에는 사진이나 동영상을 저장할 수 없는가
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

DB에는 사진이나 동영상을 저장할 수 없는가?

## 해답

저장할 수 있다. 다만 데이터베이스 내부 저장과 별도 파일·오브젝트 저장소 사용 중 어느 방식이 적합한지는 파일 크기, 접근 방식, 트랜잭션 일관성, 백업·복원과 전달 경로를 기준으로 결정해야 한다.

- 데이터베이스 저장: PostgreSQL은 `bytea` 또는 Large Object 기능으로 바이너리 데이터를 저장할 수 있다.[^1]
- 별도 저장소 사용: 파일 스토리지는 공유 경로와 파일 동작을, 오브젝트 스토리지는 키와 API 중심의 대규모 비정형 데이터 저장을 제공한다.[^2]
- 선택 기준: 작은 파일을 업무 행과 하나의 트랜잭션으로 다뤄야 한다면 DB 저장이 단순할 수 있다. 대용량 미디어를 독립적으로 확장·배포해야 한다면 파일 또는 오브젝트 저장소에 두고 DB에는 식별자와 메타데이터를 저장하는 구성이 유리할 수 있다.

## 프로젝트 판단

아래 판단은 소규모 연습 프로젝트(가상의 소규모 고객, 직원 30명 안팎의 유통업체를 가정한 DR 이전 과제)에서 요구사항과 규모를 고려해 내린 것이며, 모든 환경에 맞는 정답은 아니다.

- 맥락: 주문서 사진이 쌓이면 DB 크기와 복제 부담이 커질 수 있다.
- 판단: DB에는 이미지 경로만 저장하고 파일은 NFS 저장소 서버에 두었다. DB 복제와 백업의 크기를 작게 유지하려는 의도이다.
- 비고: BLOB 저장은 불가능해서가 아니라 운영상 권장하지 않아서 택하지 않았다.

## 관련 문서

- [[Database]]
- [[File Storage]]

## 출처

[^1]: PostgreSQL JDBC 공식 문서는 `bytea`와 Large Object라는 두 가지 바이너리 데이터 저장 방식을 설명한다. [pgJDBC, Storing Binary Data](https://jdbc.postgresql.org/documentation/binary-data/)
[^2]: 파일·블록·오브젝트 스토리지의 인터페이스와 일반적인 활용 차이. [Google Cloud, Object storage vs. block storage vs. file storage](https://cloud.google.com/discover/object-vs-block-vs-file-storage)

## 둘러보기

[[Questions/index|질문과 해답]] · [[Knowledge Base/index|Knowledge Base]] · [[Projects/index|Projects]]
