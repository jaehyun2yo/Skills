# API Specification Template — API 명세서

## 문서 구조

```markdown
# [프로젝트명] — API 명세서 (API Specification)

> 버전: 1.0 | 작성일: YYYY-MM-DD | 기반 문서: SDS v1.0, FUNC-SPEC v1.0

---

## 1. 개요

### 1.1 API 유형
- REST / GraphQL / gRPC / WebSocket (프로젝트에 맞게 선택)

### 1.2 Base URL
- Production: `https://api.example.com/v1`
- Staging: `https://api-staging.example.com/v1`

### 1.3 공통 규칙

#### 인증
```
Authorization: Bearer <access_token>
```
인증이 필요 없는 엔드포인트는 개별 명시.

#### 요청 형식
- Content-Type: `application/json`
- 날짜 형식: ISO 8601 (`YYYY-MM-DDTHH:mm:ssZ`)
- 페이지네이션: cursor 기반 / offset 기반

#### 공통 응답 형식
```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "page": 1,
    "total": 100
  }
}
```

#### 공통 에러 형식
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "이메일 형식이 올바르지 않습니다.",
    "details": [
      { "field": "email", "reason": "invalid_format" }
    ]
  }
}
```

#### HTTP 상태 코드
| 코드 | 의미 | 사용 상황 |
|------|------|----------|
| 200 | OK | 조회/수정 성공 |
| 201 | Created | 리소스 생성 성공 |
| 204 | No Content | 삭제 성공 |
| 400 | Bad Request | 요청 파라미터 오류 |
| 401 | Unauthorized | 인증 실패 |
| 403 | Forbidden | 권한 없음 |
| 404 | Not Found | 리소스 없음 |
| 409 | Conflict | 중복/충돌 |
| 422 | Unprocessable Entity | 유효성 검증 실패 |
| 429 | Too Many Requests | 레이트 리밋 초과 |
| 500 | Internal Server Error | 서버 오류 |

#### Rate Limiting
- 인증 유저: X req/min
- 비인증: Y req/min
- 헤더: `X-RateLimit-Remaining`, `X-RateLimit-Reset`

---

## 2. API 엔드포인트

### 2.1 [도메인/리소스 그룹명] (예: 인증)

#### API-001: [엔드포인트 제목]

| 항목 | 내용 |
|------|------|
| Method | `POST` |
| Path | `/auth/login` |
| 인증 | 불필요 |
| 관련 기능 | FUNC-001 |
| 관련 요구사항 | REQ-F-001 |

**Request**

Headers:
```
Content-Type: application/json
```

Body:
| 필드 | 타입 | 필수 | 설명 | 유효성 | 예시 |
|------|------|------|------|--------|------|
| email | string | Y | 사용자 이메일 | 이메일 형식 | "user@example.com" |
| password | string | Y | 비밀번호 | 8자 이상 | "********" |

```json
{
  "email": "user@example.com",
  "password": "securePassword123!"
}
```

**Response — 200 OK**
| 필드 | 타입 | 설명 |
|------|------|------|
| access_token | string | JWT 액세스 토큰 |
| refresh_token | string | 리프레시 토큰 |
| expires_in | number | 토큰 만료까지 초 |

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbG...",
    "refresh_token": "dGhpcyBp...",
    "expires_in": 3600
  }
}
```

**Error Responses**
| 상황 | 코드 | error.code | 설명 |
|------|------|-----------|------|
| 잘못된 형식 | 400 | VALIDATION_ERROR | 이메일/비밀번호 형식 오류 |
| 인증 실패 | 401 | INVALID_CREDENTIALS | 이메일/비밀번호 불일치 |
| 계정 잠김 | 403 | ACCOUNT_LOCKED | 5회 이상 실패로 잠김 |

---

#### API-002: [엔드포인트 제목]
(위와 동일한 구조)

---

## 3. WebSocket 이벤트 (해당 시)

### 연결
```
wss://api.example.com/ws?token=<access_token>
```

### 이벤트 목록
| 이벤트 | 방향 | 설명 | 페이로드 |
|--------|------|------|---------|
| message.new | Server → Client | 새 메시지 | `{ id, content, sender }` |
| typing.start | Client → Server | 타이핑 시작 | `{ channel_id }` |

---

## 4. API 추적

| API ID | FUNC ID | REQ ID | Method | Path |
|--------|---------|--------|--------|------|
| API-001 | FUNC-001 | REQ-F-001 | POST | /auth/login |
| API-002 | FUNC-001 | REQ-F-002 | POST | /auth/refresh |

---

## 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| 1.0 | YYYY-MM-DD | 초안 작성 | [이름] |
```

## 작성 가이드

- **요청/응답 예시 필수**: 스키마만이 아니라 실제 JSON 예시를 반드시 포함. 개발자가 가장 먼저 보는 것이 예시.
- **에러를 소홀히 하지 않는다**: 모든 엔드포인트에 가능한 에러 응답을 나열. 프론트엔드 개발자가 에러 처리를 구현하는 데 이 정보가 필수.
- **일관된 네이밍**: URL은 kebab-case, JSON 필드는 snake_case 등 프로젝트 컨벤션을 정하고 일관되게 적용.
- **버전 전략 명시**: URL 기반 (`/v1/`) vs 헤더 기반 vs 미사용. 초기에 결정해야 나중에 고통이 줄어든다.
- **GraphQL의 경우**: 엔드포인트 대신 Query/Mutation/Subscription 단위로 작성. 스키마 정의와 resolver 설명 포함.
