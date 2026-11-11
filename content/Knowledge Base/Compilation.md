---
title: Compilation
type: knowledge
domain: Compute
tags: [Program, Linux]
aliases: [컴파일, 컴파일 과정]
status: growing
lang: ko
created: 2026-11-09
updated: 2026-11-09
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 실행 파일과 실행의 차이|2. 실행 파일과 실행의 차이]]
- [[#3. 관련 문서와 출처|3. 관련 문서와 출처]]

## 1. 개요

Compilation(컴파일)은 사람이 작성한 고수준 언어의 소스 코드를 대상 CPU와 실행 환경이 처리할 수 있는 더 낮은 수준의 코드로 변환하는 과정이다.

컴파일을 곧바로 ‘실행 파일 만들기’와 완전히 같은 뜻으로 쓰기도 하지만, C 프로그램이 실행 파일이 되기까지는 일반적으로 전처리·컴파일·어셈블·링크 단계가 이어진다.[^gcc]

```mermaid
flowchart LR
    C["소스 코드<br/>hello.c"] --> P["전처리"]
    P --> A["컴파일<br/>어셈블리 코드"]
    A --> O["어셈블<br/>목적 파일"]
    O --> L["링크<br/>라이브러리·목적 파일 결합"]
    L --> E["실행 파일"]
```

## 2. 실행 파일과 실행의 차이

실행 파일은 디스크에 저장된 [[Program and Process|프로그램]]이다. 사용자가 실행을 요청하면 운영체제가 파일 형식을 확인하고 코드와 데이터를 가상 메모리에 배치하여 프로세스로 실행한다. 즉 컴파일은 프로그램을 준비하는 과정이고, 실행은 그 프로그램을 프로세스로 만드는 과정이다.

모든 언어가 같은 방식으로 미리 기계어 실행 파일을 만드는 것은 아니다. Python처럼 소스나 바이트코드를 인터프리터가 실행하는 방식, Java처럼 바이트코드와 가상 머신을 사용하는 방식, 실행 중 자주 쓰는 코드를 기계어로 바꾸는 JIT 방식도 있다.

[[Firmware|펌웨어]]도 개발 과정에서는 소스 코드를 컴파일·링크하여 장치가 실행할 이미지로 만들 수 있다. 완성된 펌웨어 이미지가 [[Flash Memory|플래시 메모리]]에 기록되는 것이지, 사람이 작성한 소스 코드가 그대로 CPU 명령이 되는 것은 아니다.

## 3. 관련 문서와 출처

- [[Program and Process|프로그램과 프로세스]], [[Firmware|펌웨어]], [[CPU]]

[^gcc]: [GNU Compiler Collection — Options Controlling the Kind of Output](https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
