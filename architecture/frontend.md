---
layout: default
title: Frontend Architecture
description: Co-Talk 프론트엔드 아키텍처
---

# 프론트엔드 아키텍처

[← 아키텍처 개요](./index)

---

## 목차

- [프레임워크](#프레임워크)
- [상태 관리](#상태-관리)
- [실시간 통신](#실시간-통신)
- [UI 라이브러리](#ui-라이브러리)
- [프로젝트 구조](#프로젝트-구조)

---

## 프레임워크

### React (v18+)
- 컴포넌트 기반 UI 개발
- 풍부한 생태계
- Next.js (SSR/SSG 필요 시)

### TypeScript
- 타입 안정성
- 개발 생산성 향상
- 백엔드와 타입 공유 가능

---

## 상태 관리

### Zustand (권장 - MVP 단계)
- 가볍고 간단한 상태 관리

### 대안
- **Redux Toolkit** - 복잡한 상태 관리 필요 시
- **React Query** - 서버 상태 관리

---

## 실시간 통신

### socket.io-client
- 백엔드 Socket.io와 통신
- 자동 재연결 지원

---

## UI 라이브러리

### Tailwind CSS (권장)
- 유틸리티 기반 스타일링
- 빠른 개발

### 대안
- Material-UI, Chakra UI (컴포넌트 라이브러리)

### HTTP 클라이언트
- **axios** 또는 **fetch API**

### 폼 관리
- **react-hook-form**

### 라우팅
- **React Router** (v6+)

---

## 프로젝트 구조

```
frontend/
├── src/
│   ├── components/      # 재사용 가능한 컴포넌트
│   │   ├── common/      # 공통 컴포넌트
│   │   │   ├── Button/
│   │   │   ├── Input/
│   │   │   └── Avatar/
│   │   ├── chat/        # 채팅 관련 컴포넌트
│   │   │   ├── ChatList/
│   │   │   ├── ChatRoom/
│   │   │   └── Message/
│   │   └── friend/      # 친구 관련 컴포넌트
│   │       ├── FriendList/
│   │       └── FriendSearch/
│   ├── pages/           # 페이지 컴포넌트
│   │   ├── Login/
│   │   ├── Signup/
│   │   ├── ChatList/
│   │   ├── ChatRoom/
│   │   ├── FriendList/
│   │   └── Profile/
│   ├── hooks/           # 커스텀 훅
│   │   ├── useAuth.ts
│   │   ├── useSocket.ts
│   │   └── useChat.ts
│   ├── store/           # 상태 관리 (Zustand)
│   │   ├── authStore.ts
│   │   ├── chatStore.ts
│   │   └── friendStore.ts
│   ├── services/        # API 서비스
│   │   ├── api.ts       # axios 인스턴스
│   │   ├── auth.service.ts
│   │   ├── user.service.ts
│   │   ├── friend.service.ts
│   │   └── chat.service.ts
│   ├── utils/           # 유틸리티
│   │   ├── socket.ts    # Socket.io 클라이언트
│   │   ├── date.ts      # 날짜 포맷팅
│   │   └── validation.ts
│   ├── types/           # TypeScript 타입 정의
│   │   ├── user.types.ts
│   │   ├── chat.types.ts
│   │   └── message.types.ts
│   ├── App.tsx          # 메인 앱 컴포넌트
│   ├── index.tsx        # 진입점
│   └── router.tsx       # 라우터 설정
├── public/              # 정적 파일
├── .env.example
├── package.json
├── tsconfig.json
├── tailwind.config.js
└── Dockerfile
```

---

## 다음 문서

→ [데이터베이스 설계](./database)
