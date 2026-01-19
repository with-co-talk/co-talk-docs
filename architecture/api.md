---
layout: default
title: API Design
description: Co-Talk API 설계
---

# API 설계

[← 아키텍처 개요](./index)

---

## 목차

- [REST API](#rest-api)
- [WebSocket 이벤트](#websocket-이벤트)

---

## REST API

### 인증 (Auth)

| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | `/api/auth/signup` | 회원가입 |
| POST | `/api/auth/login` | 로그인 |
| POST | `/api/auth/logout` | 로그아웃 |
| GET | `/api/auth/me` | 현재 사용자 정보 |

### 사용자 (User)

| Method | Endpoint | 설명 |
|--------|----------|------|
| GET | `/api/users/:id` | 사용자 정보 조회 |
| PUT | `/api/users/:id` | 사용자 정보 수정 |
| GET | `/api/users/search?q=keyword` | 사용자 검색 |

### 친구 (Friend)

| Method | Endpoint | 설명 |
|--------|----------|------|
| GET | `/api/friends` | 친구 목록 조회 |
| POST | `/api/friends/requests` | 친구 요청 보내기 |
| GET | `/api/friends/requests` | 친구 요청 목록 조회 |
| PUT | `/api/friends/requests/:id/accept` | 친구 요청 수락 |
| PUT | `/api/friends/requests/:id/reject` | 친구 요청 거절 |
| DELETE | `/api/friends/:id` | 친구 삭제 |

### 채팅 (Chat)

| Method | Endpoint | 설명 |
|--------|----------|------|
| GET | `/api/chats` | 채팅방 목록 조회 |
| GET | `/api/chats/:id` | 채팅방 정보 조회 |
| GET | `/api/chats/:id/messages` | 메시지 히스토리 조회 |

### 메시지 (Message)

| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | `/api/messages` | 메시지 전송 (REST, 폴백용) |
| GET | `/api/messages/:id` | 메시지 조회 |

---

## WebSocket 이벤트

### 클라이언트 → 서버

| 이벤트 | 설명 |
|--------|------|
| `connect` | 연결 |
| `disconnect` | 연결 해제 |
| `join:room` | 채팅방 입장 |
| `leave:room` | 채팅방 나가기 |
| `send:message` | 메시지 전송 |
| `typing:start` | 타이핑 시작 (향후) |
| `typing:stop` | 타이핑 중지 (향후) |

### 서버 → 클라이언트

| 이벤트 | 설명 |
|--------|------|
| `connect` | 연결 성공 |
| `disconnect` | 연결 해제 |
| `message:new` | 새 메시지 수신 |
| `message:read` | 메시지 읽음 처리 |
| `friend:online` | 친구 온라인 (향후) |
| `friend:offline` | 친구 오프라인 (향후) |

---

## 다음 문서

→ [인프라](./infrastructure)
