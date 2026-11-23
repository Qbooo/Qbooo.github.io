---
title: Knowledge Base
type: map
---
## 목차

- [[#소개|소개]]
- [[#위키 안내|위키 안내]]
- [[#질문과 해답|질문과 해답]]
- [[#인프라 기본 개념|인프라 기본 개념]]
- [[#서버와 하드웨어|서버와 하드웨어]]
- [[#가상화 기술|가상화 기술]]
- [[#운영체제와 Linux|운영체제와 Linux]]
- [[#Linux 시스템 운영|Linux 시스템 운영]]
- [[#Linux 디스크 관리|Linux 디스크 관리]]
- [[#스토리지 기초|스토리지 기초]]
- [[#네트워크 기초|네트워크 기초]]
- [[#네트워크 보안|네트워크 보안]]
- [[#분류|분류]]

## 소개
모르는 개념을 만날 때마다 내 말로 정리하는 개념 사전입니다. 문서 상태는 `seed`(한 줄 정의), `growing`(채우는 중), `evergreen`(다듬어진 문서)로 표시합니다.

## 위키 안내
- [[비오위키]]

## 질문과 해답

개념의 정의를 넘어 여러 기술을 어떻게 조합하고 선택해야 하는지 궁금하다면 [[Questions/index|Questions]]에서 질문별 해답과 적용 판단을 확인할 수 있습니다.

## 인프라 기본 개념

서버·네트워크·스토리지의 역할을 먼저 구별한 뒤 하나의 웹 요청이 각 계층을 지나는 흐름으로 연결해 읽을 수 있습니다.

- 기반 자원: [[Server|서버]] · [[Networking|네트워크]] · [[Storage|스토리지]]
- 전체 구조: [[3-Tier Architecture|3-Tier 아키텍처]]
- 요청 처리: [[Web Server|WEB]] → [[Web Application Server|WAS]] → [[Database|DB]]

## 서버와 하드웨어

서버의 역할을 이해한 뒤 실제 장비 안의 부품이 프로그램 실행·저장·통신을 어떻게 나누어 맡는지 연결해 읽을 수 있습니다.

- 연산·실행 공간: [[CPU]] · [[Memory|메모리]]
- 저장·통신: [[Disk|디스크]] · [[NIC]]
- 실제 배치: [[Physical Server Configuration|물리 서버 구성]]

## 가상화 기술

물리 서버의 자원이 하이퍼바이저를 거쳐 VM의 가상 하드웨어로 제공되는 흐름과, 물리 장비를 직접 사용하는 베어메탈 환경의 차이를 연결해 읽을 수 있습니다.

- 전체 원리: [[Virtualization|가상화]]
- 실행·중재 계층: [[Hypervisor|하이퍼바이저]]
- 가상 실행 환경: [[Virtual Machine|가상 머신(VM)]]
- 물리 실행 환경: [[Bare Metal|베어메탈]]

## 운영체제와 Linux

하드웨어 위에서 Linux 커널이 자원을 관리하고, 사용자 공간의 셸과 프로그램이 시스템 호출을 통해 커널 기능을 사용하는 흐름으로 읽을 수 있습니다.

- 전체 개념: [[Linux|리눅스]]
- 시스템 계층과 요청 흐름: [[Linux Structure|리눅스 구조]]
- 파일시스템의 경로와 역할: [[Linux Directory Structure|리눅스 디렉터리 구조]]
- 명령줄 입력과 프로그램 연결: [[CLI]]
- 명령 사용과 상태 확인: [[Linux Basic Commands|리눅스 기본 명령어]]
- 사용자·그룹과 자원 접근: [[Linux Accounts and Permissions|리눅스 계정과 권한]]
- 암호화된 원격 접속: [[SSH]]
- 부팅 계층: [[Firmware|펌웨어]] → [[BIOS and UEFI|BIOS·UEFI]] → [[Bootloader|부트로더]] → [[Kernel|커널]]
- 프로그램 실행: [[Compilation|컴파일]] → [[Program and Process|프로그램과 프로세스]]
- 명령 입출력: [[Terminal|터미널]] → [[Shell|셸]]

## Linux 시스템 운영

계정의 실행 권한에서 프로세스와 서비스의 수명, 로그와 자원 지표까지 연결하여 Linux 서버의 현재 상태를 해석할 수 있습니다.

- 실행 신원: [[Linux Accounts and Permissions|사용자·그룹 관리]]
- 실행 단위와 상태: [[Program and Process|Process]]
- 장기 서비스: [[Daemon]]
- 사건 기록: [[Linux Logging|Log]]
- 자원 관찰: [[CPU Monitoring|CPU 모니터링]] · [[Memory Monitoring|Memory 모니터링]]

## Linux 디스크 관리

커널이 인식한 블록 장치가 파티션과 파일시스템을 거쳐 하나의 디렉터리 경로로 연결되는 순서로 읽을 수 있습니다.

- 저장 장치와 블록 I/O: [[Disk|디스크]]
- 디스크 구획과 파티션 테이블: [[Partition|파티션]]
- 파일·디렉터리 관리 구조: [[Filesystem|파일시스템]] · [[Inode|inode]]
- 경로 연결과 영구 설정: [[Mount|마운트]] · [[Fstab|fstab]]
- 저장 공간 확인: [[Disk Capacity Check|디스크 용량 확인]]
- 부팅 디스크의 연결: [[BIOS and UEFI|BIOS·UEFI]] → [[Bootloader|부트로더]] → 운영체제 파티션

## 스토리지 기초

물리 매체가 운영체제의 블록 계층을 거쳐 파일로 보이는 과정과, 같은 저장 공간을 네트워크에서 블록·파일·오브젝트 가운데 어떤 인터페이스로 제공하는지 연결해 읽을 수 있습니다.

- 저장 계층: [[Disk|디스크]] → [[Linux Block Layer]] → [[Filesystem|파일시스템]] → [[VFS]]
- 디스크 구성: [[RAID]]
- 접근 방식: [[Block Storage|블록]] · [[File Storage|파일]] · [[Object Storage|오브젝트]]
- 원격 블록: [[SAN]] · [[iSCSI]]
- 원격 파일: [[NAS]] · [[NFS]] · [[SMB]]
- 오브젝트 API: [[S3 API]] · [[NGOS]] · [[ZIOS]]

## 네트워크 기초

한 LAN 안에서 프레임이 전달되는 과정부터 여러 사업자의 네트워크가 인터넷으로 연결되는 과정까지 순서대로 읽을 수 있습니다.

- 계층 모델: [[OSI and TCP-IP Models|OSI 7계층과 TCP/IP 4계층]]
- 통신 단위와 헤더: [[Encapsulation|캡슐화]]
- 주소와 전달 방식: [[IP Address and Subnet|IP 주소와 서브넷]] · [[Public and Private IP|공인 IP와 사설 IP]] · [[Unicast Broadcast Multicast Anycast|유니캐스트·브로드캐스트·멀티캐스트·애니캐스트]]
- 로컬 전달과 출구: [[ARP]] · [[Default Gateway|기본 게이트웨이]]
- 논리적 L2 분리: [[VLAN]]
- 로컬 네트워크: [[NIC]] → [[Switch|스위치]]
- 네트워크 간 전달: [[Router|라우터]] → [[IGP]] → [[OSPF]] · [[IS-IS]]
- 인터넷 연결과 경로 교환: [[Internet Routing|인터넷의 통신 구조]] → [[BGP]] → [[IXP Peering and Transit|IXP·피어링·트랜짓]]
- 사업자 핵심망: [[Backbone Network|백본 네트워크]]
- 주요 네트워크 서비스: [[DNS]] · [[NAT]] · [[Load Balancing|로드 밸런싱]]
- 서비스 분산 계층: [[L4 and L7 Load Balancing|L4·L7 로드 밸런싱]]
- 전체 개요: [[Networking|네트워크]]

## 네트워크 보안

통신 경계의 허용 정책부터 웹과 원격 접속에서 상대를 확인하고 데이터를 암호화하는 흐름으로 읽을 수 있습니다.

- 네트워크 허용 정책: [[Firewall|방화벽]] · [[Security Group]]
- 웹 보안 통신: [[HTTP and HTTPS|HTTP와 HTTPS]] → [[TLS]]
- 신뢰 기반: [[Digital Signature and Certificate|전자서명과 인증서]]
- 원격 접속: [[SSH]]

## 분류
- [[Compute 분류|Compute]]
- [[Storage 분류|Storage]]
- [[Networking 분류|Networking]]
- [[IaC 분류|IaC]]
- [[Database 분류|Database]]
- [[Container 분류|Container]]
- [[Cloud 분류|Cloud]]
- [[Security 분류|Security]]
- [[Business 분류|Business]]
