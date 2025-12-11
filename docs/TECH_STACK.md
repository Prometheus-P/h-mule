
# 🧱 TECH STACK – Human Mule

> **목표:** 모바일 앱까지 자연스럽게 확장 가능한, 길게 가져가도 안 꼬이는 스택.  
> + 2025-12 기준으로 안정적인 오픈소스만 채택.

---

## 1. 전체 아키텍처

### 1.1 Mobile / Web – Flutter

- **Framework**
  - Flutter 3.x
- **Architecture / Boilerplate (참고용)**
  - STRV Flutter 템플릿, Skelter 등 검증된 오픈소스 구조를 레퍼런스로 삼되,  
    코드는 Human Mule 도메인에 맞게 직접 작성
- **State Management**
  - Riverpod (또는 flutter_riverpod)
- **Networking**
  - Dio (Interceptor로 JWT, 로깅, 공통 에러 처리)
- **Routing**
  - go_router
- **Codegen**
  - freezed, json_serializable (DTO / 상태 객체 정의)
- **Local Storage**
  - shared_preferences (토큰/설정)
  - drift 또는 hive (오프라인 캐시가 필요해질 경우)

> 전략: “아키텍처/폴더 구조는 오픈소스 템플릿에서 모티브만 따오고,  
> 비즈니스 로직과 모델은 서비스에 최적화해서 직접 작성한다.”

---

### 1.2 Backend – Go

- **Language**
  - Go 1.22+
- **HTTP Framework**
  - chi (`github.com/go-chi/chi/v5`)
- **DB**
  - PostgreSQL
- **DB Access (공식 선택)**
  - **sqlc** (`github.com/sqlc-dev/sqlc`)  
    - SQL-first 방식
    - `.sql` 쿼리에서 타입 세이프 Go 코드를 자동 생성
- **Migrations (공식 선택)**
  - **goose** (`github.com/pressly/goose/v3`)  
    - 가볍고 단순한 DB 마이그레이션 도구
    - SQL/Go 스크립트 모두 지원, CLI/라이브러리 겸용
- **Background Job / 비동기 작업**
  - **Asynq** (`github.com/hibiken/asynq`) – Redis 기반 Task Queue  
    - 사용처: 푸시/문자 발송, 이메일, 정산 배치, 리포트 생성 등
- **Auth & Security**
  - JWT (HMAC 서명, 예: `github.com/golang-jwt/jwt/v5`)
  - Password Hash: bcrypt (`golang.org/x/crypto/bcrypt`)
  - CORS, Rate Limit, Structured Logging 기본 적용

---

### 1.3 Infra & DevOps

- **컨테이너**
  - Docker, docker-compose
- **DB Migration 파이프라인**
  - goose CLI를 CI에서 실행 (`goose up` / `goose down`)
- **CI/CD (GitHub Actions)**
  - Go: `go test ./...`
  - Flutter: `flutter test`
  - Lint:
    - Go: `golangci-lint`
    - Dart/Flutter: `flutter analyze`
- **Background Worker**
  - Asynq Worker를 별도 프로세스/컨테이너로 운영  
    (알림·정산·리포트 등 비동기 처리)

---

## 2. Flutter 앱 구조 (초기안)

```txt
apps/mobile/
├── lib/
│   ├── main.dart
│   ├── app/
│   │   ├── router.dart
│   │   └── theme.dart
│   ├── core/
│   │   ├── config/
│   │   ├── env.dart
│   │   └── error.dart
│   ├── common/
│   │   ├── widgets/
│   │   ├── services/
│   │   └── utils/
│   └── features/
│       ├── auth/
│       ├── company/
│       ├── messenger/
│       └── delivery/
└── pubspec.yaml
```

- `features/*` 단위로 도메인 모듈 분리
- 이후 dev/staging/prod 환경별 flavor 추가 가능

---

## 3. Backend 구조 (초기안)

```txt
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
│   │   ├── migrations/   # goose migration
│   │   └── queries/      # sqlc .sql 파일
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
```

---

## 4. 테스트 & 품질 도구

- **Go**
  - 테스트: 표준 `testing` 패키지 + 필요 시 `testify`
  - 정적 분석: `golangci-lint`
- **Flutter**
  - `flutter test` – 비즈니스 로직 중심 unit test
  - `flutter analyze` – 정적 분석
- **공통**
  - PR마다 GitHub Actions에서 lint + test 자동 실행

---

## 5. Open Data / External API

"서울 지하철" 특화 서비스이므로, 공공데이터 API를 공식 스택에 포함한다.

### 5.1 서울 지하철 운행 정보 Open API

- 공공데이터포털에서 제공하는 **서울 지하철 열차 운행 정보 API**
- Human Mule 용도:
  - 해당 시간·구간이 실제로 운행 중인지 검증
  - 심야/운행 종료 시간대에 매칭 제한 로직에 활용

### 5.2 GTFS 기반 대중교통 데이터 (후순위)

- 국토교통·교통 연구기관 등에서 제공하는 GTFS 데이터셋
- 활용 시점:
  - SLA 분석, 지연 패턴 분석
  - 장기적으로 경로 최적화·시뮬레이션에 활용

---

## 6. 언어 정책 요약

- **문서**: 한국어
- **코드/식별자/커밋 메시지**: 영어  
  → 해외 개발자/AI 협업, 장기 유지보수에 유리
