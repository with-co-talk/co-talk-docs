---
layout: default
title: Architecture
nav_order: 2
has_children: true
---

# Co-Talk Architecture

시스템 아키텍처 문서입니다.

---

## 목차

| 문서 | 설명 |
|------|------|
| [백엔드 구조](./backend) | Hexagonal Architecture, Spring Boot 3.5.6, WebSocket |
| [프론트엔드 구조](./frontend) | Flutter, BLoC/Cubit, Drift, GoRouter |
| [데이터베이스 설계](./database) | ERD, 16개 테이블, 인덱스, Flyway |
| [API 설계](./api) | 68개 REST 엔드포인트, STOMP WebSocket |
| [인프라](./infrastructure) | Docker Compose, 카나리아 배포, 모니터링 |

---

## 전체 아키텍처 다이어그램

```mermaid
graph TD
    Client[Flutter App<br/>Android · iOS · macOS<br/>Windows · Linux] --> Nginx[Nginx<br/>Rate Limiting · SSL]

    Nginx --> App1[app-1<br/>Spring Boot 3.5.6]
    Nginx --> App2[app-2<br/>Spring Boot 3.5.6]
    Nginx --> App3[app-3<br/>Spring Boot 3.5.6]

    App1 --> PG[(PostgreSQL 16)]
    App2 --> PG
    App3 --> PG

    App1 --> Redis[(Redis 7<br/>Cache · Pub/Sub)]
    App2 --> Redis
    App3 --> Redis

    App1 --> MinIO[(MinIO<br/>S3 Files)]
    App2 --> MinIO
    App3 --> MinIO

    Redis -.->|Pub/Sub| App1
    Redis -.->|Pub/Sub| App2
    Redis -.->|Pub/Sub| App3

    Nginx -->|/files/| MinIO

    App1 -.-> Monitoring[Prometheus · Grafana<br/>Loki · Zipkin]
    App2 -.-> Monitoring
    App3 -.-> Monitoring
```

---

## 기술 스택 요약

| Component | Technology |
|-----------|-----------|
| **Backend** | Java 25 + Spring Boot 3.5.6 (Virtual Threads) |
| **Frontend** | Flutter 3.8+ (Dart SDK ^3.8.1) |
| **Database** | PostgreSQL 16 + Spring Data JPA + QueryDSL |
| **Cache** | Redis 7 (RedisCacheManager) |
| **Real-Time** | STOMP over WebSocket + Redis Pub/Sub |
| **Storage** | MinIO (S3-compatible) |
| **Deployment** | Docker Compose on NAS (3 instances, canary rolling) |
| **CI/CD** | GitHub Actions → GHCR → deploy.sh |
| **Monitoring** | Prometheus + Grafana + Loki + Zipkin + Alertmanager |
