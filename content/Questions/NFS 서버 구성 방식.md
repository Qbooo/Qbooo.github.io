---
title: NFS 서버 구성 방식
type: question
domain: Storage
tags:
  - 질문
aliases:
  - NFS 서버는 별도로 구성해야 하는가
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

NFS 서버는 별도로 구성해야 하며, 어디에 두고 어떤 시스템이 접근해야 하는가?

## 해답

NFS는 서버가 디렉터리를 공유(export)하고, 클라이언트가 그 디렉터리를 자기 파일 시스템에 연결(mount)해서 로컬 디렉터리처럼 읽고 쓰는 방식이다.[^1][^2]

- 누가 읽는가: 파일을 읽고 쓰는 쪽은 애플리케이션인 WAS이다. WAS가 NFS를 mount한 뒤 업로드 파일을 저장하고 읽는다. DB는 파일 자체를 읽지 않고 경로 문자열만 저장하며, WAS가 DB에서 경로를 조회한 뒤 그 경로의 파일을 읽는다.
- 접근 통제: 전통적인 NFS는 인증이 mount 시점 중심이라 네트워크와 export 설정으로 접근 대상을 제한하는 것이 중요하다.[^3] 최신 RHEL 문서에는 Kerberos를 쓰는 NFS 설정 절차도 있다.[^1]
- 위치: 정해진 규칙은 없고, 접근 통제와 구역 설계에 따라 정한다.

## 프로젝트 판단

아래 판단은 소규모 연습 프로젝트(가상의 소규모 고객, 직원 30명 안팎의 유통업체를 가정한 DR 이전 과제)에서 요구사항과 규모를 고려해 내린 것이며, 모든 환경에 맞는 정답은 아니다.

- 맥락: 서브넷이 DMZ, App, DB이고, 사진 파일은 DB와 함께 지켜야 하는 데이터이다.
- 판단: NFS 저장소 서버를 DB 서브넷에 두고 App 서브넷의 WAS에서만 접근하도록 방화벽을 연다. 데이터 보관 서버를 한 구역에 모으고 접근을 최소화하려는 의도이다.
- 비고: 위치 선택은 규칙이 아니라 설계 판단이다.

## 관련 문서

- [[Mount]]
- [[Fstab]]
- [[File Storage]]
- [[Web Application Server]]

## 출처

[^1]: mount 명령으로 서버의 공유 디렉터리를 클라이언트의 마운트 지점에 연결한다. [Red Hat, Mounting NFS shares (RHEL 9)](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_file_systems/mounting-nfs-shares_managing-file-systems)
[^2]: RHEL 8의 기본 NFS 버전은 4.2이다. 최신 RHEL 9 문서에는 Kerberos를 쓰는 클라이언트 설정 절차가 있다. [Red Hat, Mounting NFS shares (RHEL 8)](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/system_design_guide/mounting-nfs-shares_system-design-guide)
[^3]: 구버전 문서 기준으로 NFS의 인증은 mount 시점에 이루어진다. 최신 버전에서는 달라질 수 있다. [Red Hat, Network File System (RHEL 3 참고서, 구버전)](https://mirror.dfw1.budgetvm.com/centos/3/docs/html/rhel-rg-en-3/ch-nfs.html)

## 둘러보기

[[Questions/index|질문과 해답]] · [[Knowledge Base/index|Knowledge Base]] · [[Projects/index|Projects]]
