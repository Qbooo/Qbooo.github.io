---
title: AWS CLI
type: knowledge
domain: Cloud
tags:
  - AWS
  - Automation
aliases:
  - "AWS Command Line Interface"
  - "AWS 명령줄 인터페이스"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-04
---

## 1. 개요

AWS CLI(AWS Command Line Interface)는 셸에서 명령으로 AWS 서비스 API를 호출하는 도구이다.

콘솔에서 수행하는 여러 관리 작업을 명령줄과 스크립트로 처리할 수 있다.[^1]

## 2. 명령이 실행되는 구조

사용자는 서비스와 작업, 매개변수를 지정한다. CLI는 설정된 자격 증명과 리전, 엔드포인트를 사용해 API를 호출하고 결과를 출력한다. 프로파일은 서로 다른 계정이나 환경의 설정을 구분하는 데 사용한다. 이 문서는 AWS CLI v2를 기본 맥락으로 하며, CLI는 Bash·PowerShell 같은 셸 안에서 실행하는 별도 도구이다.

명령줄의 리전·엔드포인트 옵션은 해당 호출의 설정을 덮어쓸 수 있다. 엔드포인트 주소를 바꾼다고 자격 증명이나 인증 방식까지 자동으로 바뀌는 것은 아니다.[^v-options]

조회와 변경은 영향이 다르다. 자원 목록 조회는 상태를 읽지만 생성·수정·삭제 명령은 실제 클라우드 상태를 바꾼다. [[Shell Script]]와 결합하면 반복 작업을 자동화할 수 있으나, 실패 처리와 실행 대상 구분도 스크립트가 책임져야 한다.[^1] API 호출 성공과 생성·변경된 자원의 최종 준비 완료는 다른 상태일 수 있다. 필요한 경우 서비스별 waiter나 상태 조회 규약을 사용한다.[^v-wait]

## 3. Terraform과의 차이

| 기준 | AWS CLI | [[Terraform]] |
|---|---|---|
| 표현 방식 | 수행할 API 작업을 명령으로 지정 | 원하는 자원 구성을 코드로 선언 |
| 상태 관리 | 일반적으로 실행 결과를 사용자가 관리 | 상태 파일로 관리 대상과 실제 자원의 관계를 추적 |
| 주된 용도 | 조회, 개별 운영 작업, 스크립트 | 반복 가능한 인프라 구성과 변경 계획 |

두 도구는 함께 사용할 수 있다. 다만 Terraform이 관리하는 자원을 CLI로 직접 바꾸면 코드와 실제 환경의 차이인 드리프트가 생길 수 있다.[^2]

## 4. Zadara에서 사용하는 맥락

zCompute는 일부 AWS API와 호환되는 인터페이스를 제공한다. AWS CLI 활용 시 AWS 기본 주소와 구분되는 Zadara 대상 엔드포인트를 설정하고 지원되는 작업과 매개변수를 기준으로 사용한다. AWS API 호환성만으로 모든 AWS CLI 명령이 동작한다고 볼 수 없다.[^3] 오래된 연동 예제의 인증·버전 설정이나 TLS 검증 생략 옵션을 그대로 운영 설정으로 채택하지 않고, 현재 환경의 신뢰할 수 있는 인증서와 호환 조건을 확인한다.

## 5. 셸 명령이 클라우드 API가 되는 과정

셸은 명령 이름과 인자를 CLI 프로세스에 전달한다. CLI는 프로파일·환경·역할 같은 설정에서 자격 증명을 얻고 요청할 서비스·리전·엔드포인트를 정한 뒤 해당 API의 요청을 만든다. AWS 인증 요청은 자격 증명과 서명 규약을 사용해 전송되며 서비스의 응답은 JSON 등의 출력과 프로세스 종료 상태로 변환된다. 셸의 인자 해석, CLI의 설정 선택, 원격 서비스의 권한 판정은 서로 다른 계층이다.[^d-config]

환경 변수에 남은 자격 증명, 다른 이름의 프로파일, AssumeRole로 얻은 임시 자격 증명은 실제 호출 신원을 바꿀 수 있다. 임시 자격 증명에는 세션 토큰과 유효기간도 관련된다. 리전은 대상 서비스 위치·서명 맥락에, 엔드포인트는 요청 주소에, 신원은 누가 호출하는지에 해당한다. 주소만 바꾼다고 신원과 서비스 권한이 자동 전환되지 않는 이유다. 실제 우선순위는 사용한 자격 증명 공급 방식의 공식 규칙을 기준으로 읽는다.[^d-config]

Zadara 호환 API를 호출할 때도 CLI는 인터페이스 도구이고 실행 결과를 결정하는 쪽은 대상 엔드포인트다. 네이티브 zCompute 토큰 API와 AWS 호환 API의 인증 방식을 혼용하지 않아야 하며, 지원·무시·미지원 매개변수의 차이는 서비스 문서에서 확인한다.[^3]

## 6. 결과 처리와 비동기 작업을 구분하기

목록 조회는 서비스가 결과를 여러 페이지로 나누어 반환할 수 있다. CLI는 지원되는 API에서 페이지 요청을 반복해 전체 결과를 모을 수 있고, 로컬 pager는 이미 받은 출력을 화면에 나누어 보여 주는 별도 기능이다. `--query` 같은 출력 가공과 서비스에 보내는 필터도 처리 위치가 다르다. 필요한 자원만 서버에서 조회하는 것과 전체 조회 후 일부만 출력하는 것은 네트워크 요청량과 실행 시간에서 차이가 날 수 있다.[^d-pagination]

생성 API의 정상 응답은 요청 접수 또는 객체 생성을 의미할 수 있지만 자원이 사용 가능한 최종 상태까지 같지는 않을 수 있다. waiter는 서비스별 상태를 반복 조회해 특정 조건을 기다리는 기능이다. 상태 반영 지연이나 throttling으로 일시 실패가 생길 때에는 서비스의 재시도·멱등성 규약과 시간 제한을 따라야 한다. 실패 응답만 보고 무조건 같은 생성 작업을 반복하면 이미 만들어진 자원을 중복 관리할 수 있다.

`command not found`는 셸이 도구를 찾지 못한 단계이고 TLS 오류는 연결 신뢰 검증, 서명 불일치는 자격 증명·서명 매개변수, AccessDenied는 권한 판정과 연결되는 현상이다. 잘못된 리전·엔드포인트·JSON 형식도 각각 별도 원인이 된다. CLI 공식 문제 해결 문서는 이런 오류를 나누어 설명한다.[^d-errors] 로그를 읽을 때는 요청이 원격 서비스에 도달했는지부터 구분하고, debug 출력에는 민감한 요청 정보가 포함될 수 있으므로 공개 문서에 그대로 넣지 않는다.

## 7. 관련 문서

- [[Terraform]]
- [[Shell Script]]
- [[Zadara Symphony API]]
- [[Unified Cloud Management Portal]]

- [[Server]]

## 8. 출처

[^1]: [AWS — What is the AWS CLI?](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)
[^2]: [HashiCorp — Terraform State](https://developer.hashicorp.com/terraform/language/state)
[^3]: [Zadara — AWS API](https://guides.zadara.com/cs-compute-guide/latest/aws-ec2-api.html)

[^v-options]: [AWS — CLI global options](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-options.html)

[^v-wait]: [AWS CLI v2 — ec2 wait](https://docs.aws.amazon.com/cli/latest/reference/ec2/wait/)

[^d-config]: [AWS — Configuring settings for the CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)

[^d-pagination]: [AWS — CLI pagination](https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-pagination.html)

[^d-errors]: [AWS — CLI troubleshooting](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
