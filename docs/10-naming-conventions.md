# 10. 네이밍·코딩 컨벤션

> 상태: **제안** · 기준일: 2026-11-16 (2026-11-16 Rust 절 추가)

코드, API, DB, 문서 전반의 이름 규칙입니다. 개발 절차 규칙은 [개발 규칙](09-development-rules.md)을 봅니다.

**적용 범위**: API·DB·메시지 명명 규칙은 언어와 무관하게 적용합니다. 코드 규칙은 Rust(§3, 코어 전략상 기본)와 TypeScript(§4, 사용하는 레포)로 나눕니다.

## 0. 언어 규칙

| 대상 | 언어 |
|---|---|
| 코드 식별자, 커밋 타입 접두어, API 필드·경로, 에러 코드 | 영어 |
| 문서 본문, 주석, 커밋 설명 | 한국어 허용 (섞어도 되지만 한 문서 안에서는 일관되게) |

## 1. 케이스 규칙 요약

| 대상 | 규칙 | 예시 |
|---|---|---|
| 파일·디렉터리 (문서, 에셋, 설정) | 소문자 kebab-case | `api-protocol.md` |
| Rust 크레이트 | 소문자 kebab-case (Cargo), snake_case (코드 내) | `hyper-macros` / `hyper_macros` |
| Rust 타입·트레이트·열거형 | PascalCase | `SceneNode`, `Renderer` |
| Rust 함수·변수·메서드·모듈 | snake_case | `load_scene`, `current_version` |
| Rust 상수·static | SCREAMING_SNAKE_CASE | `MAX_UPLOAD_BYTES` |
| proc macro | snake_case (속성·함수형 매크로명) | `#[scene]`, `define_node!` |
| TypeScript 클래스·컴포넌트·타입 | PascalCase | `ViewportPanel.tsx`, `SceneNode` |
| TypeScript 함수·변수·메서드 | camelCase | `loadScene`, `currentVersion` |
| 상수 (TS) | SCREAMING_SNAKE_CASE | `MAX_UPLOAD_BYTES` |
| 환경 변수 | SCREAMING_SNAKE_CASE, 제품 접두어 | `HYPERISH_API_BASE` |
| API JSON 필드 | camelCase | `projectId`, `createdAt` |
| API URL 경로 | 소문자 kebab-case, 복수형 리소스 | `/v1/share-links` |
| API 에러 코드 | snake_case | `project_not_found` |
| DB 테이블 | 소문자 snake_case, 복수형 | `projects`, `share_links` |
| DB 컬럼 | 소문자 snake_case, 단수형 | `created_at`, `owner_id` |
| CSS 커스텀 프로퍼티(토큰) | `--hp-범주-이름` kebab-case | `--hp-color-surface`, `--hp-space-2` |
| WebSocket 메시지 type | `도메인.행위` 소문자 | `scene.patch`, `presence.updated` |
| git 브랜치 | `타입/주제` kebab-case | `feat/dsl-parser` |
| git 태그·버전 | `v` + SemVer | `v0.3.0` |

## 2. 식별자(ID)

- 외부에 노출되는 ID는 **ULID** 문자열로 통일합니다([API 규격](07-api-protocol.md) §1). 자동 증가 정수 ID는 외부에 노출하지 않습니다.

## 3. Rust

- 표준 라이브러리와 rustfmt 기본 규칙을 따릅니다.
- `unsafe`는 꼭 필요한 곳에만 쓰고, 쓰는 곳마다 안전성 근거를 주석으로 남깁니다.
- proc macro 크레이트는 `proc-macro = true` 전용 크레이트로 분리하고, 사용자 노출은 진입 크레이트의 재수출로 통일합니다([02-hyper-framework](02-hyper-framework.md) 구조 제안).
- DSL 키워드·매크로 이름은 짧은 일반어를 우선하되, 생태계 관례와 충돌하지 않게 합니다.

## 4. TypeScript (사용하는 레포)

- 타입·인터페이스는 PascalCase, 제네릭 파라미터는 단일 대문자 또는 PascalCase(`T`, `TValue`).
- 이벤트 핸들러 prop은 `on동사`(`onSelect`), 내부 구현은 `handle동사`(`handleSelect`).
- 불리언은 `is`/`has`/`can` 접두어 (`isPublic`, `hasLicense`).
- 파일은 한 책임 하나. 컴포넌트 파일명은 컴포넌트 이름과 같게 합니다.

## 5. 디자인 토큰

- 토큰 소스는 JSON 하나로 두고, 키는 `범주.이름.변형` 순서로 깊어집니다: `color.surface.raised`, `space.2`, `font.mono`.
- CSS로 변환될 때 `--hp-` 접두어가 붙습니다: `color.surface.raised` → `--hp-color-surface-raised`.
- 토큰 체계의 위치(어느 레포가 소유하는가)는 미정입니다([열린 결정](11-open-questions.md)).

## 6. WebSocket 메시지 type

- 형태: `도메인.행위`, 전부 소문자.
- 서버→클라이언트 통지: 과거형/상태형 (`scene.patched`, `presence.updated`).
- 클라이언트→서버 요청: 명령형 (`scene.patch`, `presence.subscribe`).
- 에러 응답은 고정 타입 `error` + [API 에러 코드](07-api-protocol.md).

## 7. 명명할 때 피할 것

- 약어 남발 (`scn`, `mgr` 같은 축약 금지. 널리 쓰이는 것만 허용: `id`, `url`, `api`).
- 자료형을 이름에 넣기 (`sceneList`, `nameString` 금지).
- 의미 없는 범용어 단독 사용 (`data`, `info`, `temp` 금지).
