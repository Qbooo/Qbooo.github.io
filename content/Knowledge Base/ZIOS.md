---
title: ZIOS
type: knowledge
domain: Storage
tags: []
aliases: ["VPSA Object Storage"]
parent:
status: growing
lang: ko
featured: false
created: 2026-11-04
updated: 2026-11-04
---

## 1. 개요

ZIOS는 Zadara의 VPSA Object Storage 서비스를 가리키는 제품 명칭이다.

공식 배포 가이드와 관리 문서는 VPSA Object Storage를 ZIOS로 표기한다.[^1][^2]

이 문서에서는 확인되지 않은 약어의 원어를 임의로 풀어 쓰지 않고, 공식 문서의 제품 명칭을 기준으로 설명한다. [[Object Storage|오브젝트 스토리지]] 서비스이므로, VPSA라는 이름이 포함되어 있어도 블록·[[File Storage|파일 스토리지]]를 제공하는 [[VPSA|VPSA Storage Array]]와 구분해야 한다.

## 2. 논리 계정과 실제 저장 자원

ZIOS의 계정·사용자·컨테이너는 서비스의 데이터와 권한을 나누는 논리 단위이고, VC·스토리지 노드·드라이브는 서비스를 실행하고 데이터를 보관하는 자원이다. 22.06-SP2 소개는 Proxy+Storage VC와 선택적인 Proxy VC를 구분한다. 전자는 API 처리와 저장 역할을 결합하고 물리 스토리지 노드가 노출한 드라이브를 사용하며, 후자는 클라이언트 요청을 처리하는 자원을 추가하는 역할이다. 따라서 NGOS의 별도 Storage VC 구성도를 ZIOS에 그대로 적용하면 구성 요소를 잘못 설명할 수 있다.[^d-intro]

클라이언트는 S3·Swift API를 통해 요청하지만 계정·인증과 데이터 처리의 구체적인 경로는 인터페이스와 배포 조건에 따른다. 기존 로드밸런싱 배포 예시는 VIP를 안정된 접속 주소로 제공하고 뒤의 서비스로 요청을 나누는 구성을 다룬다. VIP는 접속 위치이며 데이터 복사본이나 사용자 권한을 뜻하지 않는다.

22.06-SP2 구성 설명을 따라 S3 요청을 개념적으로 연결하면 서명된 HTTP 요청 → 구성된 VIP·로드밸런서 → Proxy 계층의 S3 middleware·인증 처리 → Ring이 결정하는 계정·컨테이너·객체 위치 → Storage 계층과 물리 드라이브의 흐름으로 볼 수 있다. 이는 역할의 연결이며 내부 명령 순서나 성공 응답의 복제 개수를 단정하는 것은 아니다. Ring은 이름과 물리 위치의 매핑이고, Proxy와 Storage 역할은 같은 VC에 결합될 수도 있다.[^d-intro][^zios-client]

같은 클라이언트 문서의 Swift 예시는 v3 인증 엔드포인트로 토큰을 얻은 뒤 Account URL에 `X-Auth-Token`을 포함하는 요청을 보낸다. S3의 액세스 키·서명 요청과 토큰 발급 경로를 혼용하지 않아야 하며, 배포 예시의 특정 포트 번호를 모든 환경의 주소로 고정해서는 안 된다.[^zios-client] 관리 GUI·인증·객체 요청의 경로를 구분하면 “관리 화면에는 접속되지만 데이터 업로드는 실패한다”는 현상을 같은 성공·실패로 묶지 않을 수 있다.[^1]

저장 데이터는 제품의 미러링·이레이저 코딩과 장애 도메인 구성에 따라 보호된다. 같은 물리 노드 안의 드라이브만 복제하면 그 노드 전체 장애에 취약할 수 있으므로 물리 배치와 논리 보호 정책을 함께 읽는다. 보호 방식에 따른 용량 효율·복구 부하와 지원 조합은 해당 버전의 정책을 기준으로 한다.[^d-intro]

## 3. 접근 경로와 구성

Zadara가 제공하는 ZIOS 로드밸런싱 배포 가이드는 데이터 접근, 관리 GUI, 인증을 위한 접속 경로를 구분하고, 로드밸런서와 가상 IP를 구성하는 예를 보여 준다.[^1]

이는 클라이언트의 데이터 요청과 관리·인증 기능이 서로 다른 역할을 가질 수 있음을 보여 주는 구성 사례이다. 해당 문서는 특정 제품 버전을 대상으로 한 배포 예시이므로, 모든 ZIOS 환경에 동일한 포트나 토폴로지를 적용해서는 안 된다.

## 4. NGOS와의 구분

Command Center 관리 가이드는 [[Object Storage|오브젝트 스토리지]]의 IO Engine 유형을 다음과 같이 구분한다.[^2]

| 구분   | 문서에 표시된 유형                                                  |
| ---- | ----------------------------------------------------------- |
| [[NGOS]] | NextGen Object Storage                                      |
| ZIOS | VPSA Object Storage — Mini, Standard, Premium, Premium Plus |

[[NGOS]] 사용자 가이드에서는 NextGen Object Storage가 버전 23.03에서 도입되었으며 VPSA Object Storage와 함께 제공된다고 설명한다.[^3] 이를 기준으로 ZIOS는 기존 VPSA Object Storage 계열, [[NGOS]]는 차세대 [[Object Storage|오브젝트 스토리지]] 계열로 구분해 이해할 수 있다.

위 표는 관리 가이드가 사용하는 엔진 유형의 구분이며, 각 유형의 현재 신규 판매·제공 여부를 증명하는 목록은 아니다. 두 계열의 구체적인 성능 차이나 지원 종료 시점은 이 명칭만으로 알 수 없다. 운영 중인 버전과 해당 릴리스 문서를 확인해야 한다.

## 5. 성능과 장애를 요청 경로로 해석하기

Proxy 자원 확장은 요청 처리 병목을 완화할 수 있고 드라이브 추가는 저장 용량·저장 계층을 바꾼다. 두 변경이 같은 결과를 내지는 않는다. 작은 객체가 많은 부하는 요청 수·메타데이터·인증 처리에, 큰 객체 전송은 바이트 처리량·네트워크·디스크에 더 민감할 수 있다. 공식 Performance Monitor가 드라이브·사용자·계정·컨트롤러별로 관측 범위를 나누는 이유다.[^d-perf]

운영 해석에서는 요청 지연과 오류율을 객체 크기·동시성·처리량·공간 사용량에 연결한다. 연결이 안 되면 DNS·네트워크·TLS 계층, 연결 후 접근 거부라면 인증·정책, 일부 키만 실패하면 이름·객체 상태, 여러 계정에서 동시에 지연되면 공통 서비스와 후단 자원이라는 식으로 가설을 좁힐 수 있다. HTTP 403 같은 상태도 실제 제품이 반환한 오류 코드와 메시지·요청 로그를 함께 해석해야 한다.

외부 복제를 사용한다면 원본 저장 완료와 원격 반영은 다른 상태이다. 원격 장애나 대역폭 부족으로 복제가 늦어질 수 있으므로 복구 시점의 데이터 범위는 실제 동기화 상태에 달려 있다. NGOS로 이전을 검토할 때에는 이름이 바뀌는 업그레이드로만 보지 않고 API·인증·데이터 보호·객체 기능·전환 중 쓰기 처리와 검증 범위를 별도로 비교한다. 이 문서의 22.06-SP2 자료는 그런 비교의 한쪽 기준이지 현재 모든 배치의 사양표가 아니다.



## 6. Amazon S3와의 비교

ZIOS도 Amazon S3 API와 OpenStack Swift API 인터페이스를 제공한다. 공식 ZIOS 클라이언트 가이드는 지원되는 버킷·오브젝트 작업을 나열하면서 AWS 전용 기능 중 일부는 지원 범위 밖이라고 설명한다.[^zios-client]

| 항목 | ZIOS | Amazon S3 |
|---|---|---|
| 제품 | Zadara VPSA Object Storage | AWS의 관리형 [[Object Storage\|오브젝트 스토리지]] 서비스 |
| 저장 대상 | 설정한 Zadara [[Object Storage\|오브젝트 스토리지]] | AWS S3 |
| 연결 정보 | ZIOS 엔드포인트·리전·Access Key/Secret Key | AWS 엔드포인트와 AWS 자격 증명·권한 |
| API 호환성 | 해당 ZIOS 버전에서 지원하는 S3 작업 | AWS S3의 문서화된 API와 서비스 기능 |

S3 호환은 Amazon S3와 동일한 서비스라는 의미가 아니다. AWS CLI를 사용해도 요청은 설정한 엔드포인트로 전송된다. [[NGOS]]의 지원 기능을 ZIOS에도 그대로 적용하지 말고 각 가이드를 따로 확인해야 한다.[^zios-client][^aws-s3]

2026년 11월 4일 확인한 `zios-guide/latest`의 표지 버전은 22.06-SP2이다.[^v-version] URL의 latest라는 문자열만으로 최신 서비스 기능과 동일하다고 판단하면 안 된다. 일부 배포 자료는 이보다도 이전 버전을 대상으로 한다. 실제 서비스에서 필요한 기능의 지원 여부와 제한은 운영 버전의 문서 및 지원 안내를 기준으로 확인한다.
## 7. 관련 문서

- [[NGOS]]
- [[VPSA]]: 블록·[[File Storage|파일 스토리지]]

- [[Storage]]

## 8. 출처

[^1]: [Zadara — Load Balancing Zadara VPSA Object Storage (ZIOS), v1.0.1](https://www.zadara.com/wp-content/uploads/Zadara_Object_Storage_Deployment_Guide.pdf). 특정 버전 대상 배포 자료이다.
[^2]: [Zadara — Object Storage, Command Center Admin Guide](https://guides.zadara.com/admin-guide/latest/object-storage-vpsas.html)
[^3]: [Zadara — NextGen Object Storage User Guide](https://guides.zadara.com/ngos-guide/latest/index.html)

[^aws-s3]: [AWS — What is Amazon S3?](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
[^zios-client]: [Zadara — Using Object Storage Clients, ZIOS Guide](https://guides.zadara.com/zios-guide/latest/object-storage-clients.html). 가이드의 대상 버전을 확인해야 한다.

[^v-version]: [Zadara — VPSA Object Storage Guide, version 22.06-SP2](https://guides.zadara.com/zios-guide/latest/index.html)

[^d-intro]: [Zadara — ZIOS 22.06-SP2 Introduction](https://guides.zadara.com/zios-guide/latest/introduction.html)

[^d-perf]: [Zadara — ZIOS Performance Monitoring](https://guides.zadara.com/zios-guide/latest/performance-monitoring.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Storage 분류|Storage]]
