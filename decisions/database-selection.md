---
layout: default
title: Database Selection
description: PostgreSQL 선택 이유 및 비교
---

# 데이터베이스 선택 가이드

[← 기술 결정 목록](./index)

**결정**: PostgreSQL 15+
**상태**: 승인됨

---

## 목차

- [후보 데이터베이스 비교](#1-후보-데이터베이스-비교)
- [프로젝트 요구사항 분석](#2-co-talk-프로젝트-요구사항-분석)
- [최종 결정](#3-최종-결정)

---

## 1. 후보 데이터베이스 비교

### 1.1 관계형 데이터베이스 (RDBMS)

#### PostgreSQL ✅ (선택)

**장점:**
- **ACID 트랜잭션**: 완벽한 트랜잭션 보장
- **강력한 인덱싱**: B-tree, Hash, GIN, GiST 등 다양한 인덱스 타입
- **JSONB 지원**: NoSQL처럼 JSON 데이터 저장 및 쿼리 가능
- **파티셔닝**: 대용량 데이터 효율적 관리
- **Read Replica**: 읽기 부하 분산 용이
- **확장성**: 수평/수직 확장 모두 지원
- **오픈소스**: 무료, 커뮤니티 활발
- **표준 준수**: SQL 표준을 잘 따름

**단점:**
- 복잡한 설정 (하지만 유연함)
- 메모리 사용량이 MySQL보다 약간 높음

#### MySQL/MariaDB

**장점:**
- 널리 사용됨 (생태계 풍부)
- 설정 간단
- 메모리 효율적
- 읽기 성능 우수

**단점:**
- **JSON 지원 약함**: JSONB 없음, JSON 쿼리 제한적
- **파티셔닝 제한**: PostgreSQL보다 제한적
- **트랜잭션 격리 수준**: 기본 설정이 REPEATABLE READ (성능 우선)
- **확장성**: 수평 확장이 PostgreSQL보다 어려움

#### 비교 요약

| 기능 | PostgreSQL | MySQL |
|------|-----------|-------|
| ACID 트랜잭션 | ✅ 완벽 | ✅ 지원 |
| JSON/JSONB | ✅ JSONB (인덱싱 가능) | ⚠️ JSON (제한적) |
| 파티셔닝 | ✅ 강력함 | ⚠️ 제한적 |
| Read Replica | ✅ 용이 | ✅ 지원 |
| 복잡한 쿼리 | ✅ 우수 | ⚠️ 제한적 |
| 확장성 | ✅ 우수 | ⚠️ 보통 |
| 성능 | ✅ 우수 | ✅ 우수 (읽기) |

### 1.2 NoSQL 데이터베이스

#### MongoDB

**장점:**
- 유연한 스키마
- 수평 확장 용이
- JSON 문서 저장
- 빠른 개발

**단점:**
- **ACID 트랜잭션 약함**: 멀티 문서 트랜잭션은 제한적
- **관계형 데이터 모델링 어려움**: 친구 관계, 채팅방 멤버 등
- **복잡한 쿼리 제한**: JOIN 없음
- **일관성**: 최종 일관성 (Eventual Consistency)
- **읽기 복제본 설정 복잡**

**Co-Talk에 부적합한 이유:**
- 친구 관계, 채팅방 멤버 등 관계형 데이터가 많음
- 트랜잭션이 중요함 (메시지 저장 시 여러 테이블 업데이트)
- 복잡한 쿼리 필요 (친구 목록, 채팅방 목록 등)

#### Redis

**장점:**
- 매우 빠른 속도 (인메모리)
- Pub/Sub 지원
- 다양한 데이터 구조

**단점:**
- **영구 저장소 아님**: 메모리 기반 (데이터 손실 위험)
- **용량 제한**: 메모리 크기에 제한
- **복잡한 쿼리 불가**: 키-값 기반

**Co-Talk에서의 역할:**
- 캐싱 (사용자 정보, 친구 목록)
- Pub/Sub (실시간 메시지 브로드캐스팅)
- 메시지 큐 (Redis Streams)
- **주 데이터베이스로는 부적합**

---

## 2. Co-Talk 프로젝트 요구사항 분석

### 2.1 데이터 모델 특성

**관계형 데이터:**
- Users ↔ Friends (다대다 관계)
- Users ↔ ChatRooms (다대다, ChatRoomMembers)
- ChatRooms ↔ Messages (일대다)
- Users ↔ Messages (일대다)

**특징:**
- 복잡한 관계 (JOIN 필요)
- 트랜잭션 중요 (친구 추가 시 여러 테이블 업데이트)
- 정규화 필요 (데이터 일관성)

### 2.2 쿼리 패턴

**복잡한 쿼리 예시:**

1. **친구 목록 조회 (최근 메시지 포함)**
```sql
SELECT 
    u.id, u.nickname, u.avatar_url,
    m.content as last_message,
    m.created_at as last_message_time
FROM friends f
JOIN users u ON f.friend_id = u.id
LEFT JOIN (
    SELECT DISTINCT ON (chat_room_id) 
        chat_room_id, content, created_at
    FROM messages
    WHERE chat_room_id IN (
        SELECT id FROM chat_rooms 
        WHERE id IN (
            SELECT chat_room_id FROM chat_room_members 
            WHERE user_id = $1
        )
    )
    ORDER BY chat_room_id, created_at DESC
) m ON ...
WHERE f.user_id = $1 AND f.status = 'accepted'
ORDER BY m.created_at DESC;
```

2. **채팅방 목록 조회 (읽지 않은 메시지 수 포함)**
```sql
SELECT 
    cr.id,
    u.nickname as other_user_name,
    COUNT(CASE WHEN m.id IS NOT NULL AND m.read_at IS NULL THEN 1 END) as unread_count,
    MAX(m.created_at) as last_message_time
FROM chat_rooms cr
JOIN chat_room_members crm ON cr.id = crm.chat_room_id
JOIN users u ON crm.user_id = u.id AND u.id != $1
LEFT JOIN messages m ON cr.id = m.chat_room_id
WHERE crm.user_id = $1
GROUP BY cr.id, u.nickname
ORDER BY last_message_time DESC;
```

**PostgreSQL의 장점:**
- 복잡한 JOIN 쿼리 최적화
- 서브쿼리, 윈도우 함수 지원
- DISTINCT ON 같은 고급 기능

### 2.3 트랜잭션 요구사항

**친구 요청 수락 시:**
```sql
BEGIN;
  -- 1. 친구 요청 상태 업데이트
  UPDATE friend_requests SET status = 'accepted' WHERE id = $1;
  
  -- 2. Friends 테이블에 양방향 관계 추가
  INSERT INTO friends (user_id, friend_id, status) VALUES ($2, $3, 'accepted');
  INSERT INTO friends (user_id, friend_id, status) VALUES ($3, $2, 'accepted');
  
  -- 3. 채팅방 생성
  INSERT INTO chat_rooms (type) VALUES ('direct') RETURNING id;
  
  -- 4. 채팅방 멤버 추가
  INSERT INTO chat_room_members (chat_room_id, user_id) VALUES ($4, $2);
  INSERT INTO chat_room_members (chat_room_id, user_id) VALUES ($4, $3);
COMMIT;
```

**PostgreSQL의 장점:**
- 완벽한 ACID 트랜잭션
- 트랜잭션 격리 수준 세밀한 제어
- 롤백 보장

---

## 3. PostgreSQL의 Co-Talk에 특화된 장점

### 3.1 JSONB 지원

**사용 사례:**
- 사용자 프로필 확장 정보 (메타데이터)
- 메시지 메타데이터 (향후 확장)
- 채팅방 설정

**예시:**
```sql
-- 사용자 프로필에 메타데이터 저장
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255),
    nickname VARCHAR(50),
    profile_metadata JSONB  -- 유연한 확장
);

-- JSONB 인덱싱 및 쿼리
CREATE INDEX idx_users_metadata ON users USING GIN (profile_metadata);

-- 쿼리 예시
SELECT * FROM users 
WHERE profile_metadata @> '{"preferences": {"theme": "dark"}}';
```

**MySQL과 비교:**
- MySQL: JSON 타입 있으나 인덱싱 제한적, 쿼리 제한적
- PostgreSQL: JSONB로 완전한 인덱싱 및 쿼리 지원

### 3.2 파티셔닝

**메시지 테이블 파티셔닝:**
```sql
-- 월별 파티셔닝
CREATE TABLE messages (
    id UUID,
    chat_room_id UUID,
    sender_id UUID,
    content TEXT,
    created_at TIMESTAMP
) PARTITION BY RANGE (created_at);

-- 파티션 생성
CREATE TABLE messages_2024_01 PARTITION OF messages
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
CREATE TABLE messages_2024_02 PARTITION OF messages
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
```

**장점:**
- 오래된 데이터 자동 아카이빙
- 쿼리 성능 향상 (관련 파티션만 스캔)
- 관리 용이

**MySQL과 비교:**
- MySQL: 파티셔닝 지원하나 제한적
- PostgreSQL: 더 강력하고 유연한 파티셔닝

### 3.3 복잡한 인덱싱

**복합 인덱스 예시:**
```sql
-- 메시지 조회 최적화
CREATE INDEX idx_messages_room_time 
ON messages(chat_room_id, created_at DESC) 
INCLUDE (sender_id, content);

-- 친구 목록 조회 최적화
CREATE INDEX idx_friends_user_status 
ON friends(user_id, status) 
INCLUDE (friend_id);
```

**PostgreSQL의 고급 인덱스:**
- **GIN 인덱스**: JSONB, 배열 검색
- **GiST 인덱스**: 전문 검색, 지리 데이터
- **Partial 인덱스**: 조건부 인덱싱
- **Covering 인덱스**: INCLUDE 절로 추가 컬럼 포함

### 3.4 Read Replica 설정

**PostgreSQL의 장점:**
- 비동기 복제 설정 간단
- 여러 Read Replica 지원
- 지연 시간 모니터링 용이

**설정 예시:**
```sql
-- Primary 서버 설정
wal_level = replica
max_wal_senders = 3

-- Replica 서버 설정
primary_conninfo = 'host=primary_server port=5432 user=replicator'
```

### 3.5 확장성

**수직 확장:**
- 대용량 메모리 활용
- 멀티 코어 CPU 활용
- SSD 최적화

**수평 확장:**
- Read Replica로 읽기 확장
- 파티셔닝으로 데이터 분산
- 샤딩 (Citus 확장 사용 가능)

---

## 4. 성능 비교

### 4.1 읽기 성능

| 작업 | PostgreSQL | MySQL | MongoDB |
|------|-----------|-------|---------|
| 단순 조회 | ✅ 우수 | ✅ 우수 | ✅ 우수 |
| 복잡한 JOIN | ✅ 우수 | ⚠️ 보통 | ❌ 불가 |
| 집계 쿼리 | ✅ 우수 | ✅ 우수 | ⚠️ 제한적 |
| 전문 검색 | ✅ 우수 | ⚠️ 제한적 | ✅ 우수 |

### 4.2 쓰기 성능

| 작업 | PostgreSQL | MySQL | MongoDB |
|------|-----------|-------|---------|
| 단일 삽입 | ✅ 우수 | ✅ 우수 | ✅ 우수 |
| 배치 삽입 | ✅ 우수 | ✅ 우수 | ✅ 우수 |
| 트랜잭션 | ✅ 완벽 | ✅ 지원 | ⚠️ 제한적 |
| 동시성 | ✅ 우수 | ⚠️ 보통 | ✅ 우수 |

### 4.3 대규모 트래픽 처리

**10,000 TPS 처리:**
- PostgreSQL: ✅ 충분 (Read Replica + 파티셔닝)
- MySQL: ✅ 충분 (Read Replica)
- MongoDB: ✅ 충분 (Sharding)

**차이점:**
- PostgreSQL: 복잡한 쿼리도 효율적 처리
- MySQL: 단순 쿼리에서 우수, 복잡한 쿼리는 제한적
- MongoDB: 관계형 데이터 모델링 어려움

---

## 5. Co-Talk 프로젝트에서의 활용

### 5.1 데이터 모델링

**PostgreSQL의 장점:**
- **외래 키 제약**: 데이터 무결성 보장
- **체크 제약**: 상태 값 검증 (예: friend.status IN ('pending', 'accepted'))
- **유니크 제약**: 중복 방지 (예: (user_id, friend_id) 유니크)

**예시:**
```sql
CREATE TABLE friends (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    friend_id UUID REFERENCES users(id),
    status VARCHAR(20) CHECK (status IN ('pending', 'accepted', 'blocked')),
    UNIQUE(user_id, friend_id)
);
```

### 5.2 확장성 고려

**현재 (MVP):**
- 단일 PostgreSQL 인스턴스
- 기본 인덱싱

**확장 시:**
- Read Replica 추가 (읽기 부하 분산)
- 파티셔닝 (메시지 테이블)
- 샤딩 (Citus 확장, 필요 시)

### 5.3 향후 기능 확장

**그룹 채팅:**
- PostgreSQL의 배열 타입 활용 가능
- JSONB로 유연한 채팅방 설정

**메시지 검색:**
- PostgreSQL의 전문 검색 (Full-Text Search)
- 또는 Elasticsearch와 연동

**알림:**
- PostgreSQL의 LISTEN/NOTIFY 활용 가능
- 또는 Redis Pub/Sub 사용

---

## 6. 다른 데이터베이스와의 하이브리드 전략

### 6.1 PostgreSQL + Redis

**PostgreSQL:**
- 주 데이터 저장소
- 관계형 데이터
- 트랜잭션 보장

**Redis:**
- 캐싱 (사용자 정보, 친구 목록)
- Pub/Sub (실시간 메시지)
- 세션 관리

### 6.2 PostgreSQL + Elasticsearch (향후)

**PostgreSQL:**
- 주 데이터 저장소
- 관계형 데이터

**Elasticsearch:**
- 메시지 검색
- 사용자 검색
- 분석

---

## 7. 최종 선택 이유 요약

### ✅ PostgreSQL 선택 이유

1. **관계형 데이터 모델링**
   - 친구 관계, 채팅방 멤버 등 복잡한 관계
   - JOIN 쿼리 최적화

2. **트랜잭션 보장**
   - 친구 추가, 메시지 저장 시 여러 테이블 업데이트
   - ACID 트랜잭션 필수

3. **복잡한 쿼리**
   - 친구 목록 (최근 메시지 포함)
   - 채팅방 목록 (읽지 않은 메시지 수)
   - PostgreSQL의 고급 기능 활용

4. **확장성**
   - Read Replica로 읽기 확장
   - 파티셔닝으로 대용량 데이터 관리
   - 향후 샤딩 가능

5. **유연성**
   - JSONB로 NoSQL처럼 사용 가능
   - 향후 기능 확장 용이

6. **성능**
   - 복잡한 쿼리에서도 우수한 성능
   - 인덱싱 최적화

### ❌ 다른 데이터베이스가 부적합한 이유

**MySQL:**
- JSON 지원 약함
- 복잡한 쿼리 제한적
- 파티셔닝 제한적

**MongoDB:**
- 관계형 데이터 모델링 어려움
- 트랜잭션 제한적
- JOIN 불가

**Redis:**
- 영구 저장소 아님
- 복잡한 쿼리 불가

---

## 8. 결론

**PostgreSQL이 Co-Talk 프로젝트에 최적인 이유:**

1. **관계형 데이터**: 친구 관계, 채팅방 멤버 등 복잡한 관계 모델링
2. **트랜잭션**: 데이터 일관성 보장 필수
3. **복잡한 쿼리**: 친구 목록, 채팅방 목록 등 복잡한 조회
4. **확장성**: Read Replica, 파티셔닝으로 대규모 트래픽 처리
5. **유연성**: JSONB로 향후 기능 확장 용이
6. **성능**: 복잡한 쿼리에서도 우수한 성능

**하이브리드 전략:**
- PostgreSQL: 주 데이터 저장소
- Redis: 캐싱, Pub/Sub, 세션
- Elasticsearch: 검색 (향후)
