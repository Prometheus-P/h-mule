
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
```

> **원칙**  
> - 문서: **한국어**  
> - 코드 / 주석 / 식별자 / 커밋 메시지: **영어**

---

## 3. 기술 스택 (요약)

자세한 내용은 `docs/TECH_STACK.md` 참고.

- **Mobile/Web**
  - Flutter 3.x (Android / iOS / Web)
  - State: Riverpod 기반 아키텍처
  - Networking: Dio (+ Interceptor로 JWT, 로깅, 에러 처리)
  - Routing: go_router
  - Codegen: freezed, json_serializable
- **Backend**
  - Go 1.22+
  - HTTP: chi 라우터
  - DB: PostgreSQL
  - DB Layer: sqlc (SQL-first, type-safe)
  - Migration: goose
  - Background Job: Asynq (Redis 기반 작업 큐)
  - Auth: JWT (access / refresh), bcrypt password hash
- **Infra**
  - Docker / docker-compose
  - GitHub Actions (lint + test)
  - 배포: Railway / Render / Fly.io 중 택1 (초기)

---

## 4. 협업 규칙 (요약)

상세 내용은 `docs/COLLAB_RULES.md` 참고.

- **이슈 단위 작업**: 모든 작업은 GitHub Issue 먼저 생성
- **브랜치 전략**: `main` / `develop` / `feature/*` / `hotfix/*`
- **커밋 컨벤션**: Conventional Commits (`feat:`, `fix:`, `chore:` 등)
- **PR 규칙**:
  - 200~300줄 정도로 쪼개기
  - 작업 요약 + 테스트 결과 필수
- **AI 사용 규칙**:
  - 설계/스펙 먼저 → 그 다음 코드 생성
  - AI가 작성한 코드는 반드시 사람이 리뷰 후 머지

---

## 5. 로컬 개발 (초기 초안)

> 실제 명령어/환경변수는 MVP 상세 정의 후 업데이트 예정.

```bash
# 1) backend
cd apps/backend
go run ./cmd/api

# 2) mobile (Flutter)
cd apps/mobile
flutter run   # 시뮬레이터 또는 실제 디바이스 선택
```

---

## 6. 🔓 Open Source & Open Data

Human Mule은 다음 오픈소스 및 공공데이터를 적극 활용합니다.

- **Backend**
  - Go + chi Router
  - PostgreSQL
  - sqlc – SQL → 타입 세이프 Go 코드 생성
  - goose – DB 스키마 마이그레이션 관리
  - Asynq – Redis 기반 백그라운드 작업 큐 (알림/정산 등)
- **Mobile/Web**
  - Flutter 3.x
  - Riverpod / Dio / go_router / freezed / json_serializable
  - STRV / Skelter 등 공개된 Flutter 템플릿 구조를 레퍼런스로 활용
- **Open Data (Seoul Subway)**
  - 공공데이터포털의 서울 지하철 열차 운행 정보 Open API
  - GTFS 기반 대중교통 데이터 (경로 최적화·시뮬레이션 단계에서 활용)

---

## 7. 라이선스 / 기여

- 라이선스: TBD (초기에는 Private Repo 기준)
- 외부 기여는 당분간 받지 않고, **코어 팀 + AI 바이브코딩** 방식으로 개발을 진행합니다.
