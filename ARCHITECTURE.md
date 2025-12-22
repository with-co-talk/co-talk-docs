# Co-Talk 아키텍처 문서
## 버전 1.0 - MVP 초기 버전

---

## 1. 기술 스택

### 1.1 백엔드

#### 런타임 및 프레임워크
- **Node.js** (v20 LTS)
  - 실시간 통신에 적합한 비동기 I/O
  - 풍부한 생태계와 라이브러리
  - JavaScript/TypeScript 통합 개발 가능

- **Express.js** 또는 **NestJS**
  - **Express.js** (권장 - MVP 단계)
    - 가볍고 빠른 설정
    - 유연한 구조
    - 실시간 통신과의 호환성 우수
  - **NestJS** (대안)
    - TypeScript 기반
    - 구조화된 아키텍처
    - 확장성 고려 시 적합

#### 실시간 통신
- **Socket.io**
  - WebSocket 기반 실시간 양방향 통신
  - 자동 폴백 지원 (Long Polling)
  - 방(Room) 개념으로 채팅방 관리 용이
  - 연결 관리 및 재연결 로직 내장

#### 데이터베이스
- **PostgreSQL** (v15+)
  - 관계형 데이터 모델링 (친구 관계, 채팅방, 메시지)
  - ACID 트랜잭션 보장
  - JSONB 타입으로 유연한 스키마 확장 가능
  - 풍부한 인덱싱 옵션
  - 확장성 고려 시 적합

#### ORM/쿼리 빌더
- **Prisma** (권장)
  - TypeScript 기반 타입 안정성
  - 마이그레이션 관리 용이
  - 직관적인 쿼리 API
  - 또는 **TypeORM** (대안)

#### 인증 및 보안
- **JWT (jsonwebtoken)**
  - Stateless 인증
  - 토큰 기반 세션 관리
  - **bcrypt** - 비밀번호 해싱
  - **helmet** - 보안 헤더 설정
  - **express-validator** - 입력 검증

#### 기타 라이브러리
- **dotenv** - 환경 변수 관리
- **cors** - CORS 설정
- **winston** 또는 **pino** - 로깅
- **joi** 또는 **zod** - 스키마 검증

### 1.2 프론트엔드

#### 프레임워크
- **React** (v18+)
  - 컴포넌트 기반 UI 개발
  - 풍부한 생태계
  - 또는 **Next.js** (SSR/SSG 필요 시)

- **TypeScript**
  - 타입 안정성
  - 개발 생산성 향상
  - 백엔드와 타입 공유 가능

#### 상태 관리
- **Zustand** (권장 - MVP 단계)
  - 가볍고 간단한 상태 관리
  - 또는 **Redux Toolkit** (복잡한 상태 관리 필요 시)
  - 또는 **React Query** (서버 상태 관리)

#### 실시간 통신
- **socket.io-client**
  - 백엔드 Socket.io와 통신
  - 자동 재연결 지원

#### UI 라이브러리
- **Tailwind CSS** (권장)
  - 유틸리티 기반 스타일링
  - 빠른 개발
  - 또는 **Material-UI**, **Chakra UI** (컴포넌트 라이브러리)

#### HTTP 클라이언트
- **axios** 또는 **fetch API**
  - REST API 통신

#### 폼 관리
- **react-hook-form**
  - 폼 검증 및 관리

#### 라우팅
- **React Router** (v6+)

### 1.3 인프라 및 배포

#### 컨테이너화
- **Docker**
  - 개발 환경 통일
  - 배포 간소화

#### 호스팅 (초기)
- **백엔드**: 
  - **Railway**, **Render**, **Fly.io** (권장 - 간단한 배포)
  - 또는 **AWS EC2**, **DigitalOcean**
- **프론트엔드**:
  - **Vercel**, **Netlify** (권장 - 무료 티어)
- **데이터베이스**:
  - **Supabase** (PostgreSQL 호스팅, 무료 티어)
  - 또는 **Railway**, **Render** (PostgreSQL 포함)

#### CI/CD (선택)
- **GitHub Actions**
  - 자동 빌드 및 배포

### 1.4 개발 도구

#### 코드 품질
- **ESLint** - 코드 린팅
- **Prettier** - 코드 포맷팅
- **Husky** - Git hooks
- **lint-staged** - 커밋 전 검사

#### 테스팅 (향후)
- **Jest** - 단위 테스트
- **Supertest** - API 테스트
- **React Testing Library** - 컴포넌트 테스트

---

## 2. 시스템 아키텍처

### 2.1 전체 아키텍처 다이어그램

```
┌─────────────────┐
│   Web Browser   │
│   (React App)   │
└────────┬────────┘
         │
         │ HTTPS/WSS
         │
┌────────▼─────────────────────────────────────┐
│         API Gateway / Load Balancer         │
│              (Nginx / Caddy)                 │
└────────┬─────────────────────────────────────┘
         │
    ┌────┴────┐
    │         │
┌───▼───┐ ┌──▼────────┐
│ REST  │ │ WebSocket │
│ API   │ │ (Socket.io)│
│Server │ │  Server   │
└───┬───┘ └──┬────────┘
    │        │
    └───┬────┘
        │
┌───────▼────────┐
│   PostgreSQL   │
│   Database     │
└────────────────┘
```

### 2.2 컴포넌트 구조

#### 백엔드 구조 (Express.js 기준)

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
│   ├── services/         # 비즈니스 로직
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
│   ├── middleware/     # 미들웨어
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
├── .env                 # 환경 변수
├── .env.example
├── package.json
├── tsconfig.json
└── Dockerfile
```

#### 프론트엔드 구조 (React 기준)

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
├── .env
├── .env.example
├── package.json
├── tsconfig.json
├── tailwind.config.js
└── Dockerfile
```

---

## 3. 데이터베이스 설계

### 3.1 주요 엔티티

#### Users (사용자)
- id (UUID, PK)
- email (String, Unique)
- password_hash (String)
- nickname (String, Unique)
- avatar_url (String, Optional)
- created_at (Timestamp)
- updated_at (Timestamp)

#### Friends (친구 관계)
- id (UUID, PK)
- user_id (UUID, FK → Users)
- friend_id (UUID, FK → Users)
- status (Enum: 'pending', 'accepted', 'blocked')
- created_at (Timestamp)
- updated_at (Timestamp)
- Unique constraint: (user_id, friend_id)

#### FriendRequests (친구 요청)
- id (UUID, PK)
- requester_id (UUID, FK → Users)
- receiver_id (UUID, FK → Users)
- status (Enum: 'pending', 'accepted', 'rejected')
- created_at (Timestamp)
- updated_at (Timestamp)

#### ChatRooms (채팅방)
- id (UUID, PK)
- type (Enum: 'direct', 'group') - 초기에는 'direct'만
- created_at (Timestamp)
- updated_at (Timestamp)

#### ChatRoomMembers (채팅방 멤버)
- id (UUID, PK)
- chat_room_id (UUID, FK → ChatRooms)
- user_id (UUID, FK → Users)
- joined_at (Timestamp)
- last_read_at (Timestamp) - 읽음 표시용

#### Messages (메시지)
- id (UUID, PK)
- chat_room_id (UUID, FK → ChatRooms)
- sender_id (UUID, FK → Users)
- content (Text)
- message_type (Enum: 'text', 'image', 'file') - 초기에는 'text'만
- created_at (Timestamp)
- updated_at (Timestamp)
- Index: (chat_room_id, created_at)

### 3.2 관계도

```
Users ──┬── Friends (user_id)
        │
        ├── Friends (friend_id)
        │
        ├── FriendRequests (requester_id)
        │
        ├── FriendRequests (receiver_id)
        │
        ├── ChatRoomMembers (user_id)
        │
        └── Messages (sender_id)

ChatRooms ──┬── ChatRoomMembers (chat_room_id)
            │
            └── Messages (chat_room_id)
```

---

## 4. API 설계 개요

### 4.1 REST API 엔드포인트 구조

#### 인증 (Auth)
- `POST /api/auth/signup` - 회원가입
- `POST /api/auth/login` - 로그인
- `POST /api/auth/logout` - 로그아웃
- `GET /api/auth/me` - 현재 사용자 정보

#### 사용자 (User)
- `GET /api/users/:id` - 사용자 정보 조회
- `PUT /api/users/:id` - 사용자 정보 수정
- `GET /api/users/search?q=keyword` - 사용자 검색

#### 친구 (Friend)
- `GET /api/friends` - 친구 목록 조회
- `POST /api/friends/requests` - 친구 요청 보내기
- `GET /api/friends/requests` - 친구 요청 목록 조회
- `PUT /api/friends/requests/:id/accept` - 친구 요청 수락
- `PUT /api/friends/requests/:id/reject` - 친구 요청 거절
- `DELETE /api/friends/:id` - 친구 삭제

#### 채팅 (Chat)
- `GET /api/chats` - 채팅방 목록 조회
- `GET /api/chats/:id` - 채팅방 정보 조회
- `GET /api/chats/:id/messages` - 메시지 히스토리 조회

#### 메시지 (Message)
- `POST /api/messages` - 메시지 전송 (REST API, 폴백용)
- `GET /api/messages/:id` - 메시지 조회

### 4.2 WebSocket 이벤트

#### 클라이언트 → 서버
- `connect` - 연결
- `disconnect` - 연결 해제
- `join:room` - 채팅방 입장
- `leave:room` - 채팅방 나가기
- `send:message` - 메시지 전송
- `typing:start` - 타이핑 시작 (향후)
- `typing:stop` - 타이핑 중지 (향후)

#### 서버 → 클라이언트
- `connect` - 연결 성공
- `disconnect` - 연결 해제
- `message:new` - 새 메시지 수신
- `message:read` - 메시지 읽음 처리
- `friend:online` - 친구 온라인 (향후)
- `friend:offline` - 친구 오프라인 (향후)

---

## 5. 보안 고려사항

### 5.1 인증 및 인가
- JWT 토큰 기반 인증
- 토큰 만료 시간 설정 (예: 7일)
- Refresh Token 고려 (향후)
- Socket.io 연결 시 JWT 토큰 검증

### 5.2 데이터 보안
- 비밀번호 bcrypt 해싱 (salt rounds: 10)
- HTTPS/WSS 통신 필수
- SQL Injection 방지 (ORM 사용)
- XSS 방지 (입력 검증 및 이스케이프)
- CSRF 방지 (SameSite 쿠키)

### 5.3 입력 검증
- 모든 사용자 입력 검증
- 이메일 형식 검증
- 비밀번호 강도 검증
- 메시지 길이 제한 (1000자)

---

## 6. 성능 최적화

### 6.1 데이터베이스
- 인덱스 최적화
  - Users: email, nickname
  - Messages: (chat_room_id, created_at)
  - Friends: (user_id, friend_id)
- 페이지네이션 (메시지 히스토리)
- 쿼리 최적화 (N+1 문제 방지)

### 6.2 실시간 통신
- Socket.io Room 활용 (채팅방별)
- 메시지 큐잉 (고부하 시)
- 연결 풀 관리

### 6.3 프론트엔드
- 메시지 가상화 (Virtual Scrolling)
- 이미지 지연 로딩 (향후)
- 코드 스플리팅
- 메모이제이션

---

## 7. 확장성 고려사항

### 7.1 수평 확장
- Socket.io Redis 어댑터 (다중 서버 지원)
- 로드 밸런서 (Sticky Session)
- 데이터베이스 읽기 복제본

### 7.2 향후 기능 확장
- 그룹 채팅 지원 가능한 구조
- 파일 전송 (멀티파트 업로드)
- 메시지 검색 (Elasticsearch 고려)
- 알림 시스템 (Redis Pub/Sub)

---

## 8. 모니터링 및 로깅

### 8.1 로깅
- 구조화된 로깅 (JSON 형식)
- 로그 레벨 관리 (DEBUG, INFO, WARN, ERROR)
- 에러 추적 (Sentry 고려)

### 8.2 모니터링
- API 응답 시간 모니터링
- WebSocket 연결 수 모니터링
- 데이터베이스 쿼리 성능 모니터링
- 서버 리소스 모니터링

---

## 9. 배포 전략

### 9.1 초기 배포
- 단일 서버 구성
- 데이터베이스 백업 (일일 1회)
- 환경 변수 관리 (.env)

### 9.2 향후 배포
- Blue-Green 배포
- 무중단 배포
- 자동 스케일링

---

## 10. 개발 환경 설정

### 10.1 필수 도구
- Node.js (v20 LTS)
- PostgreSQL (v15+)
- Git
- Docker (선택)

### 10.2 개발 워크플로우
1. 기능 브랜치 생성
2. 로컬 개발 및 테스트
3. PR 생성 및 코드 리뷰
4. 메인 브랜치 머지
5. 자동 배포 (CI/CD)

---

**문서 작성일**: 2024년
**최종 수정일**: 2024년
**작성자**: [작성자명]

