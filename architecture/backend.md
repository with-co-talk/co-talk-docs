---
layout: default
title: Backend Architecture
description: Co-Talk 백엔드 아키텍처
---

# 백엔드 아키텍처

[← 아키텍처 개요](./index)

---

## 목차

- [런타임 및 프레임워크](#런타임-및-프레임워크)
- [실시간 통신](#실시간-통신)
- [데이터베이스](#데이터베이스)
- [인증 및 보안](#인증-및-보안)
- [프로젝트 구조](#프로젝트-구조)

---

## 런타임 및 프레임워크

### Node.js (v20 LTS)
- 실시간 통신에 적합한 비동기 I/O
- 풍부한 생태계와 라이브러리
- JavaScript/TypeScript 통합 개발 가능

### Express.js 또는 NestJS

**Express.js** (권장 - MVP 단계)
- 가볍고 빠른 설정
- 유연한 구조
- 실시간 통신과의 호환성 우수

**NestJS** (대안)
- TypeScript 기반
- 구조화된 아키텍처
- 확장성 고려 시 적합

---

## 실시간 통신

### Socket.io
- WebSocket 기반 실시간 양방향 통신
- 자동 폴백 지원 (Long Polling)
- 방(Room) 개념으로 채팅방 관리 용이
- 연결 관리 및 재연결 로직 내장

---

## 데이터베이스

### PostgreSQL (v15+)
- 관계형 데이터 모델링 (친구 관계, 채팅방, 메시지)
- ACID 트랜잭션 보장
- JSONB 타입으로 유연한 스키마 확장 가능
- 풍부한 인덱싱 옵션

### ORM/쿼리 빌더

**Prisma** (권장)
- TypeScript 기반 타입 안정성
- 마이그레이션 관리 용이
- 직관적인 쿼리 API

**TypeORM** (대안)

---

## 인증 및 보안

### JWT (jsonwebtoken)
- Stateless 인증
- 토큰 기반 세션 관리

### 보안 라이브러리
- **bcrypt** - 비밀번호 해싱
- **helmet** - 보안 헤더 설정
- **express-validator** - 입력 검증

### 기타 라이브러리
- **dotenv** - 환경 변수 관리
- **cors** - CORS 설정
- **winston** 또는 **pino** - 로깅
- **joi** 또는 **zod** - 스키마 검증

---

## 프로젝트 구조

```
backend/
├── src/
│   ├── config/          # 설정 파일
│   │   ├── database.ts
│   │   └── env.ts
│   ├── controllers/     # 컨트롤러
│   │   ├── auth.controller.ts
│   │   ├── user.controller.ts
│   │   ├── friend.controller.ts
│   │   ├── chat.controller.ts
│   │   └── message.controller.ts
│   ├── services/        # 비즈니스 로직
│   │   ├── auth.service.ts
│   │   ├── user.service.ts
│   │   ├── friend.service.ts
│   │   ├── chat.service.ts
│   │   └── message.service.ts
│   ├── models/          # 데이터 모델
│   │   ├── User.ts
│   │   ├── Friend.ts
│   │   ├── ChatRoom.ts
│   │   └── Message.ts
│   ├── routes/          # 라우트 정의
│   │   ├── auth.routes.ts
│   │   ├── user.routes.ts
│   │   ├── friend.routes.ts
│   │   ├── chat.routes.ts
│   │   └── message.routes.ts
│   ├── middleware/      # 미들웨어
│   │   ├── auth.middleware.ts
│   │   ├── error.middleware.ts
│   │   └── validation.middleware.ts
│   ├── socket/          # Socket.io 핸들러
│   │   ├── socket.handler.ts
│   │   └── socket.middleware.ts
│   ├── utils/           # 유틸리티
│   │   ├── jwt.util.ts
│   │   ├── bcrypt.util.ts
│   │   └── logger.util.ts
│   └── app.ts           # Express 앱 설정
├── prisma/              # Prisma 스키마 및 마이그레이션
│   └── schema.prisma
├── tests/               # 테스트 파일
├── .env.example
├── package.json
├── tsconfig.json
└── Dockerfile
```

---

## 다음 문서

→ [프론트엔드 구조](./frontend)
