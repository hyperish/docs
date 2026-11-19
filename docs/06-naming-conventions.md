# 06. 네이밍·코딩 컨벤션

> 상태: **제안** · 기준일: 2026-11-19

코드, API, DB, 문서 전반의 이름 규칙입니다. 개발 절차 규칙은 [개발 규칙](05-development-rules.md)을 봅니다. API·DB·메시지 명명 규칙은 언어와 무관하게 적용하고, 코드 규칙은 Rust(§3, 코어 전략상 기본)와 TypeScript(§4, 사용하는 레포)로 나눕니다.

## 0. 언어 규칙

| 대상 | 언어 |
|---|---|
| 코드 식별자, 커밋 타입 접두어, API 필드·경로, 에러 코드 | 영어 |
| 문서 본문, 주석, 커밋 설명 | 한국어 허용 (섞어도 되지만 한 문서 안에서는 일관되게) |

## 1. 케이스 규칙 요약

| 대상 | 규칙 | 예시 |
|---|---|---|
| 파일·디렉터리 (문서, 설정, 일반 모듈) | 소문자 kebab-case (단, Rust 소스 파일은 `snake_case.rs`, TS 컴포넌트 파일은 `PascalCase.tsx`) | `api-protocol.md`, `config-loader.ts` |
| Rust 크레이트 | 소문자 kebab-case (Cargo), snake_case (코드 내) — 공개 레지스트리(crates.io) 선점명 충돌 시 네임스페이스 전략은 [07-open-questions](07-open-questions.md) 따름 | `my-crate` / `my_crate` |
| Rust 타입·트레이트·열거형 및 배리언트 | PascalCase | `ParseError` |
| Rust 함수·변수·메서드·모듈·파일 | snake_case (단, UI 뷰 컴포넌트 매크로가 붙은 함수만 PascalCase 허용하며 매크로가 `#[allow(non_snake_case)]`를 자동 부여) | `load_config`, `scene_node.rs`, `CounterCard` |
| Rust 상수·static | SCREAMING_SNAKE_CASE | `MAX_RETRIES` |
| proc macro (derive) | PascalCase (Rust Reference, 2026-11-17 확인: https://doc.rust-lang.org/reference/procedural-macros.html) | `#[derive(MyItem)]` |
| proc macro (attribute, function-like) | snake_case | `#[my_attr]`, `define_item!` |
| TypeScript 클래스·컴포넌트·타입 | PascalCase | `ParseError` |
| TypeScript 함수·변수·메서드 | camelCase | `loadConfig` |
| 상수 (TS) | SCREAMING_SNAKE_CASE | `MAX_RETRIES` |
| 환경 변수 | SCREAMING_SNAKE_CASE, 접두어 | `HYPERISH_LOG_LEVEL` |
| API JSON 필드 | camelCase | `createdAt` |
| API URL 경로 | 소문자 kebab-case, 복수형 리소스 | `/v1/things` |
| API 에러 코드 | snake_case | `thing_not_found` |
| DB 테이블 | 소문자 snake_case, 복수형 | `things` |
| DB 컬럼 | 소문자 snake_case, 단수형 | `created_at` |
| 실시간 메시지 type | `대상.행위` 소문자 | `thing.update` |
| git 브랜치 | `타입/주제` kebab-case | `feat/dsl-parser` |
| git 태그·버전 | 단일 레포 릴리스는 `v` + SemVer, 개별 크레이트·패키지 독립 릴리스는 `<이름>-v` + SemVer | `v0.3.0`, `my-crate-v0.3.0` |

## 2. 식별자(ID)

- 외부에 노출되는 ID는 불투명한 문자열로 통일하고, 자동 증가 정수 ID는 외부에 노출하지 않습니다([03-api-protocol](03-api-protocol.md) §1).

## 3. Rust

- 표준 라이브러리와 rustfmt 기본 규칙, Rust API Guidelines(`RFC 430`)를 따릅니다.
- 변환 메서드는 비용과 소유권에 따라 `as_`(무비용 참조 변환), `to_`(할당·복사 변환), `into_`(소유권 소비 변환) 접두어를 구분합니다.
- `unsafe`는 꼭 필요한 곳에만 쓰고, 쓰는 곳마다 안전성 근거를 주석으로 남깁니다.
- proc macro 크레이트는 `proc-macro = true` 전용 크레이트로 분리하고, 사용자 노출은 진입 크레이트의 재수출로 통일합니다.
- DSL 키워드·매크로 이름은 짧은 일반어를 우선하되, 생태계 관례와 충돌하지 않게 합니다.
- 복수 프레임워크 레포의 `prelude::*`를 풀스택 코드에서 함께 임포트할 때 동일 심볼(`component()` 등)이 충돌하지 않도록 빌더·어트리뷰트 이름을 구분하거나 명확한 네임스페이스를 둡니다.

## 4. TypeScript (사용하는 레포)

- 타입·인터페이스는 PascalCase, 제네릭 파라미터는 단일 대문자 또는 PascalCase(`T`, `TValue`).
- 불리언은 `is`/`has`/`can` 접두어 (`isPublic`, `hasLicense`).
- 파일은 한 책임 하나. 일반 TS 모듈 파일은 `kebab-case.ts`로 짓고, 컴포넌트 파일명만 컴포넌트 이름과 같은 `PascalCase.tsx`로 짓습니다.

## 5. 명명할 때 피할 것

- 약어 남발 (`mgr` 같은 축약 금지. 널리 쓰이는 것만 허용: `id`, `url`, `api`).
- 자료형을 이름에 넣기 (`nameString` 금지).
- 의미 없는 범용어 단독 사용 (`data`, `info`, `temp` 금지).
