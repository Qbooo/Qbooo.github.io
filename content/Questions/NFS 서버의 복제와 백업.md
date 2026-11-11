---
title: NFS 서버의 복제와 백업
type: question
domain: Storage
tags:
  - 질문
aliases:
  - NFS 서버도 복제와 백업이 필요한가
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

NFS 서버도 복제와 백업이 필요한가?

## 해답

업무의 가용성과 데이터 복구 요구에 따라 필요하다. NFS 서비스를 단일 서버에만 두면 그 서버가 멈췄을 때 클라이언트의 파일 접근도 중단된다. Red Hat은 두 노드와 공유 스토리지·부동 IP를 사용하는 active/passive NFS 구성 예시를 제공한다.[^1] 완화 방법은 두 가지 축이다.

- 가용성: 이중화 서버를 두고 장애 시 서비스를 넘기는 HA NFS 클러스터를 구성할 수 있다.[^1]
- 데이터 보호: 다른 위치로 복제하고, 삭제나 손상에 대비해 복구 지점을 제공하는 백업을 별도로 둔다. Amazon EFS도 복제와 백업을 서로 다른 데이터 보호 기능으로 제공한다.[^2]

## 프로젝트 판단

아래 판단은 소규모 연습 프로젝트(가상의 소규모 고객, 직원 30명 안팎의 유통업체를 가정한 DR 이전 과제)에서 요구사항과 규모를 고려해 내린 것이며, 모든 환경에 맞는 정답은 아니다.

- 맥락: 소규모 프로젝트이고 HA 클러스터까지 구성하기는 부담이 크다.
- 판단: HA 클러스터는 범위 밖으로 두고, DR 센터의 저장소로 파일을 동기화하며 백업 박스에 이미지 사본을 보관한다. NFS 서버가 단일 장애점이라는 점은 문서에 명시한다.
- 한계: 주 센터에서 NFS 서버가 멈추면 복구 전까지 업로드 파일 접근이 끊긴다. 이 위험은 고객에게 알리고 확장 시 HA를 제안한다.

## 관련 문서

- [[File Storage]]
- [[Storage]]

## 출처

[^1]: 두 노드와 공유 스토리지·부동 IP로 active/passive NFS 서비스를 구성하는 예시. [Red Hat Documentation, An active/passive NFS Server in a High Availability Cluster](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/high_availability_add-on_administration/ch-nfsserver-haaa)
[^2]: Amazon EFS는 백업과 복제를 별도 기능으로 설명하며, 백업은 복구 지점을 제공하고 복제는 대상 파일시스템을 동기화한다. [Amazon EFS, Backing up and replicating data](https://docs.aws.amazon.com/efs/latest/ug/backup-replication.html)

## 둘러보기

[[Questions/index|질문과 해답]] · [[Knowledge Base/index|Knowledge Base]] · [[Projects/index|Projects]]
