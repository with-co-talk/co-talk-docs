---
layout: default
title: Tech Stack
nav_order: 3
has_children: true
---

# 기술 스택 결정서

대규모 트래픽 성능 목표 기반의 기술 스택입니다.

---

## 목차

| 문서 | 설명 |
|------|------|
| [백엔드 기술](./backend) | Java 25, Spring Boot 3.5.6, PostgreSQL 16, Redis 7 |
| [프론트엔드 기술](./frontend) | Flutter 3.8+, BLoC/Cubit, Drift, Dio |
| [인프라 기술](./infrastructure) | Docker Compose, Nginx, Prometheus, Grafana |

---

## 성능 목표

### 트래픽 목표

| 지표 | 목표 |
|------|------|
| 동시 접속자 | 100만+ 명 |
| DAU | 500만+ |
| 초당 메시지 (TPS) | 10,000+ |
| 피크 시간대 | 동시 접속자 200만+ |
| 일일 메시지 저장량 | 10억+ |

### 성능 요구사항

| 지표 | 목표 |
|------|------|
| API 응답 시간 | P95 < 100ms |
| 메시지 전달 지연 | < 50ms |
| 가용성 | 99.9% 이상 |
| DB 쿼리 | P95 < 50ms |

---

## 기술 스택 요약

### 백엔드
- **언어**: Java 25 (Virtual Threads, JEP 491)
- **프레임워크**: Spring Boot 3.5.6 (Spring MVC)
- **실시간 통신**: STOMP over WebSocket + Redis Pub/Sub
- **데이터베이스**: PostgreSQL 16
- **ORM**: Spring Data JPA + QueryDSL 5.1.0
- **캐싱**: Redis 7 (RedisCacheManager)
- **보안**: AES-256 메시지 암호화, JWT, Bucket4j Rate Limiting

### 프론트엔드
- **프레임워크**: Flutter 3.8+ (Dart SDK ^3.8.1)
- **상태 관리**: BLoC/Cubit (flutter_bloc)
- **플랫폼**: Android, iOS, macOS, Windows, Linux

### 인프라 (현재)
- **오케스트레이션**: Docker Compose on Synology NAS (8GB)
- **리버스 프록시**: Nginx (Rate Limiting, SSL)
- **배포**: 카나리아 롤링 (3개 인스턴스)
- **모니터링**: Prometheus + Grafana + Loki + Zipkin

---

## 마이그레이션 로드맵

| Phase | 기간 | 내용 |
|-------|------|------|
| **Phase 1: MVP** | ✅ 완료 | Spring Boot REST + STOMP WebSocket, PostgreSQL, Flutter 앱 |
| **Phase 2: 프로덕션** | ✅ 완료 | 3-인스턴스 카나리아 배포, Redis Pub/Sub, 모니터링 스택 |
| **Phase 3: 확장** | 향후 | Kubernetes 전환, Read Replica, API Gateway |
| **Phase 4: 대규모** | 향후 | DB 샤딩, Kafka, Elasticsearch, MSA 전환 |

---

## 관련 문서

기술 결정의 상세한 배경과 비교 분석

→ [기술 결정 문서 (ADR)](../decisions/)
