# Lab 03: 프로세스와 시그널

- 날짜: 2026-10-25
- 환경: Mac(iTerm2) -> VM(Ubuntu Server 24.04 ARM64, `ssh devbox`)
- 목표: 프로세스의 부모-자식 관계를 확인하고, 시그널별로 종료 결과가 어떻게 다른지 비교하기

```
bash (PID 1444)
  │  fork → exec
  ▼
sleep (PID 2735, PPID 1444)  ← kill -<시그널> 로 종료
```

## 개념 정리

### 시그널

| 시그널 | 번호 | 언제 |
|---|---|---|
| SIGHUP | 1 | 터미널이 끊길 때, 설정 다시 읽기 요청 |
| SIGINT | 2 | `Ctrl+C` |
| SIGKILL | 9 | `kill -9` |
| SIGTERM | 15 | `kill`의 기본값 |
| SIGSTOP / SIGCONT | 19 / 18 | 일시정지 / 재개 |

### 명령어

```bash
ps -ef                            # 모든 프로세스, PID/PPID가 보임
ps aux                            # 모든 프로세스, CPU%/MEM%/STAT이 보임
ps -o pid,ppid,stat,cmd -p 1234   # 원하는 칸만 골라 보기
pgrep -a sleep                    # 이름으로 PID 찾기
top                               # 실시간으로 보기 (q로 종료, 숫자 1 = CPU별)
kill -15 1234                     # SIGTERM 보내기 (= kill 1234)
kill -l                           # 시그널 목록 보기
echo $$                           # 지금 쓰고 있는 셸의 PID
```

### STAT 칸

| 값 | 의미 |
|---|---|
| R | 실행 중 |
| S | 잠든 상태(대기) |
| T | 정지됨 |
| Z | 좀비 |

## Step 1. 부모-자식 관계 확인

```
ohwm@ohwmdev:~$ cat /proc/2735/status | grep -e State -e PPid -e Uid
State:	S (sleeping)
PPid:	1444
Uid:	1000	1000	1000	1000
ohwm@ohwmdev:~$ echo $$
1444
```

### Uid가 4개인 이유

| 순서 | 이름 | 의미 |
|---|---|---|
| 1 | Real UID | 실제로 실행한 UID |
| 2 | Effective UID | 커널이 권한 검사하는 UID |
| 3 | Saved UID | 보관용 값 |
| 4 | Filesystem UID | 파일 접근 검사할 때 쓰는 값 (2번과 보통 같음) |

평범한 프로그램은 4개가 같지만, setuid 프로그램을 실행할 때 값이 서로 달라짐.

```
passwd 실행
-> 커널이 exec할 때 s 비트를 보고 Effective UID만 0(root)로 바꿈
-> Uid: 1000 0 0 0
```

- 권한 검사하는 UID가 0(root)여야 비밀번호를 바꿀 수 있고,
- Real UID가 남아 있어야 ohwm의 비밀번호만 바꾸게 할 수 있기 때문에 이렇게 된다!

## Step 2. SIGSTOP / SIGCONT

SIGSTOP을 보냈을 때, S->T 가 되고 SIGCONT가 올 때까지 멈춰 있는 상태가 된다.
정지 상태에서 kill -15를 보내면 커널은 pending signal 목록에 표시만 해두고 시그널은 프로세스가 다시 cpu를 받아 실행될 때 처리돤다.
따라서 cpu 할당이 되지 않아 종료되지 않는다.
SIGCONT를 보내야 T->S가 되고 cpu할당을 받을 수 있게 되어 그제서야 종료된다.

## Step 3. 종료 방법별 `$?` 비교

foreground에서 `sleep 100`을 실행하고 각 방법으로 kill한 뒤 `echo $?`로 결과 보기

| 종료 방법 | 시그널 | `$?` |
|---|---|---|
| `Ctrl+C` | SIGINT (2) | 130 |
| 다른 터미널에서 `kill -15` | SIGTERM (15) | 143 |
| 다른 터미널에서 `kill -9` | SIGKILL (9) | 137 |

> **`$?` = 128 + 시그널 번호**

## 트러블슈팅

**증상:** `echo $?`가 계속 `0`이 나옴

**원인:**
- `$?`가 `kill`의 종료 코드라 0이 나왔던 것이었다.
- 처음에 `sleep 100 &`로 백그라운드로 만들어서 제대로 된 실습을 못했다.
