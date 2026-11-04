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
updated: 2026-11-04
---

## 1. 개요

셸 스크립트(Shell Script)는 셸이 해석해 실행할 명령을 파일로 작성한 것이다.

셸은 명령을 해석하는 프로그램이며, Bash는 그중 하나이다. 터미널 창 자체와 셸은 구분된다.[^1][^2]

반복적인 서버 점검, 명령 실행과 결과 수집 등을 자동화할 때 사용한다. 여러 명령을 나열하는 것뿐 아니라 변수, 조건문과 반복문으로 실행 흐름을 구성할 수 있다.

## 2. 명령 한 줄이 운영체제에서 실행되는 과정

터미널은 문자 입력과 출력을 연결하는 창구이고 셸은 그 문자를 명령으로 해석한다. Bash는 입력을 토큰과 구문으로 나누고 변수·명령 치환 같은 확장을 수행한 다음 리다이렉션과 실행할 명령을 결정한다. 외부 프로그램은 운영체제 프로세스로 실행되어 인자·환경 변수·작업 디렉터리·파일 디스크립터를 전달받는다. 셸 내장 명령은 셸 자체에서 처리될 수도 있으므로 모든 명령이 반드시 같은 방식으로 자식 프로세스를 만드는 것은 아니다.[^d-operation]

표준 입력은 데이터가 들어오는 통로, 표준 출력은 일반 결과, 표준 오류는 진단 메시지 통로다. 파이프는 앞 명령의 표준 출력을 다음 명령의 표준 입력에 연결한다. 오류 출력까지 자동으로 같은 파이프를 흐른다고 가정하면 수집한 결과와 오류를 혼동한다. `>` 같은 리다이렉션은 명령이 사용할 출력 경로를 바꾸는 셸 기능이며 파일을 열거나 비우는 시점도 실행 의미의 일부다.

```bash
name='data set'
printf '<%s>\n' "$name"
```

이 예시에서 큰따옴표는 공백이 포함된 값을 하나의 인자로 전달한다. 따옴표 없는 변수 확장은 단어 분리와 파일 이름 확장의 영향을 받을 수 있다. 명령어 문자열을 다시 만들어 해석시키는 방식보다 값과 인자를 구분하는 이유가 여기에 있다. 명령 치환·서브셸에서 바꾼 상태가 부모 셸에 그대로 남는다고도 가정할 수 없다.[^d-environment]

## 3. 실행 방식

Bash 스크립트는 `bash example.sh`처럼 실행할 수 있다. 이 방식은 Bash가 읽을 수 있는 파일이면 되며 파일 자체의 실행 권한은 필요하지 않다. 실행 권한을 부여하고 `./example.sh`로 실행하는 방식에서는 첫 줄의 shebang으로 해석기를 지정할 수 있다.[^2]

```bash
#!/usr/bin/env bash
printf '현재 호스트: %s\n' "$(hostname)"
printf '현재 시간: %s\n' "$(date)"
```

*위 코드는 호스트 이름과 시간을 출력하는 예시이다. GNU/Linux의 Bash 환경을 기준으로 한다.*

`.sh` 확장자만으로 Bash 사용이 보장되는 것은 아니다. `sh example.sh`처럼 해석기를 명시하면 해당 프로그램이 파일을 읽으므로 shebang의 Bash 지정에 의존할 수 없다. Bash 전용 문법은 다른 셸에서 동작하지 않을 수 있다.

## 4. 기본 구성 요소

| 요소 | 용도 |
|---|---|
| 변수와 인자 | 설정값과 실행 시 전달된 입력 처리 |
| 조건문·반복문 | 조건에 따른 분기와 반복 실행 |
| 파이프·리다이렉션 | 명령의 입력·출력 연결과 파일 저장 |
| 종료 상태 | 명령의 성공·실패 판단 |
| 함수 | 반복되는 작업을 묶어 재사용 |

문자열의 공백과 특수문자 때문에 동작이 달라질 수 있으므로 따옴표 사용과 인자 전달 방식을 이해하는 것이 중요하다.[^1]

## 5. 반복 가능한 운영 자동화의 조건

운영 자동화는 순서대로 명령을 쓰는 것에서 더 나아가 대상·성공 조건·부분 실패 뒤의 상태를 정의한다. 예를 들어 같은 자원을 생성하는 명령을 재실행하면 중복이 생기는지, 이미 존재할 때 조회·갱신으로 처리하는지를 구분한다. 이런 멱등성 설계는 재시도가 실제로 안전한지를 결정한다. 실패 뒤 무조건 반복하면 부분 완료된 변경을 더 복잡하게 만들 수 있다.

프로세스 종료 상태와 업무 성공도 나눈다. API 도구가 요청 접수를 성공으로 반환해도 서비스 준비는 나중에 끝날 수 있다. 명령의 상태·표준 오류·대상 서비스의 최종 상태를 연결하는 로그가 있어야 어떤 단계가 실패했는지 알 수 있다. `trap`은 종료·신호 시 정리 동작을 연결하는 수단이지만 모든 실패를 복원하는 트랜잭션 기능은 아니다. `set -e`·`pipefail`은 앞서 설명한 셸 판정 규칙일 뿐 애플리케이션 성공 조건을 대신하지 않는다.

수동 실행과 예약·서비스 실행이 다른 이유도 프로세스 환경에서 찾는다. 로그인한 사용자의 PATH·환경 변수·현재 디렉터리·인증 파일·권한이 자동 실행 환경과 다르면 같은 스크립트가 다른 프로그램을 찾거나 다른 상대 경로를 연다. systemd는 WorkingDirectory·User·Environment 같은 실행 환경을 명시적으로 구성한다.[^d-systemd] 따라서 “수동으로는 된다”는 증거만으로 자동 실행 환경까지 같다고 결론 내리지 않고 실행 주체·경로·환경·입출력과 최종 상태를 함께 해석한다.



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

[^d-systemd]: [systemd — Execution environment](https://www.freedesktop.org/software/systemd/man/latest/systemd.exec.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
