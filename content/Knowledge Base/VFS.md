---
title: VFS
type: knowledge
domain: Linux
tags: [Storage, Kernel, OJT]
aliases:
  - Virtual File System
  - Virtual Filesystem Switch
  - 가상 파일시스템
parent:
status: growing
lang: ko
featured: false
created: 2026-11-18
updated: 2026-11-18
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 공통 파일 인터페이스|2. 공통 파일 인터페이스]]
- [[#3. 파일 접근의 처리 흐름|3. 파일 접근의 처리 흐름]]
- [[#4. 주요 객체|4. 주요 객체]]
- [[#5. 블록 계층과의 관계|5. 블록 계층과의 관계]]
- [[#6. 관련 문서|6. 관련 문서]]
- [[#7. 출처|7. 출처]]

## 1. 개요

VFS(Virtual File System)는 Linux 커널에서 서로 다른 파일시스템을 공통된 파일 연산으로 사용할 수 있게 중재하는 추상화 계층이다.

애플리케이션은 ext4·XFS·NFS의 내부 구현을 각각 호출하지 않고 `open`, `read`, `write`, `close` 같은 공통 시스템 호출을 사용한다. VFS는 경로와 마운트 정보를 확인해 실제 요청을 처리할 파일시스템 구현으로 연결한다.[^vfs]

VFS 자체가 디스크에 데이터를 저장하는 파일시스템은 아니다. 여러 파일시스템이 Linux의 동일한 경로·파일 인터페이스에 참여할 수 있게 하는 커널 내부의 공통 규약에 가깝다.

## 2. 공통 파일 인터페이스

Linux에서는 로컬 디스크의 ext4, 원격 서버의 NFS, 메모리 기반 tmpfs와 커널 정보를 보여 주는 procfs를 하나의 디렉터리 트리 안에서 사용할 수 있다.

```text
/
├── home      → ext4
├── data      → XFS
├── shared    → NFS
├── tmp       → tmpfs
└── proc      → procfs
```

애플리케이션은 어느 파일시스템인지 몰라도 같은 형태로 파일을 연다.

```text
open("/home/report.txt")
open("/shared/report.txt")
```

VFS는 각 경로가 속한 마운트와 파일시스템을 찾아 해당 구현의 연산으로 요청을 넘긴다. 쉽게 설명하면 VFS는 모든 저장 방식을 하나의 파일 경로 체계로 연결하는 안내 창구이다. 다만 실제 권한 검사·캐시·파일시스템 연산은 여러 커널 구성 요소와 해당 파일시스템 구현이 함께 처리한다.

## 3. 파일 접근의 처리 흐름

로컬 ext4 파일에 쓰는 일반적인 흐름은 다음과 같다.

```mermaid
flowchart LR
    A["애플리케이션"] -->|"write 시스템 호출"| B["VFS"]
    B --> C["ext4 구현"]
    C --> D["페이지 캐시·블록 매핑"]
    D --> E["Linux Block Layer"]
    E --> F["장치 드라이버"]
    F --> G["SSD·HDD"]
```

NFS로 마운트한 원격 파일은 VFS 아래의 경로가 달라진다.

```mermaid
flowchart LR
    A["애플리케이션"] -->|"write 시스템 호출"| B["VFS"]
    B --> C["NFS 클라이언트"]
    C -->|"NFS 요청"| D["네트워크"]
    D --> E["NFS 서버 파일시스템"]
    E --> F["서버의 저장장치"]
```

클라이언트의 NFS 접근이 로컬 디스크처럼 경로에 보이더라도, 클라이언트가 원격 디스크의 블록을 직접 배치하는 것은 아니다. 파일 이름·권한·파일 핸들의 처리는 NFS 프로토콜과 서버 파일시스템이 맡는다.

## 4. 주요 객체

VFS는 파일시스템 구현을 공통된 객체와 연산으로 표현한다.[^vfs]

| 객체 | 설명 |
|---|---|
| Superblock | 마운트된 파일시스템 전체의 상태와 연산 |
| Inode | 파일의 유형·권한·크기 등 파일 객체의 메타데이터 |
| Dentry | 경로의 이름과 inode를 연결하는 디렉터리 엔트리 캐시 |
| File | 프로세스가 열어 둔 파일과 현재 접근 상태 |

디스크에 저장된 ext4 inode와 커널 메모리의 VFS inode 객체는 관련되지만 같은 실체는 아니다. VFS 객체는 실행 중인 커널이 파일시스템을 공통 방식으로 다루기 위한 메모리 표현이다.

## 5. 블록 계층과의 관계

VFS와 [[Linux Block Layer|Linux 블록 계층]]은 담당하는 질문이 다르다.

```text
VFS
→ 어느 경로의 어떤 파일을 여는가?

파일시스템
→ 파일 내용과 메타데이터를 어느 블록에 배치하는가?

Linux Block Layer
→ 만들어진 블록 I/O 요청을 어느 장치 큐와 드라이버로 전달하는가?
```

모든 VFS 요청이 블록 계층으로 이어지는 것은 아니다. NFS처럼 네트워크 파일 프로토콜로 이어지거나 procfs처럼 커널 정보를 즉시 만들어 반환하는 파일시스템도 있다. 반대로 데이터베이스가 raw 블록 장치를 직접 사용하면 일반 파일시스템을 거치지 않을 수 있다.

## 6. 관련 문서

- [[Linux Structure|리눅스 구조]]
- [[Filesystem|파일시스템]]
- [[Inode|inode]]
- [[Mount|마운트]]
- [[Linux Block Layer|Linux 블록 계층]]
- [[NFS]]

## 7. 출처

[^vfs]: [Linux Kernel Documentation — Overview of the Linux Virtual File System](https://docs.kernel.org/filesystems/vfs.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Storage 분류|Storage]]
