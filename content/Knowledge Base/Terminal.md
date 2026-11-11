---
title: Terminal
type: knowledge
domain: Compute
tags: [Linux, CLI, Shell]
aliases: [터미널, 터미널 에뮬레이터]
status: growing
lang: ko
created: 2026-11-09
updated: 2026-11-09
---

## 목차

- [[#1. 개요|1. 개요]]
- [[#2. 물리 터미널과 현대 프로그램|2. 물리 터미널과 현대 프로그램]]
- [[#3. 연결 흐름|3. 연결 흐름]]
- [[#4. 관련 문서와 출처|4. 관련 문서와 출처]]

## 1. 개요

Terminal(터미널)은 사용자의 문자 입력을 컴퓨터에 전달하고 컴퓨터가 돌려준 문자와 제어 결과를 표시하는 입출력 끝점이며, 현대에는 이 동작을 소프트웨어로 구현한 터미널 에뮬레이터를 주로 뜻한다.

터미널은 명령의 의미를 판단하는 [[Shell|셸]]과 다르다. 터미널은 키 입력, 문자 표시, 커서 이동과 화면 제어를 담당하고 셸은 그 통로로 받은 `ls`, `cd` 같은 명령을 해석한다.

## 2. 물리 터미널과 현대 프로그램

![DEC VT100 물리 터미널](https://commons.wikimedia.org/wiki/Special:Redirect/file/DEC_VT100_terminal.jpg?width=960)

*DEC VT100 터미널. 사진: Jason Scott, [Wikimedia Commons](https://commons.wikimedia.org/wiki/File:DEC_VT100_terminal.jpg), [CC BY 2.0](https://creativecommons.org/licenses/by/2.0/).*

과거의 물리 터미널은 키보드와 문자 화면을 갖추고 직렬 통신 등으로 중앙 컴퓨터에 연결됐다. 셸과 업무 프로그램은 보통 중앙 컴퓨터에서 실행되고, 터미널은 입력과 표시를 맡았다. `xterm`은 VT 계열 터미널의 제어 동작을 X Window System 위에서 재현하는 터미널 에뮬레이터이다.[^xterm]

MobaXterm은 같은 계열의 화면 기능만을 뜻하지 않는다. PuTTY 기반 터미널 에뮬레이터에 SSH 클라이언트, X 서버, SFTP 탐색기와 원격 접속 도구를 묶은 제품이다.[^moba] SSH 접속 시 MobaXterm 화면은 Windows에 있어도 셸과 프로그램은 원격 Linux 서버에서 실행될 수 있다.

## 3. 연결 흐름

```text
키보드 → 터미널 에뮬레이터 → 로컬 셸

키보드 → MobaXterm → SSH → 원격 서버의 sshd → 원격 셸
```

터미널의 표준 입력·표준 출력은 프로그램과 이어질 수 있고, 프로그램이 출력한 ANSI/VT 제어 시퀀스를 터미널이 해석하여 색상과 커서 위치를 표현한다.

## 4. 관련 문서와 출처

- [[Shell|셸]], [[Linux]], [[SSH]]

[^xterm]: [XTerm official site](https://www.invisible-island.net/xterm/)
[^moba]: [MobaXterm Documentation](https://mobaxterm.mobatek.net/documentation.html)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
