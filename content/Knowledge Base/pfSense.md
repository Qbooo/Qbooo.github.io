---
title: pfSense
type: knowledge
domain: Security
tags: [Networking, Firewall, VPN]
aliases: [피에프센스, pfSense CE]
parent: Security
status: growing
lang: ko
featured: false
created: 2026-11-24
updated: 2026-11-24
---

## 목차

- [[#개요]]
- [[#네트워크 경계에서의 역할]]
- [[#OpenVPN 원격 접속]]
- [[#연결 검증]]
- [[#관련 문서]]

## 개요

pfSense는 FreeBSD 기반의 방화벽·라우터 소프트웨어로, 네트워크 사이의 트래픽을 라우팅하고 방화벽 정책을 적용하며 VPN 연결을 종료할 수 있다. 운영자는 웹 GUI에서 정책을 관리하고, 원격 사용자는 별도의 VPN 클라이언트로 접속한다. [Netgate pfSense 문서](https://docs.netgate.com/pfsense/en/latest/)

## 네트워크 경계에서의 역할

가상머신으로 배치하면 WAN 인터페이스는 외부 네트워크 방향, LAN 인터페이스는 내부 네트워크 방향에 연결한다. 클라우드에서 공인 IP가 WAN NIC의 사설 IP에 매핑된 경우 외부 클라이언트는 공인 IP로 접속하고, pfSense의 WAN 인터페이스에는 NIC의 사설 IP를 설정한다. 같은 접속에 클라우드 보안 그룹과 pfSense 방화벽 정책이 각각 적용될 수 있다. [Zadara VPN 배포 가이드](https://guides.zadara.com/cs-compute-guide/latest/vpn-edge-cloud.html)

## OpenVPN 원격 접속

Windows PC에서 OpenVPN 클라이언트가 `.ovpn` 프로필을 읽어 pfSense의 VPN 서버에 연결한다. 프로필에는 서버 주소와 접속에 필요한 인증 자료가 포함될 수 있다. 인증기관(CA)은 서버와 사용자 인증서에 서명한다. 인증서와 사용자 인증을 함께 쓰는 구성에서는 클라이언트가 서버 인증서를 검증하고 서버는 사용자 인증서와 로그인 정보를 확인한다. 이후 내부 Bastion에 접속하는 SSH는 별도의 로그인이다. [Netgate OpenVPN 원격 접속 예제](https://docs.netgate.com/pfsense/en/latest/recipes/openvpn-ra.html) · [OpenVPN 프로필 가져오기](https://openvpn.net/connect-docs/import-profile/)

| 설정 | 역할 |
|---|---|
| Tunnel Network | VPN 서버와 접속한 클라이언트가 터널 안에서 사용할 가상 주소 대역 |
| Local Network | VPN 클라이언트에 광고할 내부 목적지 경로 |
| WAN 방화벽 규칙 | 외부 PC가 VPN 서버의 수신 포트에 도달하도록 허용 |
| OpenVPN 방화벽 규칙 | 연결된 사용자가 터널 안에서 접근할 목적지와 포트를 제한 |

예를 들어 Bastion 한 대의 주소를 `/32` 경로로 광고하고, OpenVPN 규칙에서 그 주소의 TCP 22만 허용할 수 있다. [Netgate OpenVPN 방화벽 규칙](https://docs.netgate.com/pfsense/en/latest/vpn/openvpn/firewall-rules.html)

## 연결 검증

먼저 VPN 앱의 연결 상태와 할당된 터널 IP를 확인한다. 다음으로 내부 대상의 서비스 포트에 접근해 정책과 왕복 경로를 검증한다. 원본 터널 IP를 유지하는 라우팅 구성에서는 내부 대상의 응답이 pfSense로 돌아갈 경로가 필요하다. 웹 GUI 관리 경로는 VPN을 통해 접근하도록 구성할 수 있다. [Netgate 라우팅 진단](https://docs.netgate.com/pfsense/en/latest/troubleshooting/routes.html) · [Netgate 원격 GUI 관리](https://docs.netgate.com/pfsense/en/latest/recipes/remote-firewall-administration.html)

## 관련 문서

- [[Firewall|방화벽]] · [[Security Group]] · [[NAT]]
- [[Digital Signature and Certificate|전자서명과 인증서]] · [[SSH]]

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Security 분류]]
