---
title: Lift and Shift
type: knowledge
domain: Cloud
tags: []
aliases:
  - 리프트 앤 시프트
  - Rehost
parent:
status: growing
lang: ko
featured: false
created: 2026-11-05
updated: 2026-11-11
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 이전 전략 속의 위치|2. 이전 전략 속의 위치]]
- [[#3. Rehost와 Relocate의 차이|3. Rehost와 Relocate의 차이]]
- [[#4. 장점과 한계|4. 장점과 한계]]
- [[#5. 이전 뒤 최적화와 rightsizing|5. 이전 뒤 최적화와 rightsizing]]
- [[#6. 벤더별 용어 차이|6. 벤더별 용어 차이]]
- [[#7. 전략 선택의 판단 기준|7. 전략 선택의 판단 기준]]
- [[#8. 이 문서가 다루지 않는 범위|8. 이 문서가 다루지 않는 범위]]
- [[#9. 관련 문서|9. 관련 문서]]
- [[#10. 출처|10. 출처]]

## 1. 개요

Lift and Shift는 애플리케이션의 코드와 구조를 바꾸지 않고 기존 환경에서 클라우드로 그대로 옮기는 이전 방식이다. AWS는 이를 Rehost 전략의 다른 이름으로 소개하며, 애플리케이션을 변경 없이 AWS 클라우드로 옮기는 방식이라고 설명한다.[^1]

이름이 비슷한 Lift and Optimize는 다른 전략을 가리킨다. Google Cloud 문서는 Replatform을 Lift and Optimize라고 부르며, 옮긴 뒤 새 환경에 맞게 최적화하는 방식이다.[^2] 정리하면 Lift and Shift는 변경 없이 옮기는 것이고, Lift and Optimize는 옮기면서 일부를 개선하는 것이다. 이 문서는 앞의 Lift and Shift(Rehost)를 중심으로 다룬다.

이사에 견주면 가구를 해체하거나 새로 사지 않고 놓여 있던 배치 그대로 새집에 옮겨 놓는 일이다. 짐은 빨리 옮겨지지만 새집의 구조에 맞게 가구를 고르지는 않았으므로 방이 남거나 가구가 낡은 채로 남을 수 있다. 다만 이 비유는 위치가 바뀐다는 핵심만 보여 주며, 호환성 점검이나 라이선스 같은 실무 요소는 담지 못한다.

## 2. 이전 전략 속의 위치

클라우드 이전 전략은 애플리케이션을 얼마나 바꾸는지에 따라 나뉜다. AWS Prescriptive Guidance는 일곱 가지 전략을 제시하며 이를 7 Rs라고 부른다.[^1]

| 전략 | AWS의 설명 | 다른 이름 |
|---|---|---|
| Retire | 폐기하거나 보관할 애플리케이션이며 해당 서버를 종료함 | 없음 |
| Retain | 원본 환경에 두거나 아직 이전할 준비가 되지 않은 애플리케이션 | 없음 |
| Rehost | 애플리케이션을 변경 없이 AWS 클라우드로 옮김 | lift and shift |
| Relocate | 많은 서버를 온프레미스 플랫폼에서 그 플랫폼의 클라우드 버전으로 옮김 | 없음 |
| Repurchase | 애플리케이션을 다른 버전이나 제품으로 교체함 | drop and shop |
| Replatform | 옮기면서 일부 최적화를 도입함 | lift, tinker, and shift, lift and reshape |
| Refactor 또는 re-architect | 클라우드 고유 기능을 충분히 활용하도록 아키텍처를 수정함 | 없음 |

AWS는 Replatform의 예로 Microsoft SQL Server 데이터베이스를 Amazon RDS for SQL Server로 옮기는 경우를 든다.[^1] 데이터베이스의 운용을 관리형 서비스에 맡기는 정도의 변경이며 애플리케이션 전체를 다시 설계하지는 않는다.

이사로 보면 Rehost는 배치 그대로 옮기는 이사이고, Replatform은 옮기면서 낡은 냉장고만 새집의 빌트인 설비로 바꾸는 이사이다. Refactor는 새집의 구조에 맞춰 가구를 처음부터 다시 맞추는 일에 가깝다.

## 3. Rehost와 Relocate의 차이

Rehost는 애플리케이션을 변경 없이 클라우드로 옮기는 전략이다. Relocate는 많은 서버를 온프레미스 플랫폼에서 같은 플랫폼의 클라우드 버전으로 옮기는 전략이다.[^1] 두 전략은 AWS가 대규모 이전에 흔히 쓰는 전략으로 함께 꼽으며, Relocate는 플랫폼 단위로 옮긴다는 점이 다르다. 두 전략의 세부 절차 차이는 이 문서의 출처로 확인하지 못했다.

## 4. 장점과 한계

Google Cloud의 이전 시작 문서는 Rehost를 workload를 사소한 수정만 하거나 수정 없이 옮기는 방식으로 정의한다.[^2] 변경 허용 범위는 AWS가 변경 없음, Google이 사소한 수정까지로 서로 다르다.

Microsoft Learn의 Azure Migration Hub는 Rehost를 코드 변경 없이 Azure 인프라로 옮기는 방식으로 설명하며, 빠르고 위험이 낮아 속도가 가장 중요한 단순한 workload에 적합하다고 쓴다.[^3]

- 장점: 코드와 아키텍처를 바꾸지 않아, 속도가 가장 중요한 단순한 workload에서 이전이 빠르고 위험이 낮다고 Microsoft는 설명한다.[^3]
- 한계: 이전한 뒤의 workload는 클라우드에 최적화되어 있지 않다.[^2]

## 5. 이전 뒤 최적화와 rightsizing

Azure Migration Hub는 대부분의 workload 이전이 rehost 또는 replatform을 쓰며, 먼저 이전을 마친 뒤 최적화하고 현대화하라고 권한다.[^3] 이는 Microsoft 한 곳의 권고이며 모든 벤더의 공통 규칙은 아니다.

Rightsizing은 실제 사용량을 측정한 뒤 인스턴스의 종류와 크기를 사용량에 맞게 조정하는 일이다. AWS의 Rehost 안내 블로그는 AWS Compute Optimizer가 Amazon CloudWatch에 기록된 최근 14일의 사용 패턴을 검토해 EC2 인스턴스의 right-sizing 권고를 제공한다고 소개한다.[^4] AWS 공식 문서도 기본 분석 기간을 14일로 밝히며, 유료 확장 지표를 쓰면 최대 93일까지 볼 수 있다고 한다.[^5] 따라서 AWS 도구 기준으로는 이전 직후가 아니라 일정 기간의 실측 지표가 쌓인 뒤에 권고를 받는다. 이전 전에 하는 사전 용량 산정과의 구분은 이 문서의 출처로 확인하지 못했다.

## 6. 벤더별 용어 차이

세 벤더는 같은 개념을 서로 다른 이름과 범위로 부른다. 아래 Microsoft 열은 Azure Migration Hub 문서 기준이다.

| 구분 | AWS | Microsoft | Google Cloud |
|---|---|---|---|
| 변경 없이 옮김 | Rehost (lift and shift)[^1] | Rehost (lift and shift)[^3] | Rehost: lift and shift[^2] |
| 옮기며 일부 개선 | Replatform (lift, tinker, and shift, lift and reshape)[^1] | Replatform (lift, adjust, and shift)[^3] | Replatform: lift and optimize[^2] |
| 구조나 코드 개선 | Refactor 또는 re-architect (한 항목)[^1] | Refactor[^3] | Refactor: move and improve[^2] |
| 아키텍처 재설계 | 위 항목에 포함[^1] | Rearchitect[^3] | Re-architect: continue to modernize[^2] |

AWS는 Refactor와 re-architect를 한 항목으로 묶지만 Google Cloud와 Microsoft는 둘을 따로 구분하므로, 같은 행이 같은 범위를 뜻하지는 않는다. Google Cloud는 Re-architect를 코드의 구조만이 아니라 동작 방식을 바꾸는 것으로 설명하며 모놀리식 애플리케이션을 마이크로서비스로 나누는 일을 예로 든다.[^2]

Microsoft Cloud Adoption Framework의 rationalize 문서는 목록이 다르다. 이 문서에서는 Rehost를 같은 종류끼리 옮기는 like-for-like 이전으로 설명하고 lift and shift라는 표현은 찾지 못했으며, Refactor를 새 기능 추가 없이 코드의 내부 구조를 개선하는 일로 정의한다.[^6] 같은 벤더 안에서도 문서에 따라 표기와 전략 목록이 다르다. Google Cloud는 Rehost, Replatform, Refactor, Re-architect, Rebuild, Repurchase의 여섯 가지를 구분한다.[^2] 따라서 전략의 개수를 외우기보다 각 문서가 정의한 범위를 확인하는 편이 안전하다.

Lift and Optimize라는 표현은 이 문서가 확인한 AWS 전략 페이지와 Microsoft 문서에는 나오지 않고, Google Cloud 문서에서 Replatform을 가리키는 이름으로 쓰인다.[^1][^2][^3]

## 7. 전략 선택의 판단 기준

AWS 전략 페이지는 대규모 이전에 흔히 쓰는 전략으로 rehost, replatform, relocate, retire를 들고, refactor는 대규모 이전에는 권하지 않으며 이전을 마친 뒤 현대화하라고 한다.[^1]

Google Cloud는 replatform 이전이 rehost 이전보다 작업이 더 많이 들고 새 플랫폼의 테스트가 여러 차례 필요하다고 밝히며, Refactor는 rehost보다 시간이 오래 걸리고 새로운 기술 학습이 필요하다고 설명한다.[^2] 그래서 처음부터 한 번에 바꿀지, 먼저 옮기고 나눠서 개선할지는 일정, 위험 허용 범위, 애플리케이션이 클라우드 환경에 맞는지에 따라 정한다.

## 8. 이 문서가 다루지 않는 범위

이 문서는 라이선스, 애플리케이션 호환성 점검, 이전 웨이브 구성, 전략 용어의 연혁을 다루지 않는다. 이전 중 서버를 실제로 복제하고 전환하는 절차는 [[Delta Sync]]와 [[Cutover]]에서 다룬다.

## 9. 관련 문서

- [[Server Migration Types]]

- [[Cutover]]

- [[Delta Sync]]

- [[Rackware|RackWare]]

## 10. 출처

[^1]: [AWS Prescriptive Guidance — About the migration strategies](https://docs.aws.amazon.com/prescriptive-guidance/latest/large-migration-guide/migration-strategies.html)
[^2]: [Google Cloud Architecture Center — Migrate to Google Cloud: Get started](https://docs.cloud.google.com/architecture/migration-to-gcp-getting-started)
[^3]: [Microsoft Learn — Migrate to Azure](https://learn.microsoft.com/en-us/azure/migration/migrate-to-azure)
[^4]: [AWS Migration & Modernization Blog — Unlock Cloud Savings, A Rehost Migration Playbook (Part 4: Migrate – Realizing Planned Savings)](https://aws.amazon.com/blogs/migration-and-modernization/unlock-cloud-savings-a-rehost-migration-playbook-part-4-migrate-realizing-planned-savings)
[^5]: [AWS Compute Optimizer User Guide — What is AWS Compute Optimizer?](https://docs.aws.amazon.com/compute-optimizer/latest/ug/what-is-compute-optimizer.html)
[^6]: [Microsoft Learn — Cloud Adoption Framework: Rationalize the digital estate](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/digital-estate/rationalize)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Cloud 분류|Cloud]]
