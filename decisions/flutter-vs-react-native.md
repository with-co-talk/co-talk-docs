---
layout: default
title: Flutter vs React Native
description: Flutter 선택 이유 및 상세 비교
permalink: /decisions/flutter-vs-react-native
---

# Flutter vs React Native 상세 비교

[← 기술 결정 목록](./index)

**결정**: Flutter 3.0+
**상태**: 승인됨

---

## 목차

- [기능 비교](#1-기능-비교)
- [성능 비교](#2-성능-비교)
- [개발 경험](#3-개발-경험)
- [최종 결정](#4-최종-결정)

---

## 1. 기능 비교

### 1.1 실시간 통신

#### React Native
**라이브러리:**
- `socket.io-client`: Socket.io 지원
- `react-native-websocket`: WebSocket 지원
- `@react-native-async-storage/async-storage`: 로컬 저장소

**코드 예시:**
```javascript
import io from 'socket.io-client';

const socket = io('https://api.cotalk.com');
socket.on('message:new', (message) => {
  // 메시지 처리
});
```

#### Flutter
**라이브러리:**
- `socket_io_client`: Socket.io 지원
- `web_socket_channel`: WebSocket 지원
- `shared_preferences`: 로컬 저장소
- `sqflite`: SQLite 데이터베이스

**코드 예시:**
```dart
import 'package:socket_io_client/socket_io_client.dart' as IO;

final socket = IO.io('https://api.cotalk.com');
socket.on('message:new', (data) {
  // 메시지 처리
});
```

**결론: 둘 다 실시간 통신 기능 충분함**

### 1.2 상태 관리

#### React Native
**옵션:**
- Redux Toolkit
- Zustand
- React Query
- Context API

**코드 예시:**
```javascript
import { create } from 'zustand';

const useChatStore = create((set) => ({
  messages: [],
  addMessage: (message) => set((state) => ({
    messages: [...state.messages, message]
  }))
}));
```

#### Flutter
**옵션:**
- Provider
- Riverpod (권장)
- Bloc
- GetX

**코드 예시:**
```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

final messagesProvider = StateNotifierProvider<MessagesNotifier, List<Message>>((ref) {
  return MessagesNotifier();
});

class MessagesNotifier extends StateNotifier<List<Message>> {
  void addMessage(Message message) {
    state = [...state, message];
  }
}
```

**결론: 둘 다 강력한 상태 관리 옵션 제공**

### 1.3 UI 구성

#### React Native
**특징:**
- 플랫폼별 네이티브 컴포넌트 사용
- iOS는 UIKit, Android는 Material Design
- 플랫폼별 차이 존재

**라이브러리:**
- React Native Elements
- NativeBase
- UI Kitten

#### Flutter
**특징:**
- 자체 렌더링 엔진 (Skia)
- 모든 플랫폼에서 동일한 UI
- Material Design, Cupertino 위젯 제공

**장점:**
- UI 일관성 (모든 플랫폼 동일)
- 커스터마이징 용이
- 애니메이션 부드러움

**결론: Flutter가 UI 일관성과 커스터마이징에 유리**

### 1.4 성능

#### React Native
**특징:**
- JavaScript 브릿지를 통해 네이티브 코드 호출
- 네이티브 브릿지 오버헤드 존재
- 대부분의 경우 충분한 성능

**성능:**
- 스크롤: 부드러움
- 애니메이션: 좋음
- 메모리: 보통

#### Flutter
**특징:**
- 컴파일 타임 최적화
- 네이티브 코드로 컴파일
- 네이티브 성능에 가까움

**성능:**
- 스크롤: 매우 부드러움
- 애니메이션: 매우 부드러움
- 메모리: 효율적

**결론: Flutter가 성능 면에서 약간 우수**

---

## 2. 채팅앱 개발 관점 비교

### 2.1 채팅 UI 구성

#### React Native
**라이브러리:**
- `react-native-gifted-chat`: 채팅 UI 라이브러리
- `react-native-flatlist`: 리스트 최적화

**특징:**
- 채팅 UI 라이브러리 풍부
- 커스터마이징 가능

#### Flutter
**라이브러리:**
- `flutter_chat_bubble`: 채팅 버블
- `flutter_list_view`: 리스트 최적화
- 또는 직접 구현 (위젯으로 쉽게 구성)

**특징:**
- 위젯으로 쉽게 구성 가능
- 커스터마이징 매우 용이
- 애니메이션 부드러움

**결론: 둘 다 채팅 UI 구성 가능, Flutter가 커스터마이징에 유리**

### 2.2 실시간 메시지 처리

#### React Native
**구현:**
```javascript
useEffect(() => {
  socket.on('message:new', (message) => {
    setMessages(prev => [...prev, message]);
  });
  
  return () => socket.off('message:new');
}, []);
```

#### Flutter
**구현:**
```dart
socket.on('message:new', (data) {
  ref.read(messagesProvider.notifier).addMessage(
    Message.fromJson(data)
  );
});
```

**결론: 둘 다 실시간 메시지 처리 가능**

### 2.3 이미지/파일 처리

#### React Native
**라이브러리:**
- `react-native-image-picker`: 이미지 선택
- `react-native-image-cache`: 이미지 캐싱
- `react-native-fs`: 파일 시스템

#### Flutter
**라이브러리:**
- `image_picker`: 이미지 선택
- `cached_network_image`: 이미지 캐싱
- `path_provider`: 파일 경로
- `file_picker`: 파일 선택

**결론: 둘 다 이미지/파일 처리 기능 충분**

### 2.4 푸시 알림

#### React Native
**라이브러리:**
- `@react-native-firebase/messaging`: Firebase 푸시 알림
- `react-native-push-notification`: 로컬 푸시 알림

#### Flutter
**라이브러리:**
- `firebase_messaging`: Firebase 푸시 알림
- `flutter_local_notifications`: 로컬 푸시 알림

**결론: 둘 다 푸시 알림 지원**

---

## 3. 개발 생산성 비교

### 3.1 학습 곡선

#### React Native
**장점:**
- JavaScript/TypeScript 사용 (웹 개발자에게 친숙)
- React 지식 활용 가능
- 웹 개발 경험 활용 가능

**단점:**
- 네이티브 모듈 이해 필요
- 플랫폼별 차이 처리 필요

#### Flutter
**장점:**
- Dart 언어가 배우기 쉬움
- 일관된 API
- 풍부한 문서

**단점:**
- Dart 언어 학습 필요
- 웹 개발 경험 활용 어려움

**결론: React Native가 웹 개발자에게 더 친숙**

### 3.2 개발 속도

#### React Native
**특징:**
- Fast Refresh 지원
- 웹과 코드 일부 공유 가능
- 풍부한 라이브러리

#### Flutter
**특징:**
- Hot Reload 지원 (매우 빠름)
- 단일 코드베이스로 모든 플랫폼
- 풍부한 위젯

**결론: 둘 다 빠른 개발 속도, Flutter의 Hot Reload가 약간 더 빠름**

### 3.3 생태계

#### React Native
**특징:**
- JavaScript 생태계 활용 가능
- npm 패키지 사용 가능
- 풍부한 라이브러리

#### Flutter
**특징:**
- pub.dev 패키지
- 풍부한 패키지 (10만개+)
- Google 지원

**결론: 둘 다 풍부한 생태계**

---

## 4. Co-Talk 프로젝트 관점

### 4.1 React Native 선택 시

**장점:**
- React 지식 활용 가능 (웹 개발 경험)
- 웹과 코드 일부 공유 가능
- JavaScript 생태계 활용

**단점:**
- 성능이 Flutter보다 약간 낮음
- 플랫폼별 차이 처리 필요

**적합한 경우:**
- 웹 개발 경험이 있는 팀
- 웹과 모바일을 동시에 개발
- React 지식을 활용하고 싶을 때

### 4.2 Flutter 선택 시

**장점:**
- **성능 우수**: 네이티브에 가까운 성능
- **UI 일관성**: 모든 플랫폼에서 동일한 UI
- **크로스 플랫폼**: iOS, Android, Web, Desktop 모두 지원
- **개발 속도**: Hot Reload로 빠른 개발

**단점:**
- Dart 언어 학습 필요
- 웹 개발 경험 활용 어려움

**적합한 경우:**
- 성능이 중요한 경우
- UI 일관성이 중요한 경우
- iOS, Android, Web 모두 지원하고 싶을 때
- 새로운 언어 학습에 거부감이 없는 경우

---

## 5. 최종 비교표

| 항목 | React Native | Flutter |
|------|------------|---------|
| **실시간 통신** | ✅ 충분 | ✅ 충분 |
| **상태 관리** | ✅ 강력 | ✅ 강력 |
| **UI 구성** | ✅ 좋음 | ✅ 매우 좋음 |
| **성능** | ✅ 좋음 | ✅ 매우 좋음 |
| **크로스 플랫폼** | iOS, Android | iOS, Android, Web, Desktop |
| **학습 곡선** | 낮음 (React 지식) | 중간 (Dart 학습) |
| **개발 속도** | 빠름 | 매우 빠름 |
| **생태계** | 풍부 | 풍부 |
| **웹 경험 활용** | ✅ 가능 | ❌ 어려움 |

---

## 6. 결론

### Flutter는 기능이 부족하지 않음

**오히려 매우 강력함:**
- ✅ 실시간 통신 기능 충분
- ✅ 상태 관리 강력
- ✅ UI 구성 용이
- ✅ 성능 우수
- ✅ 크로스 플랫폼 지원 (iOS, Android, Web, Desktop)

### 선택 기준

**React Native 선택:**
- 웹 개발 경험이 있는 팀
- React 지식을 활용하고 싶을 때
- 웹과 모바일을 동시에 개발

**Flutter 선택:**
- 성능이 중요한 경우
- UI 일관성이 중요한 경우
- iOS, Android, Web 모두 지원하고 싶을 때
- 새로운 언어 학습에 거부감이 없는 경우

### Co-Talk 프로젝트 권장

**현재 상황:**
- 웹은 React로 개발 예정
- React 지식 활용 가능

**권장:**
- **React Native**: React 지식 활용, 웹과 코드 일부 공유
- **Flutter**: 성능과 UI 일관성 우선 시 선택 가능

**핵심:**
- Flutter는 기능이 부족하지 않음
- 오히려 성능과 UI 일관성 면에서 우수
- 선택은 팀의 경험과 우선순위에 따라 결정
