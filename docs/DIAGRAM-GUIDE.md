---
layout: default
title: 다이어그램 & 수식 가이드
description: Mermaid 다이어그램과 수학 공식 사용법
---

# 다이어그램 & 수식 가이드

이 문서에서는 Mermaid를 사용한 다이어그램과 KaTeX를 사용한 수학 공식 작성법을 설명합니다.

---

## 🎨 Mermaid 다이어그램

### 플로우차트

```mermaid
graph TD
    A[사용자 로그인] --> B{인증 성공?}
    B -->|예| C[홈 화면]
    B -->|아니오| D[로그인 실패]
    D --> A
    C --> E[채팅방 목록]
    E --> F[메시지 전송]
```

### 시퀀스 다이어그램

```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant DB
    participant Redis

    Client->>Server: 메시지 전송
    Server->>DB: 메시지 저장
    DB-->>Server: 저장 완료
    Server->>Redis: 메시지 큐에 추가
    Redis-->>Server: 큐 추가 완료
    Server-->>Client: 전송 성공
    Redis->>Client: 실시간 메시지 전달
```

### 클래스 다이어그램

```mermaid
classDiagram
    class User {
        +Long id
        +String email
        +String nickname
        +login()
        +logout()
    }
    
    class Message {
        +Long id
        +Long senderId
        +Long receiverId
        +String content
        +LocalDateTime createdAt
        +send()
    }
    
    class ChatRoom {
        +Long id
        +Long user1Id
        +Long user2Id
        +List~Message~ messages
        +addMessage()
    }
    
    User "1" --> "*" Message : sends
    ChatRoom "1" --> "*" Message : contains
    User "2" --> "1" ChatRoom : participates
```

### ER 다이어그램

```mermaid
erDiagram
    USER ||--o{ FRIEND_REQUEST : sends
    USER ||--o{ FRIEND_REQUEST : receives
    USER ||--o{ MESSAGE : sends
    USER ||--o{ CHAT_ROOM_USER : participates
    CHAT_ROOM ||--o{ CHAT_ROOM_USER : has
    CHAT_ROOM ||--o{ MESSAGE : contains
    
    USER {
        bigint id PK
        varchar email UK
        varchar password
        varchar nickname
        timestamp created_at
    }
    
    FRIEND_REQUEST {
        bigint id PK
        bigint sender_id FK
        bigint receiver_id FK
        varchar status
        timestamp created_at
    }
    
    MESSAGE {
        bigint id PK
        bigint chat_room_id FK
        bigint sender_id FK
        text content
        boolean is_read
        timestamp created_at
    }
    
    CHAT_ROOM {
        bigint id PK
        timestamp created_at
    }
    
    CHAT_ROOM_USER {
        bigint chat_room_id FK
        bigint user_id FK
    }
```

### 상태 다이어그램

```mermaid
stateDiagram-v2
    [*] --> Offline
    Offline --> Connecting: 로그인
    Connecting --> Online: 연결 성공
    Connecting --> Offline: 연결 실패
    Online --> Chatting: 채팅방 입장
    Chatting --> Online: 채팅방 나가기
    Online --> Offline: 로그아웃
    Chatting --> Offline: 연결 끊김
```

### 간트 차트

```mermaid
gantt
    title Co-Talk 개발 로드맵
    dateFormat  YYYY-MM-DD
    section Phase 1 (MVP)
        요구사항 분석           :done,    des1, 2024-01-01, 7d
        기술 스택 선정          :done,    des2, 2024-01-08, 3d
        DB 설계                :done,    des3, 2024-01-11, 5d
        백엔드 개발            :active,  dev1, 2024-01-16, 21d
        프론트엔드 개발        :active,  dev2, 2024-01-16, 21d
        통합 테스트            :         test1, 2024-02-06, 7d
        배포                   :         deploy1, 2024-02-13, 2d
    section Phase 2 (확장)
        프로필 사진 기능       :         feat1, 2024-02-15, 5d
        이미지 전송            :         feat2, 2024-02-20, 7d
        그룹 채팅              :         feat3, 2024-02-27, 14d
```

### Git 그래프

```mermaid
gitGraph
    commit id: "Initial commit"
    commit id: "Add PRD"
    branch develop
    checkout develop
    commit id: "Setup backend"
    commit id: "Setup database"
    branch feature/auth
    checkout feature/auth
    commit id: "Add login API"
    commit id: "Add signup API"
    checkout develop
    merge feature/auth
    branch feature/chat
    checkout feature/chat
    commit id: "Add chat API"
    commit id: "Add WebSocket"
    checkout develop
    merge feature/chat
    checkout main
    merge develop tag: "v1.0.0"
```

### 파이 차트

```mermaid
pie title 기술 스택 비율
    "Backend (Spring Boot)" : 35
    "Frontend (React)" : 30
    "Database (PostgreSQL)" : 15
    "Cache (Redis)" : 10
    "Infrastructure (AWS)" : 10
```

---

## 📐 수학 공식 (KaTeX)

### 인라인 수식

동시 접속자 수 처리를 위한 서버 용량 계산: $C = \frac{N \times M}{T}$

여기서:
- $C$: 초당 처리 용량
- $N$: 동시 접속자 수
- $M$: 평균 메시지 크기
- $T$: 응답 시간

### 블록 수식

**처리량(Throughput) 계산:**

$$
Throughput = \frac{Messages}{Second} = \frac{DAU \times AvgMessages}{86400}
$$

**캐시 적중률(Cache Hit Ratio):**

$$
HitRatio = \frac{CacheHits}{CacheHits + CacheMisses} \times 100\%
$$

**평균 응답 시간:**

$$
ResponseTime_{avg} = \sum_{i=1}^{n} \frac{ResponseTime_i \times Weight_i}{\sum_{j=1}^{n} Weight_j}
$$

**데이터베이스 용량 추정:**

$$
\begin{aligned}
Storage &= Users \times AvgMessages \times MessageSize \\
&= 10^6 \times 100 \times 1KB \\
&= 100GB
\end{aligned}
$$

**서버 확장 계산:**

$$
Servers_{required} = \lceil \frac{PeakLoad}{ServerCapacity} \rceil \times SafetyFactor
$$

---

## 🎯 사용 방법

### Mermaid 다이어그램 작성

마크다운 파일에서 다음과 같이 작성:

\`\`\`mermaid
graph LR
    A --> B
\`\`\`

### 수학 공식 작성

- **인라인**: `$E = mc^2$` → $E = mc^2$
- **블록**: 

```
$$
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
$$
```

---

## 💡 팁

### Mermaid
- [Mermaid 공식 문서](https://mermaid.js.org/)
- [Mermaid Live Editor](https://mermaid.live/) - 실시간 미리보기
- 다크모드 자동 지원

### KaTeX
- [KaTeX 지원 함수 목록](https://katex.org/docs/supported.html)
- LaTeX 문법 사용
- 빠른 렌더링

---

## 🚀 실전 예제

### 아키텍처 다이어그램

<div class="diagram-container" data-title="Co-Talk 시스템 아키텍처">

```mermaid
graph TB
    subgraph Client["클라이언트"]
        Web[React Web App]
        Mobile[Flutter Mobile App]
    end
    
    subgraph Gateway["API Gateway"]
        LB[Load Balancer]
        API[Spring Boot API]
    end
    
    subgraph Storage["데이터 저장소"]
        DB[(PostgreSQL)]
        Cache[(Redis Cache)]
        Queue[(Redis Queue)]
    end
    
    subgraph Messaging["실시간 메시징"]
        WS[WebSocket Server]
    end
    
    Web --> LB
    Mobile --> LB
    LB --> API
    API --> DB
    API --> Cache
    API --> Queue
    Queue --> WS
    WS --> Web
    WS --> Mobile
    
    style Client fill:#e3f2fd
    style Gateway fill:#fff3e0
    style Storage fill:#f3e5f5
    style Messaging fill:#e8f5e9
```

</div>

### 성능 지표

메시지 처리 지연시간 분포:

$$
P(X \leq x) = 1 - e^{-\lambda x}, \quad x \geq 0
$$

여기서 $\lambda = \frac{1}{mean\_latency}$

---

**이제 모든 문서에서 다이어그램과 수식을 자유롭게 사용할 수 있습니다!** ✨

