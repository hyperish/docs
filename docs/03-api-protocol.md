# 03. HTTP/JSON 어댑터 규약

> 상태: **제안** · 기준일: 2026-11-24

외부 호환 어댑터 중 **HTTP/JSON 어댑터를 채택하는 경우** 그 어댑터가 따르는 공통 규약입니다. 외부 호환이 반드시 HTTP/JSON인 것은 아닙니다. 코어·내부 규약 모델은 표준보다 성능과 수직 통합 생산성을 우선하는 스택을 쓰고([00-vision](00-vision.md)의 규약 모델 원칙), 외부 호환은 개별 어댑터로 해결하며, 이 문서는 그 어댑터 중 하나의 규약 제안입니다. 어댑터의 공통 계약은 [02-core-contracts](02-core-contracts.md) §5~§7을 따릅니다.

이 문서는 **전송·인증·오류·버전닝 같은 프로토콜 수준 규칙**만 다룹니다. 리소스 모델, 엔드포인트 목록, 수치 한도(수명, 크기, 속도)는 이 문서가 정하지 않고 이 어댑터를 구현하는 쪽이 정합니다.

## 1. 기본 전송 규약

| 항목 | 규약 |
|---|---|
| 전송 | HTTPS only. 평문 HTTP는 거부 |
| 본문 포맷 | JSON, UTF-8 인코딩 (RFC 8259, 2026-11-17 확인: https://www.rfc-editor.org/rfc/rfc8259.html) |
| 필드명 | `camelCase` |
| 경로명 | 소문자 `kebab-case` |
| 날짜·시각 | RFC 3339 / ISO 8601, UTC |
| ID | 외부에 노출하는 ID는 불투명한 문자열. 순서 가능한 ID(ULID 등)를 권장하며, 자동 증가 정수는 노출하지 않음 |
| 바이너리 | JSON 본문에 넣지 않음. 대용량 전송 원칙은 [02-core-contracts](02-core-contracts.md) §7 |
| Content-Type | 요청·응답 모두 `application/json` (RFC 8259상 charset 파라미터는 미정의이므로 필수로 요구하지 않음) |

## 2. URL 설계

- 버전 접두어를 둡니다(버전닝은 §7).
- 리소스 중심, 복수형 명사를 씁니다.
- 행위는 HTTP 메서드로 표현합니다: GET 조회 / POST 생성 / PATCH 부분 수정 / DELETE 삭제.
- 메서드로 표현 못 하는 행위만 동사형 하위 경로를 씁니다.

## 3. 인증과 권한

- 인증 방식은 클라이언트 종류에 맞춰 고르되, 자격 증명을 URL에 싣지 않습니다.
- 브라우저처럼 자격 증명이 자동 전송되는 클라이언트는 상태를 바꾸는 요청(POST/PATCH/DELETE)에 **CSRF 방어**를 둡니다. 세션 쿠키는 HttpOnly·Secure·SameSite를 설정하고, 세션 생성·조회 응답 본문으로 내려준 CSRF 토큰을 클라이언트가 `X-CSRF-Token` 헤더로 보내 검증합니다.
- 토큰 방식은 범위(scope)를 제한하고 개별 폐기가 가능해야 하며, 클라이언트는 이를 보안 저장소에 보관합니다.
- 세션 식별자, CSRF 토큰, 리프레시 토큰은 [02-core-contracts](02-core-contracts.md) §6의 자격 증명 값 규칙을 따릅니다. 세션 식별자는 CSPRNG로 만들고 엔트로피는 64비트 이상이어야 합니다(OWASP Session Management Cheat Sheet, 2026-11-24 확인: https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html). 실제 길이는 구현이 정하고 문서화합니다.
- 권한 검사와 철회 원칙은 [02-core-contracts](02-core-contracts.md) §6을 따릅니다.
- CORS는 허용한 Origin만 엽니다. `*`는 쓰지 않습니다. 크로스 서브도메인/크로스 오리진에서 쿠키 인증을 써야 하면 `Access-Control-Allow-Credentials: true`와 함께 정확한 Origin 화이트리스트 및 적절한 `SameSite` 정책을 명시합니다.

## 4. 응답 형식

- 성공: `{ "data": ... }`. 목록은 `{ "data": [...], "page": { "nextCursor": ..., "hasMore": ... } }`.
- 실패: `{ "error": { "code": "...", "message": "...", "details": { }, "requestId": "..." } }`.
- `code`는 `snake_case`의 안정된 문자열이며 클라이언트는 이 값으로 분기합니다. `message`는 사람용이라 파싱하지 않습니다.
- HTTP 상태 코드와 `code`를 함께 씁니다. 상태 코드만으로 의미를 전달하지 않습니다.
- 모든 오류 응답은 상태 코드와 무관하게 `error.requestId`에 상관관계 ID를 싣습니다.
- 재시도해도 되는 오류에는 `Retry-After`를 붙입니다. 클라이언트는 이 헤더의 유무로 재시도 가능 여부를 판단합니다.

| HTTP | 분류 | `code` 규칙 |
|---|---|---|
| 400 | 요청 형식 오류 | `invalid_request` |
| 401 | 인증 없음·만료 | `unauthenticated` |
| 403 | 권한 없음 | `forbidden`, `csrf_token_invalid` |
| 404 | 없음 | `<대상>_not_found`, `route_not_found` |
| 405 | 허용하지 않는 메서드 | `method_not_allowed` |
| 409 | 충돌 | `version_conflict`, `idempotency_key_mismatch`, `request_in_progress` |
| 413 | 본문 크기 초과 | `payload_too_large` |
| 415 | 받을 수 없는 본문 형식 | `unsupported_media_type` |
| 422 | 도메인 규칙 위반 | 도메인이 정한 `code` (위 행에 해당하지 않는 도메인 오류의 기본 상태) |
| 429 | 속도 제한 | `rate_limited` (`Retry-After` 필수) |
| 500 | 서버 오류 | `internal_error` |
| 503 | 일시적 인프라 장애 | `service_unavailable` (재시도 가능하면 `Retry-After`. RFC 9110 §15.6.4, 2026-11-24 확인: https://www.rfc-editor.org/rfc/rfc9110.html) |

## 5. 페이지네이션

- 커서 기반을 씁니다. 오프셋 페이지네이션은 데이터 변경 시 누락·중복이 생기므로 쓰지 않습니다.
- 기본 크기와 최대 크기는 구현이 정하고 문서화합니다.

## 6. 멱등성과 동시 수정

- 부수효과가 있는 생성성 요청은 `Idempotency-Key`를 받습니다. 키의 범위와 보관 기간은 구현이 정하고 문서화합니다.
  - 같은 키와 같은 본문의 재요청은 부수효과를 반복하지 않고 처음 응답을 돌려줍니다.
  - 같은 키에 다른 본문이 오면 `409 idempotency_key_mismatch`입니다.
  - 같은 키 요청이 처리 중이면 `409 request_in_progress`와 `Retry-After`를 돌려줍니다.
  - 멱등 키 조회와 응답 재사용은 인증·권한 검사 뒤에 수행하고, 키의 범위에는 인증 주체와 대상(메서드, 경로)을 포함합니다([02-core-contracts](02-core-contracts.md) §6).
- 수정이 필요한 대상은 `version`(정수)을 가지고, 수정 요청 본문에 기대 버전(`expectedVersion`)을 함께 보내 서버의 현재 버전과 다르면 `409 version_conflict`입니다(낙관적 잠금).

## 7. 버전닝

- 경로에 메이저 버전만 둡니다.
- 하위호환: 응답 필드 추가, 선택 요청 필드 추가.
- 하위호환 아님(메이저 증가 필요): 필수 요청 필드 추가, 필드 제거·이름 변경·의미 변경, 선택 필드의 필수화.
- 구버전은 폐기 전 일정 기간 병행하고 `Deprecation` 헤더와 문서로 고지합니다.

## 8. 속도 제한

- 속도 제한을 두는 경우 제한이 걸린 범위의 모든 응답에 헤더 `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset`(`Reset`은 리셋까지 남은 초 단위 정수, delta-seconds)을 싣습니다. 수치는 구현이 정합니다.
