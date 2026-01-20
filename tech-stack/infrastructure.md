---
layout: default
title: Infrastructure Tech Stack
description: Co-Talk 인프라 기술 스택
permalink: /tech-stack/infrastructure
---

# 인프라 기술 스택

[← 기술 스택 개요](./index)

---

## 목차

- [컨테이너화](#컨테이너화)
- [로드 밸런서](#로드-밸런서)
- [API Gateway](#api-gateway)
- [모니터링](#모니터링)
- [개발 도구](#개발-도구)
- [아키텍처 다이어그램](#아키텍처-다이어그램)
- [성능 최적화](#성능-최적화)
- [확장성 계획](#확장성-계획)
- [비용 예상](#비용-예상)

---

## 컨테이너화

### ✅ 결정: Docker + Kubernetes

**선택 이유**
- 표준화된 배포
- 자동 스케일링
- 롤링 업데이트
- 서비스 디스커버리

### 배포 플랫폼

**✅ 결정: AWS EKS (Elastic Kubernetes Service)**

| 장점 |
|------|
| 관리형 Kubernetes |
| 자동 스케일링 |
| 고가용성 |
| 통합 모니터링 |

**대안**: Google GKE, Azure AKS, 온프레미스 Kubernetes

---

## 로드 밸런서

### ✅ 결정: AWS ALB + NLB

| 로드 밸런서 | 용도 |
|------------|------|
| **ALB** | HTTP/HTTPS 트래픽 (REST API) |
| **NLB** | TCP/UDP 트래픽 (WebSocket) |

**기능**
- 자동 스케일링
- Health Check
- SSL Termination

**대안**: NGINX, CloudFlare Load Balancer

---

## API Gateway

### ✅ 결정: Kong 또는 AWS API Gateway

**기능**
- 라우팅 및 로드 밸런싱
- 인증/인가 (JWT 검증)
- Rate Limiting
- 요청/응답 변환

---

## 모니터링

### ✅ 결정: Prometheus + Grafana

**선택 이유**
- 오픈소스
- 풍부한 메트릭
- 커스텀 대시보드
- 알림 연동

### 추가 도구

| 용도 | 도구 |
|------|------|
| **APM** | New Relic 또는 Datadog |
| **로깅** | ELK Stack (Elasticsearch, Logstash, Kibana) |
| **분산 추적** | Jaeger |

---

## 개발 도구

### 빌드 도구
**✅ 결정: Gradle**
- Kotlin DSL 지원
- 빠른 빌드
- 의존성 관리

### 코드 품질

| 도구 | 용도 |
|------|------|
| Checkstyle, SpotBugs | Linting |
| Google Java Format | 포맷팅 |
| SonarQube | 정적 분석 |

### 테스팅

| 도구 | 용도 |
|------|------|
| JUnit 5 + Mockito | 단위 테스트 |
| Spring Boot Test | 통합 테스트 |
| Playwright | E2E 테스트 (프론트엔드) |

### CI/CD
**✅ 결정: GitHub Actions**

**파이프라인**
1. 코드 품질 검사
2. 테스트 실행
3. 빌드
4. Docker 이미지 생성
5. Kubernetes 배포

---

## 아키텍처 다이어그램

```
                    ┌─────────────┐
                    │   CDN       │
                    │  (CloudFlare)│
                    └──────┬──────┘
                           │
                    ┌──────▼────────────────────────────┐
                    │   Load Balancer (ALB/NLB)         │
                    │   - SSL Termination               │
                    │   - Health Check                   │
                    └──────┬────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
┌───────▼──────┐  ┌────────▼────────┐  ┌─────▼──────┐
│ API Gateway  │  │  WebSocket     │  │  API       │
│ (Kong)       │  │  Gateway       │  │  Servers   │
│              │  │  (Netty)       │  │  (Spring)  │
│              │  │  - 10 instances│  │  - 20 pods │
└───────┬──────┘  └────────┬────────┘  └─────┬──────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
┌───────▼──────┐  ┌────────▼────────┐  ┌─────▼──────┐
│   Redis      │  │   Redis        │  │  Redis    │
│  Cluster     │  │  Cluster       │  │  Cluster  │
│  (Cache)     │  │  (Pub/Sub)     │  │  (Streams)│
└───────┬──────┘  └────────┬────────┘  └─────┬──────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
┌───────▼──────┐  ┌────────▼────────┐  ┌─────▼──────┐
│ PostgreSQL   │  │  PostgreSQL     │  │ PostgreSQL │
│ (Primary)    │  │  (Read Replica) │  │ (Read      │
│              │  │                 │  │  Replica)  │
└──────────────┘  └─────────────────┘  └────────────┘
                           │
                    ┌──────▼──────┐
                    │ Elasticsearch│
                    │  Cluster     │
                    └──────────────┘
```

---

## 성능 최적화

### WebSocket 서버 최적화

**Netty 설정**
| 항목 | 설정 |
|------|------|
| 이벤트 루프 스레드 | CPU 코어 수 |
| 워커 스레드 | CPU 코어 수 × 2 |
| Direct Memory | 4GB+ |
| 연결당 메모리 | ~10KB |

**예상 성능**
- 단일 서버: 100만+ 동시 연결
- 클러스터 (10대): 1,000만+ 동시 연결

### 데이터베이스 최적화

**인덱스 전략**
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

**설정**
- Messages 테이블: 월별 파티셔닝
- Connection Pooling: HikariCP (최대 50 connections per instance)
- Read Replica: 읽기 쿼리만 라우팅

### 캐싱 전략

**캐시 계층**
| 계층 | 기술 | TTL | 용도 |
|------|------|-----|------|
| L1 (Local) | Caffeine | 1분 | 자주 조회되는 데이터 |
| L2 (Redis) | Redis Cluster | 1시간 | 사용자 정보, 친구 목록 |

**캐시 무효화**
- Write-Through: 쓰기 시 캐시 업데이트
- TTL 기반 자동 만료

---

## 확장성 계획

### 수평 확장

| 컴포넌트 | 초기 | 목표 | 확장 |
|----------|------|------|------|
| WebSocket 서버 | 3대 | 10대 (100만) | 50대 (1,000만) |
| API 서버 | 5 pods | 20 pods | Auto Scaling |
| DB Read Replica | 2개 | 5개 | 샤딩 (필요 시) |

### 수직 확장

| 컴포넌트 | 사양 |
|----------|------|
| WebSocket 서버 | 16 vCPU, 32GB RAM |
| API 서버 | 8 vCPU, 16GB RAM |
| PostgreSQL | 32 vCPU, 128GB RAM |

---

## 비용 예상

### 월간 인프라 비용 (AWS 기준)

| 항목 | 비용 |
|------|------|
| EKS 클러스터 | $73 |
| EC2 (WebSocket, 10대) | $1,200 |
| EC2 (API, 20 pods) | $800 |
| RDS PostgreSQL | $500 |
| ElastiCache Redis | $600 |
| Elasticsearch | $400 |
| ALB/NLB | $50 |
| 데이터 전송 | $200 |
| **총계** | **~$3,800/월** |

### 최적화

| 방법 | 예상 비용 |
|------|----------|
| Reserved Instances (30% 할인) | $2,700/월 |
| Spot Instances (70% 할인) | $1,900/월 |

---

## 리스크 및 대응

### 기술적 리스크

| 리스크 | 대응 |
|--------|------|
| Netty 학습 곡선 | Spring WebSocket으로 시작, 점진적 전환 |
| DB 병목 | Read Replica 추가, 캐싱 강화 |
| 메모리 부족 | 모니터링 강화, 자동 스케일링 |

### 운영 리스크

| 리스크 | 대응 |
|--------|------|
| 장애 대응 | 모니터링 및 알림, 자동 복구 |
| 비용 증가 | Auto Scaling, Reserved Instances |

---

## 관련 문서

→ [MSA vs 모놀리식 아키텍처 결정](../decisions/msa-vs-monolith)
