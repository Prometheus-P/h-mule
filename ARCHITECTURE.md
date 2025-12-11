

# 🏗 ARCHITECTURE – Human Mule

---

## 1. 도메인 개요

핵심 도메인:

- **Company**: 검증된 B2B 화주(법인/개인사업자)
- **CompanyUser**: Company 소속 사용자(직원)
- **Messenger**: 출퇴근 루트 기반 배송 수행하는 일반 사용자
- **Station**: 서울 지하철 역
- **RoutePreference**: 메신저 출퇴근 루트/시간대 선호
- **DeliveryRequest**: 화주가 생성한 배송 요청
- **DeliveryMatch**: 특정 메신저와 매칭된 건
- **DeliveryEventLog**: 상태 변화/이벤트 기록

---

## 2. 간단 ERD (텍스트)

```txt
Company (id, name, biz_reg_no, email, status, created_at, ...)

CompanyUser (id, company_id, email, name, role, password_hash, ...)

Messenger (id, phone, name, status, verified_at, ...)

Station (id, line, name_ko, name_en, code, lat, lng, ...)

RoutePreference (
  id, messenger_id,
  from_station_id, to_station_id,
  time_window_start, time_window_end,
  days_of_week, is_active
)

DeliveryRequest (
  id, company_id,
  from_station_id, to_station_id,
  item_type, price_total,
  status, requested_pickup_time,
  requested_drop_time,
  created_at, ...
)

DeliveryMatch (
  id, delivery_request_id, messenger_id,
  status,
  accepted_at, picked_at, dropped_at, canceled_at
)

DeliveryEventLog (
  id, delivery_match_id,
  event_type,               -- CREATED, MATCHED, PICKED, DROPPED, CANCELED ...
  event_payload_json,
  created_at
)
3. Boundary & API
3.1 Flutter App ↔ Go API
통신: HTTPS + JSON

인증:

/auth/login → JWT(access/refresh) 발급

토큰은 Flutter Dio interceptor에서 자동 첨부

3.2 주요 Endpoint 예시
/api/v1/company/*

기업 등록, 사업자번호 인증, 사용자 관리

/api/v1/messenger/*

메신저 등록, KYC 수준(휴대폰 인증), 루트 설정

/api/v1/station/*

역 리스트/검색

/api/v1/delivery/*

요청 생성/조회/취소

/api/v1/delivery/match/*

후보 메신저 조회, 수락/거절, 상태 변경

4. Delivery 상태 다이어그램
txt
코드 복사
DeliveryRequest.status
  - CREATED      (화주가 생성)
  - MATCHING     (메신저 후보에게 푸시)
  - MATCHED      (한 명이 수락)
  - IN_TRANSIT   (픽업 완료)
  - COMPLETED    (도착역 인도 완료)
  - CANCELED     (취소/실패)

DeliveryMatch.status
  - PENDING
  - ACCEPTED
  - PICKED
  - DROPPED
  - FAILED
  - CANCELED
5. MVP 범위
포함
Company / Messenger 기본 회원가입 & 인증

Station 기본 데이터셋

DeliveryRequest 생성/조회

간단 Rule-based 매칭:

루트(from→to) + 시간대 겹치는 메신저

PICKED / DROPPED 상태 전환 + EventLog 기록

Flutter 앱:

Company용: 요청 생성/상태 조회

Messenger용: 오더 수신/수락/상태 변경

제외(후순위)
복잡한 동적 라우팅/최적화

다중 매칭/멀티 스탑

정산/세금계산서/영수증 고도화

실시간 위치 트래킹(초기엔 상태 기반)

6. 배포 구조 (초기)
txt
코드 복사
[Flutter App / Flutter Web]  →  [Go API]  →  [PostgreSQL]

- Go API: Railway / Render / Fly.io
- PostgreSQL: Managed (같은 플랫폼 or Supabase/Neon)
- Flutter Web: Vercel / Cloudflare Pages 등 정적 호스팅
인증/보안은 모두 HTTPS 전제

env 기반으로 dev / staging / prod 분리
