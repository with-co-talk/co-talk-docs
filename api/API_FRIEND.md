---
layout: default
title: Friend API
parent: API Reference
nav_order: 3
---

# 친구 API

친구 요청, 친구 목록 관리, 사용자 차단을 위한 REST API 문서입니다.

---

## 목차

### 친구 요청
1. [친구 요청 전송](#1-친구-요청-전송)
2. [친구 요청 수락](#2-친구-요청-수락)
3. [친구 요청 거절](#3-친구-요청-거절)
4. [받은 친구 요청 목록 조회](#4-받은-친구-요청-목록-조회)
5. [보낸 친구 요청 목록 조회](#5-보낸-친구-요청-목록-조회)

### 친구 관리
6. [친구 목록 조회](#6-친구-목록-조회)
7. [친구 삭제](#7-친구-삭제)

### 차단
8. [사용자 차단](#8-사용자-차단)
9. [차단 해제](#9-차단-해제)
10. [차단 목록 조회](#10-차단-목록-조회)

---

## 친구 요청

**Base URL**: `/api/v1/friends`

### 1. 친구 요청 전송

다른 사용자에게 친구 요청을 보냅니다.

```http
POST /api/v1/friends/requests
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "requesterId": 1,
  "receiverId": 2
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| requesterId | Long | O | 요청자 ID (본인) |
| receiverId | Long | O | 수신자 ID |

**Response (201 Created)**:
```json
{
  "requestId": 100,
  "message": "친구 요청이 전송되었습니다."
}
```

**Error Response**:

**400 Bad Request** - 잘못된 친구 요청
```json
{
  "error": "자기 자신에게 친구 요청을 보낼 수 없습니다.",
  "code": "INVALID_FRIEND_REQUEST",
  "timestamp": "2025-01-22T10:30:00"
}
```

```json
{
  "error": "이미 친구 요청을 보냈습니다.",
  "code": "INVALID_FRIEND_REQUEST",
  "timestamp": "2025-01-22T10:30:00"
}
```

```json
{
  "error": "이미 친구 관계입니다.",
  "code": "INVALID_FRIEND_REQUEST",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

### 2. 친구 요청 수락

받은 친구 요청을 수락합니다.

```http
POST /api/v1/friends/requests/{requestId}/accept?userId={userId}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| requestId | Long | O | 친구 요청 ID (Path) |
| userId | Long | O | 요청 수락자 ID (Query) |

**Response (200 OK)**:
```json
{
  "message": "친구 요청을 수락했습니다."
}
```

**Error Response**:

**403 Forbidden** - 권한 없음
```json
{
  "error": "본인에게 온 요청만 수락할 수 있습니다.",
  "code": "FRIEND_REQUEST_ACCESS_DENIED",
  "timestamp": "2025-01-22T10:30:00"
}
```

**404 Not Found** - 친구 요청 없음
```json
{
  "error": "친구 요청을 찾을 수 없습니다.",
  "code": "FRIEND_REQUEST_NOT_FOUND",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

### 3. 친구 요청 거절

받은 친구 요청을 거절합니다.

```http
POST /api/v1/friends/requests/{requestId}/reject?userId={userId}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| requestId | Long | O | 친구 요청 ID (Path) |
| userId | Long | O | 요청 거절자 ID (Query) |

**Response (200 OK)**:
```json
{
  "message": "친구 요청을 거절했습니다."
}
```

---

### 4. 받은 친구 요청 목록 조회

사용자가 받은 대기 중인 친구 요청 목록을 조회합니다.

```http
GET /api/v1/friends/requests/received?userId={userId}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| userId | Long | O | 사용자 ID |

**Response (200 OK)**:
```json
{
  "requests": [
    {
      "id": 100,
      "requester": {
        "id": 2,
        "email": "user2@example.com",
        "nickname": "요청자",
        "avatarUrl": "https://example.com/avatar2.png",
        "onlineStatus": "ONLINE",
        "lastActiveAt": "2025-01-22T10:00:00"
      },
      "receiver": {
        "id": 1,
        "email": "user@example.com",
        "nickname": "나",
        "avatarUrl": "https://example.com/avatar.png",
        "onlineStatus": "ONLINE",
        "lastActiveAt": "2025-01-22T10:30:00"
      },
      "status": "PENDING",
      "createdAt": "2025-01-22T09:00:00"
    }
  ]
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 친구 요청 ID |
| requester | Object | 요청을 보낸 사용자 정보 |
| receiver | Object | 요청을 받은 사용자 정보 |
| status | String | 요청 상태 (`PENDING`, `ACCEPTED`, `REJECTED`) |
| createdAt | String | 요청 생성 시간 (ISO 8601) |

---

### 5. 보낸 친구 요청 목록 조회

사용자가 보낸 대기 중인 친구 요청 목록을 조회합니다.

```http
GET /api/v1/friends/requests/sent?userId={userId}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| userId | Long | O | 사용자 ID |

**Response (200 OK)**:
```json
{
  "requests": [
    {
      "id": 101,
      "requester": {
        "id": 1,
        "email": "user@example.com",
        "nickname": "나",
        "avatarUrl": "https://example.com/avatar.png",
        "onlineStatus": "ONLINE",
        "lastActiveAt": "2025-01-22T10:30:00"
      },
      "receiver": {
        "id": 3,
        "email": "user3@example.com",
        "nickname": "상대방",
        "avatarUrl": null,
        "onlineStatus": "OFFLINE",
        "lastActiveAt": "2025-01-21T15:00:00"
      },
      "status": "PENDING",
      "createdAt": "2025-01-22T08:00:00"
    }
  ]
}
```

---

## 친구 관리

### 6. 친구 목록 조회

사용자의 친구 목록을 조회합니다.

```http
GET /api/v1/friends?userId={userId}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| userId | Long | O | 사용자 ID |

**Response (200 OK)**:
```json
{
  "friends": [
    {
      "id": 2,
      "nickname": "친구1",
      "avatarUrl": "https://example.com/avatar2.png",
      "onlineStatus": "ONLINE",
      "lastActiveAt": "2025-01-22T10:00:00"
    },
    {
      "id": 3,
      "nickname": "친구2",
      "avatarUrl": null,
      "onlineStatus": "OFFLINE",
      "lastActiveAt": "2025-01-21T15:00:00"
    }
  ]
}
```

---

### 7. 친구 삭제

친구 관계를 삭제합니다.

```http
DELETE /api/v1/friends/{friendId}?userId={userId}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| friendId | Long | O | 삭제할 친구의 사용자 ID (Path) |
| userId | Long | O | 요청자 ID (Query) |

**Response (200 OK)**:
```json
{
  "message": "친구가 삭제되었습니다."
}
```

**Error Response**:

**404 Not Found** - 친구 관계 없음
```json
{
  "error": "친구 관계를 찾을 수 없습니다.",
  "code": "FRIEND_NOT_FOUND",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

## 차단

**Base URL**: `/api/v1/blocks`

### 8. 사용자 차단

특정 사용자를 차단합니다.

```http
POST /api/v1/blocks
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "blockerId": 1,
  "blockedId": 2
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| blockerId | Long | O | 차단하는 사용자 ID (본인) |
| blockedId | Long | O | 차단할 사용자 ID |

**Response (201 Created)**:
```json
{
  "message": "사용자를 차단했습니다."
}
```

**Error Response**:

**400 Bad Request** - 잘못된 차단 요청
```json
{
  "error": "자기 자신을 차단할 수 없습니다.",
  "code": "INVALID_BLOCK",
  "timestamp": "2025-01-22T10:30:00"
}
```

```json
{
  "error": "이미 차단한 사용자입니다.",
  "code": "INVALID_BLOCK",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

### 9. 차단 해제

사용자 차단을 해제합니다.

```http
DELETE /api/v1/blocks?blockerId={blockerId}&blockedId={blockedId}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| blockerId | Long | O | 차단한 사용자 ID (본인) |
| blockedId | Long | O | 차단된 사용자 ID |

**Response (200 OK)**:
```json
{
  "message": "차단을 해제했습니다."
}
```

**Error Response**:

**404 Not Found** - 차단 기록 없음
```json
{
  "error": "차단 정보를 찾을 수 없습니다.",
  "code": "BLOCK_NOT_FOUND",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

### 10. 차단 목록 조회

차단한 사용자 목록을 조회합니다.

```http
GET /api/v1/blocks?userId={userId}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| userId | Long | O | 요청 사용자 ID |

**Response (200 OK)**:
```json
{
  "blockedUsers": [
    {
      "id": 2,
      "nickname": "차단된사용자",
      "avatarUrl": "https://example.com/avatar2.png"
    }
  ]
}
```

---

## 친구 요청 상태

| 상태 | 설명 |
|------|------|
| PENDING | 대기 중 |
| ACCEPTED | 수락됨 |
| REJECTED | 거절됨 |

---

## 차단 시 동작

사용자를 차단하면 다음과 같은 동작이 발생합니다:

1. **친구 관계 해제**: 기존 친구 관계가 자동으로 삭제됩니다.
2. **친구 요청 거부**: 차단된 사용자의 친구 요청이 자동으로 거절됩니다.
3. **메시지 차단**: 차단된 사용자로부터의 메시지를 받지 않습니다.
4. **검색 제외**: 차단된 사용자는 검색 결과에서 제외됩니다.
