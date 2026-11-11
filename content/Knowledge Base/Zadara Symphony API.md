---
title: Zadara Symphony API
type: knowledge
domain: Cloud
tags:
  - Zadara
  - API
aliases:
  - "자다라 심포니 API"
  - "Symphony API"
  - "zCompute API"
  - "Symp API"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-05
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 화면 조작을 프로그램의 요청으로 표현하는 방식|2. 화면 조작을 프로그램의 요청으로 표현하는 방식]]
- [[#3. 요청과 인증|3. 요청과 인증]]
- [[#4. REST 요청의 구성과 프로젝트 범위|4. REST 요청의 구성과 프로젝트 범위]]
- [[#5. 네이티브 API와 AWS 호환 API|5. 네이티브 API와 AWS 호환 API]]
- [[#6. 운영 자동화에서의 역할|6. 운영 자동화에서의 역할]]
- [[#7. 관련 문서|7. 관련 문서]]
- [[#8. 출처|8. 출처]]

## 1. 개요

Zadara Symphony API는 Zadara zCompute의 자원을 프로그램으로 조회·관리하는 API를 가리킨다.

학습 목록의 이름을 제목으로 사용했고, 공식 가이드의 zCompute API·Symp API Explorer·Symp CLI를 기준으로 설명한다. 금융 메시징 서비스 Symphony의 API와는 구분한다.[^1]

## 2. 화면 조작을 프로그램의 요청으로 표현하는 방식

관리 화면에서 사용자가 선택하고 누르는 작업은 프로그램에서는 자원 경로·메서드·매개변수 등의 요청으로 표현된다. API는 그 요청과 응답의 규칙을 정의한다. 따라서 자동화 도구가 화면의 버튼을 그대로 누르지 않아도 문서화된 API를 사용해 자원을 조회하고 관리할 수 있다.[^1]

예를 들어 특정 프로젝트의 VM을 조회하려면 요청 대상과 프로젝트 범위를 정하고, 적절한 인증 정보와 권한으로 API를 호출한다. 응답 데이터는 다른 프로그램이 읽어 목록을 만들거나 후속 작업의 입력으로 사용할 수 있다. API Explorer는 이런 호출의 입력·출력 구조를 확인하는 도구이다.

요청을 접수했다는 응답과 자원이 실제 준비된 상태는 다를 수 있다. 호출별 응답과 상태 규약을 이해해야 자동화가 어느 단계를 완료했는지 알 수 있다. 또한 네이티브 토큰 API와 AWS 호환 API는 인터페이스·인증 방식·지원 범위가 다르므로 같은 이름의 자원을 다루더라도 구분해서 이해해야 한다.[^2]

## 3. 요청과 인증

API 클라이언트는 대상 엔드포인트로 요청을 보내고 응답을 받는다. zCompute는 Swagger 기반 API Explorer를 제공하며, 클러스터의 API 엔드포인트 목록은 UI에서 확인할 수 있다.

zCompute API에는 인증을 통해 발급받은 유효한 토큰을 사용하며, 공식 인증 예제는 `X-Auth-Token` 헤더를 이용한다. 요청 권한은 사용자와 계정·프로젝트 범위의 정책에 의해 제한된다. 토큰이 있다는 사실과 모든 자원에 대한 관리 권한이 있다는 사실은 다르다.[^1]

## 4. REST 요청의 구성과 프로젝트 범위

API 클라이언트는 엔드포인트의 주소와 자원 경로, HTTP 메서드, 필요한 헤더·본문을 조합해 요청한다. GET은 정보를 읽는 요청에, POST·PUT·PATCH·DELETE 등은 API가 정한 생성·변경·삭제 작업에 쓰일 수 있다. 메서드 이름만으로 제품의 정확한 동작을 추측하기보다 API Explorer에 표시된 필수 입력과 응답 모델을 읽는다. 응답에는 HTTP 상태와 서비스가 정의한 데이터·오류 정보가 포함될 수 있다.[^1][^d-http]

zCompute 공식 인증 예시는 identity/auth를 통해 유효한 토큰을 얻고 프로젝트 토큰을 `X-Auth-Token`에 넣어 API를 호출하는 방식을 보여 준다. API Explorer는 사용 가능한 엔드포인트와 메서드·매개변수를 읽는 창구다. 신원 인증, 프로젝트 범위와 정책이 요청에서 함께 작용하므로 같은 사용자라도 선택한 계정·프로젝트와 권한에 따라 대상 자원이 달라진다.[^1]

AWS CLI·SDK·Terraform은 API 위에서 작업을 표현하는 도구다. AWS 호환 인터페이스에서는 액세스 키와 서명 방식의 요청을 사용하며 네이티브 토큰 요청과 구분한다. AWS SigV4는 요청 세부 정보와 서비스·리전·날짜 정보를 반영하는 서명 규약이다. Zadara 호환 호출은 해당 제품의 엔드포인트·지원 호출과 매개변수 표에 맞춰야 한다.[^2][^d-sign]

## 5. 네이티브 API와 AWS 호환 API

zCompute 관리 API와 AWS 호환 API는 구분해야 한다. 전자의 토큰 인증 방식을 AWS 액세스 키·서명 방식과 혼용하면 안 된다. [[AWS CLI]]나 [[Terraform]]으로 AWS 호환 인터페이스를 사용할 때는 해당 엔드포인트와 자격 증명 구성이 필요하다.

공식 AWS API 문서는 호출별 필수·선택·무시·미지원 매개변수를 명시한다. 따라서 AWS와 비슷한 API를 제공한다는 것은 모든 AWS 서비스와 옵션을 동일하게 제공한다는 의미가 아니다.[^2]

## 6. 운영 자동화에서의 역할

API는 자원 조회, 구성 변경, 배포 도구 연동의 기반이다. 자동화는 요청 결과와 실제 자원의 최종 상태를 함께 다뤄야 한다. 일반적인 자동화 설계에서는 비동기 작업의 요청 접수와 최종 완료를 구분하고, 재시도 시 중복 생성을 고려한다. 이는 모든 Zadara API가 비동기이거나 멱등성을 제공한다는 제품 보장이 아니며, 실제 호출별 응답·상태·재시도 규약을 확인해야 한다. 인증 토큰과 키는 공개 문서나 코드에 넣지 않는다.

## 7. 관련 문서

- [[Unified Cloud Management Portal]]
- [[AWS CLI]]
- [[Terraform]]
- [[Tenant]]

- [[Server]]

## 8. 출처

[^1]: [Zadara — Authentication using zCompute APIs](https://guides.zadara.com/cs-compute-guide/latest/iam-authenticate-api.html)
[^2]: [Zadara — AWS API compatibility tables](https://guides.zadara.com/cs-compute-guide/latest/aws-ec2-api.html)

[^d-http]: [RFC 9110 — HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110.html)

[^d-sign]: [AWS — Signature Version 4](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_sigv.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
