# 05. 아키텍처: 기술 스택과 Cloudflare 인프라

> 상태: 스택 선택은 **제안**, Cloudflare 구성은 **검토 중인 방향**(미확정), 한도 수치는 2026-11-16 공식 문서 확인 · 기준일: 2026-11-16

## 스택은 두 덩어리로

"스택"은 하나가 아니라 목적이 다른 두 덩어리로 나눠 정합니다. 둘을 섞어 정하면 나중에 한쪽을 바꾸기 어려워집니다.

| 덩어리 | 범위 | 상태 |
|---|---|---|
| 웹 서비스 스택 | 랜딩·문서·갤러리 사이트, 계정·공유·프로젝트 API, 에디터 앱 셸 | 제안 (아래 추천) |
| 엔진 코어 스택 | 그래픽 엔진의 언어와 타깃 플랫폼 | **미정** |

엔진 코어 언어·타깃은 되돌리기 가장 비싼 결정이므로 이 문서에서 정하지 않습니다. 두 덩어리를 분리해 두면, 엔진 결정이 늦어져도 랜딩·계정·공유 같은 서비스는 먼저 만들 수 있습니다.

## 웹 서비스 스택 (제안)

| 구성 | 추천 | 이유 |
|---|---|---|
| 랜딩·문서·갤러리 | **Astro** | 기본이 정적이라 빠르고, 섬(island) 구조라 페이지 대부분은 가볍게 두고 랜딩의 엔진 데모처럼 무거운 부분만 인터랙티브로 심을 수 있음. Cloudflare 공식 프레임워크 가이드와 공식 어댑터(@astrojs/cloudflare) 있음 |
| 서비스 API | **Workers + Hono** | 계정·공유·프로젝트 API. Workers의 표준 Fetch API 위에서 가볍게 돌고 TypeScript로 쓰며, Wrangler로 그대로 배포. D1·R2 바인딩과 같은 런타임 |
| 에디터 앱 | **React + Vite** | DS 컴포넌트를 React로 만들면 첫 에디터와 앞으로의 벡터·셰이더·사진 툴이 재사용. Cloudflare의 React + Vite 공식 가이드 있음. Astro 사이트 안에 React 섬으로 넣는 것도 가능해 사이트와 에디터가 이어짐 |

## Cloudflare 역할 분담

렌더링 같은 GPU 작업은 브라우저(WebGPU)가 하고, 서버는 데이터와 연결만 담당하는 구조가 아래 제한들과 맞습니다.

| 서비스 | 역할 | 주요 제한 (2026-11-16 확인) |
|---|---|---|
| **R2** | 에셋 저장소: 모델, 텍스처, HDRI, 렌더 결과물 | 객체 하나당 최대 5TiB, 단일 업로드 5GiB(그 이상은 멀티파트) |
| **D1** | 메타데이터: 프로젝트·버전·사용자·공유 설정 | 데이터베이스당 최대 10GB(유료)/500MB(무료), 한 행 최대 2MB, 데이터베이스 하나는 한 번에 하나의 쿼리만 처리 |
| **Workers** | 인증, 공유 링크 서빙, API, 정적 사이트 호스팅 | CPU 시간 무료 10ms, 유료 기본 30초(최대 5분 설정 가능). 정적 에셋은 파일당 25MiB 제한 → 큰 에셋은 R2로 |
| **WebSocket + Durable Objects** | 실시간 연결 | WebSocket 연결 자체는 Workers에서 바로 가능. 여러 접속자의 상태 조율(공동 편집, 채팅방)에는 Durable Objects 필요. 상세는 [실시간 통신 규칙](07-realtime-communication.md) |

### 데이터 배치 규칙 (제안)

- 장면 데이터 JSON이 크면 **D1에는 색인(메타데이터)만 두고 본문은 R2**에 둡니다. D1의 행 크기·동시 쿼리 제한 때문입니다.
- 25MiB가 넘는 정적 파일은 Workers 정적 에셋이 아니라 R2에서 서빙합니다.

### 서버 렌더링은 별개

Workers의 CPU·메모리(128MB) 제한 때문에 클라우드 렌더 팜 같은 무거운 서버 연산은 이 구성에 올리기 어렵습니다. 첫 버전은 렌더링을 전부 클라이언트 GPU(WebGPU)에 두고, 서버 렌더링이 필요해지면 그때 별도 인프라를 검토하는 걸 제안합니다. GPU 렌더 팜을 Workers에 올린다는 주장은 아닙니다.

### 네이티브 서비스가 필요한 경우 (미정과 연결)

API 처리량과 OS 이벤트 루프·소켓 직접 제어는 다른 문제입니다. axum/netpoll 수준의 직접 제어가 실제로 필요한 영역이 확인되면, 그 영역만 네이티브 서비스로 분리하고 계정·메타데이터·에셋은 Cloudflare를 유지하는 방향을 우선 검토합니다. 실제 필요가 확인된 영역에만 적용합니다. 엔진 언어 결정과 함께 [열린 결정](11-open-questions.md)에서 추적합니다.

## 참고한 공식 문서 (2026-11-16 확인)

- Workers 한도 — https://developers.cloudflare.com/workers/platform/limits/
- R2 한도 — https://developers.cloudflare.com/r2/platform/limits/
- D1 한도 — https://developers.cloudflare.com/d1/platform/limits/
- WebSocket — https://developers.cloudflare.com/workers/runtime-apis/websockets/
- Cloudflare 프레임워크 가이드 (Astro, React + Vite) — https://developers.cloudflare.com/workers/framework-guides/
- Astro 공식 문서의 Cloudflare 배포 안내 — https://docs.astro.build/en/guides/deploy/cloudflare/
- Hono 공식 문서의 Cloudflare Workers 안내 — https://hono.dev/docs/getting-started/cloudflare-workers
