# Pixel Cup Soccer APK 분석 및 리뉴얼 개발 검토

대상: `com.atoviamestudio.ixelupoccer19.apk` (617 MB) · 분석일 2026-11-02
분석 방법: APK 멤버 목록, 매니페스트·서명 확인, IL2CPP `global-metadata.dat` 정적 디코딩(REA Evidence `ev_035f6ab0…`). 실행이나 디컴파일된 로직 확인은 하지 않았습니다. 아래 동작 설명은 클래스·메서드 이름에서 추론한 것입니다.

## 1. 정체와 빌드 정보

| 항목 | 값 | 근거 |
|---|---|---|
| 게임 | Pixel Cup Soccer (Batovi Games, 우루과이) | 클래스 접두사 `CPCS*`, `CBatoviLogoIntroState`, 문자열 `twitter.com/PixelCupSoccer`, `assets/build_info`: "Build from BATOVIB at 27/05/2024" |
| 엔진 | Unity 2022.3.62f1, IL2CPP, arm64-v8a 단일 | `data.unity3d` 헤더, `libil2cpp.so`, 메타데이터 v31 |
| 규모 | 타입 9,720 / 메서드 77,641 (게임 코드 Assembly-CSharp 1,276 타입) | IL2CPP 메타데이터 |
| 리소스 | `datapack.unity3d` 401 MB, `resources.resource` 163 MB, `data.unity3d` 26 MB | APK 목록 |

### ⚠️ 이 APK는 원본이 아닙니다
- `lib/arm64-v8a/libPDALIFE.so`, `assets/PDALIFE.bin/.conf`가 추가되어 있고, 이 라이브러리에 `A64HookFunction`(ARM64 인라인 후킹) 심볼이 있습니다 → 모드 배포 사이트(PDALIFE)가 코드를 주입한 버전입니다.
- 서명 인증서가 `CN=Android, O=Android, Mountain View` → AOSP 공개 테스트키로 재서명된 것입니다.
- 패키지명도 원본(`…batovi…pixelcupsoccer…` 계열)에서 글자를 뺀 형태로 바뀐 것으로 보입니다.
- **권장:** 실기기 설치·실행 금지. 벤치마크용 플레이는 Google Play 정품으로 하십시오.

## 2. 기술 구조

**자체 2D 프레임워크 위에 Unity를 얇게 사용**하는 구조입니다. Unity의 씬·컴포넌트 방식보다 자체 클래스 계층에 의존합니다.

- 코어: `CGame`, `CGameState`(상태 머신), `CGameObject`(240 메서드), `CSprite`(134), `CAnimatedSprite`, `CManager`, `CEventManager`, `CObjectPool`
- 수학·충돌: `CMath`(108), `CVector`(105), `CRect`, `CQuadTree`, `CAStar`, `CInfluenceMap`
- 렌더링: 도트(픽셀) 스프라이트, Smart Lighting 2D 에셋(Light2D 계열 수백 타입), CRT 효과(`CCRTEffectManager`), LUT 포스트 효과, 날씨(비·눈·바람), 아틀라스 색상 교체(`CAtlasColorSwapper`, 유니폼 색)
- 입력: Rewired(패드), 터치 가상 컨트롤러(`CVirtualController`, `CTouchController`), 키보드·마우스, Apple TV
- 멀티플랫폼 흔적: Steam(로비·업적·워크숍), iOS/iCloud, Game Center, Google Play Games(업적·스냅샷 저장), Apple TV, Switch 설정값 → **PC/콘솔/모바일 공용 코드베이스**
- 기타: ACTk 안티치트, Unity Ads, Tiled(TMX) 맵 로더, 다국어 문자열 테이블

## 3. 게임플레이 시스템 (이름 기반 추정)

| 시스템 | 주요 클래스 (메서드 수) | 내용 |
|---|---|---|
| 경기 엔진 | `CPCSPlayer`(732), `CPCSTeam`(242), `CPCSBall`(152), `CPCSMatchState`(261), `CPCSPitch`, `CPCSGoal`, `CPCSFormation` | 탑뷰 픽셀 축구, 패스·슛 파워바, 헤더/시저킥/오버헤드킥, 슬라이딩 태클, 스태미나 |
| 심판 | `CPCSReferee`(356), `CPCSLineman`(358) | 주심·선심 개체 시뮬레이션, 옐로/레드카드, 페널티 |
| AI | `CAITreeNode` 결정 트리 + `CPCS*Decision` 약 30종, `CPCSTeamAI`, `CPCSPlayerAI` | 볼 쫓기, 마킹, 포메이션 복귀, 드리블·스프린트, 골키퍼 판단 등 |
| 혼돈 요소 | `CPCSBananaPeel`, `CPCSPuddle`, `CPCSIceSpike`, `CPCSPorcupine`, `CDog`, `CPCSInvadingFan`, `CPCSFlare`, `CPCSFlyingTrash`, `CPCSAmbulance` | 경기장 장애물·관중 난입·조명탄 등 코믹 이벤트 |
| 리플레이 | `CPCSReplay`, `CPCSNewReplay`, `C*FrameReplayData` | 프레임 단위 기록 재생 |
| 대회 | `CTournament*` (조별·녹아웃·리그전, 타이브레이커) | 월드컵·코파 등 컵/리그 |
| 커리어 | `CCareer`(365), `CPCSCareerSituationsState`(117), 상황 카드·이사회·감독·XP·시즌 캘린더 | 구단 운영 + 카드형 이벤트 선택 |
| 기타 모드 | 친선, 추천 경기(Featured), 승부차기, 연습(드릴 약 25개), 온라인(`CLiveDemo*`, Steam 로비) | |
| 에디터 | 팀·선수·스킬·능력치·유니폼·감독·경기장·대회 에디터, 커스텀 깃발 | UGC 중심 (PC는 워크숍 연동) |

## 4. 수익 모델 (Android 빌드)
- 유료 프리미엄 앱 + Unity Ads(광고 액티비티 선언) 혼합으로 보입니다. 메타데이터에서 IAP(Google Billing) 클래스는 확인되지 않았습니다.
- 권한은 `INTERNET`만 선언되어 있습니다.

## 5. 리뉴얼 개발 검토

### 5.1 먼저 정해야 할 것: 권리 관계
Pixel Cup Soccer의 코드·그래픽·사운드·이름·데이터는 Batovi Games의 저작물입니다. 경로는 둘 중 하나입니다.

| 경로 | 내용 | 가능 범위 |
|---|---|---|
| A. 라이선스/공동개발 | Batovi와 계약해 IP와 원본 소스 확보 | 정식 리마스터·후속작. 디컴파일 코드 재사용이 아니라 원본 프로젝트로 작업 |
| B. 독자 IP | "아케이드 픽셀 축구" 장르를 참고한 새 게임 | 장르·룰·일반 메커니즘은 참고 가능. 에셋·이름·UI·데이터·코드 복제는 불가 |

이 APK에서 추출한 에셋이나 코드는 어느 경로에서도 쓰지 않는 것을 전제로 합니다. 경로 A라면 원본 소스를 받으면 되므로 추출할 필요가 없습니다.

### 5.2 원작의 강점 (유지·계승할 점)
1. 짧은 경기, 단순 조작(패스/슛/스프린트)에 파워바를 더한 깊이
2. 혼돈 요소(동물·관중·장애물·날씨)가 만드는 개성과 화제성
3. 심판·선심까지 개체로 움직이는 픽셀 연출과 CRT 감성
4. 대회·커리어·에디터로 이어지는 긴 콘텐츠

### 5.3 개선 기회 (리뉴얼 포인트)
| 영역 | 현황(추정) | 제안 |
|---|---|---|
| 용량 | APK 617 MB, 데이터가 단일 `datapack` | Play Asset Delivery·Addressables로 분할, 첫 설치 150 MB 이하 목표 |
| 모바일 조작 | PC/패드 우선 설계 위에 가상 패드를 추가한 구조 | 스와이프 슛, 자동 선수 전환, 원터치 모드 등 모바일 전용 조작 |
| 온라인 | 데모 수준 클라·서버와 Steam 로비 | 서버 권위 + 롤백 넷코드 실시간 PvP, 비동기 리그 |
| 메타 | 커리어 카드 이벤트, 업적 | 시즌 패스, 클럽 꾸미기, 선수 수집·육성 루프 |
| 수익 | 유료 + 광고 | F2P + 코스메틱 IAP + 보상형 광고, 혹은 프리미엄 유지 + DLC |
| 기술 부채 | 자체 프레임워크, 거대 클래스(`CPCSPlayer` 732 메서드) | ECS/컴포넌트 분리, 결정론적 시뮬레이션(리플레이·넷코드 공용) |
| 라이브 운영 | 로컬 데이터 중심 | 원격 설정, 이벤트 대회, 분석 SDK |

### 5.4 권장 기술 스택 (경로 B 기준)
- **엔진:** Unity 6 LTS (2D URP + Pixel Perfect Camera). 대안은 Godot 4 (가벼운 2D, 라이선스 비용 없음)
- **시뮬레이션:** 고정 틱(예: 60 Hz) 결정론적 물리. 직접 구현하는 간단한 2.5D 공 물리(높이 z축)가 Box2D보다 적합
- **AI:** 유틸리티 AI 또는 행동 트리 + 포메이션 슬롯 시스템
- **네트워크:** 롤백(GGPO 방식) 또는 Photon Quantum
- **백엔드:** Firebase / PlayFab / Nakama 중 택1 (계정, 리더보드, 클라우드 저장)

### 5.5 개발 범위와 일정 (대략)
| 단계 | 기간 | 산출물 | 인원 |
|---|---|---|---|
| 프리프로덕션 | 2개월 | 코어 경기 프로토타입(5v5, 패스·슛·태클·AI), 아트 스타일 확정 | 기획1·클라2·아트1 |
| 버티컬 슬라이스 | 3개월 | 11v11, 심판·카드, 혼돈 요소 3종, 대회 모드 1종, 모바일 조작 | +아트1·사운드(외주) |
| 프로덕션 | 6–8개월 | 커리어·에디터·온라인 PvP·메타·수익화·현지화 | 7–9명 |
| 소프트런칭 | 2개월 | KPI 검증(D1 40%+, D7 15%+ 목표), 밸런싱 | |

총 13–15개월, 인원 7–9명 규모로 예상합니다. 온라인 PvP를 출시 후로 미루면 3–4개월 줄일 수 있습니다.

### 5.6 주요 리스크
- **IP/법무:** 경로 B에서 원작과 너무 닮은 아트·이름·실제 국가대표 명칭 사용 → 독자 아트 디렉션, 가상 팀명 사용
- **게임 감각:** 아케이드 축구는 조작 손맛이 핵심 → 프리프로덕션에서 플레이테스트 반복
- **넷코드 난이도:** 실시간 11v11 동기화 → 결정론적 시뮬레이션을 처음부터 설계에 넣기
- **보안:** 원작도 모드판이 유통 중 → 서버 검증, Play Integrity API

## 6. 확인하지 못한 것
- 실제 로직(물리 수치, AI 파라미터, 밸런스): 메서드 본문 디컴파일·실행 미수행
- 팀·대회 데이터 수, 에셋 구성: `datapack.unity3d` 미추출
- 수익 구조의 실제 동작(광고 노출 시점 등)
- 추가 분석이 필요하면 정품 APK로 진행하는 것을 권장합니다.
