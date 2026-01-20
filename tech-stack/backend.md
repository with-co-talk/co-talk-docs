---
layout: default
title: Backend Tech Stack
description: Co-Talk 백엔드 기술 스택
permalink: /tech-stack/backend
---

# 백엔드 기술 스택

[← 기술 스택 개요](./index)

---

## 목차

- [언어 및 런타임](#언어-및-런타임)
- [웹 프레임워크](#웹-프레임워크)
- [실시간 통신](#실시간-통신)
- [데이터베이스](#데이터베이스)
- [ORM](#orm)
- [캐싱](#캐싱)
- [메시지 큐](#메시지-큐)
- [검색 엔진](#검색-엔진)

---

## 언어 및 런타임

### ✅ 결정: Java 25 LTS

**선택 이유**
- **Virtual Threads 개선**: pinning 문제 해결 등 안정성 향상
- **최신 LTS**: 2025년 9월 출시, 2033년까지 장기 지원
- **I/O 바운드 최적화**: 수백만 동시 연결 처리 가능
- **멀티스레딩**: Node.js 단일 스레드 한계 극복
- **JVM 최적화**: 10,000+ TPS 처리 가능

**Java 25 핵심 장점**
- Virtual Threads 개선
- WebSocket 최적화 (기존 대비 10-100배 효율)
- Structured Concurrency
- Pattern Matching
- Record Patterns

**대안 비교**
| 기술 | 평가 |
|------|------|
| Java 21 | ⚠️ Virtual Threads 개선사항 없음 |
| Java 24 | ⚠️ 비-LTS, 프로덕션 부적합 |
| Node.js | ❌ 단일 스레드 한계 |
| Go | ⚠️ 실시간 메시징 생태계 부족 |

---

## 웹 프레임워크

### ✅ 결정: Spring Boot 3.3+ (Spring MVC + Virtual Threads)

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

**주요 의존성**
```xml
- spring-boot-starter-web
- spring-boot-starter-websocket
- spring-boot-starter-data-jpa
- spring-boot-starter-security
- spring-boot-starter-validation
- spring-boot-starter-cache
- spring-boot-starter-actuator
```

**Virtual Threads 활성화**
```properties
spring.threads.virtual.enabled=true
```

→ [Spring MVC vs WebFlux 상세 비교](../decisions/spring-mvc-vs-webflux)

---

## 실시간 통신

### ✅ 결정: Netty 기반 커스텀 WebSocket 서버

**선택 이유**
- **최고 성능**: 수백만 동시 연결 처리 가능
- **낮은 지연시간**: 네이티브 성능에 가까움
- **메모리 효율**: 직접 메모리 관리로 낮은 메모리 사용

**대안 비교**
| 기술 | 평가 |
|------|------|
| Spring WebSocket | ⚠️ Netty보다 성능 낮음, 개발 속도 빠름 |
| Socket.io | ❌ Node.js 전용, Java 사용 불가 |

**구현 전략**
- Netty WebSocket 서버 독립 서비스로 분리
- Redis Pub/Sub로 서버 간 메시지 브로드캐스팅
- Connection Manager로 연결 상태 관리

---

## 데이터베이스

### ✅ 결정: PostgreSQL 15+ (Primary + Read Replicas)

**선택 이유**

| 기능 | 설명 |
|------|------|
| **관계형 모델링** | 친구 관계, 채팅방 멤버 등 복잡한 관계 처리 |
| **ACID 트랜잭션** | 친구 추가 시 여러 테이블 업데이트 |
| **복잡한 쿼리** | 친구 목록 + 최근 메시지, 읽지 않은 메시지 수 |
| **JSONB** | NoSQL처럼 유연한 스키마 확장 |
| **확장성** | Read Replica, 파티셔닝, 샤딩 가능 |
| **고급 인덱싱** | 복합/부분/커버링 인덱스, GIN 인덱스 |

**설정**
- Primary: 쓰기 전용
- Read Replicas: 3-5개 (읽기 부하 분산)
- Connection Pooling: HikariCP
- 파티셔닝: Messages 테이블 월별 파티셔닝

→ [데이터베이스 선택 상세 비교](../decisions/database-selection)

---

## ORM

### ✅ 결정: Spring Data JPA + QueryDSL

**선택 이유**
- Spring Boot와 완벽 통합
- 타입 안전한 쿼리 (QueryDSL)
- 자동 쿼리 최적화
- 복잡한 쿼리도 JPA로 처리 가능

---

## 캐싱

### ✅ 결정: Redis Cluster 7.0+

**사용 용도**
| 용도 | 설명 |
|------|------|
| **캐싱** | 사용자 정보, 친구 목록, 채팅방 정보 |
| **Pub/Sub** | WebSocket 서버 간 메시지 브로드캐스팅 |
| **세션** | WebSocket 연결 상태 관리 |
| **메시지 큐** | Redis Streams (초기) |

**설정**
- 클러스터 모드: 3 Master + 3 Replica
- 메모리: 32GB+ per node
- 지속성: AOF (Append Only File)

---

## 메시지 큐

### ✅ 결정: Redis Streams (초기) → Apache Kafka (확장 시)

**전략**
- **초기**: Redis Streams (간단한 설정, 낮은 지연)
- **확장 시**: Kafka (대용량, 강력한 내구성)

**Redis Streams 순서 보장**
- ✅ 트래픽 높을 때도 순서 보장됨
- 채팅방별 Stream 사용 시 완벽한 순서 보장

**Kafka 전환 기준**
- 일일 메시지 10억+ 도달 시
- 더 강력한 내구성이 필요한 경우

→ [Redis Streams 순서 보장 상세 가이드](../decisions/redis-streams-ordering)

---

## 검색 엔진

### ✅ 결정: Elasticsearch 8.0+

**선택 이유**
- 메시지 검색 기능
- 사용자 검색
- 실시간 인덱싱
- 분산 검색

**설정**
- 클러스터: 3 Master + 3 Data Node
- 샤딩: 인덱스당 5 샤드
- 리플리카: 1개

---

## 다음 문서

→ [프론트엔드 기술](./frontend)
