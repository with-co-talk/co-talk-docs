# Co-Talk 최종 기술 스택 결정서
## 대규모 트래픽 성능 목표 기반

---

## 1. 성능 목표

### 1.1 트래픽 목표
- **동시 접속자**: 100만+ 명
- **일일 활성 사용자 (DAU)**: 500만+
- **초당 메시지 전송량 (TPS)**: 10,000+
- **피크 시간대**: 동시 접속자 200만+
- **메시지 저장량**: 일일 10억+ 메시지

### 1.2 성능 요구사항
- **API 응답 시간**: P95 < 100ms
- **메시지 전달 지연**: < 50ms
- **가용성**: 99.9% 이상
- **데이터베이스 쿼리**: P95 < 50ms

---

## 2. 최종 기술 스택 결정

### 2.1 백엔드

#### 언어 및 런타임
**✅ 결정: Java 25 LTS**

**선택 이유:**
- **Virtual Threads 개선**: Java 24에서 개선된 Virtual Threads (pinning 문제 해결 등)
- **최신 LTS**: 2025년 9월 출시, 최소 8년간 장기 지원 보장 (2033년까지)
- **I/O 바운드 작업 최적화**: 수백만 동시 연결 처리 가능
- **성능 개선**: GC 및 JIT 컴파일러 지속적 개선
- **멀티스레딩**: CPU 효율적 활용 (Node.js 단일 스레드 한계 극복)
- **JVM 최적화**: 높은 처리량 (10,000+ TPS 처리 가능)
- **엔터프라이즈급 안정성**: 검증된 확장성
- **풍부한 모니터링 도구**: APM, 프로파일링

**Java 25의 핵심 장점 (대규모 트래픽에 유리):**
- **Virtual Threads 개선**: Java 24에서 해결된 pinning 문제 등 안정성 향상
- **WebSocket 최적화**: I/O 집약 작업에 기존 스레드 대비 10-100배 효율적
- **Structured Concurrency**: 동시성 프로그래밍 개선
- **Pattern Matching**: 코드 가독성 향상
- **Record Patterns**: 데이터 클래스 활용 개선
- **AI 기능 확장**: 향후 AI 기능 통합 시 유리

**대안 고려:**
- ⚠️ Java 21: 검증된 LTS이지만 Virtual Threads 개선사항 없음
- ⚠️ Java 24: Virtual Threads 개선 있으나 비-LTS, 프로덕션에 부적합
- ❌ Java 17: Virtual Threads 없음
- ❌ Node.js: 단일 스레드 한계, CPU 집약 작업에 불리
- ⚠️ Go: 성능 우수하나 실시간 메시징 생태계 부족, 팀 학습 곡선

#### 웹 프레임워크
**✅ 결정: Spring Boot 3.3+ (Java 25 기반) - Spring MVC + Virtual Threads**

**선택 이유:**
- **Spring MVC + Virtual Threads**: Java 25의 Virtual Threads로 WebFlux 수준의 동시성 확보
- **개발 생산성**: 명령형 코드 (기존 방식), 학습 곡선 낮음
- **생태계 호환성**: Spring Data JPA 등 기존 블로킹 라이브러리 모두 사용 가능
- **성능**: Virtual Threads로 수백만 동시 요청 처리 가능
- **엔터프라이즈급 프레임워크**: 풍부한 생태계 및 라이브러리
- **프로덕션 검증된 안정성**

**Spring MVC vs WebFlux 선택:**
- ✅ **Spring MVC 선택**: Virtual Threads로 충분한 성능, 개발 생산성 우수
- ❌ **WebFlux 미선택**: 리액티브 프로그래밍 학습 곡선, 코드 복잡도 증가, 모든 라이브러리 리액티브 필요

**주요 의존성:**
```xml
- spring-boot-starter-web (Spring MVC)
- spring-boot-starter-websocket
- spring-boot-starter-data-jpa
- spring-boot-starter-security
- spring-boot-starter-validation
- spring-boot-starter-cache
- spring-boot-starter-actuator (모니터링)
```

**Virtual Threads 활성화:**
```properties
# application.properties
spring.threads.virtual.enabled=true
```

**참고 문서:**
- [Spring MVC vs WebFlux 상세 비교](./SPRING-MVC-VS-WEBFLUX.md)

#### 실시간 통신
**✅ 결정: Netty 기반 커스텀 WebSocket 서버**

**선택 이유:**
- **최고 성능**: 이벤트 기반 비동기 I/O, 수백만 동시 연결 처리 가능
- **낮은 지연시간**: 네이티브 성능에 가까움
- **메모리 효율**: 직접 메모리 관리로 낮은 메모리 사용
- **확장성**: 수평 확장에 최적화

**대안 고려:**
- ⚠️ Spring WebSocket: Netty보다 성능 낮음, 하지만 개발 속도 빠름
- ❌ Socket.io: Node.js 생태계, Java에서 사용 불가

**구현 전략:**
- Netty WebSocket 서버 독립 서비스로 분리
- Redis Pub/Sub로 서버 간 메시지 브로드캐스팅
- Connection Manager로 연결 상태 관리

#### 데이터베이스
**✅ 결정: PostgreSQL 15+ (Primary + Read Replicas)**

**선택 이유:**

1. **관계형 데이터 모델링**
   - 친구 관계, 채팅방 멤버 등 복잡한 관계 (다대다)
   - JOIN 쿼리 최적화 (친구 목록, 채팅방 목록 조회)
   - 외래 키 제약으로 데이터 무결성 보장

2. **ACID 트랜잭션 보장**
   - 친구 추가 시 여러 테이블 업데이트 (트랜잭션 필수)
   - 메시지 저장 시 일관성 보장
   - 완벽한 롤백 지원

3. **복잡한 쿼리 처리**
   - 친구 목록 조회 (최근 메시지 포함)
   - 채팅방 목록 조회 (읽지 않은 메시지 수 포함)
   - 서브쿼리, 윈도우 함수, DISTINCT ON 등 고급 기능

4. **JSONB 지원**
   - NoSQL처럼 유연한 스키마 확장
   - 사용자 프로필 메타데이터, 채팅방 설정 등
   - GIN 인덱스로 빠른 검색

5. **확장성**
   - Read Replica로 읽기 부하 분산 (3-5개)
   - 파티셔닝 지원 (메시지 테이블 월별 파티셔닝)
   - 향후 샤딩 가능 (Citus 확장)

6. **고급 인덱싱**
   - 복합 인덱스, 부분 인덱스, 커버링 인덱스
   - GIN 인덱스 (JSONB, 배열 검색)
   - 쿼리 성능 최적화

**MySQL과 비교:**
- ❌ MySQL: JSON 지원 약함, 복잡한 쿼리 제한적, 파티셔닝 제한적
- ✅ PostgreSQL: JSONB 완벽 지원, 복잡한 쿼리 우수, 강력한 파티셔닝

**MongoDB와 비교:**
- ❌ MongoDB: 관계형 데이터 모델링 어려움, 트랜잭션 제한적, JOIN 불가
- ✅ PostgreSQL: 관계형 데이터 최적화, 완벽한 트랜잭션, JOIN 우수

**설정:**
- Primary: 쓰기 전용
- Read Replicas: 3-5개 (읽기 부하 분산)
- Connection Pooling: HikariCP
- 파티셔닝: Messages 테이블 월별 파티셔닝

**참고 문서:**
- [데이터베이스 선택 상세 비교](./DATABASE-SELECTION.md)

#### ORM
**✅ 결정: Spring Data JPA + QueryDSL**

**선택 이유:**
- Spring Boot와 완벽 통합
- 타입 안전한 쿼리 (QueryDSL)
- 자동 쿼리 최적화
- 복잡한 쿼리도 JPA로 처리 가능

**대안 고려:**
- ⚠️ MyBatis: 성능 우수하나 보일러플레이트 많음
- ❌ JOOQ: 학습 곡선, 복잡도 증가

#### 캐싱
**✅ 결정: Redis Cluster 7.0+**

**선택 이유:**
- 인메모리 캐싱으로 빠른 응답 시간
- Pub/Sub로 실시간 메시지 브로드캐스팅
- Streams로 메시지 큐 역할
- 클러스터 모드로 고가용성

**사용 용도:**
1. **캐싱**: 사용자 정보, 친구 목록, 채팅방 정보
2. **Pub/Sub**: WebSocket 서버 간 메시지 브로드캐스팅
3. **세션**: WebSocket 연결 상태 관리
4. **메시지 큐**: Redis Streams (초기), 필요 시 Kafka로 전환

**설정:**
- 클러스터 모드: 3 Master + 3 Replica (최소)
- 메모리: 32GB+ per node
- 지속성: AOF (Append Only File)

#### 메시지 큐
**✅ 결정: Redis Streams (초기) → Apache Kafka (확장 시)**

**선택 이유:**
- **초기**: Redis Streams로 충분 (간단한 설정, 낮은 지연)
- **순서 보장**: Redis Streams도 순서 보장됨 (트래픽과 무관)
- **확장 시**: Kafka로 전환 (대용량, 더 강력한 내구성)

**Redis Streams 순서 보장:**
- ✅ **트래픽이 낮을 때뿐만 아니라 높을 때도 순서 보장됨**
- 채팅방별 Stream 사용 시 완벽한 순서 보장
- 컨슈머 그룹 내에서 순서대로 처리됨

**전환 기준:**
- 일일 메시지 10억+ 도달 시
- 더 강력한 내구성이 필요한 경우
- 더 복잡한 스트림 처리가 필요한 경우

**참고 문서:**
- [Redis Streams 순서 보장 상세 가이드](./REDIS-STREAMS-ORDERING.md)

#### 검색 엔진
**✅ 결정: Elasticsearch 8.0+**

**선택 이유:**
- 메시지 검색 기능
- 사용자 검색
- 실시간 인덱싱
- 분산 검색

**설정:**
- 클러스터: 3 Master + 3 Data Node
- 샤딩: 인덱스당 5 샤드
- 리플리카: 1개

### 2.2 프론트엔드

#### 프레임워크
**✅ 결정: React 18.3+ with TypeScript**

**선택 이유:**
- 컴포넌트 기반 개발
- 풍부한 생태계
- 타입 안정성 (TypeScript)
- 백엔드와 타입 공유 가능
- 최신 보안 패치 포함

**보안 이슈 대응:**
- 최신 버전 사용 (React 18.3+)
- 정기적 업데이트 및 npm audit
- 보안 모범 사례 준수 (XSS, CSRF 방지)

**플랫폼 전략:**
- **웹 (MVP)**: React + PWA
- **모바일 (확장)**: Flutter (iOS, Android, Web 모두 지원)
- **데스크톱 (선택)**: Flutter Desktop 또는 Electron

**Flutter 선택 이유:**
- 성능 우수 (네이티브에 가까운 성능)
- UI 일관성 (모든 플랫폼에서 동일한 UI)
- 크로스 플랫폼 (iOS, Android, Web, Desktop 모두 지원)
- 실시간 통신 기능 충분 (socket_io_client)
- Hot Reload로 빠른 개발

**참고 문서:**
- [프론트엔드 플랫폼 전략 상세 가이드](./FRONTEND-PLATFORM-STRATEGY.md)
- [Flutter vs React Native 상세 비교](./FLUTTER-VS-REACT-NATIVE.md)

#### 상태 관리
**✅ 결정: Zustand + React Query**

**선택 이유:**
- **Zustand**: 클라이언트 상태 관리 (가볍고 빠름)
- **React Query**: 서버 상태 관리 (캐싱, 동기화)

**대안 고려:**
- ⚠️ Redux Toolkit: 복잡도 증가, MVP에는 과함

#### 실시간 통신
**✅ 결정: Socket.io-client (초기) → Native WebSocket (최적화 시)**

**선택 이유:**
- **초기**: Socket.io-client로 빠른 개발 (자동 재연결, 폴백)
- **최적화 시**: Native WebSocket으로 전환 (더 낮은 지연)

#### UI 라이브러리
**✅ 결정: Tailwind CSS + Headless UI**

**선택 이유:**
- 유틸리티 기반으로 빠른 개발
- 번들 크기 최소화
- 커스터마이징 용이
- Headless UI로 접근성 확보

**대안 고려:**
- ⚠️ Material-UI: 번들 크기 큼, 커스터마이징 어려움

#### 빌드 도구
**✅ 결정: Vite**

**선택 이유:**
- 빠른 개발 서버
- 빠른 빌드 속도
- 최적화된 번들

### 2.3 인프라

#### 컨테이너화
**✅ 결정: Docker + Kubernetes**

**선택 이유:**
- 표준화된 배포
- 자동 스케일링
- 롤링 업데이트
- 서비스 디스커버리

#### 로드 밸런서
**✅ 결정: AWS Application Load Balancer (ALB) + Network Load Balancer (NLB)**

**선택 이유:**
- **ALB**: HTTP/HTTPS 트래픽 (REST API)
- **NLB**: TCP/UDP 트래픽 (WebSocket)
- 자동 스케일링
- Health Check

**대안:**
- NGINX (온프레미스)
- CloudFlare Load Balancer

#### API Gateway
**✅ 결정: Kong 또는 AWS API Gateway**

**선택 이유:**
- 라우팅 및 로드 밸런싱
- 인증/인가 (JWT 검증)
- Rate Limiting
- 요청/응답 변환

#### 모니터링
**✅ 결정: Prometheus + Grafana**

**선택 이유:**
- 오픈소스
- 풍부한 메트릭
- 커스텀 대시보드
- 알림 연동

**추가 도구:**
- **APM**: New Relic 또는 Datadog
- **로깅**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **분산 추적**: Jaeger

#### 배포 플랫폼
**✅ 결정: AWS EKS (Elastic Kubernetes Service)**

**선택 이유:**
- 관리형 Kubernetes
- 자동 스케일링
- 고가용성
- 통합 모니터링

**대안:**
- Google GKE
- Azure AKS
- 온프레미스 Kubernetes

### 2.4 개발 도구

#### 빌드 도구
**✅ 결정: Gradle**

**선택 이유:**
- Kotlin DSL 지원
- 빠른 빌드
- 의존성 관리

**대안:**
- Maven (기존 프로젝트 호환성)

#### 코드 품질
**✅ 결정:**
- **Linting**: Checkstyle, SpotBugs
- **포맷팅**: Google Java Format
- **정적 분석**: SonarQube

#### 테스팅
**✅ 결정:**
- **단위 테스트**: JUnit 5 + Mockito
- **통합 테스트**: Spring Boot Test
- **E2E 테스트**: Playwright (프론트엔드)

#### CI/CD
**✅ 결정: GitHub Actions**

**선택 이유:**
- GitHub 통합
- 무료 (Public repo)
- 풍부한 액션

**파이프라인:**
1. 코드 품질 검사
2. 테스트 실행
3. 빌드
4. Docker 이미지 생성
5. Kubernetes 배포

---

## 3. 아키텍처 다이어그램

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
│  6 nodes     │  │  6 nodes       │  │  6 nodes  │
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
                    │  6 nodes     │
                    └──────────────┘
```

---

## 4. 성능 최적화 전략

### 4.1 WebSocket 서버 최적화

**Netty 설정:**
- **이벤트 루프 스레드**: CPU 코어 수
- **워커 스레드**: CPU 코어 수 * 2
- **메모리**: Direct Memory 4GB+
- **연결당 메모리**: ~10KB

**예상 성능:**
- 단일 서버: 100만+ 동시 연결
- 클러스터 (10대): 1,000만+ 동시 연결

### 4.2 데이터베이스 최적화

**인덱스 전략:**
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

**파티셔닝:**
- Messages 테이블: 월별 파티셔닝
- 오래된 파티션: S3로 아카이빙

**Connection Pooling:**
- HikariCP: 최대 50 connections per instance
- Read Replica: 읽기 쿼리만 라우팅

### 4.3 캐싱 전략

**캐시 계층:**
1. **L1 (Local Cache)**: Caffeine (애플리케이션 내부)
   - TTL: 1분
   - 용도: 자주 조회되는 데이터

2. **L2 (Redis Cache)**: Redis Cluster
   - TTL: 1시간
   - 용도: 사용자 정보, 친구 목록

**캐시 무효화:**
- Write-Through: 쓰기 시 캐시 업데이트
- TTL 기반 자동 만료

### 4.4 메시지 처리 최적화

**비동기 처리:**
1. 메시지 수신 → Redis Streams에 즉시 저장
2. WebSocket으로 즉시 전달
3. 백그라운드에서 PostgreSQL에 비동기 저장

**배치 처리:**
- 메시지 저장: 100개씩 배치
- Redis Pipeline 사용

---

## 5. 확장성 계획

### 5.1 수평 확장

**WebSocket 서버:**
- 초기: 3대
- 목표: 10대 (100만 동시 접속자)
- 확장: 50대 (1,000만 동시 접속자)

**API 서버:**
- 초기: 5 pods
- 목표: 20 pods
- Auto Scaling: CPU 70% 기준

**데이터베이스:**
- 초기: Primary + 2 Read Replicas
- 목표: Primary + 5 Read Replicas
- 샤딩: 사용자 ID 기반 (필요 시)

### 5.2 수직 확장

**서버 사양:**
- **WebSocket 서버**: 16 vCPU, 32GB RAM
- **API 서버**: 8 vCPU, 16GB RAM
- **PostgreSQL**: 32 vCPU, 128GB RAM

---

## 6. 비용 예상

### 6.1 인프라 비용 (월간, 대략적)

**AWS 기준:**
- EKS 클러스터: $73
- EC2 인스턴스 (WebSocket): $1,200 (10대)
- EC2 인스턴스 (API): $800 (20 pods)
- RDS PostgreSQL: $500 (Primary + Replicas)
- ElastiCache Redis: $600 (6 nodes)
- Elasticsearch: $400 (6 nodes)
- ALB/NLB: $50
- 데이터 전송: $200

**총계: 약 $3,800/월**

**최적화 후:**
- Reserved Instances: 30% 할인 → $2,700/월
- Spot Instances: 70% 할인 → $1,900/월

---

## 7. 마이그레이션 로드맵

### Phase 1: MVP (1-3개월)
- Spring Boot 기반 REST API
- 기본 WebSocket (Spring WebSocket)
- PostgreSQL 단일 인스턴스
- Redis 단일 인스턴스

### Phase 2: 최적화 (3-6개월)
- Netty WebSocket 서버 도입
- Redis Cluster 구성
- Read Replica 추가
- 캐싱 전략 도입

### Phase 3: 확장 (6-12개월)
- Kubernetes 마이그레이션
- API Gateway 도입
- 모니터링 구축
- Auto Scaling 설정
- Flutter 모바일 앱 개발 (iOS, Android)

### Phase 4: 대규모 (12개월+)
- 데이터베이스 샤딩
- Kafka 도입
- Elasticsearch 도입
- 마이크로서비스 전환
- Flutter Web/Desktop 지원 (선택)

---

## 8. 리스크 및 대응 방안

### 8.1 기술적 리스크

**리스크: Netty 학습 곡선**
- 대응: Spring WebSocket으로 시작, 점진적 전환

**리스크: 데이터베이스 병목**
- 대응: Read Replica 추가, 캐싱 강화

**리스크: 메모리 부족**
- 대응: 모니터링 강화, 자동 스케일링

### 8.2 운영 리스크

**리스크: 장애 대응**
- 대응: 모니터링 및 알림, 자동 복구

**리스크: 비용 증가**
- 대응: Auto Scaling, Reserved Instances

---

## 9. 최종 기술 스택 요약

### 백엔드
- **언어**: Java 25 LTS
- **프레임워크**: Spring Boot 3.2+
- **실시간 통신**: Netty (WebSocket)
- **데이터베이스**: PostgreSQL 15+ (Primary + Read Replicas)
- **ORM**: Spring Data JPA + QueryDSL
- **캐싱**: Redis Cluster 7.0+
- **메시지 큐**: Redis Streams → Kafka
- **검색**: Elasticsearch 8.0+

### 프론트엔드
- **웹 (MVP)**: React 18.3+ + TypeScript
  - 상태 관리: Zustand + React Query
  - 실시간 통신: Socket.io-client → Native WebSocket
  - 스타일링: Tailwind CSS + Headless UI
  - 빌드 도구: Vite

- **모바일 (확장)**: Flutter 3.0+
  - 언어: Dart
  - 상태 관리: Riverpod 또는 Bloc
  - 실시간 통신: socket_io_client
  - UI: Material Design / Cupertino
  - 크로스 플랫폼: iOS, Android, Web, Desktop

### 인프라
- **컨테이너**: Docker
- **오케스트레이션**: Kubernetes (AWS EKS)
- **로드 밸런서**: AWS ALB/NLB
- **API Gateway**: Kong
- **모니터링**: Prometheus + Grafana
- **로깅**: ELK Stack
- **분산 추적**: Jaeger

### 개발 도구
- **빌드**: Gradle
- **테스팅**: JUnit 5 + Mockito
- **CI/CD**: GitHub Actions
- **코드 품질**: Checkstyle, SonarQube

---

**문서 작성일**: 2024년
**최종 수정일**: 2024년
**작성자**: [작성자명]

