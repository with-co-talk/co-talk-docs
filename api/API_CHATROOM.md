---
layout: default
title: ChatRoom API
parent: API Reference
nav_order: 4
---

# 채팅방 API

채팅방 관리를 위한 REST API 문서입니다.

**Base URL**: `/api/v1/chat/rooms`

---

## 목차

1. [1:1 채팅방 생성](#1-11-채팅방-생성)
2. [그룹 채팅방 생성](#2-그룹-채팅방-생성)
3. [채팅방 목록 조회](#3-채팅방-목록-조회)
4. [채팅방 멤버 목록 조회](#4-채팅방-멤버-목록-조회)
5. [채팅방 나가기](#5-채팅방-나가기)
6. [읽음 표시](#6-읽음-표시)
7. [그룹 채팅방 멤버 초대](#7-그룹-채팅방-멤버-초대)
8. [채팅방 이름 변경](#8-채팅방-이름-변경)
9. [공지사항 설정](#9-공지사항-설정)
10. [공지사항 삭제](#10-공지사항-삭제)
11. [관리자 임명](#11-관리자-임명)
12. [관리자 해제](#12-관리자-해제)
13. [멤버 강제 퇴장](#13-멤버-강제-퇴장)

---

## 1. 1:1 채팅방 생성

두 사용자 간의 1:1 채팅방을 생성합니다.

### Request

```http
POST /api/v1/chat/rooms
Content-Type: application/json
```

**Body**:
```json
{
  "userId1": 1,
  "userId2": 2
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| userId1 | Long | O | 첫 번째 사용자 ID |
| userId2 | Long | O | 두 번째 사용자 ID |

### Response

**201 Created**
```json
{
  "roomId": 100,
  "message": "채팅방이 생성되었습니다."
}
```

---

## 2. 그룹 채팅방 생성

여러 사용자가 참여하는 그룹 채팅방을 생성합니다.

### Request

```http
POST /api/v1/chat/rooms/group
Content-Type: application/json
```

**Body**:
```json
{
  "creatorId": 1,
  "roomName": "개발팀 채팅방",
  "memberIds": [2, 3, 4]
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| creatorId | Long | O | 생성자 ID (자동으로 관리자 권한 부여) |
| roomName | String | O | 채팅방 이름 |
| memberIds | List\<Long\> | O | 초대할 멤버 ID 목록 |

### Response

**201 Created**
```json
{
  "roomId": 101,
  "message": "그룹 채팅방이 생성되었습니다."
}
```

---

## 3. 채팅방 목록 조회

사용자가 참여 중인 채팅방 목록을 조회합니다.

### Request

```http
GET /api/v1/chat/rooms?userId={userId}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| userId | Long | O | 사용자 ID |

### Response

**200 OK**
```json
{
  "rooms": [
    {
      "id": 100,
      "name": "채팅방1",
      "type": "DIRECT",
      "createdAt": "2025-01-22T10:30:00",
      "lastMessage": "마지막 메시지입니다",
      "lastMessageAt": "2025-01-22T10:30:00",
      "unreadCount": 5,
      "otherUserId": 2,
      "otherUserNickname": "상대방",
      "otherUserAvatarUrl": "https://example.com/avatar.png"
    },
    {
      "id": 101,
      "name": "개발팀 채팅방",
      "type": "GROUP",
      "createdAt": "2025-01-22T09:00:00",
      "lastMessage": "안녕하세요",
      "lastMessageAt": "2025-01-22T09:30:00",
      "unreadCount": 0,
      "otherUserId": null,
      "otherUserNickname": null,
      "otherUserAvatarUrl": null
    }
  ]
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 채팅방 ID |
| name | String | 채팅방 이름 |
| type | String | 채팅방 유형 (`DIRECT`: 1:1, `GROUP`: 그룹) |
| createdAt | String | 채팅방 생성 시간 (ISO 8601) |
| lastMessage | String | 마지막 메시지 내용 |
| lastMessageAt | String | 마지막 메시지 시간 |
| unreadCount | Long | 안읽은 메시지 수 |
| otherUserId | Long | 1:1 채팅방에서 상대방 ID (그룹은 null) |
| otherUserNickname | String | 1:1 채팅방에서 상대방 닉네임 |
| otherUserAvatarUrl | String | 1:1 채팅방에서 상대방 프로필 이미지 |

---

## 4. 채팅방 멤버 목록 조회

채팅방의 멤버 목록을 조회합니다. 관리자가 먼저 정렬되어 반환됩니다.

### Request

```http
GET /api/v1/chat/rooms/{roomId}/members?userId={userId}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| roomId | Long | O | 채팅방 ID (Path) |
| userId | Long | O | 요청 사용자 ID (Query) |

### Response

**200 OK**
```json
{
  "members": [
    {
      "userId": 1,
      "nickname": "관리자",
      "avatarUrl": "https://example.com/admin.png",
      "role": "ADMIN"
    },
    {
      "userId": 2,
      "nickname": "멤버1",
      "avatarUrl": null,
      "role": "MEMBER"
    }
  ]
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| userId | Long | 멤버 사용자 ID |
| nickname | String | 닉네임 |
| avatarUrl | String | 프로필 이미지 URL (nullable) |
| role | String | 역할 (`ADMIN`: 관리자, `MEMBER`: 일반 멤버) |

### Error Response

**403 Forbidden** - 채팅방 멤버가 아닌 경우
```json
{
  "error": "채팅방 멤버가 아닙니다.",
  "code": "ACCESS_DENIED",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

## 5. 채팅방 나가기

채팅방에서 나갑니다.

### Request

```http
POST /api/v1/chat/rooms/{roomId}/leave?userId={userId}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| roomId | Long | O | 채팅방 ID (Path) |
| userId | Long | O | 사용자 ID (Query) |

### Response

**200 OK**
```json
{
  "message": "채팅방을 나갔습니다."
}
```

---

## 6. 읽음 표시

채팅방의 메시지를 읽음 처리합니다.

### Request

```http
POST /api/v1/chat/rooms/{roomId}/read?userId={userId}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| roomId | Long | O | 채팅방 ID (Path) |
| userId | Long | O | 사용자 ID (Query) |

### Response

**200 OK**
```json
{
  "message": "읽음 처리되었습니다."
}
```

---

## 7. 그룹 채팅방 멤버 초대

그룹 채팅방에 새로운 멤버를 초대합니다.

### Request

```http
POST /api/v1/chat/rooms/{roomId}/invite
Content-Type: application/json
```

**Body**:
```json
{
  "inviterId": 1,
  "inviteeIds": [5, 6]
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| inviterId | Long | O | 초대하는 사용자 ID |
| inviteeIds | List\<Long\> | O | 초대할 사용자 ID 목록 |

### Response

**200 OK**
```json
{
  "message": "멤버를 초대했습니다."
}
```

---

## 8. 채팅방 이름 변경

그룹 채팅방의 이름을 변경합니다. **관리자 권한 필요**

### Request

```http
PUT /api/v1/chat/rooms/{roomId}/name
Content-Type: application/json
```

**Body**:
```json
{
  "userId": 1,
  "newName": "새로운 채팅방 이름"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| userId | Long | O | 요청자 ID (관리자) |
| newName | String | O | 변경할 이름 |

### Response

**200 OK**
```json
{
  "name": "새로운 채팅방 이름",
  "message": "채팅방 이름이 변경되었습니다."
}
```

---

## 9. 공지사항 설정

채팅방 공지사항을 설정합니다. **관리자 권한 필요**

### Request

```http
POST /api/v1/chat/rooms/{roomId}/announcement
Content-Type: application/json
```

**Body**:
```json
{
  "userId": 1,
  "announcement": "중요 공지사항입니다."
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| userId | Long | O | 요청자 ID (관리자) |
| announcement | String | O | 공지 내용 |

### Response

**200 OK**
```json
{
  "announcement": "중요 공지사항입니다.",
  "message": "공지사항이 설정되었습니다."
}
```

---

## 10. 공지사항 삭제

채팅방 공지사항을 삭제합니다. **관리자 권한 필요**

### Request

```http
DELETE /api/v1/chat/rooms/{roomId}/announcement?userId={userId}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| roomId | Long | O | 채팅방 ID (Path) |
| userId | Long | O | 요청자 ID (Query, 관리자) |

### Response

**200 OK**
```json
{
  "message": "공지사항이 삭제되었습니다."
}
```

---

## 11. 관리자 임명

멤버를 관리자로 임명합니다. **관리자 권한 필요**

### Request

```http
POST /api/v1/chat/rooms/{roomId}/admins/{targetUserId}?userId={userId}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| roomId | Long | O | 채팅방 ID (Path) |
| targetUserId | Long | O | 관리자로 임명할 사용자 ID (Path) |
| userId | Long | O | 요청자 ID (Query, 관리자) |

### Response

**200 OK**
```json
{
  "userId": 2,
  "role": "ADMIN",
  "message": "관리자로 임명되었습니다."
}
```

---

## 12. 관리자 해제

관리자 권한을 해제합니다. **관리자 권한 필요**

### Request

```http
DELETE /api/v1/chat/rooms/{roomId}/admins/{targetUserId}?userId={userId}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| roomId | Long | O | 채팅방 ID (Path) |
| targetUserId | Long | O | 권한 해제할 사용자 ID (Path) |
| userId | Long | O | 요청자 ID (Query, 관리자) |

### Response

**200 OK**
```json
{
  "userId": 2,
  "role": "MEMBER",
  "message": "관리자 권한이 해제되었습니다."
}
```

---

## 13. 멤버 강제 퇴장

채팅방에서 멤버를 강제 퇴장시킵니다. **관리자 권한 필요**

### Request

```http
DELETE /api/v1/chat/rooms/{roomId}/members/{targetUserId}?userId={userId}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| roomId | Long | O | 채팅방 ID (Path) |
| targetUserId | Long | O | 강제 퇴장시킬 사용자 ID (Path) |
| userId | Long | O | 요청자 ID (Query, 관리자) |

### Response

**200 OK**
```json
{
  "message": "멤버가 강제 퇴장되었습니다."
}
```

### Error Response

**400 Bad Request** - 1:1 채팅방에서 시도한 경우
```json
{
  "error": "1:1 채팅방에서는 멤버를 강제 퇴장시킬 수 없습니다.",
  "code": "BAD_REQUEST",
  "timestamp": "2025-01-22T10:30:00"
}
```

**400 Bad Request** - 자기 자신을 퇴장시키려는 경우
```json
{
  "error": "자기 자신을 강제 퇴장시킬 수 없습니다.",
  "code": "BAD_REQUEST",
  "timestamp": "2025-01-22T10:30:00"
}
```

**403 Forbidden** - 관리자가 아닌 경우
```json
{
  "error": "관리자만 멤버를 강제 퇴장시킬 수 있습니다.",
  "code": "ACCESS_DENIED",
  "timestamp": "2025-01-22T10:30:00"
}
```

**403 Forbidden** - 대상이 채팅방 멤버가 아닌 경우
```json
{
  "error": "대상 사용자가 채팅방 멤버가 아닙니다.",
  "code": "ACCESS_DENIED",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

## 공통 에러 응답

### 401 Unauthorized

인증되지 않은 요청 또는 토큰 만료

```json
{
  "error": "자신의 리소스만 접근할 수 있습니다.",
  "code": "UNAUTHORIZED",
  "timestamp": "2025-01-22T10:30:00"
}
```

### 404 Not Found

채팅방을 찾을 수 없는 경우

```json
{
  "error": "채팅방을 찾을 수 없습니다. ID: 999",
  "code": "CHAT_ROOM_NOT_FOUND",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

## 권한 체계

| 역할 | 설명 | 권한 |
|------|------|------|
| ADMIN | 관리자 | 이름 변경, 공지 설정/삭제, 관리자 임명/해제, 멤버 강제 퇴장 |
| MEMBER | 일반 멤버 | 메시지 전송, 채팅방 나가기, 멤버 초대 |

- 그룹 채팅방 생성자는 자동으로 ADMIN 권한 부여
- 관리자는 다른 멤버를 관리자로 임명 가능
- 1:1 채팅방에서는 관리자 기능 사용 불가
