---
layout: default
title: Backend Tech Stack
parent: Tech Stack
nav_order: 1
---

# 백엔드 기술 스택

[← 기술 스택 개요](./index)

---

## 목차

- [언어 및 런타임](#언어-및-런타임)
- [웹 프레임워크](#웹-프레임워크)
- [실시간 통신](#실시간-통신)
- [데이터베이스](#데이터베이스)
- [ORM 및 마이그레이션](#orm-및-마이그레이션)
- [캐싱](#캐싱)
- [보안](#보안)
- [메시지 큐](#메시지-큐)
- [의존성 버전](#의존성-버전)

---

## 언어 및 런타임

### ✅ 결정: Java 25

**선택 이유**
- **Virtual Threads 개선 (JEP 491)**: Pinning 문제 해결, `synchronized` 블록에서도 carrier thread 해제
- **최신 LTS**: 2025년 9월 출시, 2033년까지 장기 지원
- **I/O 바운드 최적화**: 수백만 동시 연결 처리 가능
- **멀티스레딩**: Node.js 단일 스레드 한계 극복
- **JVM 최적화**: 10,000+ TPS 처리 가능

**Virtual Threads 활성화**
```yaml
spring:
  threads:
    virtual:
      enabled: true
```

**JVM 설정** (Docker)
```
-XX:+UseContainerSupport
-XX:MaxRAMPercentage=75.0
-XX:InitialRAMPercentage=50.0
-XX:+UseG1GC
-XX:+UseStringDeduplication
```

**대안 비교**
| 기술 | 평가 |
|------|------|
| Java 21 | ⚠️ Virtual Threads pinning 문제 존재 (JEP 491 미포함) |
| Java 24 | ⚠️ 비-LTS, 프로덕션 부적합 |
| Node.js | ❌ 단일 스레드 한계 |
| Go | ⚠️ 실시간 메시징 생태계 부족 |

---

## 웹 프레임워크

### ✅ 결정: Spring Boot 3.5.6 (Spring MVC + Virtual Threads)

**선택 이유**
- **Spring MVC + Virtual Threads**: WebFlux 수준의 동시성 확보
- **개발 생산성**: 명령형 코드, 학습 곡선 낮음
- **생태계 호환성**: Spring Data JPA 등 기존 라이브러리 사용 가능
- **성능**: 수백만 동시 요청 처리 가능

**Spring MVC vs WebFlux**
| | Spring MVC | WebFlux |
|--|------------|---------|
| 코드 스타일 | ✅ 명령형 (쉬움) | 리액티브 (복잡) |
| 성능 | ✅ Virtual Threads로 충분 | 높음 |
| 라이브러리 호환 | ✅ 모든 라이브러리 | 리액티브 전용 필요 |

**주요 Spring Boot Starters**
- `spring-boot-starter-web` — REST API
- `spring-boot-starter-websocket` — STOMP over WebSocket
- `spring-boot-starter-data-jpa` — 데이터 액세스
- `spring-boot-starter-security` — 인증/인가
- `spring-boot-starter-validation` — 입력 검증
- `spring-boot-starter-cache` — Redis 캐싱
- `spring-boot-starter-actuator` — 메트릭/헬스체크

**Graceful Shutdown**
```yaml
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

→ [Spring MVC vs WebFlux 상세 비교](../decisions/spring-mvc-vs-webflux)

---

## 실시간 통신

### ✅ 결정: STOMP over WebSocket + Redis Pub/Sub

**현재 구현**
- **프로토콜**: STOMP over WebSocket at `/ws` (SockJS fallback)
- **브로커**: SimpleBroker (로컬) + Redis Pub/Sub (멀티 인스턴스 팬아웃)
- **인증**: STOMP CONNECT 시 JWT 검증
- **인가**: SUBSCRIBE 시 채팅방 멤버십 체크

**멀티 인스턴스 동작**
```
Instance 1 ──publish──> Redis Pub/Sub ──subscribe──> Instance 1 → WebSocket clients
Instance 2 ──publish──> Redis Pub/Sub ──subscribe──> Instance 2 → WebSocket clients
Instance 3 ──publish──> Redis Pub/Sub ──subscribe──> Instance 3 → WebSocket clients
```

**Redis 채널 구조**
| 채널 패턴 | 용도 |
|-----------|------|
| `chat:room:{roomId}` | 채팅 메시지 |
| `chat:room:{roomId}:reaction` | 리액션 이벤트 |
| `chat:room:{roomId}:event` | 룸 이벤트 (READ, TYPING, DELETE, UPDATE) |
| `user:event:{userId}` | 사용자 이벤트 (채팅 목록 업데이트) |

**Transport 제한**
| 항목 | 설정 |
|------|------|
| 메시지 크기 | 128KB |
| 전송 버퍼 | 1MB |
| 전송 타임아웃 | 20초 |

**이벤트 타입**: MESSAGE, REACTION_ADDED, REACTION_REMOVED, TYPING, STOP_TYPING, READ, MESSAGE_DELETED, MESSAGE_UPDATED, LINK_PREVIEW_UPDATED, USER_LEFT, USER_JOINED

**스키마 버전관리**: 모든 WebSocket 메시지에 `schemaVersion` + `eventId` 포함 (클라이언트 중복 제거 지원)

**스케일 플랜**
| 단계 | 기술 | 시기 |
|------|------|------|
| 현재 | STOMP + Redis Pub/Sub | ✅ 운영 중 |
| 확장 | Netty 기반 커스텀 서버 | 100만+ 동시 연결 시 |

---

## 데이터베이스

### ✅ 결정: PostgreSQL 16

**선택 이유**

| 기능 | 설명 |
|------|------|
| **관계형 모델링** | 친구 관계, 채팅방 멤버 등 복잡한 관계 처리 |
| **ACID 트랜잭션** | 친구 추가 시 여러 테이블 업데이트 |
| **복잡한 쿼리** | 친구 목록 + 최근 메시지, 읽지 않은 메시지 수 |
| **JSONB** | NoSQL처럼 유연한 스키마 확장 |
| **GIN 인덱스** | 메시지 전문 검색 (tsvector) |

**현재 설정**
- Alpine 이미지, 512MB 메모리 제한 (프로덕션)
- Connection Pooling: HikariCP
- 프로덕션에서 외부 포트 미노출

**스케일 플랜**
- Read Replicas: 읽기 부하 분산
- Messages 테이블 월별 파티셔닝
- 샤딩 (필요 시)

→ [데이터베이스 선택 상세 비교](../decisions/database-selection)

---

## ORM 및 마이그레이션

### ✅ 결정: Spring Data JPA + QueryDSL 5.1.0 + Flyway

**Spring Data JPA**
- Spring Boot와 완벽 통합
- Repository 패턴으로 데이터 액세스 추상화

**QueryDSL 5.1.0 (Jakarta)**
- 타입 안전한 동적 쿼리
- 복잡한 조건 검색에 활용

**Flyway**
- 스키마 버전 관리 (V1: 초기 스키마 16개 테이블, V2: 인덱스)
- SQL 기반 마이그레이션
- 환경별 자동 실행

---

## 캐싱

### ✅ 결정: Redis 7 (RedisCacheManager)

**사용 용도**
| 용도 | 설명 |
|------|------|
| **캐싱** | RedisCacheManager (멀티 인스턴스 캐시 공유) |
| **Pub/Sub** | WebSocket 서버 간 메시지 브로드캐스팅 |
| **Rate Limiting** | Bucket4j 백엔드 (사용자/IP별 제한) |

**캐시 TTL**
| 캐시 | TTL |
|------|-----|
| USER | 1시간 |
| CHAT_ROOM | 30분 |
| STATISTICS | 5분 |

**Redis 설정**
- Alpine 이미지, allkeys-lru 정책
- 128MB (프로덕션) / 256MB (개발)
- 프로덕션에서 외부 포트 미노출

---

## 보안

### 인증/인가
- **JWT** (HMAC-SHA256): Access Token + Refresh Token
- **BCrypt**: 비밀번호 해싱
- **Spring Security**: Stateless 세션, 역할 기반 접근 제어

### 데이터 보호
- **AES-256**: 메시지 내용 저장 시 암호화 (`EncryptedStringConverter`)
- **환경변수**: 모든 시크릿 외부 관리 (DB_PASSWORD, JWT_SECRET, ENCRYPTION_KEY)

### 보안 헤더
- HSTS (1년, preload)
- CSP (Content Security Policy)
- X-Frame-Options DENY
- X-Content-Type-Options nosniff

### Rate Limiting (다중 계층)
| 계층 | 도구 | 설정 |
|------|------|------|
| Nginx | limit_req | auth 5r/m, WS 10r/s, general 30r/s |
| Application | Bucket4j + Redis | login 5/min, signup 3/min, file upload 10/min + 50/hr |

---

## 메시지 큐

### ✅ 결정: Redis Pub/Sub (현재) → Apache Kafka (확장 시)

**현재**: Redis Pub/Sub
- 채팅방별 채널로 메시지 팬아웃
- 3개 인스턴스 간 실시간 동기화

**확장 시**: Apache Kafka
- 일일 메시지 10억+ 도달 시 전환
- 강력한 내구성 및 순서 보장

→ [Redis Streams 순서 보장 상세 가이드](../decisions/redis-streams-ordering)

---

## 의존성 버전

| Component | Version |
|-----------|---------|
| Java | 25 |
| Spring Boot | 3.5.6 |
| spring-dependency-management | 1.1.7 |
| QueryDSL | 5.1.0 (jakarta) |
| JJWT | 0.12.5 |
| springdoc-openapi | 2.8.14 |
| Firebase Admin SDK | 9.2.0 |
| AWS SDK S3 (MinIO) | 2.25.16 BOM |
| Bucket4j | 8.10.1 |
| Redisson | 3.51.0 |
| jsoup | 1.17.2 |
| Loki Logback | 1.4.2 |
| Logstash Logback Encoder | 8.0 |
| Micrometer (Prometheus) | managed |
| Micrometer Tracing (Brave) | managed |
| Flyway | managed |
| Testcontainers | 1.20.1 |
| ArchUnit | 1.4.1 |
| JaCoCo | 0.8.14 |

---

## 다음 문서

→ [프론트엔드 기술](./frontend)
