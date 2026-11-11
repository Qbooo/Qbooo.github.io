---
title: Flash Memory
type: knowledge
domain: Compute
tags: [Hardware, Memory, Storage]
aliases: [플래시 메모리]
status: growing
lang: ko
created: 2026-11-09
updated: 2026-11-09
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. NOR와 NAND|2. NOR와 NAND]]
- [[#3. 부팅과의 관계|3. 부팅과의 관계]]
- [[#4. 관련 문서와 출처|4. 관련 문서와 출처]]

## 1. 개요

Flash Memory(플래시 메모리)는 전원이 끊겨도 데이터를 유지하면서 전기적으로 지우고 다시 기록할 수 있는 비휘발성 반도체 메모리이다.

메인보드의 [[Firmware|펌웨어]] 저장 칩, SSD, USB 메모리와 메모리 카드에 사용된다. 같은 플래시 계열이어도 용도와 읽기·쓰기 방식은 다르다.

## 2. NOR와 NAND

| 종류 | 특징 | 대표 용도 |
|---|---|---|
| NOR Flash | 임의 위치의 코드를 읽기 쉬워 장치가 저장된 코드를 직접 실행하는 용도에 적합 | BIOS·UEFI와 임베디드 펌웨어 저장 |
| NAND Flash | 큰 용량을 비교적 낮은 비용으로 구성하고 페이지·블록 단위로 관리 | SSD, USB 메모리, 메모리 카드 |

플래시는 보통 읽기보다 쓰기와 지우기에 더 큰 제약이 있고, 셀을 무한히 다시 쓸 수 없다. SSD는 컨트롤러가 오류 정정, 불량 블록 관리와 쓰기 분산을 수행하여 이러한 물리적 특성을 가린다. 따라서 ‘플래시 메모리’라는 부품과 사용자가 보는 ‘SSD’라는 저장장치는 같은 말이 아니다.[^micron]

## 3. 부팅과의 관계

메인보드에는 BIOS·UEFI 펌웨어 이미지가 저장된 플래시 칩이 있다. 전원 직후 CPU가 펌웨어 실행을 시작한다고 해서 CPU가 일반 파일처럼 칩 전체를 탐색하는 것은 아니다. CPU와 칩셋이 정한 초기 주소 매핑과 리셋 동작에 따라 정해진 시작 코드를 실행하고, 펌웨어가 나머지 하드웨어를 초기화한다.

## 4. 관련 문서와 출처

- [[Firmware|펌웨어]], [[BIOS and UEFI|BIOS·UEFI]], [[Disk]], [[Memory]]

[^micron]: [Micron — NOR and NAND Flash Guide](https://assets.micron.com/adobe/assets/urn:aaid:aem:e98bc653-e42d-45a7-9b8c-03cb3d488f05/original/as/nor-nand-flash-guide.pdf)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
