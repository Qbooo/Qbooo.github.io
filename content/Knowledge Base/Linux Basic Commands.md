---
title: Linux Basic Commands
type: knowledge
domain: Compute
tags: [Linux, CLI, OJT]
aliases: [리눅스 기본 명령어, Linux 명령어]
parent: Linux
status: growing
lang: ko
featured: false
created: 2026-11-09
updated: 2026-11-09
---

## 1. 개요

Linux 기본 명령어는 셸에서 현재 위치와 파일·프로세스·시스템 상태를 조회하고 필요한 변경을 요청하는 내장 명령 또는 사용자 공간 프로그램이다.

명령어는 [[Kernel|커널]]이 이름을 직접 이해하는 특별한 단어가 아니다. [[Shell|셸]]이 명령줄을 해석한 뒤 `cd` 같은 내장 명령을 직접 처리하거나 `ls`, `grep` 같은 외부 프로그램을 찾아 실행한다. 외부 프로그램은 시스템 호출을 통해 파일·프로세스·장치 정보를 커널에 요청한다.

```text
명령 [옵션] [인수·대상]

ls   -la    /etc
│     │       └─ 조회할 경로
│     └───────── 출력 방식을 바꾸는 옵션
└─────────────── 실행할 명령
```

모든 옵션을 외우기보다 명령의 목적과 입력·출력 구조를 알고 `--help`와 `man`으로 세부 사용법을 확인하는 것이 중요하다.[^coreutils]

## 2. 위치와 디렉터리 이동

| 명령 | 역할 | 예시 |
|---|---|---|
| `pwd` | 현재 작업 디렉터리의 절대 경로 출력 | `pwd` |
| `ls` | 디렉터리 항목과 파일 정보 조회 | `ls -la /etc` |
| `cd` | 현재 셸의 작업 디렉터리 변경 | `cd /var/log` |

`cd`는 현재 셸의 위치를 바꿔야 하므로 셸 내장 명령이다. `ls`는 일반적으로 `/usr/bin/ls`에 있는 외부 프로그램이다.

```bash
cd /var/log    # 절대 경로로 이동
cd ..          # 상위 디렉터리로 이동
cd ~           # 현재 사용자의 홈으로 이동
cd -           # 직전 작업 디렉터리로 이동
```

경로에서 `.`은 현재 디렉터리, `..`은 상위 디렉터리, `~`는 현재 사용자의 홈 디렉터리를 의미한다. `~` 확장은 셸이 처리한다.

### `ls`의 대표 옵션

```bash
ls             # 현재 디렉터리 이름 목록
ls -l          # 권한·소유자·크기·시간 등 자세한 정보
ls -a          # 점으로 시작하는 숨김 항목 포함
ls -lh         # 크기를 사람이 읽기 쉬운 단위로 표시
ls -la /etc    # 옵션을 결합하고 대상 경로 지정
```

`-la`는 `-l -a`와 같은 뜻이다. 옵션의 의미는 Bash가 아니라 `ls` 프로그램이 해석한다.

## 3. 파일과 디렉터리 생성·복사·이동·삭제

| 명령 | 역할 | 예시 |
|---|---|---|
| `mkdir` | 디렉터리 생성 | `mkdir project` |
| `touch` | 파일이 없으면 빈 파일 생성, 있으면 시간 정보 갱신 | `touch note.txt` |
| `cp` | 파일·디렉터리 복사 | `cp report.txt backup/` |
| `mv` | 파일·디렉터리 이동 또는 이름 변경 | `mv old.txt new.txt` |
| `rm` | 파일 또는 지정한 디렉터리 항목 삭제 | `rm note.txt` |
| `rmdir` | 비어 있는 디렉터리 삭제 | `rmdir empty-dir` |
| `ln` | 하드 링크 또는 심볼릭 링크 생성 | `ln -s target link` |

### 주요 사용 형태

```bash
mkdir -p project/logs       # 중간 디렉터리까지 생성
cp -r source-dir backup/    # 디렉터리 재귀 복사
mv report.txt archive/      # 파일 이동
mv old-name new-name        # 이름 변경
rm file.txt                 # 파일 삭제
rm -r old-dir               # 디렉터리와 내부 항목을 재귀 삭제
```

`rm`은 일반적으로 휴지통을 거치지 않는다. 특히 `-r`, `-f`, 와일드카드와 관리자 권한을 함께 사용할 때는 실행 전에 현재 위치와 대상 경로를 확인해야 한다. 복사·이동도 같은 이름의 대상 파일을 덮어쓸 수 있으므로 중요한 작업에서는 대상 존재 여부를 먼저 확인한다.

`touch`의 본래 기능은 파일의 접근·수정 시간을 갱신하는 것이다. 파일이 없을 때 기본 동작으로 빈 파일이 생성되기 때문에 연습과 간단한 파일 준비에 자주 사용된다.

## 4. 파일 내용 확인

| 명령 | 역할 | 적합한 상황 |
|---|---|---|
| `cat` | 파일 내용을 이어서 표준 출력 | 짧은 파일 전체 확인 |
| `less` | 긴 내용을 페이지 단위로 탐색 | 긴 로그·설정 읽기 |
| `head` | 앞부분 출력 | 헤더·초기 행 확인 |
| `tail` | 뒷부분 출력 | 최신 로그 확인 |
| `wc` | 행·단어·바이트 수 계산 | 결과 개수 파악 |

```bash
cat /etc/hosts
less /var/log/app.log
head -n 20 file.txt
tail -n 50 app.log
tail -f app.log          # 파일에 추가되는 내용을 계속 확인
wc -l file.txt           # 행 수 출력
```

큰 파일을 `cat`으로 한꺼번에 출력하면 필요한 부분을 찾기 어렵다. 긴 파일은 `less`, 계속 추가되는 로그는 `tail -f`가 적합하다. 단, 로그 위치와 열람 권한은 서비스와 배포판 구성에 따라 다르다.

## 5. 검색과 결과 가공

| 명령 | 역할 | 예시 |
|---|---|---|
| `grep` | 입력이나 파일에서 패턴과 일치하는 행 검색 | `grep "error" app.log` |
| `find` | 디렉터리 트리에서 조건에 맞는 파일 검색 | `find /etc -name '*.conf'` |
| `sort` | 입력받은 행 정렬 | `sort names.txt` |
| `uniq` | 연속해서 중복되는 행을 묶거나 제거 | `sort names.txt \| uniq` |
| `cut` | 구분자나 문자 위치를 기준으로 필드 선택 | `cut -d: -f1 /etc/passwd` |

```bash
grep -i "error" app.log            # 대소문자를 구분하지 않고 검색
grep -n "error" app.log            # 일치한 행 번호 표시
grep -r "Listen" /etc/nginx        # 디렉터리 아래를 재귀 검색
find /var/log -type f -name '*.log' # 이름과 유형으로 파일 검색
```

`grep`은 기본적으로 일치한 문자열만이 아니라 패턴을 포함한 행 전체를 출력한다. `find`는 파일 이름만 찾는 명령이 아니라 경로·유형·크기·시간 등 여러 조건으로 디렉터리 트리를 탐색하는 프로그램이다.

## 6. 표준 입출력·파이프·리다이렉션

프로세스는 기본적으로 표준 입력(stdin), 표준 출력(stdout), 표준 오류(stderr)를 사용한다. `|`, `>`, `>>`는 프로그램 옵션이 아니라 Bash가 처리하는 문법이다.[^bash]

```bash
grep "error" /var/log/app.log | sort > errors.txt
```

```text
로그 파일
   ↓
grep이 error가 포함된 행 출력
   ↓ 표준 출력
   | 파이프
   ↓ 표준 입력
sort가 행 단위 정렬
   ↓ 표준 출력
errors.txt에 덮어쓰기
```

| 문법 | 역할 |
|---|---|
| `A \| B` | A의 표준 출력을 B의 표준 입력에 연결 |
| `A > file` | 표준 출력으로 파일을 새로 쓰거나 덮어씀 |
| `A >> file` | 표준 출력을 파일 끝에 추가 |
| `A 2> file` | 표준 오류를 파일에 기록 |
| `A && B` | A가 성공한 경우에만 B 실행 |
| `A || B` | A가 실패한 경우에만 B 실행 |

파이프는 단순한 ‘그리고’가 아니라 두 프로그램의 데이터 통로를 연결한다. 성공했을 때 다음 명령을 실행하는 논리적인 ‘그리고’에 가까운 문법은 `&&`이다. 자세한 원리는 [[CLI]]에서 다룬다.

## 7. 사용자와 권한 확인

| 명령 | 역할 | 예시 |
|---|---|---|
| `whoami` | 현재 유효 사용자 이름 확인 | `whoami` |
| `id` | UID·GID와 소속 그룹 확인 | `id` |
| `groups` | 사용자의 그룹 목록 확인 | `groups` |
| `ls -l` | 파일 유형·권한·소유자·그룹 확인 | `ls -l report.txt` |
| `chmod` | 권한 변경 | `chmod 640 report.txt` |
| `chown` | 소유자·그룹 변경 | `chown alice:developers report.txt` |

권한 변경 명령은 파일을 읽는 명령보다 영향이 크다. 숫자 권한과 `rwx`, 파일과 디렉터리에서 권한의 의미, root와 `sudo`의 차이는 계정·권한 문서에서 별도로 설명한다.

## 8. 프로세스와 시스템 상태 확인

| 명령 | 역할 | 예시 |
|---|---|---|
| `ps` | 특정 시점의 프로세스 목록 확인 | `ps aux` |
| `top` | 프로세스와 CPU·메모리 상태를 계속 갱신 | `top` |
| `kill` | PID로 프로세스에 신호 전송 | `kill 1234` |
| `uname` | 커널·시스템 정보 출력 | `uname -a` |
| `hostname` | 시스템 호스트 이름 확인 | `hostname` |
| `free` | 메모리 사용량 확인 | `free -h` |
| `uptime` | 실행 시간과 부하 평균 확인 | `uptime` |

`kill`은 이름과 달리 항상 강제 종료만 하는 명령이 아니다. 기본적으로 종료를 요청하는 `SIGTERM` 신호를 보내며, 프로세스가 정리할 기회를 줄 수 있다. `kill -9`는 `SIGKILL`을 보내 커널이 즉시 종료시키므로 일반 종료가 되지 않는 경우에만 신중하게 사용한다.

## 9. 디스크·파일시스템 확인

| 명령 | 역할 | 핵심 구분 |
|---|---|---|
| `df -hT` | 마운트된 파일시스템의 전체·사용·여유 공간 | 파일시스템 관점 |
| `du -sh PATH` | 특정 디렉터리·파일이 차지하는 용량 | 경로 관점 |
| `lsblk` | 블록 장치·파티션·마운트 지점 | 장치 관점 |
| `findmnt` | 마운트 관계와 옵션 | 마운트 관점 |

```bash
df -hT
du -sh /var/log
lsblk
findmnt /home
```

`df`의 사용량과 `du`로 더한 파일 크기가 항상 정확히 같지는 않다. 삭제했지만 프로세스가 아직 열고 있는 파일, 파일시스템 예약 공간, 메타데이터와 스파스 파일 등이 차이를 만들 수 있다. 디렉터리와 마운트의 관계는 [[Linux Directory Structure|리눅스 디렉터리 구조]]에서 설명한다.

## 10. 명령의 정체와 도움말 확인

```bash
type cd             # 셸 내장 명령인지 확인
type -a ls          # 같은 이름으로 찾을 수 있는 후보 모두 표시
command -v grep     # 셸이 실행할 명령 확인
ls --help           # 간단한 도움말
man ls              # 매뉴얼 페이지
```

```text
cd is a shell builtin
ls is aliased to 'ls --color=auto'
ls is /usr/bin/ls
```

이는 `cd`가 셸 내부 기능이고, `ls`에는 별칭이 적용될 수 있으며, 실제 외부 프로그램도 존재한다는 뜻이다.

## 11. 안전하게 명령을 사용하는 기준

- 먼저 `pwd`, `ls`, `find`, `findmnt` 같은 조회 명령으로 현재 위치와 대상을 확인한다.
- 상대 경로가 모호하면 절대 경로를 사용하고 공백이 있는 경로는 따옴표로 묶는다.
- `rm -r`, `chmod -R`, `chown -R`처럼 하위 항목 전체에 영향을 주는 옵션은 대상 범위를 다시 확인한다.
- 오류를 없애려고 바로 `sudo`를 붙이지 말고 권한 부족의 원인과 필요한 변경 범위를 확인한다.
- 명령 예시는 환경에 그대로 복사하기 전에 옵션과 대상 경로를 `man`·`--help`로 검토한다.

## 12. 관련 문서

- [[Linux]], [[Linux Structure|리눅스 구조]], [[Linux Directory Structure|리눅스 디렉터리 구조]]
- [[CLI]], [[Shell|셸]], [[Terminal|터미널]], [[Shell Script|셸 스크립트]]
- [[Program and Process|프로그램과 프로세스]], [[Kernel|커널]], [[Disk]]
- [[Linux Accounts and Permissions|리눅스 계정과 권한]]
- [[SSH]]

## 13. 출처

[^coreutils]: [GNU Project — Coreutils Manual](https://www.gnu.org/software/coreutils/manual/coreutils.html)
[^bash]: [GNU Project — Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html)

- [GNU Project — GNU Grep Manual](https://www.gnu.org/software/grep/manual/grep.html)
- [GNU Project — Findutils Manual](https://www.gnu.org/software/findutils/manual/html_mono/find.html)
- [Linux man-pages project](https://man7.org/linux/man-pages/)

## 둘러보기

[[Knowledge Base/index|전체 개념 목록]] · [[Compute 분류|Compute]]
