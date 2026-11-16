# 03. ring: 백엔드

> 상태: 레포 역할은 **확정**(2026-11-16), 구조·절차는 **제안** · 기준일: 2026-11-16

ring은 하이퍼리쉬의 백엔드 레포입니다([00-vision](00-vision.md)). 공통 계약은 [06-core-contracts](06-core-contracts.md)를 따릅니다.

## 역할 (확정)

- 하이퍼리쉬 서비스의 백엔드를 담당합니다.

## 계층 책임 (제안)

ring이 소유하는 것과 소유하지 않는 것을 나눕니다.

- **소유**: 인증·세션, 리소스의 수명 관리(생성·보관·삭제), 외부 어댑터의 서빙, 실시간 룸의 조정, 저장소 바인딩.
- **소유하지 않음**: 도메인 모델의 정의(코어는 hyper가 소유). ring은 코어 모델을 변환해 저장·전달할 뿐, 모델의 의미를 재정의하지 않습니다.

## 구성과 의존 방향 (제안)

```
ring/
  core/       # 서버 코어: 내부 모델, 상태, 수명 관리 (외부 규약을 모름)
  adapters/   # 외부 호환 어댑터들 (예: HTTP/JSON, WebSocket)
  rooms/      # 실시간 룸 조정자
  storage/    # 저장소 바인딩
```

- 의존 방향: adapters → core. core는 adapters를 모릅니다. 외부 규약의 타입이 core에 스며들지 않게 합니다([06-core-contracts](06-core-contracts.md) §1, §5).
- 어댑터가 외부 규약으로 **HTTP/JSON이나 WebSocket을 채택하는 경우**, 각각 [07-api-protocol](07-api-protocol.md)과 [08-realtime-communication](08-realtime-communication.md)의 제안 규약을 따릅니다. 이 둘은 채택 시 적용할 규약이며, 외부 호환이 항상 HTTP/WS라는 뜻은 아닙니다.
- 내부 모델 간 통신은 성능·수직 통합 생산성 우선 원칙([00-vision](00-vision.md))에 따라 별도 스택을 쓸 수 있습니다. 구체 스택은 미정입니다.

## 빌드·테스트·진단 흐름 (제안)

- **테스트**:
  - core: 단위 테스트(수명·상태 규칙).
  - adapters: 외부 규약 문서의 형식·에러 코드·버전닝을 검증하는 계약 테스트. 규약 문서가 테스트의 원천입니다.
  - 멱등성·재연결: [07](07-api-protocol.md) §6, [08](08-realtime-communication.md) §6의 시나리오를 그대로 재현하는 통합 테스트.
- **진단**: 모든 외부 응답에 상관관계 ID를 포함하고, 구조화 로그로 requestId를 이어 붙입니다([06-core-contracts](06-core-contracts.md) §2).
- **오류·취소·동시성**: [06-core-contracts](06-core-contracts.md) §2~§4를 따릅니다. 룸 조정자 구조는 §4의 동시성 메커니즘 중 하나이며, 룸 상태에 맞는 구조를 선택합니다.

## 미정: ring과 기존 Cloudflare 인프라 검토안의 관계

이전 검토안은 계정·메타데이터·에셋 같은 서비스를 Cloudflare(Workers + Hono, D1, R2)로 구성하는 방향이었습니다. ring(백엔드) 레포의 생성은 이 역할 분담이 다시 열렸음을 뜻합니다.

- ring이 Cloudflare 구성을 **대체**하는지, 아니면 Cloudflare가 담당하던 영역과 **공존·분담**하는지는 미정입니다([열린 결정](11-open-questions.md)).
- 결정 전까지는 어느 한쪽을 폐기하는 구조로 설계하지 않습니다. 저장소 바인딩(storage/)을 교체 가능하게 두어 결정이 늦어져도 core가 막히지 않게 합니다.
