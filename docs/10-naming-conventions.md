# 10. 네이밍·코딩 컨벤션

> 상태: **제안** · 기준일: 2026-11-16

코드, API, DB, 문서 전반의 이름 규칙입니다. 개발 절차 규칙은 [개발 규칙](09-development-rules.md)을 봅니다.

**적용 범위**: 코드 관련 규칙(§3 등)은 TypeScript/JavaScript에 적용됩니다. 엔진 코어 언어는 아직 미정이므로([열린 결정](11-open-questions.md)), 그 언어의 컨벤션은 결정 후 별도 절이나 문서로 추가합니다. API·DB·토큰·메시지 명명 규칙은 언어와 무관하게 적용됩니다.

## 0. 언어 규칙

| 대상 | 언어 |
|---|---|
| 코드 식별자, 커밋 타입 접두어, API 필드·경로, 에러 코드 | 영어 |
| 문서 본문, 주석, 커밋 설명 | 한국어 허용 (섞어도 되지만 한 문서 안에서는 일관되게) |

## 1. 케이스 규칙 요약

| 대상 | 규칙 | 예시 |
|---|---|---|
| 파일·디렉터리 (문서, 에셋, 설정) | 소문자 kebab-case | `api-protocol.md`, `scene-loader.ts` |
| React 컴포넌트 / 클래스 / 타입 | PascalCase | `ViewportPanel.tsx`, `SceneNode` |
| 함수·변수·메서드 | camelCase | `loadScene`, `currentVersion` |
| 상수 | SCREAMING_SNAKE_CASE | `MAX_UPLOAD_BYTES` |
| 환경 변수 | SCREAMING_SNAKE_CASE, 제품 접두어 | `HYPERISH_API_BASE` |
| API JSON 필드 | camelCase | `projectId`, `createdAt` |
| API URL 경로 | 소문자 kebab-case, 복수형 리소스 | `/v1/share-links` |
| API 에러 코드 | snake_case | `project_not_found` |
| DB 테이블 | 소문자 snake_case, 복수형 | `projects`, `share_links` |
| DB 컬럼 | 소문자 snake_case, 단수형 | `created_at`, `owner_id` |
| CSS 커스텀 프로퍼티(토큰) | `--hp-범주-이름` kebab-case | `--hp-color-surface`, `--hp-space-2` |
| WebSocket 메시지 type | `도메인.행위` 소문자 | `scene.patch`, `presence.updated` |
| git 브랜치 | `타입/주제` kebab-case | `feat/scene-autosave` |
| git 태그·버전 | `v` + SemVer | `v0.3.0` |

## 2. 식별자(ID)

- 외부에 노출되는 ID는 **ULID** 문자열로 통일합니다([API 규격](06-api-protocol.md) §1). 자동 증가 정수 ID는 외부에 노출하지 않습니다.

## 3. TypeScript / React

- 타입·인터페이스는 PascalCase, 제네릭 파라미터는 단일 대문자 또는 PascalCase(`T`, `TValue`).
- 이벤트 핸들러 prop은 `on동사`(`onSelect`), 내부 구현은 `handle동사`(`handleSelect`).
- 불리언은 `is`/`has`/`can` 접두어 (`isPublic`, `hasLicense`).
- 파일은 한 책임 하나. 컴포넌트 파일명은 컴포넌트 이름과 같게 합니다.

## 4. 디자인 토큰

- 토큰 소스는 JSON 하나([디자인 시스템](04-design-system.md))이고, 키는 `범주.이름.변형` 순서로 깊어집니다: `color.surface.raised`, `space.2`, `font.mono`.
- CSS로 변환될 때 `--hp-` 접두어가 붙습니다: `color.surface.raised` → `--hp-color-surface-raised`.

## 5. WebSocket 메시지 type

- 형태: `도메인.행위`, 전부 소문자.
- 서버→클라이언트 통지: 과거형/상태형 (`scene.patched`, `presence.updated`).
- 클라이언트→서버 요청: 명령형 (`scene.patch`, `presence.subscribe`).
- 에러 응답은 고정 타입 `error` + [API 에러 코드](06-api-protocol.md).

## 6. 명명할 때 피할 것

- 약어 남발 (`scn`, `mgr` 같은 축약 금지. 널리 쓰이는 것만 허용: `id`, `url`, `api`).
- 자료형을 이름에 넣기 (`sceneList`, `nameString` 금지).
- 의미 없는 범용어 단독 사용 (`data`, `info`, `temp` 금지).
