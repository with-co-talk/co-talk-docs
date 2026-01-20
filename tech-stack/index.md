---
layout: default
title: Tech Stack
description: Co-Talk 최종 기술 스택 결정서
permalink: /tech-stack/
---

# 기술 스택 결정서

대규모 트래픽 성능 목표 기반의 기술 스택입니다.

---

## 목차

| 문서 | 설명 |
|------|------|
| [백엔드 기술](./backend) | Java 25, Spring Boot, Netty, PostgreSQL |
| [프론트엔드 기술](./frontend) | React, Flutter, Zustand |
| [인프라 기술](./infrastructure) | Kubernetes, Redis, Elasticsearch |

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
- **언어**: Java 25 LTS
- **프레임워크**: Spring Boot 3.3+
- **실시간 통신**: Netty (WebSocket)
- **데이터베이스**: PostgreSQL 15+ (Primary + Read Replicas)
- **ORM**: Spring Data JPA + QueryDSL
- **캐싱**: Redis Cluster 7.0+
- **메시지 큐**: Redis Streams → Kafka
- **검색**: Elasticsearch 8.0+

### 프론트엔드
- **웹 (MVP)**: React 18.3+ + TypeScript
- **모바일 (확장)**: Flutter 3.0+

### 인프라
- **컨테이너**: Docker + Kubernetes (AWS EKS)
- **로드 밸런서**: AWS ALB/NLB
- **API Gateway**: Kong
- **모니터링**: Prometheus + Grafana

---

## 마이그레이션 로드맵

| Phase | 기간 | 내용 |
|-------|------|------|
| **Phase 1: MVP** | 1-3개월 | Spring Boot REST, 기본 WebSocket, 단일 DB |
| **Phase 2: 최적화** | 3-6개월 | Netty WebSocket, Redis Cluster, Read Replica |
| **Phase 3: 확장** | 6-12개월 | Kubernetes, API Gateway, Flutter 앱 |
| **Phase 4: 대규모** | 12개월+ | DB 샤딩, Kafka, Elasticsearch, MSA 전환 |

---

## 관련 문서

기술 결정의 상세한 배경과 비교 분석

→ [기술 결정 문서 (ADR)](../decisions/)
