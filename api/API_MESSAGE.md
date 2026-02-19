---
layout: default
title: Message API
parent: API Reference
nav_order: 5
---

# 채팅 메시지 API

채팅 메시지 전송, 조회, 수정, 삭제 및 반응(이모지)을 위한 REST API 문서입니다.

---

## 목차

### 메시지 (Message)
1. [메시지 전송](#1-메시지-전송)
2. [파일/이미지 메시지 전송](#2-파일이미지-메시지-전송)
3. [메시지 히스토리 조회](#3-메시지-히스토리-조회)
4. [메시지 수정](#4-메시지-수정)
5. [메시지 삭제](#5-메시지-삭제)
6. [메시지 답장](#6-메시지-답장)
7. [메시지 전달](#7-메시지-전달)

### 반응 (Reaction)
8. [메시지 반응 추가](#8-메시지-반응-추가)
9. [메시지 반응 제거](#9-메시지-반응-제거)
10. [메시지 반응 조회](#10-메시지-반응-조회)

### 검색 (Search)
11. [채팅방 내 메시지 검색](#11-채팅방-내-메시지-검색)
12. [전체 채팅방 메시지 검색](#12-전체-채팅방-메시지-검색)

---

## 메시지 (Message)

**Base URL**: `/api/v1/chat/messages`

### 1. 메시지 전송

채팅방에 텍스트 메시지를 전송합니다.

```http
POST /api/v1/chat/messages
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "chatRoomId": 100,
  "senderId": 1,
  "content": "안녕하세요!"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| chatRoomId | Long | O | 채팅방 ID |
| senderId | Long | O | 발신자 ID |
| content | String | O | 메시지 내용 |

**Response (201 Created)**:
```json
{
  "id": 1000,
  "chatRoomId": 100,
  "senderId": 1,
  "content": "안녕하세요!",
  "type": "TEXT",
  "status": "SENT",
  "createdAt": "2025-01-22T10:30:00",
  "updatedAt": null
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| id | Long | 메시지 ID |
| chatRoomId | Long | 채팅방 ID |
| senderId | Long | 발신자 ID |
| content | String | 메시지 내용 |
| type | String | 메시지 타입 (`TEXT`, `IMAGE`, `FILE`, `SYSTEM`) |
| status | String | 메시지 상태 (`SENT`, `EDITED`, `DELETED`) |
| createdAt | String | 생성 시간 (ISO 8601) |
| updatedAt | String | 수정 시간 (nullable) |

---

### 2. 파일/이미지 메시지 전송

채팅방에 파일 또는 이미지를 전송합니다.

```http
POST /api/v1/chat/messages/file
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "chatRoomId": 100,
  "senderId": 1,
  "fileUrl": "https://storage.example.com/files/image.png",
  "fileName": "image.png",
  "fileSize": 102400,
  "contentType": "image/png",
  "thumbnailUrl": "https://storage.example.com/thumbnails/image_thumb.png"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| chatRoomId | Long | O | 채팅방 ID |
| senderId | Long | O | 발신자 ID |
| fileUrl | String | O | 파일 URL |
| fileName | String | O | 파일명 |
| fileSize | Long | O | 파일 크기 (bytes) |
| contentType | String | O | MIME 타입 |
| thumbnailUrl | String | - | 썸네일 URL (이미지/동영상) |

**Response (201 Created)**:
```json
{
  "id": 1001,
  "chatRoomId": 100,
  "senderId": 1,
  "content": null,
  "type": "IMAGE",
  "status": "SENT",
  "fileUrl": "https://storage.example.com/files/image.png",
  "fileName": "image.png",
  "fileSize": 102400,
  "thumbnailUrl": "https://storage.example.com/thumbnails/image_thumb.png",
  "createdAt": "2025-01-22T10:31:00",
  "updatedAt": null
}
```

---

### 3. 메시지 히스토리 조회

채팅방의 메시지 히스토리를 커서 기반으로 조회합니다.

```http
GET /api/v1/chat/messages/rooms/{roomId}?userId={userId}&beforeMessageId={cursor}&size={size}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 기본값 | 설명 |
|----------|------|------|--------|------|
| roomId | Long | O | - | 채팅방 ID (Path) |
| userId | Long | O | - | 요청 사용자 ID |
| beforeMessageId | Long | - | - | 이 메시지 ID 이전의 메시지 조회 (커서) |
| size | Integer | - | 20 | 조회할 메시지 수 |

**Response (200 OK)**:
```json
{
  "messages": [
    {
      "id": 1000,
      "chatRoomId": 100,
      "senderId": 1,
      "senderNickname": "사용자1",
      "senderAvatarUrl": "https://example.com/avatar1.png",
      "content": "안녕하세요!",
      "type": "TEXT",
      "status": "SENT",
      "replyToMessageId": null,
      "forwardedFromMessageId": null,
      "createdAt": "2025-01-22T10:30:00",
      "updatedAt": null
    },
    {
      "id": 999,
      "chatRoomId": 100,
      "senderId": 2,
      "senderNickname": "사용자2",
      "senderAvatarUrl": null,
      "content": "반갑습니다!",
      "type": "TEXT",
      "status": "SENT",
      "replyToMessageId": null,
      "forwardedFromMessageId": null,
      "createdAt": "2025-01-22T10:29:00",
      "updatedAt": null
    }
  ],
  "nextCursor": 999,
  "hasMore": true
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| messages | Array | 메시지 목록 (최신순) |
| nextCursor | Long | 다음 페이지 조회용 커서 (nullable) |
| hasMore | Boolean | 더 많은 메시지 존재 여부 |

---

### 4. 메시지 수정

본인이 보낸 메시지를 수정합니다.

```http
PUT /api/v1/chat/messages/{messageId}
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "userId": 1,
  "content": "수정된 메시지입니다."
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| userId | Long | O | 요청 사용자 ID |
| content | String | O | 수정할 내용 |

**Response (200 OK)**:
```json
{
  "id": 1000,
  "chatRoomId": 100,
  "senderId": 1,
  "content": "수정된 메시지입니다.",
  "type": "TEXT",
  "status": "EDITED",
  "createdAt": "2025-01-22T10:30:00",
  "updatedAt": "2025-01-22T10:35:00"
}
```

**Error Response**:

**403 Forbidden** - 본인 메시지만 수정 가능
```json
{
  "error": "본인의 메시지만 수정할 수 있습니다.",
  "code": "MESSAGE_ACCESS_DENIED",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

### 5. 메시지 삭제

본인이 보낸 메시지를 삭제합니다.

```http
DELETE /api/v1/chat/messages/{messageId}?userId={userId}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| messageId | Long | O | 삭제할 메시지 ID (Path) |
| userId | Long | O | 요청 사용자 ID (Query) |

**Response (200 OK)**:
```json
{
  "message": "메시지가 삭제되었습니다."
}
```

---

### 6. 메시지 답장

특정 메시지에 답장합니다.

```http
POST /api/v1/chat/messages/{messageId}/reply
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "senderId": 1,
  "content": "네, 알겠습니다!"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| senderId | Long | O | 발신자 ID |
| content | String | O | 답장 내용 |

**Response (201 Created)**:
```json
{
  "id": 1002,
  "chatRoomId": 100,
  "senderId": 1,
  "content": "네, 알겠습니다!",
  "type": "TEXT",
  "status": "SENT",
  "replyToMessageId": 1000,
  "createdAt": "2025-01-22T10:32:00",
  "updatedAt": null
}
```

---

### 7. 메시지 전달

메시지를 다른 채팅방으로 전달합니다.

```http
POST /api/v1/chat/messages/{messageId}/forward
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "senderId": 1,
  "targetChatRoomId": 200
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| senderId | Long | O | 발신자 ID |
| targetChatRoomId | Long | O | 전달할 대상 채팅방 ID |

**Response (201 Created)**:
```json
{
  "id": 2000,
  "chatRoomId": 200,
  "senderId": 1,
  "content": "안녕하세요!",
  "type": "TEXT",
  "status": "SENT",
  "forwardedFromMessageId": 1000,
  "createdAt": "2025-01-22T10:33:00",
  "updatedAt": null
}
```

---

## 반응 (Reaction)

**Base URL**: `/api/v1/chat/messages/{messageId}/reactions`

### 8. 메시지 반응 추가

메시지에 이모지 반응을 추가합니다.

```http
POST /api/v1/chat/messages/{messageId}/reactions
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "emoji": "👍"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| emoji | String | O | 이모지 (유니코드) |

**Response (201 Created)**:
```json
{
  "id": 500,
  "messageId": 1000,
  "userId": 1,
  "emoji": "👍",
  "createdAt": "2025-01-22T10:34:00"
}
```

**Error Response**:

**400 Bad Request** - 유효하지 않은 이모지
```json
{
  "error": "유효하지 않은 이모지입니다.",
  "code": "INVALID_EMOJI",
  "timestamp": "2025-01-22T10:30:00"
}
```

---

### 9. 메시지 반응 제거

메시지에서 이모지 반응을 제거합니다.

```http
DELETE /api/v1/chat/messages/{messageId}/reactions
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "emoji": "👍"
}
```

**Response (200 OK)**:
```json
{
  "message": "반응이 제거되었습니다."
}
```

---

### 10. 메시지 반응 조회

메시지의 모든 반응을 이모지별로 그룹핑하여 조회합니다.

```http
GET /api/v1/chat/messages/{messageId}/reactions
Authorization: Bearer {accessToken}
```

**Response (200 OK)**:
```json
[
  {
    "emoji": "👍",
    "count": 3,
    "users": [
      {"id": 1, "nickname": "사용자1"},
      {"id": 2, "nickname": "사용자2"},
      {"id": 3, "nickname": "사용자3"}
    ],
    "reactedByMe": true
  },
  {
    "emoji": "❤️",
    "count": 1,
    "users": [
      {"id": 2, "nickname": "사용자2"}
    ],
    "reactedByMe": false
  }
]
```

| 필드 | 타입 | 설명 |
|------|------|------|
| emoji | String | 이모지 |
| count | Integer | 반응 수 |
| users | Array | 반응한 사용자 목록 |
| reactedByMe | Boolean | 본인 반응 여부 |

---

## 검색 (Search)

**Base URL**: `/api/v1/messages/search`

### 11. 채팅방 내 메시지 검색

특정 채팅방 내에서 키워드로 메시지를 검색합니다.

```http
GET /api/v1/messages/search?chatRoomId={roomId}&userId={userId}&keyword={keyword}&page={page}&size={size}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 기본값 | 설명 |
|----------|------|------|--------|------|
| chatRoomId | Long | O | - | 채팅방 ID |
| userId | Long | O | - | 요청 사용자 ID |
| keyword | String | O | - | 검색 키워드 (최대 255자) |
| page | Integer | - | 0 | 페이지 번호 |
| size | Integer | - | 20 | 페이지 크기 |

**Response (200 OK)**:
```json
{
  "messages": [
    {
      "id": 1000,
      "chatRoomId": 100,
      "chatRoomName": "개발팀",
      "senderId": 1,
      "senderNickname": "사용자1",
      "content": "검색된 메시지 내용입니다.",
      "type": "TEXT",
      "createdAt": "2025-01-22T10:30:00"
    }
  ]
}
```

---

### 12. 전체 채팅방 메시지 검색

사용자가 속한 모든 채팅방에서 키워드로 메시지를 검색합니다.

```http
GET /api/v1/messages/search/all?userId={userId}&keyword={keyword}&page={page}&size={size}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 기본값 | 설명 |
|----------|------|------|--------|------|
| userId | Long | O | - | 요청 사용자 ID |
| keyword | String | O | - | 검색 키워드 (최대 255자) |
| page | Integer | - | 0 | 페이지 번호 |
| size | Integer | - | 20 | 페이지 크기 |

**Response (200 OK)**:
```json
{
  "messages": [
    {
      "id": 1000,
      "chatRoomId": 100,
      "chatRoomName": "개발팀",
      "senderId": 1,
      "senderNickname": "사용자1",
      "content": "검색된 메시지 내용입니다.",
      "type": "TEXT",
      "createdAt": "2025-01-22T10:30:00"
    },
    {
      "id": 2000,
      "chatRoomId": 200,
      "chatRoomName": "친구와의 대화",
      "senderId": 2,
      "senderNickname": "사용자2",
      "content": "다른 방에서 검색된 메시지",
      "type": "TEXT",
      "createdAt": "2025-01-21T15:00:00"
    }
  ]
}
```

---

## 메시지 타입

| 타입 | 설명 |
|------|------|
| TEXT | 텍스트 메시지 |
| IMAGE | 이미지 메시지 |
| FILE | 파일 메시지 |
| SYSTEM | 시스템 메시지 (입장, 퇴장 등) |

## 메시지 상태

| 상태 | 설명 |
|------|------|
| SENT | 전송됨 |
| EDITED | 수정됨 |
| DELETED | 삭제됨 |

---

## 페이지네이션

### 커서 기반 (메시지 히스토리)

메시지 히스토리 조회는 커서 기반 페이지네이션을 사용합니다.

```dart
// Flutter 예시
Future<void> loadMoreMessages() async {
  final response = await api.getMessageHistory(
    roomId: roomId,
    userId: userId,
    beforeMessageId: lastMessageId, // 커서
    size: 20,
  );

  if (response.hasMore) {
    lastMessageId = response.nextCursor;
  }
}
```

### 오프셋 기반 (메시지 검색)

메시지 검색은 오프셋 기반 페이지네이션을 사용합니다.

```dart
// Flutter 예시
Future<void> searchMessages(int page) async {
  final response = await api.searchMessages(
    chatRoomId: roomId,
    userId: userId,
    keyword: searchKeyword,
    page: page,
    size: 20,
  );
}
```
