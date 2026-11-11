---
title: Bootloader
type: knowledge
domain: Compute
tags: [Boot, OS, Linux]
aliases: [부트로더, 부트 로더]
status: growing
lang: ko
created: 2026-11-09
updated: 2026-11-11
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 다른 계층과의 관계|2. 다른 계층과의 관계]]
- [[#3. Linux에서 GRUB을 관리할 수 있는 이유|3. Linux에서 GRUB을 관리할 수 있는 이유]]
- [[#4. 관련 문서와 출처|4. 관련 문서와 출처]]

## 1. 개요

Bootloader(부트로더)는 플랫폼 펌웨어가 실행한 뒤 운영체제 커널과 부팅에 필요한 데이터를 메모리에 적재하고 커널에 제어권을 넘기는 프로그램이다.

PC Linux에서 대표적인 부트로더는 GNU GRUB이다. 여러 커널이나 운영체제 중 하나를 선택하고, 커널 명령줄과 `initramfs` 위치 같은 부팅 정보를 전달할 수 있다.[^grub]

## 2. 다른 계층과의 관계

```mermaid
flowchart LR
    F["BIOS·UEFI<br/>플랫폼 초기화"] --> B["부트로더<br/>커널 선택·적재"]
    B --> K["Linux 커널<br/>자원 관리자 시작"]
    K --> I["PID 1<br/>사용자 공간 시작"]
```

- [[BIOS and UEFI|BIOS·UEFI]]는 운영체제보다 먼저 플랫폼을 준비하고 부팅 대상을 찾는다.
- 부트로더는 어떤 커널을 어떤 옵션으로 실행할지 결정하고 메모리에 적재한다.
- [[Kernel|커널]]은 CPU·메모리·장치를 관리하고 첫 사용자 공간 프로세스를 시작한다.

UEFI 시스템에서는 부트로더 파일이 디스크의 EFI System Partition에 저장될 수 있다. UEFI 펌웨어 자체는 메인보드의 플래시 메모리에 있고 부트로더는 디스크에 있다는 점을 구분해야 한다.

```mermaid
flowchart LR
    U["메인보드<br/>UEFI 펌웨어"] --> E["디스크<br/>EFI System Partition"]
    E --> B["부트로더<br/>.efi 실행 파일"]
    B --> O["운영체제 파티션<br/>커널과 시스템 파일"]
```

Windows에서는 Windows Boot Manager가 부팅 구성 정보를 읽고 C 파티션의 Windows 로더와 커널로 실행을 이어 간다. Linux에서는 GRUB이나 systemd-boot 등이 커널과 `initramfs`를 선택해 실행할 수 있다. 부트로더가 운영체제 본체인 것은 아니다.

## 3. Linux에서 GRUB을 관리할 수 있는 이유

GRUB과 Linux는 별도 프로그램이지만 같은 디스크의 파일과 설정을 매개로 이어진다. Linux로 부팅한 뒤 관리 도구가 GRUB 설정 파일이나 EFI System Partition의 부트 파일을 갱신할 수 있다. 다음 부팅에서는 펌웨어와 GRUB이 그 결과를 읽는다. Linux 안에서 설정할 수 있다는 사실이 GRUB이 Linux 커널 일부라는 뜻은 아니다.

배포판이 생성한 파일과 사용자가 수정해야 하는 원본 설정의 위치는 다를 수 있다. 따라서 `/boot/grub/grub.cfg` 같은 생성 결과를 직접 고치는 대신 배포판이 정한 설정과 갱신 명령을 따르는 것이 일반적이다.

## 4. 관련 문서와 출처

- [[BIOS and UEFI|BIOS·UEFI]], [[Firmware|펌웨어]], [[Kernel|커널]], [[Linux]]
- [[Disk|디스크]], [[Partition|파티션]], [[Filesystem|파일시스템]]

[^grub]: [GNU Project — GNU GRUB Manual](https://www.gnu.org/software/grub/manual/grub/grub.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
