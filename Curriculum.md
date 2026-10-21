# 클라우드 DevOps 커리큘럼

> 2026년 10월 11일 · @ohwm

## 목차

- [학습 원칙과 전체 로드맵](#학습-원칙과-전체-로드맵)
- [Phase 0 · 학습 환경 세팅](#phase-0--학습-환경-세팅-1012--1018)
- [Phase 1–2 · 리눅스 실전과 네트워크](#phase-12--리눅스-실전과-네트워크-1019--1122)
- [Phase 3 · Docker](#phase-3--docker-1123--1130-여행-후-1223--13)
- [Phase 4–5 · AWS와 Terraform](#phase-45--aws와-terraform-14--214)
- [Phase 6 · CI/CD](#phase-6--cicd-215--228)
- [Phase 7–8 · Kubernetes와 관측성](#phase-78--kubernetes와-관측성-3월--6월-학기-병행)
- [3학년 학기 중과 그 이후](#3학년-학기-중과-그-이후-2027년-3월--)
- [Claude Code로 공부 능률 올리기](#claude-code로-공부-능률-올리기)
- [하루·주간 루틴](#하루주간-루틴)
- [참고 자료](#참고-자료-공식-문서-우선)

---

## 학습 원칙과 전체 로드맵

전역 직후부터 3학년 개강(2027년 3월)까지 풀타임(하루 6시간+)으로 써서, 개강 전에 **Linux → 네트워크 → Docker → AWS → Terraform → CI/CD**까지 한 바퀴 돌리는 것이 목표다.

- 12월 1일–22일은 유럽 여행으로 공부를 쉰다.
- Kubernetes와 관측성(모니터링)은 학기 중에 수업(OS·시스템프로그래밍·네트워크·정보보호)과 엮어서 진행한다.

### 다섯 가지 원칙

1. **개념 30%, 손 70%.** 읽고 끝내지 않는다. 모든 주제는 직접 띄우고, 망가뜨리고, 고쳐본다.
2. **프로젝트 하나를 끝까지 키운다.** NBA 경기 알림 봇 하나를 Phase마다 업그레이드한다.
   Python 스크립트 → 컨테이너 → AWS 배포 → Terraform으로 인프라 코드화 → GitHub Actions 자동 배포 → Kubernetes → 모니터링·알림. 이게 곧 포트폴리오가 된다.
3. **"왜?"를 세 번 묻는다.** 명령어를 외우지 말고, 그 명령이 OS·네트워크 수준에서 무엇을 하는지 설명할 수 있어야 다음 단계로 넘어간다.
4. **모든 것을 기록한다.** 매일 TIL, Phase마다 블로그 글 1편. 면접에서 "이거 해봤다"를 증명하는 자료가 된다.
5. **Claude는 튜터지 대필가가 아니다.** 코드·설정은 직접 쓰고, Claude에게는 힌트·리뷰·퀴즈·장애 시나리오를 받는다. ([Claude Code 섹션](#claude-code로-공부-능률-올리기) 참고)

### 한눈에 보는 일정

> 주 단위, 진도에 따라 ±1주 조정

| Phase | 기간 | 주제 | 끝나면 할 수 있는 것 |
|---|---|---|---|
| 0 | 10/12 – 10/18 (1주) | 환경 세팅 + Claude Code + Git 기초 | 맥북에서 VM·터미널·GitHub로 공부 환경이 돌아간다 |
| 1 | 10/19 – 11/8 (3주) | 리눅스 실전·셸 스크립트·Git·Python 스크립팅 | 서버에 SSH로 들어가 서비스를 띄우고 로그로 문제를 찾는다 |
| 2 | 11/9 – 11/22 (2주) | 네트워크 (TCP/IP, DNS, HTTP/TLS, Nginx) | "브라우저에 URL을 치면 일어나는 일"을 패킷 수준으로 설명한다 |
| 3-1 | 11/23 – 11/30 (1주) | Docker 1주차: 이미지·컨테이너·볼륨 | 컨테이너를 띄우고 데이터가 볼륨에 남는지 확인한다 |
| ✈️ | 12/1 – 12/22 | 유럽 여행 (공부 쉼) | 떠나기 전 켜진 클라우드 리소스 없음 (AWS 계정은 여행 후 생성) |
| 3-2 | 12/23 – 1/3 (1.5주) | Docker 1주차 복습 + 2–3주차: Dockerfile·Compose | 알림 봇을 컨테이너로 만들고 Compose로 DB와 함께 띄운다 |
| 4 | 1/4 – 1/31 (4주) | AWS | VPC부터 직접 설계해 알림 봇을 배포한다 |
| 5 | 2/1 – 2/14 (2주) | Terraform | 4주 동안 클릭으로 만든 인프라를 코드로 다시 만든다 |
| 6 | 2/15 – 2/28 (2주) | CI/CD (GitHub Actions) | push 하면 테스트 → 이미지 빌드 → 배포가 자동으로 된다 |
| 7 | 3월 – 4월 (학기 병행) | Kubernetes | Minikube·EKS에 알림 봇을 올리고 롤링 업데이트를 한다 |
| 8 | 5월 – 6월 (학기 병행) | 관측성: Prometheus·Grafana·로그 | 대시보드와 알림으로 장애를 먼저 발견한다 |

---

## Phase 0 · 학습 환경 세팅 (10/12 – 10/18)

첫 주는 공부할 "작업대"를 만드는 주다. 이 세팅이 앞으로 20주의 속도를 결정한다.

### 할 일

1. **맥북 초기화 후 기본 도구**: Homebrew → iTerm2(또는 기본 터미널) → zsh 설정 → VS Code → Git
2. **Git·GitHub 기초**: init / add / commit / push / pull / branch / merge, SSH 키 등록, 커밋 메시지 규칙. 학습 저장소 `devops-study`를 만든다. ([저장소 구조](#1-설치와-저장소-구조) 참고)
3. **리눅스 VM**: UTM + Ubuntu Server (최신 LTS, ARM64). AWS 기본 이미지와 Docker 베이스 이미지에서 가장 많이 쓰는 배포판이라 이것 하나로 간다.
4. **Claude Code 설치 + CLAUDE.md 튜터 설정** ([아래 섹션](#2-claudemd-튜터-모드-설정) 그대로 따라 하기)
5. **AWS 계정은 아직 만들지 않는다.** Phase 4에서 예산 알림부터 걸고 시작한다. (EC2 과금 사고 재발 방지)

### 완료 기준

- [x] 터미널에서 VM에 ssh로 접속된다
- [x] `devops-study` 저장소에 첫 커밋이 올라가 있다
- [x] Claude Code가 CLAUDE.md를 읽고 힌트만 주는 튜터처럼 답한다

---

## Phase 1–2 · 리눅스 실전과 네트워크 (10/19 – 11/22)

리눅스마스터 2급은 "용어"를 알게 해줬다. 여기서는 그 지식을 서버를 운영하는 손으로 바꾼다. DevOps의 모든 도구는 결국 리눅스 프로세스와 네트워크 위에서 돈다.

### Phase 1 · 리눅스 실전 + 셸 + Python (3주)

| 주차 | 주제 | 실습 |
|---|---|---|
| 1주 | 파일시스템·권한·사용자, 프로세스(ps, top, kill, 시그널), 패키지 관리 | 사용자별 권한이 다른 공유 디렉터리 만들기 |
| 2주 | systemd(systemctl, unit 파일), 로그(journalctl, /var/log), SSH 키·sshd_config, cron | 직접 만든 스크립트를 systemd 서비스로 등록하고 재부팅 후 자동 실행 확인 |
| 3주 | Bash 스크립트(변수·조건·반복·종료코드), grep / awk / sed / jq, 파이프 | 로그에서 에러를 집계해 리포트하는 스크립트 |

> **Python 병행 (하루 1시간)**: 프로그래머스 문제로 문법을 익히고, 3주차부터 `nba_api`로 "오늘 경기 결과를 출력하는 스크립트"를 만든다. 이게 **알림 봇 v0.1**이다. 이어서 requests, 환경변수, 로깅, 예외 처리까지.

### Phase 2 · 네트워크 (2주)

| 주차 | 주제 | 실습 |
|---|---|---|
| 1주 | OSI/TCP-IP, IP·서브넷·CIDR, 라우팅, TCP 3-way handshake, 포트, NAT | ip, ss, ping, traceroute, tcpdump로 실제 패킷 보기 |
| 2주 | DNS, HTTP/HTTPS, TLS 인증서, 리버스 프록시·로드밸런서 | VM에 Nginx 리버스 프록시 + 백엔드 2대 띄워 부하 분산, dig·curl -v로 추적 |

> CIDR은 AWS VPC 설계에서 그대로 다시 나오니 여기서 확실히 잡는다.

### 완료 기준

- [ ] 서비스가 안 뜰 때 systemctl status → journalctl → 포트 확인 순서로 원인을 찾는다
- [ ] 10.0.0.0/16을 /24 서브넷 4개로 나누고 각 범위를 말할 수 있다
- [ ] "브라우저에 URL을 치면 일어나는 일"을 DNS·TCP·TLS·HTTP 순서로 설명한다
- [ ] 알림 봇 v0.1이 cron으로 하루 한 번 돌아간다

---

## Phase 3 · Docker (11/23 – 11/30, 여행 후 12/23 – 1/3)

1주차는 여행 전에, 2–3주차는 여행에서 돌아와 1주차를 짧게 복습한 뒤 이어서 한다.

컨테이너는 **"격리된 리눅스 프로세스"**다. 이 한 문장을 직접 증명하는 것이 이 Phase의 목표이고, 3월 시스템프로그래밍·OS 수업과 C 미니 컨테이너 런타임 프로젝트의 복선이 된다.

| 주차 | 주제 | 실습 |
|---|---|---|
| 1주 | 이미지 vs 컨테이너, run / exec / logs / ps, 볼륨, 포트 매핑, 레지스트리 | Nginx·PostgreSQL 컨테이너 띄우고 데이터가 볼륨에 남는지 확인 |
| 2주 | Dockerfile, 레이어 캐시, 멀티스테이지 빌드, 이미지 경량화, .dockerignore, 비루트 사용자 | 알림 봇 v0.2: Dockerfile 작성 → 이미지 크기를 단계별로 줄이며 기록 |
| 3주 | Docker 네트워크(bridge/host), Docker Compose, 헬스체크, 환경변수·시크릿 | 알림 봇 + PostgreSQL + Nginx를 Compose 하나로 띄우기 |

> **속을 들여다보는 실습**: 컨테이너 안 프로세스를 호스트에서 `ps`로 찾기, `/proc/<pid>/ns` 확인, `unshare`로 네임스페이스 직접 만들어보기, cgroup으로 메모리 제한 걸고 OOM 일으켜 보기. 이걸 해본 사람과 안 해본 사람은 면접에서 확연히 갈린다.

> **참고**: Apple Silicon(ARM)에서 만든 이미지는 x86 서버에서 안 돌 수 있다. `docker buildx`로 멀티 아키텍처 빌드를 이때 익혀두면 Phase 4에서 덜 헤맨다.

### 완료 기준

- [ ] 알림 봇 이미지를 Docker Hub(또는 GHCR)에 push 했다
- [ ] `docker compose up` 한 줄로 전체 스택이 뜬다
- [ ] "컨테이너와 VM의 차이"를 네임스페이스·cgroup으로 설명한다
- [ ] 블로그 글 1편: 이미지 크기를 몇 MB에서 몇 MB로 줄였는지

---

## Phase 4–5 · AWS와 Terraform (1/4 – 2/14)

순서가 핵심이다. **먼저 콘솔에서 손으로 만들어 원리를 이해하고, 그다음 Terraform으로 똑같이 다시 만든다.** 처음부터 코드로 하면 무엇을 만드는지 모른 채 복붙하게 된다.

### Phase 4 · AWS (4주)

> **Day 1 안전장치 (반드시 먼저)**
> 루트 계정 MFA → 루트 대신 IAM 사용자(또는 IAM Identity Center) → AWS Budgets로 월 예산 알림 → 실습 끝나면 리소스 삭제하는 습관.
> 하루 공부를 마칠 때 "켜져 있는 것 확인"을 체크리스트에 넣는다.

| 주차 | 주제 | 실습 |
|---|---|---|
| 1주 | IAM(사용자·역할·정책, 최소 권한), EC2, 보안 그룹, 키페어, EBS | EC2에 SSH 접속해 알림 봇 컨테이너 실행 |
| 2주 | VPC 직접 설계: 퍼블릭/프라이빗 서브넷, 라우팅 테이블, IGW, NAT, 보안 그룹 vs NACL | Phase 2 CIDR 지식으로 2-AZ VPC를 손으로 그린 뒤 그대로 구축 |
| 3주 | S3, RDS, ALB, Auto Scaling Group, Route 53, ACM | ALB 뒤에 ASG로 봇 API를 두고 인스턴스를 강제로 죽여 자동 복구 확인 |
| 4주 | CloudWatch(지표·로그·알람), ECR, ECS(Fargate) 맛보기, 비용 구조 | 알림 봇을 ECR + ECS로 배포, 에러 시 CloudWatch 알람 |

> **자격증**: 이 Phase가 끝날 즈음 AWS Solutions Architect – Associate를 준비하면 효율이 좋다. (응시 전 최신 시험 버전 확인) 자격증 자체보다 "설계 이유를 말할 수 있는가"가 중요하다.

### Phase 5 · Terraform (2주)

| 주차 | 주제 | 실습 |
|---|---|---|
| 1주 | HCL 문법, provider·resource·variable·output, init / plan / apply / destroy, state | Phase 4의 VPC를 Terraform으로 재현 |
| 2주 | 원격 state(S3 + 잠금), 모듈, 환경 분리(dev/prod), fmt / validate, drift | VPC·ALB·ECS를 모듈로 나누고 destroy → apply로 10분 안에 전체 재구축 |

### 완료 기준

- [ ] 2-AZ VPC 구조도를 손으로 그리고 각 구성요소의 역할을 설명한다
- [ ] 알림 봇이 AWS에서 돌고, 장애 시 알람이 온다
- [ ] `terraform destroy` 후 `apply` 한 번으로 인프라 전체가 복구된다
- [ ] 이번 달 AWS 비용이 예산 안에 있다

---

## Phase 6 · CI/CD (2/15 – 2/28)

지금까지 손으로 하던 "테스트 → 빌드 → 배포"를 사람 손 없이 돌게 만든다. 여기서부터 진짜 DevOps다.

| 주차 | 주제 | 실습 |
|---|---|---|
| 1주 | CI 개념, GitHub Actions(workflow·job·step·runner), 캐시, 매트릭스, 시크릿 | PR마다 lint + pytest 자동 실행, 실패하면 머지 불가 (브랜치 보호) |
| 2주 | CD, OIDC로 AWS 인증(액세스 키 저장 금지), 이미지 태깅 전략, Terraform plan을 PR 코멘트로, 롤백 | main 머지 → ECR push → ECS 배포 자동화, 이전 버전으로 롤백 연습 |

> **보안 습관**: 시크릿을 코드·로그에 남기지 않기, 이미지 취약점 스캔(Trivy) 단계 추가. 3월 정보보호 수업과 바로 연결된다.

### 완료 기준

- [ ] main에 push 하면 5분 안에 새 버전이 배포된다
- [ ] 저장소 어디에도 AWS 액세스 키가 없다 (OIDC 사용)
- [ ] 일부러 테스트를 깨뜨렸을 때 배포가 막힌다

---

## Phase 7–8 · Kubernetes와 관측성 (3월 – 6월, 학기 병행)

### Phase 7 · Kubernetes (3–4월, 학기 병행)

로컬(Minikube 또는 kind)에서 충분히 연습한 뒤, 마지막에만 EKS를 잠깐 띄운다. EKS는 켜두는 것만으로 비용이 나가므로 실습 후 바로 `terraform destroy`.

| 주차 | 주제 | 실습 |
|---|---|---|
| 1주 | 아키텍처(컨트롤 플레인·kubelet·etcd), Pod, Deployment, ReplicaSet, Service, kubectl | 알림 봇 Deployment 작성, Pod를 지워도 다시 살아나는 것 확인 |
| 2주 | ConfigMap·Secret, Ingress, 리소스 requests/limits, liveness/readiness probe, HPA | 롤링 업데이트·롤백, 일부러 OOMKilled·CrashLoopBackOff 만들고 해결 |
| 3주 | Helm, 네임스페이스·RBAC, PV/PVC, EKS(Terraform으로), GitOps(Argo CD) 맛보기 | 알림 봇을 Helm 차트로 만들고 Argo CD로 Git 변경이 자동 반영되게 |

### Phase 8 · 관측성 (5–6월, 학기 병행)

"모니터링은 장애를 고객보다 먼저 아는 것"이 목표다.

1. **지표**: Prometheus + Grafana (kube-prometheus-stack), 봇에 커스텀 메트릭 추가
2. **로그**: Loki 또는 CloudWatch Logs로 중앙 수집
3. **알림**: Alertmanager → Slack/Discord
4. **SRE 개념**: SLI·SLO·에러 버짓, 골든 시그널(지연·트래픽·에러·포화도). Google의 무료 SRE 책을 이때 읽는다.

### 완료 기준

- [ ] 알림 봇이 Kubernetes에서 돌고 Helm으로 배포된다
- [ ] `kubectl describe`·`logs`·`events`로 Pod 장애 원인을 찾는다
- [ ] Grafana 대시보드에 봇의 SLO와 에러 버짓이 보인다
- [ ] 장애를 일부러 내면 알림이 먼저 온다

---

## 3학년 학기 중과 그 이후 (2027년 3월 –)

"업계 최고"는 도구를 많이 아는 사람이 아니라 **장애가 났을 때 OS·네트워크 바닥까지 내려가서 원인을 찾는 사람**이다. 학기 수업이 바로 그 바닥을 채워준다.

### 수업과 연결하기

| 수업 | DevOps와 연결되는 지점 | 같이 할 것 |
|---|---|---|
| 운영체제 | 프로세스·스케줄링·메모리·파일시스템 = 컨테이너와 리소스 제한의 원리 | OOM, CPU throttling을 지표로 관찰 |
| 시스템프로그래밍 | fork / exec, 시스템콜, 시그널 | C 미니 컨테이너 런타임 (namespace + cgroup + chroot) |
| 컴퓨터네트워크 | TCP 혼잡 제어, 라우팅, DNS | Kubernetes 네트워크(CNI, kube-proxy) 파헤치기 |
| 정보보호 | 암호화, 인증, 취약점 | IAM 최소 권한, 시크릿 관리, 컨테이너 보안 |

> **C 준비**: 1–2월에 하루 30분씩 C 기초(포인터, 구조체, 메모리)를 미리 해두면 학기와 런타임 프로젝트가 훨씬 수월하다.

### 자격증 (도구가 아니라 증명용)

1. **AWS Solutions Architect – Associate**: 1–2월
2. **CKA (Certified Kubernetes Administrator)**: 여름방학. 실기 시험이라 실력 증명에 가장 좋다.
3. 이후 선택: Terraform Associate, AWS DevOps Engineer – Professional

### 포트폴리오 두 축

1. **NBA 알림 봇 플랫폼**: Terraform + EKS + GitHub Actions + Argo CD + Prometheus. README에 구조도, 비용, 장애 대응 기록(포스트모템)까지.
2. **C 미니 컨테이너 런타임**: "Docker를 쓸 줄 아는 사람"이 아니라 "Docker가 어떻게 동작하는지 아는 사람"임을 증명.

### 취업 준비 (여름 – 4학년)

- 타깃 회사(쿠팡플레이, 무신사, 크래프톤, 왓챠, 티빙) 기술 블로그를 주 1편씩 읽고 그들의 인프라 고민을 정리한다.
- 채용 공고의 요구 기술을 표로 모아 빈칸을 채운다. (백엔드 언어 선택도 이때 공고를 보고 결정: Go 또는 Java 가능성 높음)
- 여름방학 인턴·부트캠프·오픈소스 기여(문서·작은 이슈부터)로 실무 경험 1줄 만들기

---

## Claude Code로 공부 능률 올리기

핵심은 하나다. **Claude Code에게 "대신 해줘"가 아니라 "내가 하는 걸 봐줘"를 시킨다.** 코드를 받아 붙여넣으면 20주 뒤에 남는 게 없다. 대신 튜터·리뷰어·장애 출제자로 쓰면 혼자 공부보다 몇 배 빠르다. 

### 1. 설치와 저장소 구조

공식 문서(docs.claude.com)의 설치 명령으로 설치하고, 터미널에서 `claude` 실행 후 Pro 계정으로 로그인한다. 학습 저장소는 이렇게 만든다.

```
devops-study/
├── CLAUDE.md            # Claude가 매번 읽는 튜터 규칙
├── .claude/
│   ├── settings.json    # 위험한 명령 차단
│   └── commands/        # 나만의 슬래시 커맨드
├── til/                 # 날짜별 TIL (2026-10-12.md ...)
├── labs/                # Phase별 실습 (01-linux, 02-network, ...)
├── notes/               # 개념 정리, 틀린 퀴즈 모음
└── nba-bot/             # 관통 프로젝트
```

### 2. CLAUDE.md: 튜터 모드 설정

Claude Code는 저장소의 CLAUDE.md를 매 세션 자동으로 읽는다. 여기에 학습 규칙을 적어두면 매번 설명할 필요가 없다.

Phase가 바뀔 때마다 "현재 진도"만 고친다.

### 3. 나만의 슬래시 커맨드

`.claude/commands/` 안에 마크다운 파일을 만들면 `/파일이름`으로 호출된다.

| 커맨드 | 하는 일 | 언제 |
|---|---|---|
| `/quiz` | 오늘 TIL을 읽고 개념 확인 문제 5개 출제, 틀린 것은 `notes/wrong.md`에 기록 | 매일 마무리 |
| `/break` | VM이나 Compose 환경에 일부러 장애를 하나 심고 증상만 알려줌 (원인은 내가 찾음) | 주 2–3회 |
| `/review` | 방금 쓴 스크립트·Dockerfile·Terraform을 보안·모범사례 기준으로 리뷰만 | 실습 직후 |
| `/interview` | 현재 Phase 주제로 면접관 역할, 꼬리질문까지 | 주말 |

예시 `.claude/commands/break.md`:

```markdown
현재 진도(CLAUDE.md 참고)에 맞는 장애 시나리오를 하나 만들어라.
- labs/ 안의 실습 환경에만 적용하고, 무엇을 바꿨는지는 notes/break-answers.md에만 적어라.
- 나에게는 "사용자 입장에서 보이는 증상"만 알려라.
- 내가 원인을 말하면 맞는지 판정하고, 틀리면 힌트 1단계만 줘라.
```

> `/break`가 가장 강력하다. SRE 면접은 결국 "이 장애를 어떻게 찾겠냐"를 묻는데, 이걸 매주 실전처럼 연습하게 된다.

### 4. 안전장치: settings.json

AWS 단계부터는 Claude가 실수로 비싼 리소스를 만들거나 지우지 못하게 막는다.

> `apply`·`destroy`는 항상 내 손으로 친다. 그게 공부다.

### 5. 이럴 때 이렇게 쓴다

- **에러가 났을 때**: 에러 전문을 주고 "원인 후보 3개와 각각 확인하는 명령만 알려줘". 확인은 직접.
- **문서·설정 파일을 읽을 때**: "이 Dockerfile을 한 줄씩, 각 줄이 레이어와 캐시에 미치는 영향 중심으로 설명해줘".
- **영어 공식 문서**: 먼저 직접 읽고, 이해 안 되는 문단만 물어본다. 영어 문서 독해력도 같이 큰다.
- **개념 질문·계획 점검**: 터미널이 필요 없는 질문은 이 프로젝트의 claude.ai 채팅에서 하고, Claude Code는 실습에 집중시킨다.

### 6. Pro 사용량 아끼기

Pro는 일정 시간 단위로 사용 한도가 있다.

- 주제가 바뀌면 `/clear`로 대화를 비워 맥락을 가볍게 한다.
- 질문은 "무엇을 시도했고 어디서 막혔는지"를 담아 한 번에 정확히 한다. 짧은 질문을 여러 번 하는 것보다 한도를 훨씬 덜 쓴다.

---

## 하루·주간 루틴

하루 6시간을 **"개념 → 실습 → 프로젝트 → 복습"** 순서로 고정한다. 매일 무엇을 할지 고민하는 시간을 없애는 게 목적이다.

### 하루 (평일)

| 시간 | 블록 | 내용 |
|---|---|---|
| 오전 2h | 개념 | 공식 문서·책으로 오늘 주제 학습, 모르는 것만 Claude에게 질문 |
| 오전 1h | 실습 | `labs/`에서 직접 따라 하고 일부러 망가뜨려 보기 |
| 오후 2h | 프로젝트 | 알림 봇에 오늘 배운 것 적용 |
| 저녁 1h | Python·영어 | 프로그래머스 1–2문제 또는 영어 기술 문서 1편 |
| 마무리 15분 | 복습 | TIL 작성 → `/quiz` → 커밋 (AWS 단계엔 켜진 리소스 확인) |

### 주간

- **토요일**: `/break` 장애 실습 2회 + `/interview` 모의 면접
- **일요일 오전**: 회고 30분 (이번 주 완료 기준 체크, 다음 주 계획, CLAUDE.md 진도 갱신). 오후는 쉰다. 20주는 마라톤이다.
- **Phase가 끝날 때마다**: 블로그 글 1편 + 완료 기준 전부 체크 후 다음 Phase로

### 막혔을 때 규칙

1. 30분 혼자
2. 힌트 1단계
3. 30분 더
4. 힌트 2단계
5. 2시간 넘게 한 곳에 막히면 메모해두고 다음 주제로 넘어가, 다음 날 다시 본다.

---

## 참고 자료 (공식 문서 우선)

| 분야 | 자료 |
|---|---|
| 전체 지도 | roadmap.sh의 DevOps 로드맵 |
| 리눅스·Docker | 각 공식 문서, Docker 공식 Get Started |
| AWS | AWS Skill Builder 무료 강의, Well-Architected Framework |
| Terraform | HashiCorp 공식 튜토리얼 |
| Kubernetes | kubernetes.io 공식 튜토리얼, CKA 대비 실습은 killer.sh |
| SRE | Google SRE 책 (sre.google에서 무료 공개) |
