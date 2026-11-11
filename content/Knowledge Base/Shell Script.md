---
title: Shell Script
type: knowledge
domain: Compute
tags: []
aliases: ["셸 스크립트", "쉘 스크립트"]
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-11
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 셸과 스크립트가 필요한 이유|2. 셸과 스크립트가 필요한 이유]]
- [[#3. 명령 한 줄이 운영체제에서 실행되는 과정|3. 명령 한 줄이 운영체제에서 실행되는 과정]]
- [[#4. 기본 구성 요소|4. 기본 구성 요소]]
- [[#5. 실행 방식|5. 실행 방식]]
- [[#6. 클라우드 업무에서의 활용|6. 클라우드 업무에서의 활용]]
- [[#7. 관련 문서|7. 관련 문서]]
- [[#8. 출처|8. 출처]]

## 1. 개요

셸 스크립트(Shell Script)는 셸이 해석해 실행할 명령을 모아 둔 파일이다.

셸은 명령을 해석하는 프로그램이며, Bash는 그중 하나이다. 터미널 창 자체와 셸은 구분된다.[^1][^2]

반복적인 서버 점검, 명령 실행과 결과 수집 등을 자동화할 때 사용한다. 여러 명령을 나열하는 것뿐 아니라 변수, 조건문과 반복문으로 실행 흐름을 구성할 수 있다.

## 2. 셸과 스크립트가 필요한 이유

운영체제에는 파일 복사, 프로세스 조회, 텍스트 처리처럼 작은 기능을 수행하는 여러 명령이 있다. 셸은 사용자가 이 명령을 실행하고 입출력을 연결하는 환경을 제공한다. 스크립트는 그 작업 순서와 조건을 파일에 남겨 같은 절차를 반복할 수 있게 한다.[^1][^d-operation]

예를 들어 여러 서버의 상태를 수집하는 작업은 명령 실행, 결과 저장, 조건에 따른 분류로 나눌 수 있다. 셸 스크립트는 각 기능을 처음부터 새로 구현하기보다 기존 프로그램을 조합한다. 파이프로 텍스트 처리 도구를 연결하는 방식은 이런 조합을 보여 주는 대표적인 예이다.

스크립트는 단순한 명령 목록을 넘어 변수·분기·반복·함수로 실행 흐름을 표현한다. 다만 Bash와 다른 셸의 문법이 모두 같은 것은 아니며, 스크립트가 사용할 프로그램과 실행 환경도 결과에 영향을 준다. 셸 스크립트를 이해하는 핵심은 명령, 데이터, 실행 흐름을 어떻게 연결하는지에 있다.[^2][^d-environment]

## 3. 명령 한 줄이 운영체제에서 실행되는 과정

터미널은 문자 입력과 출력을 연결하는 창구이고 셸은 그 문자를 명령으로 해석한다. Bash는 입력을 토큰과 구문으로 나누고 변수·명령 치환 같은 확장을 수행한 다음 리다이렉션과 실행할 명령을 결정한다. 외부 프로그램은 운영체제 프로세스로 실행되어 인자·환경 변수·작업 디렉터리·파일 디스크립터를 전달받는다. 셸 내장 명령은 셸 자체에서 처리될 수도 있으므로 모든 명령이 반드시 같은 방식으로 자식 프로세스를 만드는 것은 아니다.[^d-operation]

표준 입력은 데이터가 들어오는 통로, 표준 출력은 일반 결과, 표준 오류는 진단 메시지 통로다. 파이프는 앞 명령의 표준 출력을 다음 명령의 표준 입력에 연결한다. 오류 출력까지 자동으로 같은 파이프를 흐른다고 가정하면 수집한 결과와 오류를 혼동한다. `>` 같은 리다이렉션은 명령이 사용할 출력 경로를 바꾸는 셸 기능이며 파일을 열거나 비우는 시점도 실제 실행 결과에 영향을 준다.

```bash
name='data set'
printf '<%s>\n' "$name"
```

이 예시에서 큰따옴표는 공백이 포함된 값을 하나의 인자로 전달한다. 따옴표 없는 변수 확장은 단어 분리와 파일 이름 확장의 영향을 받을 수 있다. 명령어 문자열을 다시 만들어 해석시키는 방식보다 값과 인자를 구분하는 이유가 여기에 있다. 명령 치환·서브셸에서 바꾼 상태가 부모 셸에 그대로 남는다고도 가정할 수 없다.[^d-environment]

## 4. 기본 구성 요소

| 요소 | 용도 |
|---|---|
| 변수와 인자 | 설정값과 실행 시 전달된 입력 처리 |
| 조건문·반복문 | 조건에 따른 분기와 반복 실행 |
| 파이프·리다이렉션 | 명령의 입력·출력 연결과 파일 저장 |
| 종료 상태 | 명령의 성공·실패 판단 |
| 함수 | 반복되는 작업을 묶어 재사용 |

문자열의 공백과 특수문자 때문에 동작이 달라질 수 있으므로 따옴표 사용과 인자 전달 방식을 이해하는 것이 중요하다.[^1]

## 5. 실행 방식

Bash 스크립트는 `bash example.sh`처럼 실행할 수 있다. 이 방식은 Bash가 읽을 수 있는 파일이면 되며 파일 자체의 실행 권한은 필요하지 않다. 실행 권한을 부여하고 `./example.sh`로 실행하는 방식에서는 첫 줄의 shebang으로 해석기를 지정할 수 있다.[^2]

```bash
#!/usr/bin/env bash
printf '현재 호스트: %s\n' "$(hostname)"
printf '현재 시간: %s\n' "$(date)"
```

*위 코드는 호스트 이름과 시간을 출력하는 예시이다. GNU/Linux의 Bash 환경을 기준으로 한다.*

`.sh` 확장자만으로 Bash 사용이 보장되는 것은 아니다. `sh example.sh`처럼 해석기를 명시하면 해당 프로그램이 파일을 읽으므로 shebang의 Bash 지정에 의존할 수 없다. Bash 전용 문법은 다른 셸에서 동작하지 않을 수 있다.

## 6. 클라우드 업무에서의 활용

*활용 예시: 여러 서버의 디스크 사용량 수집, 서비스 상태 점검, API 호출 결과 저장, 실습 환경의 반복 구성.*

스크립트가 명령을 끝까지 실행했다는 사실과 작업이 성공했다는 사실은 다를 수 있다. Bash에서 종료 상태 0은 성공, 비0은 실패나 조건 불충족 등을 나타내며 의미는 명령의 규약에 따른다. 기본 파이프라인 상태는 마지막 명령의 상태라 앞선 실패를 놓칠 수 있다. `pipefail`은 이 판정 방식을 바꾸며, `set -e`에도 조건문·명령 목록 등 예외가 있으므로 개별 작업 결과 검증을 대신하지 않는다.[^v-exit][^v-pipe][^v-set]

## 7. 관련 문서

- [[AI Infrastructure Architect|AI 인프라 아키텍트]]: 운영 자동화를 포함하는 인프라 설계 역할

- [[AWS CLI]]
- [[Terraform]]

- [[Server]]

## 8. 출처

[^1]: [GNU Bash — What is a shell?](https://www.gnu.org/software/bash/manual/html_node/What-is-a-shell_003f.html)
[^2]: [GNU Bash — Shell Scripts](https://www.gnu.org/software/bash/manual/html_node/Shell-Scripts.html)

[^v-exit]: [GNU Bash — Exit Status](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html)

[^v-pipe]: [GNU Bash — Pipelines](https://www.gnu.org/s/bash/manual/html_node/Pipelines.html)

[^v-set]: [GNU Bash — The Set Builtin: errexit exceptions](https://www.gnu.org/s/bash/manual/html_node/The-Set-Builtin.html)

[^d-operation]: [GNU Bash — Shell Operation](https://www.gnu.org/software/bash/manual/html_node/Shell-Operation.html)

[^d-environment]: [GNU Bash — Command Execution Environment](https://www.gnu.org/s/bash/manual/html_node/Command-Execution-Environment.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
