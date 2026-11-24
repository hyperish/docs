---
name: hyperish-spec
description: 하이퍼리쉬 프레임워크 패밀리(hyve, orbic, serge 레포)에서 코드를 작성하거나 구조·DSL·어댑터·DX를 설계할 때 사용하는 범용 명세 스킬. 작업에 해당하는 명세 문서를 찾아 읽고, 문서 상태 규칙(확정/제안/미정)을 적용해 명세와 어긋나는 코드를 만들지 않는다. 하이퍼리쉬 관련 작업이면 항상 먼저 이 스킬을 연다.
---

# Hyperish 명세 스킬

빌드 에이전트가 하이퍼리쉬 코드를 작성하기 전에 범용 명세를 찾아 읽고 적용하기 위한 진입점이다.

## 명세 소스와 경로 해석

- 범용 명세의 단일 원천: **https://github.com/hyperish/docs** (public).
- 이 스킬에 적힌 문서 경로(`docs/...`)는 **docs 저장소 루트 기준**이다.
- 명세 내용을 다른 파일에 복사해 두지 않는다. 항상 원천 저장소에서 읽는다.

## docs가 다루는 것과 다루지 않는 것

- `hyperish/docs`는 **모든 제품과 레포가 공유하는 추상 계약과 공통 규칙**이다.
- 특정 제품의 설계나 특정 코드 레포(`hyve`, `orbic`, `serge`)의 세부 도메인·레퍼런스·크레이트 구조는 해당 레포의 `docs/00-overview.md` ~ `05-implementation-plan.md`에 있다.
- 필요한 세부가 문서에 없다고 지어내지 않는다. 미정 사항이나 설계 변경은 먼저 브리핑하고 리뷰 승인을 받은 뒤에만 반영한다.

## 명세 revision 고정 (재현성)

- 작업을 시작할 때 `hyperish/docs`의 **커밋 SHA를 기록**하고(`git -C docs rev-parse HEAD`), 작업 결과물(PR 설명, 커밋 메시지 본문 `Spec-Revision: hyperish/docs@<SHA>`)에 남긴다.

## 문서 상태 규칙 (가장 먼저 적용)

- **확정**: 명시적으로 결정된 사실. 확정과 충돌하는 구조·코드를 작성하거나 임의로 바꾸지 않는다.
- **제안**: 따르되, 더 나은 대안이 있으면 근거와 함께 제시하고 리뷰 승인을 받는다.
- **미정**: 임의로 정하지 않는다. 공통 미정은 `docs/07-open-questions.md`, 레포별 미정은 각 레포의 `docs/04-open-questions.md`를 확인하고 질문한다.

## 작업별로 읽을 문서

| 하려는 작업 | 읽을 문서 |
|---|---|
| 범위 판단, 레포 역할표, 확정된 엔지니어링 방향, 충돌 해소 기준, Phase 완료 기준 확인 | `docs/00-vision.md` |
| DX 설계, 컴포넌트 조립성·아키텍처 고점·표현 수단 판단 | `docs/01-dx-principles.md` |
| 계층을 넘는 설계 (코어 `ref` 소유권, 오류, 취소, 동시성, 어댑터 경계, 권한·철회, 대용량 전송) | `docs/02-core-contracts.md` |
| HTTP/JSON 어댑터 채택 시 | `docs/03-api-protocol.md` |
| WebSocket 어댑터 채택 시 | `docs/04-realtime-communication.md` |
| 모노레포 구조(`apps/`, `crates/`), 브랜치, 커밋, 버전닝, 포크·`unit optimization` 절차 | `docs/05-development-rules.md` |
| 이름 짓기 및 언어별 코드 컨벤션 | `docs/06-naming-conventions.md` |
| 공통 미정 사항 확인 | `docs/07-open-questions.md` |
| 현재 Phase의 작업 순서와 레포별 완료 기준 | **해당 코드 레포의 `docs/05-implementation-plan.md`** |
| 개별 코드 레포(`hyve`, `orbic`, `serge`)의 도메인·DX·크레이트·어댑터 세부 | **해당 코드 레포의 `README.md` 및 `docs/00`~`05`** |

## 작업 절차

1. 명세 revision을 고정한다.
2. 작업에 해당하는 범용 명세(`hyperish/docs`)와 대상 코드 레포의 문서(`docs/00`~`05`)를 찾아 전문을 읽는다.
3. 확정 사항은 따르고, 제안 변경이나 미정 사항 결정이 필요하면 먼저 안을 제시해 승인을 받은 뒤 반영한다.
4. 커밋은 `docs/05-development-rules.md`를 따른다: Conventional Commits, 한 커밋은 한 논리 변경.
