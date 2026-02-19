---
layout: default
title: Frontend Tech Stack
parent: Tech Stack
nav_order: 2
---

# 프론트엔드 기술 스택

[← 기술 스택 개요](./index)

---

## 목차

- [플랫폼 전략](#플랫폼-전략)
- [Flutter 프레임워크](#flutter-프레임워크)
- [상태 관리](#상태-관리)
- [실시간 통신](#실시간-통신)
- [네트워킹](#네트워킹)
- [로컬 데이터베이스](#로컬-데이터베이스)
- [의존성 관리](#의존성-관리)
- [빌드 및 배포](#빌드-및-배포)

---

## 플랫폼 전략

### ✅ 결정: Flutter (단일 코드베이스, 전 플랫폼)

| 플랫폼 | 지원 상태 |
|--------|----------|
| **Android** | ✅ 운영 중 |
| **iOS** | ✅ 운영 중 |
| **macOS** | ✅ 운영 중 |
| **Windows** | ✅ 운영 중 (MSIX 패키징) |
| **Linux** | ✅ 운영 중 |

**Flutter 선택 이유**
- 단일 코드베이스로 5개 플랫폼 지원
- 네이티브에 가까운 성능
- 모든 플랫폼에서 동일한 UI/UX
- Hot Reload로 빠른 개발
- Material 3 위젯 라이브러리

→ [프론트엔드 플랫폼 전략 상세](../decisions/frontend-platform-strategy)
→ [Flutter vs React Native 비교](../decisions/flutter-vs-react-native)

---

## Flutter 프레임워크

### ✅ 결정: Flutter 3.8+ (Dart SDK ^3.8.1)

| 항목 | 선택 |
|------|------|
| **Framework** | Flutter 3.8+ |
| **Language** | Dart SDK ^3.8.1 |
| **Architecture** | Clean Layered (Presentation → Domain ← Data) |
| **Design System** | Material 3 |
| **Theme** | Light / Dark / System |
| **Primary Color** | Violet #8B5CF6 |

**아키텍처 레이어**

| 레이어 | 역할 |
|--------|------|
| **Presentation** | Pages (UI) + BLoCs/Cubits (상태 관리) |
| **Domain** | 순수 Dart 엔티티 + 추상 Repository 인터페이스 |
| **Data** | Repository 구현체, Remote/Local 데이터 소스 |
| **Core** | 네트워킹, 라우팅, 테마, 서비스, 에러 처리 |
| **DI** | get_it + injectable (컴파일 타임 안전, 환경별 등록) |

---

## 상태 관리

### ✅ 결정: BLoC/Cubit (flutter_bloc)

**선택 이유**
- 엄격한 Event/State 분리로 예측 가능한 상태
- Equatable로 불변 상태 객체
- 테스트 용이 (bloc_test)
- 대규모 앱에서 검증된 패턴

**주요 BLoC/Cubit**

| 이름 | 범위 | 역할 |
|------|------|------|
| `AuthBloc` | `@injectable` | 인증, 로그인, 회원가입, 프로필, WebSocket 연결 |
| `ChatListBloc` | `@lazySingleton` | 채팅방 목록, 실시간 업데이트 |
| `ChatRoomBloc` | `@injectable` | 메시지, 리액션, 타이핑, 프레즌스, 파일 업로드 |
| `FriendBloc` | `@injectable` | 친구, 요청, 검색, 차단/숨김 |
| `ThemeCubit` | `@lazySingleton` | 라이트/다크/시스템 테마 |
| `ChatSettingsCubit` | `@lazySingleton` | 폰트 크기, 자동 다운로드, 타이핑 표시 |
| `NotificationSettingsCubit` | `@lazySingleton` | 알림 설정, DND, 미리보기 모드 |
| `AppLockCubit` | `@lazySingleton` | 생체인증 잠금 (30초 grace period) |

**대안 비교**
| 기술 | 평가 |
|------|------|
| Riverpod | ⚠️ BLoC 대비 구조적 강제력 낮음 |
| GetX | ❌ 테스트 어려움, 아키텍처 미강제 |
| Provider | ⚠️ 대규모 앱에서 복잡도 증가 |

---

## 실시간 통신

### ✅ 결정: STOMP over WebSocket (stomp_dart_client)

**WebSocketService** (Facade 패턴)

| 내부 매니저 | 역할 |
|-------------|------|
| `ConnectionManager` | 연결/해제, 지수 백오프 자동 재연결 |
| `SubscriptionManager` | 토픽 구독 관리, 재연결 시 복원 |
| `MessageSender` | STOMP 프레임 전송 |
| `PayloadParser` | 수신 프레임을 타입 이벤트로 역직렬화 |

**이벤트 스트림**
- `messageStream` — 채팅 메시지
- `reactionStream` — 리액션 추가/제거
- `readStream` — 읽음 확인
- `typingStream` — 타이핑 표시
- `onlineStatusStream` — 온라인 상태
- `messageDeletedStream` — 메시지 삭제
- `messageUpdatedStream` — 메시지 수정
- `linkPreviewStream` — 링크 프리뷰
- `chatRoomUpdateStream` — 채팅 목록 업데이트

**주요 기능**
- 이벤트 중복 제거 (EventDedupeCache with TTL)
- 재연결 시 gap recovery (누락 메시지 복구)
- 독립적인 토큰 갱신

---

## 네트워킹

### ✅ 결정: Dio (REST Client)

**주요 기능**

| 기능 | 설명 |
|------|------|
| **Auth Interceptor** | JWT 자동 삽입, 401 토큰 갱신 (QueuedInterceptor) |
| **Certificate Pinning** | SHA-256 인증서 검증 (프로덕션 전용) |
| **환경별 URL** | dev=localhost:8080, prod=NAS HTTPS |
| **에러 처리** | 한국어 사용자 친화적 에러 메시지 매핑 |

---

## 로컬 데이터베이스

### ✅ 결정: Drift (SQLite) + FTS5

**테이블**
| 테이블 | 용도 |
|--------|------|
| `Messages` | 오프라인 메시지 캐시 |
| `ChatRooms` | 채팅방 메타데이터 캐시 |
| `MessageReactions` | 리액션 데이터 캐시 |

**FTS5 전문 검색**: 메시지 내용 + 파일명 검색용 가상 테이블 (sync trigger)

**오프라인 우선 전략**:
1. 채팅방 진입 → SQLite 캐시 로드 (즉시 표시)
2. 서버에서 최신 데이터 fetch → 캐시 병합/갱신 → UI 업데이트
3. WebSocket 메시지 수신 → 캐시 추가 + UI 반영
4. 재연결 시 → gap recovery로 누락 메시지 복구

---

## 의존성 관리

### ✅ 결정: get_it + injectable (DI)

**환경별 등록**
- `mobile` — FCM, biometric, 전체 기능
- `desktop` — window_manager, NoOp FCM, 데스크톱 알림

### 주요 의존성 (42개)

| 카테고리 | 패키지 |
|----------|---------|
| **State** | `flutter_bloc`, `equatable`, `rxdart` |
| **Network** | `dio`, `stomp_dart_client` |
| **Storage** | `flutter_secure_storage`, `shared_preferences`, `drift` |
| **DI** | `get_it`, `injectable` |
| **Routing** | `go_router` |
| **Media** | `image_picker`, `image_cropper`, `pro_image_editor`, `video_player`, `chewie`, `emoji_picker_flutter` |
| **Push** | `firebase_core`, `firebase_messaging`, `flutter_local_notifications` |
| **Security** | `local_auth`, `firebase_app_check` |
| **Desktop** | `window_manager` |
| **UI** | `cached_network_image`, `shimmer`, `flutter_slidable` |
| **Serialization** | `json_annotation`, `json_serializable` |

---

## 빌드 및 배포

**개발 도구**
| 도구 | 용도 |
|------|------|
| `build_runner` | 코드 생성 (injectable, json_serializable, drift) |
| `flutter_launcher_icons` | 앱 아이콘 생성 |
| `msix` | Windows MSIX 패키징 |
| `mocktail` + `bloc_test` | 단위/BLoC 테스트 |

**환경 설정** (`--dart-define`)
- `ENV=dev` / `ENV=staging` / `ENV=prod`
- 환경별 API URL 자동 전환

---

## 다음 문서

→ [인프라 기술](./infrastructure)
