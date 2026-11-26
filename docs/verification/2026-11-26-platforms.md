# 플랫폼 인수 결과 · 2026-11-26

기준 명세는 `hyperish/docs@a9045284fe8b11d2a2213ea42b29839bf493ab9e`다. 변경 중인 세 저장소의 소스를 비공개 검증 저장소에 복사해 Windows와 Xcode에서 실행했다. 플랫폼별 링크는 해당 테스트가 실행된 커밋을 가리킨다. [소스 기준 커밋과 파일 해시](https://github.com/hyperish/framework-acceptance-20261126-0949/blob/21cd8778c91f8f6b33f7421b54f01ac0fd60d964/verification/sources.json)

같은 파일 해시로 확인한 공개 저장소의 소스도 커밋별로 조회할 수 있다. [Hyve 소스](https://github.com/hyperish/hyve/tree/ab3b833cdf5bfd5ea32d5f20b92fc198447d70d8) · [Orbic 소스](https://github.com/hyperish/orbic/tree/d606b1a03405d83c4ac086c58d8d8c2434237474) · [Serge 소스](https://github.com/hyperish/serge/tree/d58cdedd6fc7aa71c1532b3c2e709819d9e70ba8)

## 공개 저장소의 병합 검증

Hyve의 `main`에서 레지스트리 의존성과 형제 저장소 소스로 각각 새 풀스택 앱을 만들었다. 두 앱에서 `hyve test`만 실행해 노트 생성, 빈 본문 거부와 DB 무변경, 목록 조회를 확인했다. Orbic의 `main`에서는 PostgreSQL·Valkey를 연결한 통합 테스트와 공개 크레이트 패키징이 통과했다. [Hyve 실행](https://github.com/hyperish/hyve/actions/runs/45749159133) · [Orbic 실행](https://github.com/hyperish/orbic/actions/runs/45748278292)

Opale은 3D 뷰포트의 명세와 구현 계획을 병합했다. 해당 커밋에서 기존 스캐폴드의 포맷 검사, Clippy와 테스트가 통과했다. [Opale 실행](https://github.com/hyperish/opale/actions/runs/45743674469)

로컬 Pixel 6 에뮬레이터에서는 상태 표시줄과 카메라 컷아웃이 상단 128px을 차지했다. 첫 링크의 중앙은 83px에 있어 터치를 받지 못했다. Android가 보고한 안전 영역을 적용한 뒤 중앙이 211px로 이동했고 한 번 터치해 화면을 열 수 있었다. Pixel 6와 Pixel Fold 모두 키보드로 입력한 제목 전체가 화면에 나타난 뒤 저장을 눌렀고, SQLite의 소유자와 제목이 화면의 프로젝트와 일치했다. 인메모리 세션을 쓰는 서버를 재시작한 뒤, 두 번째 저장은 거부됐고 DB에는 첫 프로젝트만 남았다. Pixel Fold의 힌지를 180도에서 90도로 바꾸자 AndroidX는 반쯤 접힌 상태를, NDK 센서는 90도를 보고했다. [Pixel 6 실행](https://github.com/hyperish/serge/actions/runs/45765716844/job/139734629846) · [Pixel Fold 실행](https://github.com/hyperish/serge/actions/runs/45765716844/job/139734572676)

Hyve·Orbic·Serge의 병합된 `main`을 함께 체크아웃한 macOS에서 Chrome 158로 프로젝트를 저장했다. 로그인 후 저장한 제목이 화면과 별도로 연 SQLite 연결에서 일치했다. 브라우저의 세션 쿠키를 지우자 서버가 두 번째 저장을 거부했고 DB에는 첫 프로젝트만 남았다. iPhone 시뮬레이터에서는 별도의 UIKit 앱에서 버튼을 눌러 수신 앱을 열었다. JSON과 WASM/rkyv에서 각 경로에 고유한 프로젝트 URL이 앱 델리게이트에 그대로 전달됐다. [Chrome 실행](https://github.com/hyperish/serge/actions/runs/45765716844/job/139734571185) · [iOS WKWebView 실행](https://github.com/hyperish/serge/actions/runs/45765716844/job/139734621261)

Serge의 `main`에서 8개 CI 작업이 모두 통과했다. iOS 시스템 UI 검사에서는 단축어가 복사한 JSON을 시스템 클립보드에서 직접 읽어 제목 `Saved project`와 진행률 0.75를 대조했다. 홈 화면과 잠금 화면의 위젯 배치도 확인했다. [iOS 시스템 UI 실행](https://github.com/hyperish/serge/actions/runs/45765716844/job/139734573336) · [Serge 전체 실행](https://github.com/hyperish/serge/actions/runs/45765716844)

병합 직후 Linux 검사에서는 Weston의 XWayland 메뉴를 클릭한 뒤 응답을 기다리다 시간 초과됐다. Linux ARM64에서 동일한 메뉴 조작을 세 번 반복해 메뉴 막대와 우클릭 메뉴가 명령을 전달하는지 확인했다. 같은 병합 커밋의 공개 Linux 검사도 재실행에서 통과했다. [첫 실행](https://github.com/hyperish/serge/actions/runs/45765716844/job/139726023228) · [Linux 재실행](https://github.com/hyperish/serge/actions/runs/45765716844/job/139734570126)

## 앱과 서버

| 환경 | 관찰한 결과 |
|---|---|
| 새 풀스택 앱 | 새 디렉터리에서 `hyve init notes-live --template fullstack` 후 `hyve test`로 노트 본문·ID 저장, 빈 본문 거부와 DB 무변경, 목록 조회 3건 통과 |
| macOS와 Chrome | 프로덕션 SSG 번들과 Orbic 서버를 띄웠다. 비로그인 보호 경로가 401을 반환했다. 브라우저에서 로그인해 프로젝트를 저장하고 화면에서 조회했다. 별도 SQLite 연결로 소유자와 제목을 대조했다. |
| 릴리스 서버 두 프로세스 | 로컬 SQLite 파일 하나와 임시 Valkey 8.1.3을 공유했다. 클라이언트 8개가 로그인한 상태로 프로젝트 256개를 나눠 저장했다. 첫 루프백 실행은 초당 2,124.8건, 지연 시간 p50 0.98ms, p95 13.07ms였다. 같은 시나리오를 다시 실행했을 때는 초당 444.8건, p50 11.29ms, p95 49.48ms였다. 두 프로세스와 재시작한 프로세스에서 전체 ID·제목이 일치했다. 로그아웃 후 다른 프로세스가 같은 자격 증명을 401로 거부했다. |
| PostgreSQL·SQLite·Valkey | 격리된 PostgreSQL과 SQLite에서 작업 큐·스케줄·SAGA 릴리스 테스트 4건을 통과했다. Valkey 릴리스 테스트에서는 동시 요청, TTL 0, 연결 장애 후 복구, 복제본 간 세션 철회 4건을 통과했다. |

두 서버 검증은 `serge`에서 `cargo build --release --locked -p serge-playground --features cache-redis --bin playground-server` 후 `python3 apps/serge-playground/tests/backend_release.py target/release/playground-server`로 재실행할 수 있다. 테스트가 임시 DB와 루프백 Valkey 컨테이너를 생성하고 정리한다.

세 저장소에서 `cargo test --workspace --locked`를 실행해 Hyve 181건, Orbic 67건, Serge 49건이 통과했다. Orbic에는 격리된 PostgreSQL 18을 연결했다. 기본 실행의 ignored 13건은 이 합계에서 제외했다. 별도로 Hyve CLI의 OASdiff 1.41.0 호환성 검사, Serge의 릴리스 백엔드 테스트 5건, ignored로 지정된 Chrome UI 저장 테스트 1건을 실행했다.

## 네이티브 플랫폼

| 환경 | 관찰한 결과 |
|---|---|
| Windows Server 2025, 빌드 26100 | 별도 프로세스의 공유 메모리와 named pipe RPC가 동작했다. HWND의 플로팅 창 위치와 크기를 재시작 후 복원했다. F8이 네이티브 메뉴 명령을 전달했고 작업표시줄 진행률을 갱신·해제했다. WebView2에서 JSON과 WASM/rkyv 모두 저장·다시 열기·권한 거부·284×162 크기 검사를 통과했다. MSI와 NSIS를 각각 설치하고 실행한 뒤 제거했다. 일회용 Authenticode 인증서로 서명한 두 설치본과 설치된 실행 파일을 Windows SDK로 검증했다. 실행 파일의 한 바이트를 바꾼 복사본은 검증에 실패했다. [Windows 창과 IPC 실행](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45736124040/job/139629714670) · [WebView2 기본·새 프로필 재검증](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45737535705/job/139634294014) · [설치본 실행](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45731642449/job/139614857738) · [서명과 변조 검사](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45735064395/job/139626213471) |
| macOS AppKit | 화면에서 ‘Projects’ 상태 버튼을 찾아 포인터로 클릭했다. 실제 팝오버의 콘텐츠 창과 너비를 확인하고 같은 위치를 다시 눌러 닫았다. Dock 진행률과 메뉴·상태 항목 명령도 통과했다. [AppKit 실행](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45733271392/job/139620283456) |
| Linux ARM64, Weston | XWayland와 DISPLAY가 없는 전용 컴포지터에서 두 창이 독립적으로 그려지고 닫혔다. D-Bus 상태 항목에 연결한 셸이 명령을 전달했다. 같은 환경의 창 메뉴 요청은 `Linux window menus require an X11 parent`를 반환했다. GTK 창 메뉴·자식 웹뷰는 X11 또는 XWayland를 사용한다. |
| Android 35 에뮬레이터 (이전 실행) | APK와 테스트 서명한 AAB에서 만든 APK를 각각 실행해 터치·키보드, 앱 재개, 인증된 프로젝트 저장, 거부와 SQLite 재조회를 확인했다. 지문 등록·거부와 DocumentsUI 파일 왕복도 시스템 화면에서 검사했다. Pixel Fold에서는 힌지 각도 변화와 화면 속 화면 전환을 확인했다. |
| Xcode 디바이스 SDK | 생성한 앱이 실제 iPhoneOS SDK에서 arm64 Mach-O로 링크됐다. 개발 팀과 프로필이 없는 상태에서 `hyve build --format ipa`를 실행하자 Xcode가 서명 단계에서 거부했고 IPA가 생성되지 않았다. [디바이스 빌드 및 서명 거부](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45733986354/job/139622661446) |
| iPhone과 WKWebView | iOS 26.2에서 WidgetKit 확장을 포함한 카운터 앱의 1206×2622 렌더 프레임이 시뮬레이터 스크린숏 크기와 일치했다. iOS 18.5에서 WKWebView의 284×162 CSS 뷰포트, 저장 1회, 새로고침 후 복원, 거부된 요청, 인라인 스크립트 차단을 JSON과 WASM/rkyv 양쪽에서 확인했다. SpringBoard에서 ‘열기’를 선택한 후 앱이 원래 프로젝트 URL을 받았다. [프레임 비교](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45735280892/job/139626919006) · [WKWebView와 딥링크](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45735438278/job/139627429095) |

Windows에서 처음부터 브라우저 초기화와 메시지 왕복을 합쳐 35초를 재던 실행이 한 차례 시간 초과됐다. 검사를 두 구간으로 나눈 뒤 WebView2의 기본 프로필과 별도 데이터 폴더에서 JSON과 WASM/rkyv를 각각 실행했다. 네 번 모두 같은 뷰포트와 저장·복원·거부를 확인했다. [WebView2 재실행](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45737535705/job/139634294014)

### iOS 18.5 시스템 UI

실제 시뮬레이터의 홈 화면에 생성된 WidgetKit 위젯을 추가해 두 번째로 저장한 제목 `Saved project`가 표시되는지 검사했다. 별도 App Group 파일에서 같은 상태와 진행률 0.75를 읽었다. WidgetCenter가 홈 화면에 설치된 `systemSmall` 위젯을 보고했다. 생성한 `ReadExportHomeIntent.perform()`가 같은 JSON을 반환했다. Apple 단축어 편집기에서 `Read ExportHome`과 `Copy to Clipboard`를 추가해 실행하고 시스템 클립보드의 제목·진행률까지 대조했다. ActivityKit이 발급한 ID로 Live Activity를 시작·갱신·종료했다. 위젯은 iOS 17부터 시스템이 제거할 수 있는 배경을 선언한다. [iOS SDK와 XCUITest 실행](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45740544602/job/139644181486) · [Apple의 위젯 배경 규칙](https://developer.apple.com/documentation/widgetkit/displaying-the-right-widget-background)

Files에서 내보낸 문서를 다시 가져와 바이트를 대조하고, 취소 버튼이 앱에 취소 상태를 돌려주는지 확인했다. 날짜 선택·시간 취소, 공유 시트의 복사와 시스템 클립보드 읽기, 설정 앱 전환 후 백그라운드 저장도 통과했다. 얼굴 미등록 시의 사용 불가와 등록 후 모의 Face ID 성공, `simctl push`로 주입한 알림의 작업 버튼을 눌러 앱에 `archive` 명령이 도착하는 것까지 확인했다. [테스트 실행](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45740544602/job/139644181486)

잠금 화면 편집기에서 `ios-sdk, ExportHome`의 직사각형 위젯을 선택하고 배경화면 변경을 저장했다. 앱을 다시 열어 OS의 `WidgetCenter.getCurrentConfigurations`를 조회했을 때 `ExportHome`의 `accessoryRectangular` 배치가 등록돼 있었다. 최종 통합 실행에서 시스템 UI 8건, 단축어 1건, 잠금 화면 1건으로 XCUITest 10건이 모두 통과했다. [통합 실행](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45740544602/job/139644181486) · [잠금 화면과 WidgetCenter 단독 실행](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45739295496/job/139640081109)

## 릴리스 비교

각 칸은 같은 머신·입력에서 측정한 프레임워크 경로와 비교 대상의 중앙값을 순서대로 나타낸다. macOS는 M5 Pro, Linux ARM64는 ARM 가상 머신, x86-64는 Ubuntu 24.04와 Windows Server 2025 호스팅 러너다. 시간이 짧을수록 빠르다. [Linux·Windows 릴리스 실행](https://github.com/hyperish/framework-acceptance-20261126-0949/actions/runs/45733853023)

| 워크로드와 비교 대상 | macOS ARM64 | Linux ARM64 | Linux x86-64 | Windows x86-64 |
|---|---:|---:|---:|---:|
| Hyve 모델 JSON 디코딩 / serde_json (µs/페이지) | 17.03 / 15.83 | 20.09 / 19.90 | 53.43 / 44.02 | 46.29 / 40.99 |
| Orbic 본문 4 KiB 수집 / collect·to_bytes (ns) | 44 / 58 | 21 / 29 | 77 / 76 | 71 / 102 |
| 캐시 권한 갱신 / 단일 뮤텍스 (ns) | 55 / 107 | 32 / 84 | 41 / 120 | 39 / 44 |
| 캐시 응답 해석 / fred 튜플 (ns) | 90 / 102 | 70 / 73 | 188 / 212 | 229 / 280 |
| OpenAPI 문서 조립 / utoipa (µs) | 71 / 103 | 78 / 109 | 163 / 231 | 390 / 468 |
| Serge 단색 프레임 / Vello 배경 그리기 (µs) | 13.24 / 36.94 | 12.53 / 37.74 | 32.36 / 72.84 | 53.82 / 106.44 |
| Serge 레이아웃 / TaffyTree (µs) | 168 / 204 | 161 / 199 | 521 / 626 | 539 / 727 |

네 환경 모두 이 입력에서 Hyve 모델의 JSON 디코딩이 serde_json보다 오래 걸렸다. Orbic의 4 KiB 본문 수집은 Linux x86-64에서 두 경로가 1ns 차이였다. 플랫폼 간 절대 시간은 CPU와 러너 부하의 영향도 받는다.

M5 Pro에서 GPU 완료와 결과 읽기까지 포함한 프레임 중앙값은 직접 Metal 경로가 211.74µs, wgpu가 226.07µs였다. 단색 CPU 프레임과 GPU 비교는 서로 다른 워크로드다.

## 실기기와 배포

전용 모바일 기기, APNs·FCM 테스트 프로젝트, 배포 서명 프로필이 제공되지 않았다. 실기기 푸시 수신과 생체인증, Apple 카메라·마이크·Pencil·진동, 실제 BLE 주변기기 호환성, 배포 서명과 스토어 인수는 별도 검증 대상으로 남는다.
