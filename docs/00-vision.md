# 00. 방향

> **확정** · 기준일 2026-11-26

## 이 명세의 범위

- 이 문서 세트는 하이퍼리쉬 프레임워크 패밀리 **전 범위를 범용적으로** 다룹니다. 중심은 **DX(개발자 경험)와 응용/개발 계층**입니다.
- 범용이란 제품별 사례를 모아 두는 것이 아니라, **모든 제품과 코드 레포가 공유하는 추상 계약**을 뜻합니다.
- 개별 제품의 기획·화면·UX와 개별 코드 레포의 도메인·레퍼런스·내부 구조 세부는 각 제품과 각 코드 레포의 문서(`docs/00-overview.md` ~ `05-implementation-plan.md`)가 다룹니다.

## 코드 레포 (탐색용 역할표, 2026-11-26)

| 레포 | 역할 |
|---|---|
| **hyve** | 올인원 개발 프레임워크 & 툴체인 (범용 코어 모듈 + CLI) |
| **orbic** | 백엔드 프레임워크 |
| **serge** | 크로스플랫폼 UI 프레임워크 (Web DOM & Native GPU/CPU) |
| **opale** | 앱과 에디터에 내장하는 실시간 3D 뷰포트 렌더러 |

각 레포의 세부 범위와 레퍼런스는 해당 레포의 `docs/00-overview.md`에 있습니다. `opale`의 첫 납품 단위는 임베디드 뷰포트이며 독자 일정으로 개발합니다. 아래 Phase 1~3은 세 코어 레포의 마일스톤입니다. [opale 제품 범위](https://github.com/hyperish/opale/blob/main/docs/00-overview.md)

## 확정 엔지니어링 방향 (2026-11-19)

### 구현 전략

- 코어는 대부분 **Rust**로 작성하며, Rust 코드 레포는 개별 레포 단위로 `apps/`, `crates/` 모노레포([05-development-rules](05-development-rules.md) §1)로 운영합니다.
- 처음부터 전부 새로 만들지 않습니다. 성능 위주의 검증된 라이브러리·프레임워크를 **포크하거나 기반으로 삼아 `unit optimization`**([05-development-rules](05-development-rules.md) §7)을 거칩니다.
- 수직 통합을 위해 **`hyve`의 코어를 단일 기준(`ref`)으로 삼아 구현**하고, **그 위에 외부·타깃 호환 어댑터를 두는 구조**([02-core-contracts](02-core-contracts.md))로 작업합니다.
- **DSL 채택 이유는 개발 생산성 때문**입니다. UI 뷰 마크업(`serge`)처럼 중첩 트리 가독성과 컴파일 타임 템플릿 분리·핫리로드 이점이 확실한 곳은 **proc macro DSL로 고정**하고, 컴포넌트·라우트·가드 조립(`orbic`, `serge`)처럼 일반 Rust 코드가 깔끔한 곳은 **메서드 체이닝/함수**를 씁니다([01-dx-principles](01-dx-principles.md)).

### 규약 모델 원칙

- 코어와 내부에서 사용하는 규약 모델은 **표준 준수보다 성능과 수직 통합 시 생산성이 극대화되는 스택**을 씁니다.
- **외부 호환은 개별 어댑터를 구축**하는 방식으로 해결합니다. 내부 규약을 외부 표준에 맞추느라 성능·생산성을 포기하지 않습니다.
- 설계안이 갈리거나 문서가 충돌할 때도 같은 기준으로 고릅니다. 핫패스의 복사·할당·잠금·변환이 적은 쪽, 한 번 선언해 전 계층이 그대로 쓰는 쪽을 택합니다. 다만 [02-core-contracts](02-core-contracts.md)의 안전·권한 계약(검증, 권한 검사, 손실 금지)은 성능을 이유로 약화하지 않습니다.
- 구체적인 내부 포맷·스택은 각 코드 레포의 `docs/02-architecture.md`가 확정하고, [07-open-questions](07-open-questions.md)는 그 결정의 색인을 유지합니다.

## 마일스톤 (Phase)

| Phase | 버전 | 범위 |
|---|---|---|
| 1 | `v0.1.0` | 최소 풀스택 수직 슬라이스: 코어, 백엔드 기본기, Web DOM, Desktop Native(GPU + CPU 폴백) |
| 2 | `v0.2.0` | 상태 보존 HMR, L2 캐시, 스토리지, 잡, SSR/SSG, 데스크톱 시스템 API, 공유 메모리 IPC |
| 3 | `v0.3.0+` | 바이너리 핫패칭, 스토어 패키징, WebTransport, MSA, 모바일 풀 SDK, 위젯 코드젠 |

각 Phase에 들어가는 크레이트는 각 코드 레포의 `docs/04-open-questions.md`가, 작업 순서와 레포별 완료 기준은 `docs/05-implementation-plan.md`가 정합니다.

### Phase 1 완료 기준 (확정, 2026-11-24)

다음이 동작해야 `v0.1.0`을 붙입니다. 이 수직 슬라이스의 테스트는 `serge` 레포의 `apps/serge-playground`에 둡니다.

1. 공유 크레이트에 선언한 모델 하나와 인라인 서버 함수 하나를 UI 화면에서 호출하면, 백엔드 컴포넌트의 가드를 거쳐 DB에 기록되고 결과가 화면에 반영됩니다. 같은 화면 코드가 Web DOM과 Desktop Native에서 동작합니다. 확인 방법은 타깃마다 다릅니다.
   - Desktop Native: 서버와 UI를 한 프로세스에 두고 인프로세스 전송과 CPU 렌더 경로로 포트 없이 테스트합니다.
   - Web DOM: 서버는 브라우저 타깃으로 빌드되지 않으므로, 실제 서버를 띄우고 원격 전송으로 확인합니다.
2. 그 호출은 고속 레일과 표준 레일에서 같은 결과와 같은 오류를 냅니다.
3. Desktop Native는 GPU와 CPU 폴백에서 같은 화면을 그립니다.
4. 세 코어 레포가 각자의 완료 기준을 충족하고 CI(포맷, 린트, 테스트, 선언한 최소 Rust 버전 빌드)를 통과합니다.

### 구현·인수 기록 (2026-11-26)

[플랫폼별 실측값과 실행 기록](verification/2026-11-26-platforms.md)에 러너, 실제 OS 동작, 로컬 릴리스 부하, 기기·서명 조건을 정리했습니다.

| 대상 | 실행한 인수 시나리오 | 남은 인수 항목 |
|---|---|---|
| Phase 1 풀스택 | 공개 crates.io 의존성만 쓰는 새 디렉터리에서 `hyve init` 후 `hyve test`로 SQLite 노트 생성·검증 거부·목록 조회. 데스크톱 CPU와 실제 Chrome에서 같은 프로젝트 화면으로 로그인·저장·거부·DB 재조회. Metal과 CPU 프레임 비교. winnow 토큰 처리, ariadne 소스 인덱스, bytemuck의 POD 바이트 접근, rkyv 정렬 버퍼 소스 편입·비교. Orbic의 SQLx 컬럼 디코더가 결과 열 위치를 재사용하고 utoipa 경로 조립은 메서드별 복사를 제거. Hyper 기반 본문 수집에서 단일 프레임의 바이트 버퍼를 재사용하고 스트리밍 크기 제한을 검증. quick_cache의 샤드 선택을 비퇴거 저장소에 적용하고 fred의 캐시 응답 파싱을 편입. Serge의 Taffy 트리·Parley 글리프 위치·tachys 텍스트 노드 갱신 경로를 편입해 원본과 비교. Vello CPU 초기화 경로에서 불투명 배경을 직접 채우고 54개 화면의 결과를 원본과 대조. Apple 직접 Metal 배치와 wgpu의 출력 바이트를 비교. macOS ARM64, Linux ARM64·x86-64와 Windows x86-64에서 원본 대비 릴리스 시간 측정 | 다음 버전 태그·배포 |
| Phase 2 개발·백엔드 | 웹·네이티브 상태 복원과 빌드 실패 복구. 다중 스레드의 공유 시그널 변경을 UI 스레드에서 반영. Valkey 연결을 끊고 다른 서버에서 세션을 철회한 뒤 재연결 상태 검사. TTL 0인 조회는 L1과 Valkey에서 진행 중이던 호출만 결과를 공유하고 다음 호출은 새로 읽는지 확인. 디스크·로컬 S3 호환 서버에서 파일 무결성과 멀티파트 재개. SQLite·PostgreSQL에서 작업 임대·재시도·타이머. 실제 OpenAPI 내보내기로 호환 변경 통과·호환성 위반 거부. 로컬 릴리스 서버 두 개에서 동시 저장 256건, 재시작 후 전체 ID·목록, 다른 서버에서의 세션 철회를 확인. PostgreSQL·SQLite·Valkey 릴리스 테스트 통과 | 로컬 릴리스 인수 완료 |
| Phase 2 UI와 IPC | 프로덕션 SSR/SSG·가드·스트림 취소·교체 실패 복구. Chrome 하이드레이션과 UI→SQLite. macOS·Linux 프로세스 간 공유 메모리와 Unix 소켓 RPC. Windows 바이너리 두 개를 Wine에서 실행해 공유 메모리 수명·레이아웃과 named pipe의 단독 바인딩·교환 검증. 두 macOS 창에서 CPU·Metal 독립 렌더링. 플로팅 창을 옮기고 크기를 바꾼 뒤 새 프로세스에서 실제 창의 위치·크기·항상 위 레벨이 복원되는지 검증. 키보드와 접근성 트리의 상태 일치, AppKit 메뉴 막대·상태 메뉴 명령 전달. 잠금 화면에서 높이 0인 상태 막대에 등록된 팝오버 액션을 앱 창의 버튼으로 실행해 실제 콘텐츠 표시·닫기 검증. Dock의 0·42·100% 표시·해제. Linux D-Bus에서 런처 진행률 갱신과 해제를 별도 연결로 수신하고 Windows 작업표시줄 COM 경로를 교차 빌드. Linux X11과 Weston의 XWayland에서 F8과 메뉴 막대 클릭으로 같은 명령을 받고 우클릭 메뉴의 실제 항목을 선택. Weston은 컴포지터의 키보드·포인터로 입력. 순수 Wayland에서 두 창의 독립 렌더·종료와 별도 D-Bus 연결을 통한 트레이 명령 전달을 확인. X11에서 파일 선택기·알림창, 별도 프로세스의 클립보드 읽기와 전역 단축키, 실제 Secret Service 자격 증명 저장·교체·삭제 검증. Chrome에서 960→390px 전환 시 탐색 DOM 유지, 모달 포커스 순환·복원과 스와이프 닫기. 두 외부 뷰포트의 클립·겹침·세대 변경을 CPU와 Metal의 픽셀로 대조. Chrome에서 WebGPU 장치 상실 후 같은 위치에 CPU 프레임 표시. Windows Server 2025에서 공유 메모리, named pipe, 플로팅 창 재시작 복원, F8과 작업표시줄을 실행. 잠금 해제된 Mac에서 화면에 표시된 상태 버튼을 포인터로 눌러 팝오버를 열고 닫음 | 순수 Wayland의 창 메뉴는 X11 부모가 필요하다는 오류를 반환 |
| Phase 3 실시간·분산 | Chrome·네이티브에서 실제 WebTransport 데이터그램 힌트 및 개별 단방향 명령 스트림, 같은 명령 재전송 시 단일 커밋. UDP 차단 시 WebSocket 전환, JSON 레일 복구. 두 DB에서 티켓 단회 사용과 재전송 중복 방지. 서비스 세 개의 RPC·가드·JSON 폴백·분산 트레이스·복제본 장애 대응. 두 DB의 트랜잭션 이벤트 버스·웹훅 수신과 SAGA 보상. 로컬 SMTP·메일 API·HTTP/2 APNs Live Activity·FCM 요청 검증 | 실기기 푸시 수신 |
| Phase 3 패키징·모바일 | macOS 앱·DMG 생성, ad hoc 서명과 실행. Linux arm64에서 DEB와 AppImage를 만들고 각각 추출한 바이너리와 AppRun을 실행. Rust 위젯 상태를 SwiftUI·Glance로 생성하고 Android SDK 35로 AAR 컴파일. 생성된 Swift 소스 구문 검사와 iOS 시뮬레이터 타깃의 Rust 호스트 검사. 생성된 앱에 Xcode 시뮬레이터 빌드·서명 IPA 내보내기 경로 제공. macOS arm64에서 렌더 함수를 핫패치한 뒤 동일 PID·상태 유지, 컴파일 오류 후 기존 창 유지, 타입 변경 후 초기값 확인. Ed25519 서명 델타로 실행 파일 교체 및 실행. Android 35 에뮬레이터에서 APK와 서명한 AAB에서 생성한 APK 각각으로 접근성·터치·키보드·재개·저장·인가 거부·SQLite 검증. Windows에서 MSI·NSIS 설치·실행·제거와 일회용 Authenticode 인증서로 서명한 두 설치본·설치된 실행 파일 검증, 변조된 파일 거부를 확인. Xcode가 생성한 iPhone 앱에 WidgetKit 확장을 넣고 iOS 26.2에서 실제 화면과 같은 1206×2622 프레임을 확인. iPhoneOS SDK arm64 빌드와 서명 자격 없는 IPA 요청의 거부도 확인 | 배포용 서명과 스토어 제출, 모바일 실기기 인수 |
| Phase 3 기기 API·위젯 | Android 35 센서·AAudio PCM 링 수신과 구독 종료, 시스템 HTTPS 인증, 인증서 거부, AndroidKeyStore·클립보드·네이티브 선택/날짜/시간 대화상자. Android 시스템 지문 대화상자에서 미등록 지문 거부, 등록한 지문 인증, 뒤로 가기 취소 결과와 OS 지문 센서의 수락·거부 카운터 확인. Android DocumentsUI에서 문서 생성·기록·재선택·읽기·취소. Android의 두 백그라운드 작업 중 하나가 만료된 뒤 홈 화면으로 이동해 저장한 파일을 확인하고 마지막 핸들을 해제해 서비스 종료 확인. AndroidKeyStore의 P-256 키 재개방·서명·교체와 Mac Secure Enclave의 실제 P-256 서명·변조 거부·개인 키 내보내기 거부. Android 앱에 Glance 위젯과 진행 알림을 함께 패키징. OS 위젯에 두 상태를 표시하고, 알림 권한 거부·진행률 갱신·취소 확인. 교체된 로컬 알림의 작업 버튼을 시스템 알림창에서 눌러 프로젝트 명령이 한 번만 전달되는지 검사. Apple의 실제 UserNotifications 객체에서 명령 식별자, 표시 이름, 카테고리를 확인. iOS 홈·잠금 화면 WidgetKit 타임라인과 Shortcuts App Intent, Live Activity 시작·갱신·종료 브리지 생성. iOS 시뮬레이터 타깃에서 UIKit 창별 대화상자·날짜/시간 선택·공유·클립보드·파일 가져오기/내보내기·백그라운드 실행과 UserNotifications 바인딩 컴파일. iOS 18.5 시뮬레이터에서 App Group에 발행한 두 번째 상태와 Live Activity 시작·갱신·종료 확인. 시스템 Files 내보내기·가져오기·취소, 홈 화면에 추가한 위젯의 저장된 제목, Apple 단축어 편집기에서 실행한 App Intent의 JSON과 시스템 클립보드 일치, 미등록 생체인증의 사용 불가와 등록 후 모의 얼굴 인증 성공을 확인. 잠금 화면 편집기에서 직사각형 위젯을 추가하고 WidgetCenter가 배치를 보고하는지 확인 | 실기기 생체인증·푸시 |
| Phase 3 카메라·입력·창 | Android 35에서 펜과 지우개의 OS 입력으로 필압·기울기·이동·취소와 손가락 입력 제외를 확인. UIKit의 Apple Pencil 병합 이벤트 경로는 시뮬레이터 타깃으로 컴파일. Android Camera2 프레임 두 장을 외부 GLES 텍스처로 그려 픽셀 확인. Vulkan에서도 서로 다른 타임스탬프의 AHardwareBuffer 두 장을 가져와 계산 셰이더로 각각 8×8 픽셀의 불투명도와 색상 변화를 확인. 두 에뮬레이터의 BLE GATT 서비스 발견·읽기·쓰기·변경 알림·연결 해제와 등록 URI로 앱 실행·진동 패턴 요청을 확인. 별도 Android MIDI 서비스와 macOS CoreMIDI 포트에서 두 MIDI 메시지를 왕복. Pixel Fold 에뮬레이터 힌지를 180°에서 90°로 움직여 Jetpack의 접힘 영역·분리 상태와 NDK 힌지 각도를 확인하고 NativeActivity를 PiP로 전환. Apple에서는 합성 IOSurface의 BGRA 픽셀을 wgpu/Metal에서 읽고 실제 AVAudioPCMBuffer의 두 채널 형식으로 링의 PCM 값 확인. Core Haptics에서 세기·쉼이 있는 0.4초 파형 파싱. iPhone 시뮬레이터의 공유 시트에서 Copy를 선택해 실제 클립보드 내용을 읽고, 설정 앱으로 전환한 뒤 백그라운드 쓰기를 확인. 주입한 알림의 시스템 작업 버튼이 앱에 명령을 전달 | 실물 GATT 주변기기, Apple 실기기 카메라·마이크·펜슬·진동과 푸시 수신 |
| Phase 3 선택형 웹뷰 | Android 35 시스템 WebView, macOS WKWebView와 Linux X11의 WebKitGTK에서 JSON과 WASM/rkyv 양쪽으로 호스트 저장 직후 문서를 다시 열고 이전 결과·상태 복원, 권한 거부, 단일 실행과 인라인 스크립트 차단 확인. macOS·Linux에서 창 안의 웹뷰를 284×162로 바꾸고 문서에 전달된 크기 측정. Weston의 XWayland에서도 두 전송 방식으로 같은 크기·저장·복원 검증. Windows WebView2와 UIKit WKWebView 호스트, 딥링크를 포함한 생성 iOS 인수 앱을 대상 플랫폼용으로 교차 검사. Windows WebView2와 iOS 18.5 WKWebView에서 두 전송 방식 모두 실제 저장·복원·인가 거부와 크기 검증 통과. iOS에서 시스템 확인창을 열어 같은 앱으로 프로젝트 URL 전달 | 지원 OS의 선택형 웹뷰 실행 인수 완료 |

공통 기능은 개발 PC에서 실행합니다. 선택한 어댑터의 인수에는 임시 DB나 로컬 호환 서버를 사용합니다. 위 표의 인수는 배포 계정이나 특정 호스팅 사업자에 의존하지 않습니다.

## 문서 구성

- DX 설계 원칙 → [01-dx-principles](01-dx-principles.md)
- 계층을 넘는 공통 계약 → [02-core-contracts](02-core-contracts.md)
- 외부 어댑터 규약(채택하는 경우에만 적용) → [03](03-api-protocol.md), [04](04-realtime-communication.md)
- 절차와 이름 규칙 → [05](05-development-rules.md), [06](06-naming-conventions.md)
- 공통 미정 사항 → [07](07-open-questions.md)
