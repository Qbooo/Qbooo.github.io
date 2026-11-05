---
title: Harbor
type: knowledge
domain: Container
tags:
  - Container
  - Registry
aliases:
  - "하버"
  - "컨테이너 레지스트리"
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-05
---

## 1. 개요

Harbor는 컨테이너 이미지와 OCI 아티팩트를 저장·배포하고 접근을 제어하는 오픈소스 레지스트리이다.

이미지 저장에 더해 취약점 검사, 서명 연계, 복제와 감사 기능 등을 제공한다.[^1] OCI 아티팩트에는 OCI 형식의 Helm chart도 포함될 수 있으며, 예전 chart 저장 방식과 현재 OCI 기능을 버전 구분 없이 혼용해서는 안 된다.[^v-oci]

## 2. 이미지 저장소가 필요한 이유

컨테이너 실행 환경은 실행할 이미지가 필요하다. 개발자가 만든 이미지를 매번 파일로 복사해 전달하면 버전·배포 위치·접근 권한을 함께 관리하기 어렵다. 레지스트리는 이미지를 이름으로 저장하고 여러 실행 환경이 같은 산출물을 가져가도록 하는 공통 배포 지점이다.[^1]

예를 들어 빌드 시스템이 애플리케이션 이미지를 Harbor에 올리면 테스트 환경과 운영 환경이 필요한 버전을 가져올 수 있다. 이미지의 실제 내용을 식별하는 다이제스트와 사람이 붙이는 태그를 구분하면 어떤 산출물을 배포했는지 더 명확히 설명할 수 있다.[^v-tags]

Harbor는 저장·배포 기능에 프로젝트별 접근 제어, 취약점 검사, 복제 같은 관리 기능을 더한다. 이미지를 보관하는 레지스트리와 그 이미지를 실행하는 컨테이너 런타임은 역할이 다르다. 검사나 서명도 산출물의 신뢰를 판단하는 기능이며 애플리케이션의 모든 동작을 보증하는 기능은 아니다.[^v-scan]

## 3. 이미지가 배포되는 흐름

개발·빌드 시스템이 이미지를 만들면 레지스트리에 push하고, 실행 환경은 필요한 이미지를 pull한다. Harbor는 이 흐름에서 배포할 산출물의 보관과 배포를 담당한다. 컨테이너 실행 자체나 GPU 작업 스케줄링은 실행 플랫폼의 역할이다.

프로젝트는 저장소와 접근 권한을 묶는 단위이다. 저장소는 이미지 버전들을 담으며, 태그는 사람이 읽기 쉬운 이름이고 다이제스트는 콘텐츠를 식별한다. 같은 태그가 다른 이미지로 이동할 수 있으므로 재현 가능한 배포에서는 다이제스트와 태그 정책이 중요하다. Harbor는 프로젝트에 태그 불변 규칙을 구성할 수 있다.[^v-tags]

## 4. 레지스트리를 구성하는 프로세스와 저장소

Harbor는 이미지를 담은 디렉터리 하나가 아니라 여러 기능을 조합한 서비스다. Registry는 아티팩트의 manifest와 blob을 제공하고, Core는 인증·프로젝트·정책 등 관리 기능을 담당하며, Job Service는 복제·검사 같은 백그라운드 작업을 처리한다. 데이터베이스는 사용자·프로젝트·아티팩트 등의 관리 정보를 보관하고 Redis는 캐시·작업 관련 상태 등에 사용된다. 실제 blob은 파일시스템이나 구성한 외부 저장소에 놓일 수 있다. 공식 아키텍처 문서의 구성 역할과 배포 방식의 차이를 구분해야 한다.[^d-architecture]

컨테이너 클라이언트가 push·pull을 시작하면 Registry가 인증을 요구하는 응답을 반환하고 클라이언트가 권한 범위를 가진 토큰을 얻어 다시 요청하는 흐름을 사용할 수 있다. 토큰은 저장소에 대한 pull·push 같은 허용된 작업을 나타낸다. `docker login` 성공과 특정 프로젝트에 push할 권한이 있다는 사실이 다른 이유다.[^d-token]

이미지의 데이터 계층은 blob으로 저장되고 manifest는 필요한 blob과 구성 정보를 가리킨다. Pull은 manifest·digest와 필요한 blob을 가져오는 동작이고 컨테이너 실행은 그 뒤 런타임의 역할이다. 태그는 manifest를 가리키는 이름이므로 같은 태그의 대상이 달라질 수 있는 반면 digest는 콘텐츠 식별에 사용된다.

## 5. 보안과 복제

자동화용 project robot account는 해당 프로젝트의 권한 범위에서 작업하고, system robot account는 관리자가 지정한 여러 프로젝트·시스템 권한을 가질 수 있다. Robot account는 비밀값으로 OCI 클라이언트나 API에 인증하며 UI 로그인 계정과는 다르다.[^v-robot][^v-system]

취약점 검사는 연결된 취약점 검사기(scanner)와 검사 설정·데이터베이스에 의존한다. 검사 결과와 서명 검증을 배포 정책에 연결할 수 있다.[^v-scan] 검사에서 문제가 발견되지 않았다는 사실만으로 애플리케이션 전체가 안전하다고 단정할 수는 없다.

복제 규칙은 조건에 맞는 아티팩트를 다른 레지스트리로 전달한다. 복제는 배포 위치를 늘리는 기능이며, 레지스트리의 모든 설정과 메타데이터를 복구하는 백업 체계와는 구분해야 한다.[^2]

## 6. AI 인프라에서의 역할

[[AI Factory|AI 팩토리]]에서는 학습·추론용 컨테이너의 버전을 관리하는 데 사용할 수 있다. 대규모 학습 데이터와 모든 모델 파일을 반드시 컨테이너 이미지에 넣는 것은 아니다. 데이터 특성에 따라 [[Object Storage]]와 [[File Storage]]를 함께 사용한다.

## 7. 관련 문서

- [[AI Factory]]
- [[GPUaaS]]
- [[Object Storage]]
- [[File Storage]]

- [[Server]]

- [[Storage]]

## 8. 출처

[^1]: [Harbor — 공식 프로젝트](https://github.com/goharbor/harbor)
[^2]: [Harbor — Configuring Replication](https://goharbor.io/docs/main/administration/configuring-replication/)

[^v-oci]: [Harbor — Working with OCI Artifacts](https://goharbor.io/docs/main/working-with-projects/working-with-oci/)

[^v-tags]: [Harbor — Tag Immutability Rules](https://goharbor.io/docs/main/working-with-projects/working-with-images/create-tag-immutability-rules/)

[^v-robot]: [Harbor — Project Robot Accounts](https://goharbor.io/docs/main/working-with-projects/project-configuration/create-robot-accounts/)

[^v-system]: [Harbor — System Robot Accounts](https://goharbor.io/docs/main/administration/robot-accounts/)

[^v-scan]: [Harbor — Vulnerability Scanning](https://goharbor.io/docs/main/administration/vulnerability-scanning/)

[^d-architecture]: [Harbor — Architecture Overview](https://github.com/goharbor/harbor/wiki/Architecture-Overview-of-Harbor)

[^d-token]: [CNCF Distribution — Token Authentication Specification](https://distribution.github.io/distribution/spec/auth/token/)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Container 분류|Container]]
