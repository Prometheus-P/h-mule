# Human Mule (가칭)

> **서울 지하철 기반 B2B 초소형 급송 물류 플랫폼**  
> “당신의 출근길이 누군가의 급한 마감이 됩니다.”

---

## 1. 프로젝트 개요

Human Mule은 **서울 지하철을 이용해, 검증된 기업의 서류·소형 물품을 역 ↔ 역으로 급송**하는 B2B 특화 크라우드소싱 물류 서비스입니다.

- 퀵서비스보다 **저렴(약 30~40%↓)**
- 택배보다 **압도적으로 빠름(1~2시간급)**
- **B2B 인증 + 오픈 패키지 + 검수 프로세스**로 안전·규제 리스크 최소화

---

## 2. 리포지토리 구조(초기안)

```txt
human-mule/
├── apps/
│   ├── mobile/          # Flutter (Android / iOS / Web)
│   └── backend/         # Go API 서버
├── docs/
│   ├── TECH_STACK.md
│   ├── ARCHITECTURE.md
│   ├── COLLAB_RULES.md
│   ├── VIBE_CODING.md
│   └── ROADMAP.md
├── infra/
│   ├── docker-compose.yml
│   └── migrations/      # DB 마이그레이션 스크립트
├── .github/
│   ├── ISSUE_TEMPLATE/
│   └── PULL_REQUEST_TEMPLATE.md
├── README.md
└── LICENSE (TODO)
원칙

문서: 한국어

코드 / 주석 / 식별자 / 커밋 메시지: 영어

3. 기술 스택 (요약)
자세한 내용은 docs/TECH_STACK.md을 참고하세요.

Mobile/Web

Flutter 3 (Android / iOS / Web)

State: Riverpod or Bloc

Networking: Dio

Routing: go_router

Backend

Go 1.22+

HTTP Framework: chi (or Fiber) – 경량 라우터

DB: PostgreSQL

DB Access: sqlc (SQL-first, type-safe)

Auth: JWT (access / refresh), bcrypt password hash

Infra

Docker / docker-compose

GitHub Actions (lint + test)

배포: Railway / Render / Fly.io 중 택1 (초기)

4. 협업 규칙 (요약)
상세 버전은 docs/COLLAB_RULES.md 참고.

이슈 단위 작업: 뭐든 GitHub Issue 먼저.

브랜치 전략: main / develop / feature/* / hotfix/*

커밋 컨벤션: Conventional Commits (feat:, fix:, chore: …)

PR 규칙:

200~300줄 정도로 쪼개기

설명 + 테스트 결과 필수

AI 사용 규칙:

설계/스펙 → 그 다음에 코드.

AI 코드 그대로 머지 금지, 반드시 사람이 리뷰.

5. 로컬 개발 (거친 초안)
실제 명령어/환경변수는 MVP 정의 후 업데이트 예정.

bash
코드 복사
# 1) backend
cd apps/backend
go run ./cmd/api

# 2) mobile (Flutter)
cd apps/mobile
flutter run   # 시뮬레이터 or 디바이스 선택
6. 라이선스 / 기여
라이선스: TBD (초기에는 private 기준)

외부 기여 X, 코어 팀 + AI 바이브코딩으로 밀어붙임.

yaml
코드 복사

---

```md
<!-- docs/TECH_STACK.md -->

# 🧱 TECH STACK – Human Mule

> **목표:** 모바일 앱까지 자연스럽게 확장 가능한, 길게 가져가도 안 꼬이는 스택.

---

## 1. 전체 아키텍처

### 1.1 Mobile / Web – Flutter

- **Framework**: Flutter 3
- **Target**:
  - Android
  - iOS
  - Web (B2B 대시보드용 PWA)
- **State Management**:
  - Riverpod (선호) 또는 Bloc – 테스트/분리 용이
- **Networking**:
  - Dio (Interceptor로 JWT, 로깅, 에러 공통 처리)
- **Routing**:
  - go_router – 라우팅/딥링크 일관된 처리
- **Local Storage**:
  - shared_preferences (토큰/설정)
  - drift/hive (오프라인 캐시 필요 시)

> **이유:**  
> - 한 코드베이스로 **Android/iOS/Web** 커버  
> - 디자인/애니메이션 자유도 높고, 배포 속도 빠름  
> - 모바일이 메인인 서비스라 Flutter 선택이 합리적

---

### 1.2 Backend – Go

- **Language**: Go 1.22+
- **HTTP Framework**: chi (or Fiber)
  - 경량, 미들웨어 구조 단순, 유지보수 쉬움
- **DB**: PostgreSQL
- **DB Layer**: sqlc
  - SQL을 진짜로 쓰면서도 Go struct로 타입 세이프티 확보
- **Config**:
  - viper or 자체 config loader
- **Auth & Security**:
  - JWT (HMAC 서명) – access / refresh 토큰
  - Password: bcrypt
  - CORS, rate limit, structured logging

> **이유:**  
> - 네가 이미 Go 친숙 + 지하철/위치 기반 서비스는 **동접/동시성**이 중요 → Go 딱 맞음  
> - sqlc로 DB 접근 계층 강하게 잡아두면, 나중에 리팩토링/리포트 작업에도 유리

---

### 1.3 Infra & DevOps

- **컨테이너**: Docker, docker-compose
- **CI/CD**: GitHub Actions
  - Go: `go test ./...`
  - Flutter: `flutter test`
  - Lint: `golangci-lint`, `flutter analyze`
- **Deploy (초기)**:
  - Backend: Railway / Render / Fly.io
  - DB: 같은 곳 Managed Postgres 또는 Supabase/Neon
- **Monitoring (후순위)**:
  - Basic logging → 이후 Sentry / Grafana / Prometheus 고려

---

## 2. Flutter 앱 구조(초기안)

```txt
apps/mobile/
├── lib/
│   ├── main.dart
│   ├── app/
│   │   ├── router.dart
│   │   └── theme.dart
│   ├── features/
│   │   ├── auth/
│   │   ├── company/
│   │   ├── messenger/
│   │   └── delivery/
│   ├── common/
│   │   ├── widgets/
│   │   ├── services/
│   │   └── utils/
│   └── core/
│       ├── config/
│       ├── network/
│       └── env.dart
└── pubspec.yaml
features/* 단위로 도메인 모듈 분리

환경별 flavor (dev / staging / prod) 나중에 추가

3. Backend 구조(초기안)
txt
코드 복사
apps/backend/
├── cmd/
│   └── api/
│       └── main.go
├── internal/
│   ├── config/
│   ├── http/
│   │   ├── router.go
│   │   ├── handlers/
│   │   └── middleware/
│   ├── db/
│   │   ├── migrations/
│   │   └── queries/    # sqlc .sql 파일
│   ├── domain/
│   │   ├── company/
│   │   ├── messenger/
│   │   ├── station/
│   │   └── delivery/
│   └── pkg/
│       ├── auth/
│       ├── logger/
│       └── utils/
└── go.mod
4. 테스트 & 품질
Go

go test ./...

sqlc로 생성된 코드 포함 unit test

Flutter

flutter test – 비즈니스 로직/unit 테스트 우선

Widget test는 MVP 이후 점진적으로

Lint / Format

Go: gofmt, golangci-lint

Flutter: dart format, flutter analyze

5. 언어 정책 요약
문서: 한국어

코드/식별자/커밋 메시지: 영어
일관성, 해외 개발자/AI 협업 고려
