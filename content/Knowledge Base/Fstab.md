---
title: fstab
type: knowledge
domain: Storage
tags: [Linux, Filesystem, OJT]
aliases: [Fstab, /etc/fstab, 파일시스템 테이블]
status: growing
lang: ko
created: 2026-11-11
updated: 2026-11-11
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 한 줄의 여섯 필드|2. 한 줄의 여섯 필드]]
- [[#3. UUID를 사용하는 이유|3. UUID를 사용하는 이유]]
- [[#4. 기본 옵션과 선택 옵션|4. 기본 옵션과 선택 옵션]]
- [[#5. 마지막 0 2|5. 마지막 0 2]]
- [[#6. 안전한 반영과 확인|6. 안전한 반영과 확인]]
- [[#7. 관련 문서|7. 관련 문서]]
- [[#8. 출처|8. 출처]]

## 1. 개요

`/etc/fstab`은 Linux에서 연결할 파일시스템·경로·옵션을 선언하며, 부팅 과정과 `mount -a`가 참조하는 설정 파일이다.

현재 마운트 상태 그 자체를 저장하는 파일이라기보다 **원하는 파일시스템 배치**를 적는 설정표이다. systemd 기반 시스템에서는 fstab 항목이 mount unit으로 변환되어 처리될 수 있다.[^fstab]

## 2. 한 줄의 여섯 필드

```fstab
UUID=5db84688-14e5-4d90-8a42-5c3cd8a11e7f /data ext4 defaults 0 2
```

| 순서 | 예시 | 의미 |
|---:|---|---|
| 1 | `UUID=...` | 마운트할 장치·파일시스템 식별자 |
| 2 | `/data` | 마운트 지점 |
| 3 | `ext4` | 파일시스템 종류 |
| 4 | `defaults` | 마운트 옵션 |
| 5 | `0` | 전통적인 `dump` 백업 대상 여부 |
| 6 | `2` | 부팅 시 파일시스템 검사 순서 |

공백이 들어간 경로는 단순히 따옴표로 감싸는 것이 아니라 `\040`처럼 이스케이프해야 한다. 주석은 `#`으로 시작한다.[^fstab]

## 3. UUID를 사용하는 이유

`/dev/sdb1` 같은 이름은 하드웨어 검색 순서가 달라지면 바뀔 수 있다. 파일시스템 UUID는 장치 이름이 달라져도 같은 파일시스템을 찾는 데 유리하다.[^mount]

```bash
lsblk -f
sudo blkid /dev/sdb1
```

| 식별자 | 대상 | `mkfs` 실행 시 |
|---|---|---|
| `UUID` | 파일시스템 | 새 파일시스템을 만들면 바뀔 수 있음 |
| `PARTUUID` | GPT 등의 파티션 항목 | 파티션 항목이 유지되면 일반적으로 유지 |

디스크를 블록 단위로 복제하면 UUID까지 복제될 수 있으므로 같은 서버에 원본과 복제본을 함께 연결할 때 중복 여부를 확인한다.

## 4. 기본 옵션과 선택 옵션

`defaults`의 실제 기본값은 커널과 파일시스템에 의존하지만 일반적으로 `rw`, `suid`, `dev`, `exec`, `auto`, `nouser`, `async`가 언급된다.[^fstab]

| 옵션 | 의미 |
|---|---|
| `rw` / `ro` | 읽기·쓰기 / 읽기 전용 |
| `auto` / `noauto` | `mount -a`와 부팅 자동 마운트 대상 / 제외 |
| `nofail` | 장치가 없어도 그 실패를 필수 부팅 실패로 취급하지 않음 |
| `nodev` | 장치 파일 해석 제한 |
| `nosuid` | SUID·SGID 효과 제한 |
| `noexec` | 해당 마운트에서 직접 실행 제한 |
| `x-systemd.device-timeout=10s` | systemd에서 장치를 기다릴 시간 제한 예시 |

`nofail`은 외장·선택적 데이터 디스크에 유용할 수 있지만, 필수 데이터 디스크가 빠진 상태로 애플리케이션까지 실행되면 루트 디스크의 빈 `/data`에 잘못 기록할 수 있다. 부팅 성공과 서비스 정상 동작은 별개의 요구사항이다.

## 5. 마지막 `0 2`

다섯 번째 필드는 전통적인 `dump` 도구의 백업 대상 여부이다. `0`은 제외를 의미한다. 여섯 번째 필드는 `fsck` 검사 순서이다.[^fstab]

```fstab
UUID=ROOT-UUID /     ext4 defaults 0 1
UUID=DATA-UUID /data ext4 defaults 0 2
```

| 값 | 의미 |
|---:|---|
| `0` | 검사 대상에서 제외 |
| `1` | 일반적으로 루트 파일시스템을 먼저 검사 |
| `2` | 루트 이후 다른 로컬 파일시스템 검사 |

파일시스템마다 검사 도구와 부팅 처리 방식이 다르므로 모든 항목에 기계적으로 같은 값을 쓰지 않는다.

## 6. 안전한 반영과 확인

마운트 지점을 먼저 만든다.

```bash
sudo mkdir -p /data
```

현재 장치와 UUID를 재확인하고 fstab을 수정한 뒤, 재부팅 전에 실제 마운트를 시험한다.

```bash
sudo mount -a
findmnt /data
df -hT /data
```

`mount -a`는 문법만 검사하는 명령이 아니라 fstab 항목을 실제로 마운트하려고 시도한다. 운영 환경에서는 기존 마운트와 서비스 영향을 먼저 확인한다. systemd 기반 시스템에서는 fstab 수정 후 `systemctl daemon-reload`가 권장될 수 있다.[^fstab]

잘못된 장치, 존재하지 않는 마운트 지점, 오타 난 파일시스템 종류와 옵션은 부팅 지연이나 긴급 모드 진입을 일으킬 수 있다. 원격 서버에서는 콘솔·복구 경로를 준비한 뒤 변경한다.

## 7. 관련 문서

- [[Mount|마운트]] · [[Filesystem|파일시스템]] · [[Partition|파티션]]
- [[Disk Capacity Check|디스크 용량 확인]] · [[Linux Directory Structure|Linux 디렉터리 구조]]

## 8. 출처

[^fstab]: util-linux, [fstab(5)](https://man7.org/linux/man-pages/man5/fstab.5.html), 여섯 필드, UUID 권고, 기본·`nofail` 옵션과 검사 순서.
[^mount]: util-linux, [mount(8)](https://man7.org/linux/man-pages/man8/mount.8.html), 장치 이름의 불안정성과 UUID·LABEL 사용 권고.

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Storage 분류|Storage]]
