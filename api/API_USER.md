---
layout: default
title: User API
parent: API Reference
nav_order: 2
---

# 사용자 API

사용자 정보 조회 및 프로필 관리를 위한 REST API 문서입니다.

**Base URL**: `/api/v1/users`

---

## 목차

1. [내 정보 조회](#1-내-정보-조회)
2. [사용자 검색](#2-사용자-검색)
3. [프로필 수정](#3-프로필-수정)
4. [온라인 상태 업데이트](#4-온라인-상태-업데이트)
5. [마지막 접속 시간 업데이트](#5-마지막-접속-시간-업데이트)

---

## 1. 내 정보 조회

현재 로그인한 사용자의 정보를 조회합니다.

### Request

```http
GET /api/v1/users/me
Authorization: Bearer {accessToken}
```

### Response

**200 OK**
```json
{
  "id": 1,
  "email": "user@example.com",
  "nickname": "닉네임",
  "avatarUrl": "https://example.com/avatar.png",
  "onlineStatus": "ONLINE",
  "lastActiveAt": "2025-01-22T10:30:00"
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 사용자 ID |
| email | String | 이메일 |
| nickname | String | 닉네임 |
| avatarUrl | String | 프로필 이미지 URL (nullable) |
| onlineStatus | String | 온라인 상태 (`ONLINE`, `OFFLINE`, `AWAY`, `DO_NOT_DISTURB`) |
| lastActiveAt | String | 마지막 접속 시간 (ISO 8601) |

### Error Response

**401 Unauthorized** - 인증 필요
```json
{
  "error": "인증이 필요합니다.",
  "code": "UNAUTHORIZED",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

## 2. 사용자 검색

닉네임에 포함된 키워드로 사용자를 검색합니다.

### Request

```http
GET /api/v1/users/search?query={keyword}
Authorization: Bearer {accessToken}
```

또는

```http
GET /api/v1/users/search?nickname={keyword}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| query | String | △ | 검색 키워드 (우선순위 높음) |
| nickname | String | △ | 닉네임 검색 키워드 |

> **Note**: `query` 또는 `nickname` 중 하나는 필수입니다. 둘 다 있으면 `query`가 우선됩니다.

### Response

**200 OK**
```json
{
  "users": [
    {
      "id": 2,
      "email": "user2@example.com",
      "nickname": "검색된사용자",
      "avatarUrl": "https://example.com/avatar2.png",
      "onlineStatus": "ONLINE",
      "lastActiveAt": "2025-01-22T10:00:00"
    },
    {
      "id": 3,
      "email": "user3@example.com",
      "nickname": "다른사용자",
      "avatarUrl": null,
      "onlineStatus": "OFFLINE",
      "lastActiveAt": "2025-01-21T15:30:00"
    }
  ]
}
```

### Error Response

**400 Bad Request** - 검색 키워드 누락
```json
{
  "error": "query 또는 nickname 파라미터 중 하나는 필수입니다.",
  "code": "INVALID_ARGUMENT",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

## 3. 프로필 수정

사용자 프로필(닉네임, 아바타)을 수정합니다.

### Request

```http
PUT /api/v1/users/{userId}/profile
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "nickname": "새닉네임",
  "avatarUrl": "https://example.com/new-avatar.png"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| nickname | String | - | 변경할 닉네임 |
| avatarUrl | String | - | 변경할 프로필 이미지 URL |

### Response

**200 OK**
```json
{
  "message": "프로필이 수정되었습니다."
}
```

### Error Response

**401 Unauthorized** - 자신의 프로필만 수정 가능
```json
{
  "error": "자신의 리소스만 접근할 수 있습니다.",
  "code": "UNAUTHORIZED",
  "timestamp": "2025-01-22T10:30:00"
}
```

**409 Conflict** - 닉네임 중복
```json
{
  "error": "이미 사용 중인 닉네임입니다.",
  "code": "DUPLICATE_NICKNAME",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

## 4. 온라인 상태 업데이트

사용자의 온라인 상태를 업데이트합니다.

### Request

```http
PUT /api/v1/users/{userId}/online-status
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "status": "ONLINE"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| status | String | O | 온라인 상태 |

**온라인 상태 종류**:

| 상태 | 설명 |
|------|------|
| ONLINE | 온라인 |
| OFFLINE | 오프라인 |
| AWAY | 자리 비움 |
| DO_NOT_DISTURB | 방해 금지 |

### Response

**200 OK**
```json
{
  "message": "온라인 상태가 업데이트되었습니다."
}
```

### Error Response

**401 Unauthorized** - 자신의 상태만 변경 가능

---

## 5. 마지막 접속 시간 업데이트

사용자의 마지막 접속 시간을 현재 시간으로 업데이트합니다.

### Request

```http
PUT /api/v1/users/{userId}/last-active
Authorization: Bearer {accessToken}
```

### Response

**200 OK**
```json
{
  "message": "마지막 접속 시간이 업데이트되었습니다."
}
```

### Error Response

**401 Unauthorized** - 자신의 시간만 업데이트 가능

---

## 온라인 상태 관리

### 권장 구현 방식

1. **앱 포그라운드 진입 시**: `ONLINE` 상태로 업데이트
2. **앱 백그라운드 진입 시**: `AWAY` 또는 `OFFLINE` 상태로 업데이트
3. **주기적 heartbeat**: 30초~1분 간격으로 마지막 접속 시간 업데이트
4. **방해 금지 모드**: 사용자가 수동으로 `DO_NOT_DISTURB` 설정

### Flutter 예시

```dart
// 앱 라이프사이클 감지
class AppLifecycleObserver extends WidgetsBindingObserver {
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    switch (state) {
      case AppLifecycleState.resumed:
        userApi.updateOnlineStatus(userId, 'ONLINE');
        break;
      case AppLifecycleState.paused:
        userApi.updateOnlineStatus(userId, 'AWAY');
        break;
      default:
        break;
    }
  }
}

// Heartbeat 타이머
Timer.periodic(Duration(seconds: 30), (_) {
  userApi.updateLastActive(userId);
});
```
