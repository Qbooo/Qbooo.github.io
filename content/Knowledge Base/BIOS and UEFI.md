---
title: BIOS and UEFI
type: knowledge
domain: Compute
tags: [Firmware, Boot, OS]
aliases: [BIOS, UEFI, 바이오스, BIOS와 UEFI, BIOS·UEFI]
status: growing
lang: ko
created: 2026-11-09
updated: 2026-11-09
---

## 1. 개요

BIOS와 UEFI는 운영체제보다 먼저 실행되어 컴퓨터의 기본 하드웨어를 초기화하고 부팅할 프로그램을 선택하는 플랫폼 펌웨어 방식이다.

BIOS는 전통적인 PC 부팅 인터페이스이고 UEFI는 이를 대체하는 현대적인 규격이다. 일상적으로 UEFI 설정 화면도 ‘BIOS 화면’이라고 부르지만 기술적으로는 구분된다.

## 2. 부팅에서 하는 일

```text
전원 인가
  ↓
CPU가 정해진 시작 지점의 펌웨어 실행
  ↓
메모리·CPU·부팅 장치 등 초기화
  ↓
부팅 순서와 부팅 항목 확인
  ↓
부트로더 실행
```

UEFI의 Boot Manager는 NVRAM에 저장된 `BootOrder`와 부팅 항목에 따라 UEFI 드라이버와 운영체제 로더 같은 UEFI 애플리케이션을 불러온다.[^uefi] UEFI가 운영체제를 스스로 완전히 실행하는 것이 아니라, 준비를 마친 뒤 [[Bootloader|부트로더]]에 제어권을 넘긴다고 이해하면 된다.

## 3. BIOS와 UEFI의 차이

| 관점 | 전통 BIOS | UEFI |
|---|---|---|
| 부트 코드 발견 | 부팅 디스크의 정해진 영역에 의존 | EFI System Partition의 파일과 NVRAM 부팅 항목 사용 가능 |
| 인터페이스 | 오래된 PC 호환 방식 | 표준화된 부팅·런타임 서비스 제공 |
| 파티션과 확장성 | MBR 방식과 함께 쓰인 경우가 많음 | GPT와 함께 쓰는 구성이 일반적 |
| 보안 기능 | 표준 Secure Boot 없음 | Secure Boot 체인을 구성할 수 있음 |

UEFI와 GPT, BIOS와 MBR은 자주 함께 사용되지만 완전히 같은 개념은 아니다. 전자는 펌웨어와 부팅 인터페이스이고 후자는 디스크 파티션 정보 형식이다.

## 4. 관련 문서와 출처

- [[Firmware|펌웨어]], [[Flash Memory|플래시 메모리]], [[Bootloader|부트로더]], [[Linux]]

[^uefi]: [UEFI Forum — UEFI Specification 2.11, Boot Manager](https://uefi.org/specs/UEFI/2.11/03_Boot_Manager.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
