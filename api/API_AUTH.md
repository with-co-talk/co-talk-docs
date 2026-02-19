---
layout: default
title: Auth API
parent: API Reference
nav_order: 1
---

# 인증/계정 API

인증, 계정 관리, 약관 동의를 위한 REST API 문서입니다.

---

## 목차

### 인증 (Auth)
1. [회원가입](#1-회원가입)
2. [로그인](#2-로그인)
3. [토큰 갱신](#3-토큰-갱신)
4. [로그아웃](#4-로그아웃)
5. [소셜 로그인](#5-소셜-로그인)

### 비밀번호 (Password)
6. [비밀번호 재설정 요청](#6-비밀번호-재설정-요청)
7. [비밀번호 재설정 토큰 검증](#7-비밀번호-재설정-토큰-검증)
8. [비밀번호 재설정](#8-비밀번호-재설정)

### 계정 (Account)
9. [회원 탈퇴](#9-회원-탈퇴)

### 약관 (Terms)
10. [약관 동의](#10-약관-동의)
11. [마케팅 수신 동의 철회](#11-마케팅-수신-동의-철회)
12. [약관 동의 상태 조회](#12-약관-동의-상태-조회)
13. [필수 약관 동의 확인](#13-필수-약관-동의-확인)

---

## 인증 (Auth)

**Base URL**: `/api/v1/auth`

### 1. 회원가입

새로운 사용자를 등록합니다.

```http
POST /api/v1/auth/signup
Content-Type: application/json
```

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "password123!",
  "nickname": "닉네임"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| email | String | O | 이메일 (유효한 이메일 형식) |
| password | String | O | 비밀번호 (최소 8자) |
| nickname | String | O | 닉네임 |

**Response (201 Created)**:
```json
{
  "userId": 1,
  "message": "회원가입이 완료되었습니다."
}
```

**Error Response**:
- `400 Bad Request` - 잘못된 요청 (유효성 검사 실패)
- `409 Conflict` - 이메일 또는 닉네임 중복

---

### 2. 로그인

이메일과 비밀번호로 로그인하여 JWT 토큰을 발급받습니다.

```http
POST /api/v1/auth/login
Content-Type: application/json
```

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "password123!"
}
```

**Response (200 OK)**:
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 86400
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| accessToken | String | API 요청에 사용할 액세스 토큰 |
| refreshToken | String | 토큰 갱신에 사용할 리프레시 토큰 |
| expiresIn | Integer | 액세스 토큰 만료 시간 (초) |

**Error Response**:
- `401 Unauthorized` - 인증 실패 (이메일 또는 비밀번호 오류)

---

### 3. 토큰 갱신

Refresh Token으로 새로운 Access Token을 발급받습니다.

```http
POST /api/v1/auth/refresh
Content-Type: application/json
```

**Request Body**:
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response (200 OK)**:
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Error Response**:
- `401 Unauthorized` - 유효하지 않은 Refresh Token

---

### 4. 로그아웃

현재 사용자의 모든 Refresh Token을 폐기합니다.

```http
POST /api/v1/auth/logout
Authorization: Bearer {accessToken}
```

**Response (200 OK)**: 빈 응답

**Error Response**:
- `401 Unauthorized` - 인증 필요

---

### 5. 소셜 로그인

**Base URL**: `/api/v1/auth/oauth`

OAuth 제공자(카카오, 구글, 애플)를 통해 로그인합니다. 신규 사용자는 자동으로 회원가입됩니다.

```http
POST /api/v1/auth/oauth/login
Content-Type: application/json
```

**Request Body**:
```json
{
  "provider": "KAKAO",
  "oauthId": "123456789",
  "email": "user@example.com",
  "nickname": "닉네임",
  "avatarUrl": "https://example.com/avatar.png"
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| provider | String | O | OAuth 제공자 (`KAKAO`, `GOOGLE`, `APPLE`) |
| oauthId | String | O | OAuth 제공자에서 발급한 사용자 ID |
| email | String | - | 이메일 (제공자에서 제공 시) |
| nickname | String | - | 닉네임 |
| avatarUrl | String | - | 프로필 이미지 URL |

**Response (200 OK)**:
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "isNewUser": true,
  "userId": 1
}
```

| 필드 | 타입 | 설명 |
|------|------|------|
| accessToken | String | JWT 액세스 토큰 |
| isNewUser | Boolean | 신규 사용자 여부 |
| userId | Long | 사용자 ID |

---

## 비밀번호 (Password)

**Base URL**: `/api/v1/password`

### 6. 비밀번호 재설정 요청

이메일로 비밀번호 재설정 링크를 발송합니다.

```http
POST /api/v1/password/reset-request
Content-Type: application/json
```

**Request Body**:
```json
{
  "email": "user@example.com"
}
```

**Response (200 OK)**:
```json
{
  "message": "비밀번호 재설정 링크가 이메일로 발송되었습니다. 이메일을 확인해주세요."
}
```

> **Note**: 보안상 이메일 존재 여부와 관계없이 동일한 응답을 반환합니다.

---

### 7. 비밀번호 재설정 토큰 검증

비밀번호 재설정 토큰의 유효성을 검증합니다.

```http
GET /api/v1/password/reset-validate?token={token}
```

| 파라미터 | 타입 | 필수 | 설명 |
|----------|------|------|------|
| token | String | O | 검증할 토큰 |

**Response (200 OK)**:
```json
{
  "valid": true
}
```

---

### 8. 비밀번호 재설정

토큰을 사용하여 새 비밀번호로 변경합니다.

```http
POST /api/v1/password/reset
Content-Type: application/json
```

**Request Body**:
```json
{
  "token": "reset-token-here",
  "newPassword": "newPassword123!"
}
```

**Response (200 OK)**:
```json
{
  "message": "비밀번호가 성공적으로 변경되었습니다."
}
```

**Error Response**:
- `400 Bad Request` - 유효하지 않거나 만료된 토큰

---

## 계정 (Account)

**Base URL**: `/api/v1/account`

### 9. 회원 탈퇴

비밀번호 확인 후 계정을 삭제합니다.

```http
DELETE /api/v1/account/{userId}
Content-Type: application/json
Authorization: Bearer {accessToken}
```

**Request Body**:
```json
{
  "password": "currentPassword123!"
}
```

**Response (200 OK)**:
```json
{
  "message": "회원 탈퇴가 완료되었습니다."
}
```

**Error Response**:
- `401 Unauthorized` - 비밀번호 불일치

---

## 약관 (Terms)

**Base URL**: `/api/v1/terms`

### 10. 약관 동의

이용약관 및 개인정보처리방침에 동의합니다.

```http
POST /api/v1/terms/agree
Content-Type: application/json
```

**Request Body**:
```json
{
  "userId": 1,
  "agreements": [
    {
      "termsType": "SERVICE",
      "version": "1.0",
      "agreed": true
    },
    {
      "termsType": "PRIVACY",
      "version": "1.0",
      "agreed": true
    },
    {
      "termsType": "MARKETING",
      "version": "1.0",
      "agreed": false
    }
  ]
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| userId | Long | O | 사용자 ID |
| agreements | Array | O | 약관 동의 목록 |
| agreements[].termsType | String | O | 약관 유형 (`SERVICE`, `PRIVACY`, `MARKETING`) |
| agreements[].version | String | O | 약관 버전 |
| agreements[].agreed | Boolean | O | 동의 여부 |

**Response (200 OK)**:
```json
{
  "message": "약관 동의가 완료되었습니다."
}
```

---

### 11. 마케팅 수신 동의 철회

마케팅 정보 수신 동의를 철회합니다.

```http
DELETE /api/v1/terms/marketing/{userId}
Authorization: Bearer {accessToken}
```

**Response (200 OK)**:
```json
{
  "message": "마케팅 수신 동의가 철회되었습니다."
}
```

---

### 12. 약관 동의 상태 조회

사용자의 약관 동의 상태를 조회합니다.

```http
GET /api/v1/terms/status/{userId}
Authorization: Bearer {accessToken}
```

**Response (200 OK)**:
```json
{
  "items": [
    {
      "termsType": "SERVICE",
      "termsVersion": "1.0",
      "agreed": true,
      "required": true
    },
    {
      "termsType": "PRIVACY",
      "termsVersion": "1.0",
      "agreed": true,
      "required": true
    },
    {
      "termsType": "MARKETING",
      "termsVersion": "1.0",
      "agreed": false,
      "required": false
    }
  ]
}
```

---

### 13. 필수 약관 동의 확인

필수 약관에 동의했는지 확인합니다.

```http
GET /api/v1/terms/check/{userId}
Authorization: Bearer {accessToken}
```

**Response (200 OK)**:
```json
{
  "agreed": true
}
```

---

## 공통 사항

### 인증 헤더

로그인 후 발급받은 Access Token을 Authorization 헤더에 포함합니다.

```
Authorization: Bearer {accessToken}
```

### 약관 유형

| 유형 | 설명 | 필수 여부 |
|------|------|----------|
| SERVICE | 서비스 이용약관 | 필수 |
| PRIVACY | 개인정보처리방침 | 필수 |
| MARKETING | 마케팅 정보 수신 | 선택 |

### OAuth 제공자

| 제공자 | 설명 |
|--------|------|
| KAKAO | 카카오 로그인 |
| GOOGLE | 구글 로그인 |
| APPLE | 애플 로그인 |
