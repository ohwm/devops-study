# Lab 04: 패키지 관리 (apt / dpkg)

- 날짜: 2026-10-26
- 환경: Mac(iTerm2) -> VM(Ubuntu Server `resolute` ARM64, `ssh devbox`)
- 목표: `apt update`와 `apt upgrade`의 차이를 확인하고, 패키지가 어디서 와서 어디에 설치되는지 추적하기
- 덤: 실습 중 발생한 VM 시계 장애 해결

```
[저장소 서버] ──HTTP──▶ apt update  ──▶ /var/lib/apt/lists/      (메뉴판)
                        apt install ──▶ /var/cache/apt/archives/ (.deb 다운로드)
                                          │
                                          ▼ dpkg
                                    /usr/bin/... 에 파일 설치
                                    /var/lib/dpkg/status 에 기록 (설치 기록)
```

## 개념 정리

### apt vs dpkg

| 층 | 도구 | 하는 일 |
|---|---|---|
| 위 | apt | 저장소에서 찾기, 의존성 계산, 다운로드 |
| 아래 | dpkg | `.deb`를 풀어서 설치, 기록 |

### update vs upgrade

| 명령 | 비유 | 실제로 하는 일 |
|---|---|---|
| `apt update` | 메뉴판 새로 받기 | 저장소에서 `InRelease`/`Packages`를 받아 GPG 서명 검증 후 `/var/lib/apt/lists/`에 저장. 프로그램은 안 바뀜 |
| `apt upgrade` | 실제로 주문하기 | 새 버전 `.deb` 다운로드 → dpkg로 설치 |

- `update` 후 "N packages can be upgraded"는 `dpkg/status`(설치된 버전)와 `lists`(최신 버전)를 비교한 결과
- `update`는 메뉴판 전체를 받으므로 인자를 받지 않음 (`apt update htop` → 에러). 패키지 하나만 올리려면 `apt install --only-upgrade <패키지>`
- 메뉴판이 오래되면 미러에서 이미 지워진 버전을 요청해 404로 설치 실패, 또는 보안 패치 빠진 옛 버전 설치

### 명령어

```bash
sudo apt update                 # 패키지 목록 갱신
apt list --upgradable           # 업그레이드 가능한 것 보기
apt show htop                   # 패키지 정보
apt-cache policy htop           # Installed / Candidate / 어느 저장소
sudo apt install htop           # 설치
sudo apt remove htop            # 삭제 (설정 파일 남김)
sudo apt purge htop             # 삭제 + 설정 파일까지
sudo apt autoremove             # 안 쓰는 auto 의존성 정리

dpkg -L htop                    # 패키지가 설치한 파일 목록
dpkg -S /usr/bin/ls             # 파일이 어느 패키지 소속인지
which htop / type -a htop       # 실행 파일 위치
```

## Step 1. `apt update`

접속하는 서버:

- `kr.archive.ubuntu.com` (resolute, resolute-updates, resolute-backports)
- `security.ubuntu.com` (resolute-security)
- ARM64인데도 `ports.ubuntu.com`이 아니라 `kr.archive.ubuntu.com`에서 arm64 목록을 받고 있음

```
ohwm@ohwmdev:~$ sudo apt update
Hit:1 http://kr.archive.ubuntu.com/ubuntu resolute InRelease
Hit:2 http://security.ubuntu.com/ubuntu resolute-security InRelease
Hit:3 http://kr.archive.ubuntu.com/ubuntu resolute-updates InRelease
Hit:4 http://kr.archive.ubuntu.com/ubuntu resolute-backports InRelease
39 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

- `Hit`: 서버 목록이 내 것과 같아서 다시 안 받음
- `Get`: 새로 받음
- update만 했으므로 39개는 아직 업그레이드되지 않음

## Step 2. `htop` 추적

### Installed vs Candidate

```
ohwm@ohwmdev:~$ apt-cache policy htop
htop:
  Installed: 3.4.1-5build2
  Candidate: 3.4.1-5build2
  Version table:
 *** 3.4.1-5build2 500
        500 http://kr.archive.ubuntu.com/ubuntu resolute/main arm64 Packages
        100 /var/lib/dpkg/status
```

| 줄 | 출처 |
|---|---|
| `Installed:` | `/var/lib/dpkg/status` (실제로 설치된 버전) |
| `Candidate:` | `/var/lib/apt/lists/` (설치하면 깔릴 버전) |
| `***` | 지금 설치된 버전 |
| 500, 100 | 우선순위(priority) |

- 39개 업그레이드 대상 = 설치는 돼 있지만 Installed(옛 버전) ≠ Candidate(새 버전)
- 설치 안 된 패키지는 `Installed: (none)`

### 설치

```
ohwm@ohwmdev:~$ apt install htop
Error: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)
Error: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?
ohwm@ohwmdev:~$ sudo apt install htop
htop is already the newest version (3.4.1-5build2).
htop set to manually installed.
```

- `13` = `EACCES`. lock 파일이 root 소유라 `open()`이 거부됨 → `sudo` 필요
- lock은 두 apt가 동시에 dpkg 기록을 고치지 못하게 막는 장치
- htop은 Ubuntu Server에 기본 설치돼 있었음
- `manually installed`: auto(의존성으로 깔림) → manual(직접 설치)로 기록 변경. `autoremove`는 auto만 지우므로 이제 htop은 지워지지 않음

### 실행 파일 위치

```
ohwm@ohwmdev:~$ type -a htop
htop is /usr/bin/htop
htop is /bin/htop
ohwm@ohwmdev:~$ dpkg -S /usr/bin/htop
htop: /usr/bin/htop
```

- 셸은 `PATH`의 폴더를 앞에서부터 찾고, 처음 찾은 것을 fork → exec
- `PATH` 앞쪽(`/usr/local/bin`)에 옛 버전이 있으면 apt로 업그레이드해도 옛 버전이 실행됨
- bash는 찾은 경로를 기억(hash)함 → `hash -r`로 캐시 비우기
- 두 개로 보이지만 같은 파일: `/bin`은 `usr/bin`을 가리키는 심볼릭 링크 (usrmerge)
  - 심볼릭 링크 = 다른 경로가 적힌 작은 파일. 커널이 path resolution 중에 따라감
  - 링크 자체 권한(`lrwxrwxrwx`)은 검사하지 않고, 도착한 대상의 권한을 effective UID로 검사

### 설치된 파일 개수

```bash
dpkg -L htop | wc -l
```

결과: 33줄 (폴더 포함이라 파일 수와 같지 않음)

### `ps`는 어느 패키지?

```bash
dpkg -S /usr/bin/ps
```

결과: `procps` (= /proc + ps). `ps`, `top`, `pgrep`, `kill`, `free` 등 `/proc`를 읽어서 보여주는 도구 모음

## 트러블슈팅: VM 시계 장애

### 증상

`sudo apt update`에서 보안 업데이트 저장소가 거부됨

```
Error: Release file for http://kr.archive.ubuntu.com/ubuntu/dists/resolute-updates/InRelease is not valid yet (invalid for another 14h 3min 6s). Updates for this repository will not be applied.
Error: Release file for http://security.ubuntu.com/ubuntu/dists/resolute-security/InRelease is not valid yet (invalid for another 20h 1min 26s). Updates for this repository will not be applied.
```

- `InRelease` 안의 `Date:`(목록 생성 시각)가 VM 시계보다 미래라서 apt가 거부
- 오래되거나 조작된 목록을 막는 보안 장치 → 보안 업데이트가 적용되지 않는 상태

### 진단

**1) 시계 확인**

```
ohwm@ohwmdev:~$ timedatectl
               Local time: Sat 2026-10-24 18:54:48 UTC
           Universal time: Sat 2026-10-24 18:54:48 UTC
                 RTC time: Sun 2026-10-25 12:45:16
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: no
```

- 시스템 시계와 RTC가 크게 차이 남
- NTP 서비스는 active인데 synchronized는 no

**2) 어떤 NTP 서비스가 도는지 확인**

```
ohwm@ohwmdev:~$ journalctl -u systemd-timesyncd
-- No entries --
ohwm@ohwmdev:~$ systemctl status chrony systemd-timesyncd
Unit systemd-timesyncd.service could not be found.
● chrony.service - chrony, an NTP client/server
     Loaded: loaded (/usr/lib/systemd/system/chrony.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-10-24 15:39:31 UTC; 3h 27min ago
...
Oct 24 18:49:32 ohwmdev chronyd[1057]: Root distance of 185.125.190.122 (1.ntp.ubuntu.com) exceeds maxdistance of >
Oct 24 18:52:36 ohwmdev chronyd[1057]: Jitter of 91.189.91.111 (ntp-bootstrap.ubuntu.com) exceeds maxjitter of 1.0>
Oct 24 18:52:48 ohwmdev chronyd[1057]: Jitter of 185.125.190.122 (1.ntp.ubuntu.com) exceeds maxjitter of 1.000 sec>
Oct 24 18:59:04 ohwmdev chronyd[1057]: Selected source 91.189.91.112 (3.ntp.ubuntu.com)
Oct 24 18:59:04 ohwmdev chronyd[1057]: System clock wrong by 43559.473016 seconds
```

- 이 VM은 timesyncd가 아니라 chrony를 사용 (`journalctl`이 비어 있던 이유 = 설치 안 된 서비스)
- `Loaded:` = 설치 / enable 여부, `Active:` = 실행 여부
- NTP 서버를 선택했으므로 네트워크(UDP 123)는 정상 → 서버 추가 같은 설정 변경은 원인과 무관
- chrony는 시계가 약 12시간 틀린 걸 알고 있음

### 원인

1. Mac이 잠자기에 들어가면 하이퍼바이저가 VM의 vCPU를 멈춤 → 커널이 시간을 세지 못하고, 깨어나면 멈춘 시각부터 이어서 셈
2. 커널은 RTC를 부팅 때 한 번만 읽음. RTC는 Mac 시계를 보여주므로 정확했음
3. chrony는 기본적으로 서비스 시작 직후 몇 번의 측정에서만 step(점프)을 허용하고, 이후에는 slew(천천히)만 함
4. 서비스 시작(15:39) 이후 시계가 밀렸으므로 slew로 12시간을 따라잡는 중이었음 → 며칠 걸림

| 방법 | 위험 |
|---|---|
| step (점프) | 로그 시각 역전 / 구멍, 타임아웃·cron 오작동 |
| slew (천천히) | 순서는 지켜지지만 틀린 시각이 오래 유지 |

### 조치

```
ohwm@ohwmdev:~$ sudo chronyc makestep
200 OK
```

### 확인

```
ohwm@ohwmdev:~$ timedatectl status
               Local time: Mon 2026-10-26 06:06:53 UTC
           Universal time: Mon 2026-10-26 06:06:53 UTC
                 RTC time: Mon 2026-10-26 06:06:54
System clock synchronized: yes
              NTP service: active
```

- 시스템 시계와 RTC 차이 1초 이내, synchronized: yes
- `sudo apt update` 에러 사라짐

### 배운 점

- 검색해서 나온 해결법(`/etc/chrony.conf`, `chronyd`)은 RHEL 계열 경로였음 → 배포판 확인 필수
- 처방 전에 `systemctl status` → `journalctl` 순서로 원인부터 확인
- 자주 멈추는 개발용 VM은 `makestep`의 횟수 제한을 음수로 두어 자동화할 수 있음. 운영 서버는 뒤로 점프하는 위험 때문에 보통 기본값 유지
- 시계가 틀리면 apt뿐 아니라 TLS 인증서 검증, AWS API 서명도 실패
