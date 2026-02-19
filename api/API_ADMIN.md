---
layout: default
title: Admin API
parent: API Reference
nav_order: 7
---

# 관리자/신고 API

신고 기능 및 관리자 전용 API 문서입니다.

> **Note**: 관리자 API (`/api/v1/admin/*`)는 `ADMIN` 역할을 가진 사용자만 접근 가능합니다.

---

## 목차

### 신고 (일반 사용자)
1. [사용자 신고](#1-사용자-신고)
2. [메시지 신고](#2-메시지-신고)
3. [내 신고 목록 조회](#3-내-신고-목록-조회)

### 관리자 - 신고 관리
4. [대기 중인 신고 목록 조회](#4-대기-중인-신고-목록-조회)
5. [신고 목록 조회 (상태별)](#5-신고-목록-조회-상태별)
6. [신고 처리](#6-신고-처리)

### 관리자 - 사용자 관리
7. [전체 사용자 목록 조회](#7-전체-사용자-목록-조회)
8. [사용자 정지](#8-사용자-정지)
9. [사용자 활성화](#9-사용자-활성화)

### 관리자 - 통계
10. [관리자 통계 조회](#10-관리자-통계-조회)

---

## 신고 (일반 사용자)

**Base URL**: `/api/v1/reports`

### 1. 사용자 신고

특정 사용자를 신고합니다.

```http
POST /api/v1/reports/users?reporterId={reporterId}
Authorization: Bearer {accessToken}
Content-Type: application/json
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| reporterId | Long | O | 신고자 ID (Query) |

**Request Body**:
```json
{
  "reportedUserId": 2,
  "reason": "HARASSMENT",
  "description": "반복적인 욕설과 비방을 합니다."
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| reportedUserId | Long | O | 신고 대상 사용자 ID |
| reason | String | O | 신고 사유 |
| description | String | - | 상세 설명 |

**신고 사유 (reason)**:

| 사유 | 설명 |
|------|------|
| HARASSMENT | 괴롭힘/욕설 |
| SPAM | 스팸/광고 |
| INAPPROPRIATE_CONTENT | 부적절한 콘텐츠 |
| IMPERSONATION | 사칭 |
| FRAUD | 사기 |
| OTHER | 기타 |

**Response (201 Created)**:
```json
{
  "id": 100,
  "reporterId": 1,
  "targetType": "USER",
  "targetId": 2,
  "reason": "HARASSMENT",
  "description": "반복적인 욕설과 비방을 합니다.",
  "status": "PENDING",
  "createdAt": "2025-01-22T10:30:00"
}
```

---

### 2. 메시지 신고

특정 메시지를 신고합니다.

```http
POST /api/v1/reports/messages?reporterId={reporterId}
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body**:
```json
{
  "reportedMessageId": 1000,
  "reason": "INAPPROPRIATE_CONTENT",
  "description": "불법 정보가 포함되어 있습니다."
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| reportedMessageId | Long | O | 신고 대상 메시지 ID |
| reason | String | O | 신고 사유 |
| description | String | - | 상세 설명 |

**Response (201 Created)**:
```json
{
  "id": 101,
  "reporterId": 1,
  "targetType": "MESSAGE",
  "targetId": 1000,
  "reason": "INAPPROPRIATE_CONTENT",
  "description": "불법 정보가 포함되어 있습니다.",
  "status": "PENDING",
  "createdAt": "2025-01-22T10:31:00"
}
```

---

### 3. 내 신고 목록 조회

내가 신고한 내역을 조회합니다.

```http
GET /api/v1/reports/my?userId={userId}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| userId | Long | O | 사용자 ID |

**Response (200 OK)**:
```json
{
  "reports": [
    {
      "id": 100,
      "reporterId": 1,
      "targetType": "USER",
      "targetId": 2,
      "reason": "HARASSMENT",
      "description": "반복적인 욕설과 비방을 합니다.",
      "status": "PENDING",
      "createdAt": "2025-01-22T10:30:00"
    },
    {
      "id": 101,
      "reporterId": 1,
      "targetType": "MESSAGE",
      "targetId": 1000,
      "reason": "INAPPROPRIATE_CONTENT",
      "description": "불법 정보가 포함되어 있습니다.",
      "status": "RESOLVED",
      "createdAt": "2025-01-22T10:31:00"
    }
  ]
}
```

---

## 관리자 - 신고 관리

**Base URL**: `/api/v1/admin`

> 모든 관리자 API는 `ADMIN` 역할이 필요합니다.

### 4. 대기 중인 신고 목록 조회

처리 대기 중인 신고 목록을 조회합니다.

```http
GET /api/v1/admin/reports/pending
Authorization: Bearer {accessToken}
```

**Response (200 OK)**:
```json
{
  "reports": [
    {
      "id": 100,
      "reporterId": 1,
      "reporterNickname": "신고자닉네임",
      "targetType": "USER",
      "targetId": 2,
      "targetNickname": "신고대상닉네임",
      "reason": "HARASSMENT",
      "description": "반복적인 욕설과 비방을 합니다.",
      "status": "PENDING",
      "adminId": null,
      "adminNote": null,
      "createdAt": "2025-01-22T10:30:00",
      "processedAt": null
    }
  ]
}
```

---

### 5. 신고 목록 조회 (상태별)

상태별 신고 목록을 조회합니다.

```http
GET /api/v1/admin/reports?status={status}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| status | String | - | 신고 상태 (없으면 전체 조회) |

**신고 상태 (status)**:

| 상태 | 설명 |
|------|------|
| PENDING | 대기 중 |
| INVESTIGATING | 조사 중 |
| RESOLVED | 처리 완료 |
| DISMISSED | 기각 |

**Response (200 OK)**:
```json
{
  "reports": [
    {
      "id": 100,
      "reporterId": 1,
      "reporterNickname": "신고자닉네임",
      "targetType": "USER",
      "targetId": 2,
      "targetNickname": "신고대상닉네임",
      "reason": "HARASSMENT",
      "description": "반복적인 욕설과 비방을 합니다.",
      "status": "RESOLVED",
      "adminId": 999,
      "adminNote": "경고 조치 완료",
      "createdAt": "2025-01-22T10:30:00",
      "processedAt": "2025-01-22T11:00:00"
    }
  ]
}
```

---

### 6. 신고 처리

신고를 처리합니다.

```http
POST /api/v1/admin/reports/{reportId}/process
Authorization: Bearer {accessToken}
Content-Type: application/json
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| reportId | Long | O | 신고 ID (Path) |

**Request Body**:
```json
{
  "adminId": 999,
  "status": "RESOLVED",
  "adminNote": "경고 조치 완료. 재발 시 계정 정지 예정."
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| adminId | Long | O | 처리 관리자 ID |
| status | String | O | 처리 상태 (`INVESTIGATING`, `RESOLVED`, `DISMISSED`) |
| adminNote | String | - | 관리자 메모 |

**Response (200 OK)**:
```json
{
  "id": 100,
  "reporterId": 1,
  "reporterNickname": "신고자닉네임",
  "targetType": "USER",
  "targetId": 2,
  "targetNickname": "신고대상닉네임",
  "reason": "HARASSMENT",
  "description": "반복적인 욕설과 비방을 합니다.",
  "status": "RESOLVED",
  "adminId": 999,
  "adminNote": "경고 조치 완료. 재발 시 계정 정지 예정.",
  "createdAt": "2025-01-22T10:30:00",
  "processedAt": "2025-01-22T11:00:00"
}
```

---

## 관리자 - 사용자 관리

### 7. 전체 사용자 목록 조회

모든 사용자 목록을 조회합니다.

```http
GET /api/v1/admin/users?status={status}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| status | String | - | 사용자 상태 (없으면 전체 조회) |

**사용자 상태 (status)**:

| 상태 | 설명 |
|------|------|
| ACTIVE | 활성 |
| SUSPENDED | 정지됨 |
| DELETED | 탈퇴됨 |

**Response (200 OK)**:
```json
{
  "users": [
    {
      "id": 1,
      "email": "user@example.com",
      "nickname": "사용자1",
      "status": "ACTIVE",
      "role": "USER",
      "createdAt": "2025-01-01T10:00:00",
      "lastActiveAt": "2025-01-22T10:30:00"
    },
    {
      "id": 2,
      "email": "suspended@example.com",
      "nickname": "정지된사용자",
      "status": "SUSPENDED",
      "role": "USER",
      "createdAt": "2025-01-05T10:00:00",
      "lastActiveAt": "2025-01-20T15:00:00"
    }
  ]
}
```

---

### 8. 사용자 정지

사용자를 정지 처리합니다.

```http
POST /api/v1/admin/users/{userId}/suspend
Authorization: Bearer {accessToken}
Content-Type: application/json
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| userId | Long | O | 정지할 사용자 ID (Path) |

**Request Body**:
```json
{
  "adminId": 999,
  "reason": "반복적인 신고 접수로 인한 정지 조치"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| adminId | Long | O | 관리자 ID |
| reason | String | O | 정지 사유 |

**Response (200 OK)**:
```json
{
  "id": 2,
  "email": "suspended@example.com",
  "nickname": "정지된사용자",
  "status": "SUSPENDED",
  "role": "USER",
  "createdAt": "2025-01-05T10:00:00",
  "lastActiveAt": "2025-01-20T15:00:00"
}
```

---

### 9. 사용자 활성화

정지된 사용자를 활성화합니다.

```http
POST /api/v1/admin/users/{userId}/activate?adminId={adminId}
Authorization: Bearer {accessToken}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| userId | Long | O | 활성화할 사용자 ID (Path) |
| adminId | Long | O | 관리자 ID (Query) |

**Response (200 OK)**:
```json
{
  "id": 2,
  "email": "user2@example.com",
  "nickname": "사용자2",
  "status": "ACTIVE",
  "role": "USER",
  "createdAt": "2025-01-05T10:00:00",
  "lastActiveAt": "2025-01-20T15:00:00"
}
```

---

## 관리자 - 통계

### 10. 관리자 통계 조회

전체 시스템 통계를 조회합니다.

```http
GET /api/v1/admin/statistics
Authorization: Bearer {accessToken}
```

**Response (200 OK)**:
```json
{
  "totalUsers": 10000,
  "activeUsers": 9500,
  "suspendedUsers": 50,
  "deletedUsers": 450,
  "totalChatRooms": 5000,
  "totalMessages": 1000000,
  "pendingReports": 25,
  "todayNewUsers": 100,
  "todayActiveUsers": 3000
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| totalUsers | Long | 전체 사용자 수 |
| activeUsers | Long | 활성 사용자 수 |
| suspendedUsers | Long | 정지된 사용자 수 |
| deletedUsers | Long | 탈퇴 사용자 수 |
| totalChatRooms | Long | 전체 채팅방 수 |
| totalMessages | Long | 전체 메시지 수 |
| pendingReports | Long | 대기 중인 신고 수 |
| todayNewUsers | Long | 오늘 신규 가입자 수 |
| todayActiveUsers | Long | 오늘 활성 사용자 수 |

---

## 에러 응답

### 403 Forbidden - 권한 없음

관리자가 아닌 사용자가 관리자 API에 접근하는 경우:

```json
{
  "error": "Access Denied",
  "code": "FORBIDDEN",
  "timestamp": "2025-01-22T10:30:00"
}
```

### 400 Bad Request - 잘못된 신고

중복 신고 등 잘못된 요청:

```json
{
  "error": "이미 신고한 대상입니다.",
  "code": "INVALID_REPORT",
  "timestamp": "2025-01-22T10:30:00"
}
```

```json
{
  "error": "자기 자신을 신고할 수 없습니다.",
  "code": "INVALID_REPORT",
  "timestamp": "2025-01-22T10:30:00"
}
```
