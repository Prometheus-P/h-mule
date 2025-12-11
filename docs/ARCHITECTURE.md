
# 🏗 ARCHITECTURE – Human Mule

---

## 1. 도메인 개요

핵심 도메인 개념:

- **Company**: 검증된 B2B 화주(법인/개인사업자)
- **CompanyUser**: Company 소속 사용자(직원)
- **Messenger**: 출퇴근 루트 기반으로 배송을 수행하는 일반 사용자
- **Station**: 서울 지하철 역
- **RoutePreference**: 메신저 출퇴근 루트/시간대 선호 정보
- **DeliveryRequest**: 화주가 생성한 배송 요청
- **DeliveryMatch**: 특정 DeliveryRequest ↔ Messenger 매칭 정보
- **DeliveryEventLog**: 상태 변화/이벤트 기록

---

## 2. 간단 ERD (텍스트 버전)

```txt
Company (
  id, name, biz_reg_no, email, status,
  created_at, updated_at, ...
)

CompanyUser (
  id, company_id, email, name, role,
  password_hash, status,
  created_at, updated_at, ...
)

Messenger (
  id, phone, name, status,
  verified_at, created_at, updated_at, ...
)

Station (
  id, line, name_ko, name_en, code,
  lat, lng, ...
)

RoutePreference (
  id, messenger_id,
  from_station_id, to_station_id,
  time_window_start, time_window_end,
  days_of_week, is_active
)

DeliveryRequest (
  id, company_id,
  from_station_id, to_station_id,
  item_type,            -- DOCUMENT, SAMPLE, SMALL_ITEM 등
  price_total,
  status,               -- CREATED, MATCHING, MATCHED, IN_TRANSIT, COMPLETED, CANCELED
  requested_pickup_time,
  requested_drop_time,
  created_at, updated_at, ...
)

DeliveryMatch (
  id, delivery_request_id, messenger_id,
  status,                 -- PENDING, ACCEPTED, PICKED, DROPPED, FAILED, CANCELED
  accepted_at, picked_at, dropped_at, canceled_at
)

DeliveryEventLog (
  id, delivery_match_id,
  event_type,             -- CREATED, MATCHED, PICKED, DROPPED, CANCELED ...
  event_payload_json,
  created_at
)
```

---

## 3. 기술 컴포넌트 매핑

### 3.1 Backend

- **HTTP / API 레이어**
  - chi Router
  - 공통 미들웨어:
    - Auth (JWT)
    - Request Logging
    - Recovery (panic 대비)
    - CORS

- **Persistence 레이어**
  - PostgreSQL
  - sqlc로 생성된 Repository 레이어
  - goose 기반 schema migration (`infra/migrations/*.sql`)

- **Background Job 레이어**
  - Asynq Client:
    - `NotifyCompanyDeliveryCompleted`
    - `SendDailySettlementReport`
    - `SendMessengerPayoutReady`
  - Asynq Worker:
    - `worker/notification_worker.go`
    - `worker/settlement_worker.go`

---

## 4. 외부 데이터 및 연동

### 4.1 서울 지하철 Open API

- 목적:
  - 매칭/운영 정책에 실제 열차 운행 여부/시간대 반영
- v1 사용 방식 (MVP+α):
  - 운영 대시보드에서 사후 분석용으로 활용
  - 심야/운행 종료 시간대의 패턴 파악
- v2 이후:
  - DeliveryRequest 생성 시점에
    - 요청 시간 + 예상 이동 시간 + 운행 종료 시간 검증
    - 운행이 끝난 구간/시간대에는 요청 차단 또는 경고

### 4.2 GTFS / 기타 교통 데이터

- 활용 시점:
  - SLA 분석, 지연 패턴, 피크타임 분석
  - 장기적으로 경로 최적화 및 시뮬레이션

---

## 5. Flutter App 구조 + 플로우

### 5.1 공통 구조

- STRV/Skelter 스타일 레이어드 아키텍처 참고:
  - `core/` – env, theme, router, error 핸들링
  - `common/` – 공통 위젯/유틸/서비스
  - `features/` – 도메인별 모듈 (auth, company, messenger, delivery)
- HTTP Client:
  - Dio + Interceptor (Auth / Logging / Error mapping)

### 5.2 Company 앱 주요 플로우

1. 로그인 (JWT 기반)
2. DeliveryRequest 생성
3. 상태 조회 (CREATED → MATCHING → MATCHED → IN_TRANSIT → COMPLETED)
4. 완료 시 알림(푸시 또는 카카오톡 등) 수신  
   → 실제 발송은 Asynq Job에서 처리

### 5.3 Messenger 앱 주요 플로우

1. 로그인 및 기본 정보/인증 완료
2. 출퇴근 루트(RoutePreference) 설정 (Station 선택)
3. 매칭 후보 목록 조회
4. 오더 수락 (ACCEPTED) → 픽업(PICKED) → 전달(DROPPED)
5. 완료 후 정산 대기 및 내역 조회

---

## 6. Delivery 상태 다이어그램

```txt
DeliveryRequest.status
  - CREATED     : 화주가 생성
  - MATCHING    : 메신저 후보에게 푸시 발송 중
  - MATCHED     : 한 명이 수락하여 매칭 확정
  - IN_TRANSIT  : 픽업 완료 후 이동 중
  - COMPLETED   : 도착역 인도 완료
  - CANCELED    : 취소/실패

DeliveryMatch.status
  - PENDING     : 후보에게 알림만 간 상태
  - ACCEPTED    : 메신저가 수락
  - PICKED      : 픽업 완료
  - DROPPED     : 도착역 인도 완료
  - FAILED      : 노쇼/사고 등 실패
  - CANCELED    : 취소
```

각 상태 변화는 `DeliveryEventLog`에 기록되어,  
분쟁/CS/리포트에 활용한다.

---

## 7. 배포 구조 (초기)

```txt
[Flutter App / Flutter Web]  →  [Go API]  →  [PostgreSQL]

- Go API: Railway / Render / Fly.io
- PostgreSQL: 같은 플랫폼 Managed Postgres 또는 Supabase/Neon
- Flutter Web: Vercel / Cloudflare Pages 등 정적 호스팅
```

- 인증/보안: 모든 통신은 HTTPS 전제
- 환경 분리: env 기반 `dev / staging / prod` 구성

---

## 8. MVP 범위 정리

### 포함

- Company / Messenger 기본 가입·인증
- Station 기본 데이터셋 로딩 및 검색
- DeliveryRequest 생성/조회
- 출퇴근 루트 기반 단순 Rule 매칭
- PICKED / DROPPED 상태 전환 + EventLog 기록
- Flutter 앱:
  - Company용: 배송 요청 생성/조회
  - Messenger용: 오더 수신/수락/진행/완료

### 제외 (후순위)

- 복잡한 동적 라우팅/최적화
- 다중 매칭/멀티 스탑
- 고급 정산/세금계산서/영수증 기능
- 실시간 위치 트래킹 (초기엔 상태 기반으로만 처리)

---
