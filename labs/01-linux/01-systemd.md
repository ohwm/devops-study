# Lab 01: systemd와 로그

- 날짜: 2026-10-21
- 환경: Mac(iTerm2) -> VM(Ubuntu Server 24.04 ARM64, `ssh devbox`)
- 목표: 서비스 상태와 로그를 확인하고, 비밀번호 로그인이 막혔는지 로그로 검증한다.

서비스가 죽었을 때 가장 먼저 보는 두 가지 -> `systemctl`, `journalctl`

## 실습 1. 서비스 상태 보기 (VM)

```bash
systemctl status ssh
systemctl list-units --type=service --state=running
```

- `status`: 서비스 하나의 상태(active/inactive), PID, 최근 로그
- `list-units --type=service --state=running`: 실행 중인 서비스 목록 (`q`로 나가기)

Q. 실행 중인 서비스 중 지난 세션에 만났던 서비스 2개는?
A.

## 실습 2. 로그 실시간으로 보기

VM 창:

```bash
sudo journalctl -u ssh -f
```

Mac 새 창:

```bash
ssh devbox
```

- `-u ssh`: ssh 서비스(unit)의 로그만
- `-f`: follow, 새 로그를 실시간 출력 (`Ctrl + C`로 종료)

결과 (VM 로그):

```
Oct 21 10:36:00 ohwmdev sshd-session[1575]: Accepted publickey for ohwm from 192.168.64.1 port 54759 ssh2: ED25519 SHA256:9WMqS/9GWYaAxIT2YWo5FcR/yFdcCeMF5LKCzX8d89A
```

| 부분 | 의미 |
|---|---|
| `Accepted publickey` | 키 인증으로 로그인 성공 |
| `for ohwm` | 로그인한 계정 |
| `from 192.168.64.1` | 접속한 곳 = Mac (VM은 192.168.64.2) |
| `port 54759` | Mac 쪽 임시 포트(ephemeral port) |
| `ED25519 SHA256:...` | 사용한 클라이언트(Mac) 키의 fingerprint |

## 실습 3. 비밀번호 로그인 차단 확인

`journalctl -f` 창은 켜둔 채로, Mac 다른 창:

```bash
ssh -o PubkeyAuthentication=no devbox
```

결과 (Mac):

```
ohwm@192.168.64.2: Permission denied (publickey).
```

결과 (VM 로그):

```
Oct 21 10:37:48 ohwmdev sshd-session[1658]: Connection closed by authenticating user ohwm 192.168.64.1 port 54760 [preauth]
```

- `(publickey)`: 서버가 publickey 방식만 받음 -> 비밀번호 로그인 꺼짐 확인
- `[preauth]`: 인증이 끝나기 전에 연결이 끊김

## 참고. 지난 로그 검색

```bash
sudo journalctl -u ssh --since "1 hour ago"
sudo journalctl -u ssh --since today | grep Accepted
sudo journalctl -p err -b
```

- `--since`: 시작 시점 지정
- `-p err`: 에러 이상만
- `-b`: 이번 부팅 이후

## 관련

- 비밀번호 로그인 끄기 설정: `/etc/ssh/sshd_config.d/00-no-password.conf`
- 최종 적용 값 확인: `sudo sshd -T | grep -i passwordauthentication`
- TIL: `til/2026-10-21.md`
