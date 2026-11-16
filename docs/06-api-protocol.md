# 06. API 프로토콜 규격

> 상태: **제안** · 기준일: 2026-11-16

서비스 API(계정·프로젝트·공유·에셋)의 프로토콜 규약입니다. "proto"는 **프로토콜 규격**을 의미하며, protobuf 채택을 뜻하지 않습니다. protobuf 같은 바이너리 프로토콜은 엔진 코어 언어·타깃 결정 이후 SDK 영역에서 별도 검토합니다. 실시간 통신(WebSocket) 규약은 [07-realtime-communication](07-realtime-communication.md)이 담당합니다.

## 1. 기본 전송 규약

| 항목 | 규약 |
|---|---|
| 전송 | HTTPS only. 평문 HTTP는 거부 |
| 본문 포맷 | JSON, UTF-8 |
| 필드명 | `camelCase` |
| 경로명 | 소문자 `kebab-case` |
| 날짜·시각 | RFC 3339 / ISO 8601, UTC (`2026-11-16T01:00:00Z`) |
| ID | ULID 문자열 (`01J...`), 생성 시각 정렬 가능 |
| 바이너리 | API 본문에 넣지 않음. 에셋은 R2 직접 업로드 URL로 분리(§8) |
| Content-Type | 요청·응답 모두 `application/json; charset=utf-8` |

## 2. URL 설계

- 버전 접두어: `/v1/...` (버전닝은 §7)
- 리소스 중심, 복수형 명사: `/v1/projects`, `/v1/projects/{id}/versions`
- 행위는 HTTP 메서드로 표현: GET 조회 / POST 생성 / PATCH 부분 수정 / DELETE 삭제
- 메서드로 표현 못 하는 행위만 동사형 하위 리소스: `POST /v1/projects/{id}/share-links`, `POST /v1/assets/{id}/resume-upload`

예시 자원(초기 범위):

```
GET    /v1/me
GET    /v1/projects
POST   /v1/projects
GET    /v1/projects/{projectId}
PATCH  /v1/projects/{projectId}
DELETE /v1/projects/{projectId}
GET    /v1/projects/{projectId}/versions
POST   /v1/projects/{projectId}/versions
POST   /v1/projects/{projectId}/share-links
DELETE /v1/share-links/{shareLinkId}
GET    /v1/shared/{token}              # 공유 링크로 보기 (무인증)
POST   /v1/assets/upload-urls          # R2 직접 업로드용 서명 URL 발급
POST   /v1/assets/{assetId}/complete   # 업로드 완료 통보 (§8)
```

## 3. 인증

클라이언트 종류에 따라 두 가지 방식을 씁니다.

### 웹 (사이트·에디터): 쿠키 세션

- 로그인하면 서버가 세션을 **HttpOnly + Secure + SameSite=Lax(또는 Strict) 쿠키**로 내려줍니다. HttpOnly는 JS가 쿠키 값을 읽는 것을 막아 세션 토큰 탈취 경로를 줄일 뿐, XSS 자체를 방어하지는 않습니다.
- 브라우저는 크로스 사이트 요청에도 쿠키를 **자동으로 실어 보내므로**, 상태를 바꾸는 요청(POST/PATCH/DELETE)에는 **CSRF 방어**가 필요합니다: 서버가 세션에 묶인 CSRF 토큰을 발급·보관하고(synchronizer token 방식), 클라이언트는 그 토큰을 `X-CSRF-Token` 헤더로 보내며, 서버는 세션의 값과 비교 검증합니다. SameSite 속성과 함께 사용합니다.
- 세션은 짧은 수명(제안: 접근 세션 1시간)과 갱신 세션(제안: 30일, 사용 시 회전)의 조합.

### CLI·SDK: Bearer 토큰

- `Authorization: Bearer <sdkToken>` 헤더.
- 장기 토큰에 스코프(`read:projects`, `write:projects` 등)를 부여하고, 사용자 설정에서 개별 폐기 가능.
- CLI는 토큰을 OS 키체인 등 보안 저장소에 보관하고 평문 파일에 두지 않습니다.

### 공통

- **권한 검사**: 모든 요청은 서버가 리소스 소유자·역할을 검사합니다. 공유 링크 토큰(`/v1/shared/{token}`)은 해당 프로젝트 열람 전용이며 다른 API에는 쓸 수 없습니다. 역할 모델은 [저장·공유·권한](08-storage-sharing.md) §6.
- **CORS**: API는 허용된 Origin(사이트·에디터 도메인)만 허용합니다. 무인증 공개 경로에도 `*`는 쓰지 않고 필요한 Origin만 열어 두는 것을 원칙으로 합니다.
- 무인증 접근이 허용되는 것은 공유 링크 조회처럼 명시된 경로뿐입니다.

## 4. 응답 형식

### 성공

```json
{ "data": { "id": "01J...", "name": "새 프로젝트" } }
```

목록:

```json
{
  "data": [ { "id": "01J..." } ],
  "page": { "nextCursor": "01J...", "hasMore": true }
}
```

### 실패

```json
{
  "error": {
    "code": "project_not_found",
    "message": "프로젝트를 찾을 수 없습니다.",
    "details": { "projectId": "01J..." }
  }
}
```

- `code`: `snake_case`의 안정된 문자열. 클라이언트는 이 값으로 분기하며, `message`는 사람용이라 파싱하지 않습니다.
- HTTP 상태 코드와 `code`는 함께 씁니다. 상태 코드만으로 의미를 전달하지 않습니다.

### 에러 코드 체계

| HTTP | code 예시 | 의미 |
|---|---|---|
| 400 | `invalid_request` | 본문·파라미터 형식 오류 |
| 401 | `unauthenticated` | 인증 없음·만료 |
| 403 | `forbidden` | 권한 없음 (CSRF 토큰 불일치 포함) |
| 404 | `*_not_found` | 리소스 없음 |
| 409 | `version_conflict` | 동시 수정 충돌(버전 불일치) |
| 409 | `idempotency_key_mismatch` | 같은 멱등 키에 다른 본문 |
| 409 | `request_in_progress` | 같은 멱등 키 요청이 처리 중. `Retry-After` 헤더 동봉 |
| 413 | `asset_too_large` | 업로드 크기 초과 |
| 429 | `rate_limited` | 레이트리밋. `Retry-After` 헤더 필수 |
| 500 | `internal_error` | 서버 오류. `requestId` 포함 |

## 5. 페이지네이션

- **커서 기반**: `?cursor=...&limit=50`. `limit` 기본 50, 최대 100.
- 오프셋 페이지네이션은 쓰지 않습니다(데이터 변경 시 누락·중복).

## 6. 멱등성과 동시성

- 생성·업로드 완료 요청은 `Idempotency-Key` 헤더를 받습니다.
  - 키의 유효 범위는 **사용자 + 엔드포인트**이고, 보관 수명은 24시간(제안)입니다. 수명이 지난 키의 재요청은 새 요청으로 처리합니다.
  - **완료된 요청의 재생**: 같은 키 + 같은 본문으로 다시 보내면 서버는 부수효과를 반복하지 않고 **저장해 둔 첫 응답을 그대로 반환**합니다(상태 코드 포함).
  - 같은 키에 **다른 본문**이 오면 `409 idempotency_key_mismatch`.
  - 같은 키 요청이 **아직 처리 중**이면 `409 request_in_progress`와 `Retry-After`를 반환하고, 클라이언트는 잠시 뒤 같은 키로 다시 보냅니다.
- 리소스는 `version`(정수) 필드를 가집니다. 수정 요청은 기대 버전을 함께 보내고, 다르면 `409 version_conflict`입니다(낙관적 잠금).

## 7. 버전닝

- 경로에 메이저 버전만: `/v1`.
- **하위호환(같은 버전에서 허용)**: 응답 필드 추가, 선택적(optional) 요청 필드 추가.
- **하위호환 아님(메이저 버전 증가 필요)**: 필수 요청 필드 추가, 필드 제거·이름 변경·의미 변경, 기존 선택 필드의 필수화.
- 구버전 폐기는 최소 1개 메이저 버전 동안 병행하고, `Deprecation` 헤더와 문서로 고지합니다.

## 8. 업로드, 레이트리밍과 크기 제한

- 레이트리밍 응답 헤더: `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset`.
- API JSON 본문은 1MB 이하를 원칙으로 합니다. 장면 본문·에셋 바이너리는 R2로:
  1. `POST /v1/assets/upload-urls`로 서명된 업로드 URL을 받는다.
  2. 클라이언트가 R2에 직접 올린다. 5GiB 이상 파일은 멀티파트와 이어 올리기를 사용([에디터 UX](02-editor-ux.md)의 복구 규칙과 연결).
  3. `POST /v1/assets/{assetId}/complete`로 완료를 알린다. 서버는 **객체 존재·크기·체크섬을 확인한 뒤** 에셋을 활성화한다. 완료 통보가 왔다고 검증 없이 활성화하지 않는다.
- 완료 통보 없이 남은 업로드(고아 객체)는 주기적으로 정리합니다(제안: 7일 유예 후 삭제).

## 9. 공유 콘텐츠 서빙과 철회

- **R2 버킷이나 객체를 직접 공개하지 않습니다.** 공유 콘텐츠(공유 링크 페이지, 렌더 결과물 포함)는 항상 Workers를 거쳐 서빙하고, Worker가 **요청마다 권한과 철회 여부를 확인**합니다.
- 따라서 공유 링크 삭제·비활성화는 다음 요청부터 **즉시 차단**됩니다. 이 보장의 경로는 고정입니다: 공유 콘텐츠 응답은 **no-store**(외부 CDN 캐시에 맡기지 않음) + **요청별 권한·철회 검사**. 성능상 캐시를 도입하더라도 그것은 이 보장과 분리된 비공개·인증 콘텐츠용 최적화로만 다룹니다.
- 다운로드용 R2 서명 URL을 발급하는 경우: URL의 남은 수명 동안은 철회 후에도 접근이 가능하므로, 서명 URL 수명을 짧게 제한(제안: 15분 이하)하고 이 잔여 노출을 문서에 명시합니다.

## 10. 삭제

- `DELETE`는 즉시 영구 삭제가 아니라 보관(휴지통)으로 이동시키는 soft delete입니다. 보관 기간과 복구 규칙은 [저장·공유·권한](08-storage-sharing.md) §7을 따릅니다.
