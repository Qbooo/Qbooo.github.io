---
title: cloud-init
type: knowledge
domain: IaC
tags: [IaC, Terraform]
aliases: []
parent: IaC 분류
status: growing
lang: ko
featured: false
created: 2026-11-24
updated: 2026-11-24
---

## 목차

- [[#개요]]
- [[#초기 계정과 인증]]
- [[#부팅 후 검증]]
- [[#관련 문서]]

## 개요

cloud-init은 지원되는 클라우드 이미지에서 VM 부팅 시 사용자 데이터와 플랫폼 메타데이터를 읽어 운영체제를 초기화하는 도구다. Terraform이 전달한 `user_data`를 이미지 안의 cloud-init이 해석해 계정, 공개키, 패키지 설정 등에 적용할 수 있다. [cloud-init 사용자 데이터 형식](https://docs.cloud-init.io/en/latest/explanation/format/index.html)

## 초기 계정과 인증

YAML Cloud Config는 첫 줄에 `#cloud-config`를 붙여 형식을 알린다. `users` 항목 안의 `ssh_authorized_keys`에는 공개키를 등록하고, 접속자는 대응하는 개인키를 자기 PC에 보관한다. 콘솔 로그인용 암호 설정과 SSH 비밀번호 로그인 허용 여부는 각각 다룬다. 암호 해시도 민감한 값이라 사용자 데이터와 원본 파일의 접근 범위를 관리한다. [cloud-init 모듈 참조](https://docs.cloud-init.io/en/latest/reference/modules.html)

## 부팅 후 검증

VM의 생성 성공은 클라우드 자원 생성 결과다. 게스트에서 `cloud-init status --long`을 확인하고 필요하면 `/var/log/cloud-init.log`와 `/var/log/cloud-init-output.log`를 본다. 최초 부팅에만 실행되는 모듈도 있으므로 기존 VM의 사용자 데이터만 바꿀 때는 적용 시점을 확인한다. [cloud-init 로그](https://docs.cloud-init.io/en/latest/reference/user_files.html)

## 관련 문서

- [[Terraform]]
- [[Terraform Resource와 Data Source]]
- [[SSH]]

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[IaC 분류]]
