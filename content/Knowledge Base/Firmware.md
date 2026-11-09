---
title: Firmware
type: knowledge
domain: Compute
tags: [Hardware, Boot, OS]
aliases: [펌웨어]
status: growing
lang: ko
created: 2026-11-09
updated: 2026-11-09
---

## 1. 개요

Firmware(펌웨어)는 특정 하드웨어를 초기화·제어하고 다른 소프트웨어와 통신하게 하는, 장치의 비휘발성 메모리에 저장되는 소프트웨어이다.

펌웨어는 하드웨어 그 자체가 아니라 하드웨어 가까이에서 동작하는 프로그램이다. PC 메인보드의 [[BIOS and UEFI|BIOS·UEFI]], SSD 컨트롤러 코드, NIC 펌웨어, 서버의 BMC 펌웨어가 모두 사례이다. 전원이 꺼져도 유지되어야 하므로 ROM이나 [[Flash Memory|플래시 메모리]]에 저장되는 경우가 많다.[^nist]

## 2. 일반 프로그램과의 차이

| 구분 | 펌웨어 | 일반 애플리케이션 |
|---|---|---|
| 주된 대상 | 특정 장치와 플랫폼 | 사용자의 업무와 기능 |
| 저장 위치 | 장치의 비휘발성 메모리 | SSD·디스크의 파일시스템 |
| 실행 시점 | 전원 직후 또는 장치 초기화 시점부터 가능 | 운영체제가 준비된 뒤가 일반적 |
| 갱신 방법 | 제조사 이미지와 전용 갱신 절차 | 패키지 설치·파일 교체 |

경계가 절대적인 것은 아니다. 현대 펌웨어도 규모가 큰 소프트웨어이며 갱신할 수 있다. 다만 갱신 중 전원이 끊기거나 잘못된 이미지를 적용하면 장치가 부팅되지 않을 수 있어 일반 프로그램 업데이트보다 복구 절차가 중요하다.

## 3. 부팅에서의 역할

CPU는 전원이 들어오면 아키텍처가 정한 시작 위치에서 코드를 실행한다. 메인보드 펌웨어는 CPU·메모리와 부팅에 필요한 장치를 초기화하고, 부팅 항목을 선택하여 [[Bootloader|부트로더]]로 제어권을 넘긴다. 펌웨어가 Linux 안에 포함된 것이 아니라 Linux보다 먼저 실행되는 별도 계층이다.

## 4. 관련 문서와 출처

- [[BIOS and UEFI|BIOS·UEFI]], [[Flash Memory|플래시 메모리]], [[Bootloader|부트로더]], [[Linux]]

[^nist]: [NIST CSRC Glossary — Firmware](https://csrc.nist.gov/glossary/term/firmware)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
