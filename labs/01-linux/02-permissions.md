# Lab 02: 사용자별 권한이 다른 공유 폴더

- 날짜: 2026-10-22
- 환경: Mac(iTerm2) -> VM(Ubuntu Server 24.04 ARM64, `ssh devbox`)
- 목표: dev 그룹(alice, bob)만 읽기·쓰기 가능하고 charlie는 접근할 수 없는 `/srv/shared` 만들기
- 도전: alice가 만든 파일을 bob도 수정 가능하게 (특수 권한)

```
alice ─┐
       ├─ dev 그룹 ──→ /srv/shared  (읽기·쓰기 O)
bob   ─┘
charlie ─────────────→ /srv/shared  (접근 X)
```

## Step 1. 사용자 만들기

```bash
sudo useradd -m -s /bin/bash alice
sudo useradd -m -s /bin/bash bob
sudo useradd -m -s /bin/bash charlie
sudo passwd alice    # bob, charlie도 같은 방법
```

- `-m`: 홈 디렉터리 생성 (값을 받지 않는 flag)
- `-s /bin/bash`: 로그인 셸 지정

결과:

```
alice:x:1001:1001::/home/alice:/bin/bash
bob:x:1002:1002::/home/bob:/bin/bash
charlie:x:1003:1003::/home/charlie:/bin/bash

drwxr-x---  2 alice   alice   4096 ... alice
drwxr-x---  2 bob     bob     4096 ... bob
drwxr-x---  2 charlie charlie 4096 ... charlie
```

- UID는 ohwm(1000) 다음부터 1001, 1002, 1003
- 사용자마다 같은 이름의 그룹이 생김 = UPG(User Private Group)
- 홈 폴더 750: 다른 사용자는 남의 홈에 들어갈 수 없음

### 겪은 문제: 홈 폴더가 이미 있음

```
useradd: warning: the home directory /home/alice already exists.
useradd: Not copying any file from skel directory into it.
```

- 원인: 옵션 없이 만든 alice를 `userdel`로 지울 때 홈 폴더가 남아 있었음
- `/etc/skel`의 기본 파일(.bashrc, .profile)이 복사되지 않음
- 해결: `sudo userdel -r alice`(홈 폴더까지 삭제) 후 다시 생성

### sudo가 필요한 이유

```
$ userdel -r alice
userdel: Permission denied.
userdel: cannot lock /etc/passwd; try again later.
```

- 계정 관리 명령은 root만 쓸 수 있는 `/etc/passwd`, `/etc/shadow`, `/etc/group`을 수정한다
- 수정 전에 lock 파일을 만드는데, `/etc`에 파일을 만들 수 있는 건 root뿐

### 관련 파일 권한

```
-rw-r--r-- 1 root root   ... /etc/passwd
-rw-r----- 1 root shadow ... /etc/shadow
```

- passwd: 모두 읽기 가능 (UID -> 이름 변환에 필요), 쓰기는 root만 (UID를 0으로 바꾸는 것 방지)
- shadow: 비밀번호 해시 저장, others 권한 없음
- passwd 두 번째 칸의 `x` = 비밀번호는 shadow에 있다는 뜻

## Step 2. dev 그룹 만들고 alice, bob 넣기

```bash
sudo groupadd dev
sudo usermod -aG dev alice
sudo usermod -aG dev bob
```

결과:

```
dev:x:1004:alice,bob
uid=1001(alice) gid=1001(alice) groups=1001(alice),1004(dev)
uid=1003(charlie) gid=1003(charlie) groups=1003(charlie)
```

### 겪은 문제: `-a` 없이 `-G`

- `-G`는 보조 그룹을 추가가 아니라 교체한다
- 실험: `usermod -G test alice` -> dev 멤버 목록에서 alice가 사라짐
- ohwm에게 하면 sudo 그룹에서 빠져 관리자 권한을 잃는다
- `-a`는 혼자 쓰지 못하고 `-G`와 함께 써야 한다 (`-a -G` = `-aG`)
- GID 숫자(1004) 대신 그룹 이름(dev)을 쓰는 게 좋다

### 기본 그룹 vs 보조 그룹

| 종류 | 기록 위치 |
|---|---|
| 기본 그룹(primary) | `/etc/passwd` 4번째 칸(GID) |
| 보조 그룹(supplementary) | `/etc/group` 마지막 칸(멤버 목록) |

## Step 3. /srv/shared 만들고 권한 설정

```bash
sudo mkdir /srv/shared
sudo chgrp dev /srv/shared
sudo chmod 770 /srv/shared
```

결과:

```
drwxrwx--- 2 root dev 4096 ... /srv/shared
```

### 폴더에서 r, w, x의 의미

폴더 = "파일 이름 -> inode 번호" 표

| 권한 | 폴더일 때 |
|---|---|
| r | 표의 이름 칸을 읽는다 (목록) |
| w | 표에 줄을 추가·삭제한다 (단, x 필요) |
| x | 표를 통과해 inode까지 간다 (cd, 파일 접근) |

- 파일 삭제 권한은 파일이 아니라 폴더의 w+x에서 온다
- 755는 dev에게 쓰기가 없고 charlie에게 r-x가 열려서 목표와 맞지 않음
- root는 권한 검사를 건너뛴다 (`sudo ls /srv/shared`는 성공)

## Step 4. 사용자별 검증

```bash
su - alice    # - : 새로 로그인한 것처럼 환경과 그룹 정보를 불러옴
```

| # | 누구로 | 할 일 | 결과 |
|---|---|---|---|
| 1 | alice | `/srv/shared`에 파일 넣기 | 성공 |
| 2 | bob | 목록 보기 | 성공 |
| 3 | bob | `alice.txt` 수정 | `Permission denied` |
| 4 | charlie | `ls`, `cd` | `Permission denied` |

### 3번이 실패한 이유

```
-rw-rw-r--  alice  alice   alice.txt
```

- 커널은 소유자 -> 그룹 -> 그 외 순서로 한 자리만 골라 검사
- bob은 소유자(alice)도 아니고 그룹(alice)의 멤버도 아님 -> 그 외(`r--`) 적용
- 권한 숫자가 아니라 소유 그룹이 문제

### 겪은 문제: 상대 경로

- `su - bob` 직후 위치는 `/home/bob`
- `echo "bob" >> alice.txt` -> `/home/bob/alice.txt`를 새로 만들어 성공한 것처럼 보임
- 검증은 절대 경로를 쓰거나 `cd` 후 `pwd`로 확인

## 도전. setgid로 그룹 상속

```bash
sudo chown -R :dev /srv/shared    # 기존 파일 그룹 변경 (일회성)
sudo chmod 2770 /srv/shared       # setgid
```

결과:

```
drwxrws---  root  dev   .
-rw-rw-r--  alice dev   alice.txt    <- chown -R로 바꾼 기존 파일
-rw-rw-r--  alice alice alice2.txt   <- setgid 전에 만든 파일
-rw-rw-r--  alice dev   alice3.txt   <- setgid 후에 만든 파일
```

bob이 `alice3.txt` 수정 성공.

- `chown -R`은 지금 있는 파일만 바꾼다. 새 파일은 다시 만든 사람의 기본 그룹
- setgid 폴더: 새 파일은 만든 사람의 그룹 대신 폴더의 그룹을 물려받음
- 하위 폴더도 setgid를 물려받음
- `mv`는 같은 디스크 안에서 이름표만 옮기므로 원래 그룹이 유지됨 -> 검증은 폴더 안에서 직접 생성
- 새 파일이 그룹 쓰기 가능했던 건 umask 002 덕분 (umask 022였다면 setgid만으로는 부족)

### 특수 권한 3종

| 이름 | 숫자 | 표시 | 효과 |
|---|---|---|---|
| setuid | 4 | 소유자 x 자리 `s` | 파일 소유자 권한으로 실행 (예: `/usr/bin/passwd`) |
| setgid | 2 | 그룹 x 자리 `s` | 폴더에 걸면 새 파일이 폴더의 그룹을 물려받음 |
| sticky | 1 | others x 자리 `t` | 폴더에 걸면 파일 소유자만 삭제 가능 (예: `/tmp`) |

## 관련

- 보안 점검: `find / -perm -4000 -type f 2>/dev/null` (setuid 파일 목록)
- TIL: `til/2026-10-22.md`
