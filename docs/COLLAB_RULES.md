
# 🤝 COLLAB RULES – 협업 규칙

> “나중에 사람 늘어나도 안 터지게, 처음부터 규칙을 문서로 박아둔다.”

---

## 1. 언어 정책

- 문서(README, docs/*): **한국어**
- 코드 / 주석 / 식별자 / 커밋 메시지: **영어**

---

## 2. 브랜치 전략

- `main`
  - 항상 배포 가능한 상태
- `develop`
  - 다음 배포 후보
- `feature/*`
  - 새로운 기능 (`feature/delivery-request-api`)
- `hotfix/*`
  - 운영 중 긴급 수정

> **직접 `main`에 push 금지.**  
> 모든 변경은 PR을 통해서만 반영한다.

---

## 3. Issue 규칙

- 모든 작업은 **반드시 GitHub Issue를 하나 잡고 시작**한다.
- 템플릿 예시:

```txt
[Type] Short summary

## Background
- Why this is needed

## Tasks
- [ ] Task 1
- [ ] Task 2

## Acceptance Criteria
- What should work when done
- Test conditions, etc.
```

- 태그 예시:
  - `type:feat`, `type:fix`, `type:chore`, `type:infra`, `type:docs`
  - `prio:high`, `prio:normal`, `prio:low`

---

## 4. 커밋 컨벤션

Conventional Commits 사용:

- `feat: add delivery request creation API`
- `fix: handle invalid station id`
- `docs: update architecture diagram`
- `chore: bump go version`
- `refactor: simplify matching service`

---

## 5. PR 규칙

- PR은 **작게** (이상적 200~300줄)
- PR 템플릿:

```txt
## Summary
- What & why

## Changes
- Bullet list of key changes

## Test
- [ ] go test ./...
- [ ] flutter test
- [ ] manual local test (describe)

## Related Issues
- Closes #123
```

- 리뷰 포인트:
  - 비즈니스 로직/도메인 용어 일관성
  - 에러 처리 / 로그 적절성
  - 테스트 포함 여부

---

## 6. 스타일 & 도구

- **Go**
  - `gofmt`, `golangci-lint`
- **Flutter**
  - `dart format`, `flutter analyze`
- **CI**
  - PR 생성 시 자동으로 lint + test 실행

---

## 7. 우선순위 규칙

1. 장애 / 버그 (서비스 멈추는 문제)
2. 핵심 플로우(MVP 배송 플로우)에 직결되는 기능
3. 보안 / 데이터 무결성
4. 개발 경험(DX) 개선
5. 리팩토링 / 아키텍처 개선
6. 실험적 기능

---

## 8. 의사결정 방법

- 슬랙/카톡에서 말로 합의한 것도, **최종은 Issue 또는 ADR로 기록**
- “감정/취향”보다 **데이터/운영/비즈니스 임팩트**를 우선한다.

---
