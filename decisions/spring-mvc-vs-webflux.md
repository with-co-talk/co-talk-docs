---
layout: default
title: Spring MVC vs WebFlux
description: Spring MVC 선택 이유 및 비교
parent: Architecture Decision Records
permalink: /decisions/spring-mvc-vs-webflux
---

# Spring MVC vs WebFlux 선택 가이드

[← 기술 결정 목록](./index)

**결정**: Spring MVC + Virtual Threads
**상태**: 승인됨

---

## 목차

- [핵심 차이점](#1-핵심-차이점)
- [성능 비교](#2-성능-비교)
- [개발 생산성](#3-개발-생산성)
- [최종 결정](#4-최종-결정)

---

## 1. 핵심 차이점

### 1.1 Spring MVC (서블릿 스택)

**특징:**
- **블로킹 I/O**: 요청당 스레드 할당 (기존 방식)
- **서블릿 기반**: Servlet API 사용
- **동기식 프로그래밍**: 명령형 코드 (Imperative)
- **스레드 모델**: 요청당 OS 스레드 1개

**의존성:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**코드 예시:**
```java
@RestController
public class UserController {
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id); // 블로킹 호출
    }
}
```

### 1.2 Spring WebFlux (리액티브 스택)

**특징:**
- **논블로킹 I/O**: 이벤트 루프 기반
- **Netty 기반**: 서블릿 없이 Netty 사용
- **리액티브 프로그래밍**: 함수형 코드 (Reactive)
- **스레드 모델**: 적은 수의 이벤트 루프 스레드

**의존성:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

**코드 예시:**
```java
@RestController
public class UserController {
    @GetMapping("/users/{id}")
    public Mono<User> getUser(@PathVariable Long id) {
        return userService.findById(id); // 리액티브 반환
    }
}
```

---

## 2. 성능 비교 (시대별)

### 2.1 Virtual Threads 이전 시대 (Java 17 이하)

#### Spring MVC (전통)
**문제점:**
- 요청당 OS 스레드 1개 할당
- 스레드 풀 크기 제한 (보통 200-500)
- I/O 대기 중에도 스레드 점유
- 동시 요청 수 = 스레드 풀 크기

**성능:**
- 동시 요청: ~500개 (스레드 풀 크기에 따라)
- 메모리: 스레드당 ~1MB
- **결론: 동시성에 큰 제약**

#### Spring WebFlux
**장점:**
- 적은 수의 이벤트 루프 스레드 (CPU 코어 수)
- 논블로킹 I/O로 높은 동시성
- 메모리 효율적

**성능:**
- 동시 요청: 수만~수십만 개 가능
- 메모리: 매우 효율적

**결론: Virtual Threads 이전에는 WebFlux가 확실히 유리했음**

### 2.2 Virtual Threads 이후 시대 (Java 21+)

#### Spring MVC + Virtual Threads
**혁신:**
- **Virtual Threads**: 경량 스레드 (수백만 개 가능)
- **기존 코드 유지**: Spring MVC 코드 그대로 사용
- **자동 변환**: 블로킹 코드가 Virtual Thread에서 실행

**성능:**
- 동시 요청: 수백만 개 가능 (Virtual Threads 덕분)
- 메모리: Virtual Thread당 ~1KB (기존 스레드 대비 1000배 효율적)
- 코드 복잡도: 기존과 동동 (변경 없음)

**결론: WebFlux와 성능 차이가 거의 없어짐**

### 2.3 현재 상황 (Java 25)

#### Spring MVC + Virtual Threads vs Spring WebFlux

**성능 비교:**
- **동시성**: 거의 동일 (둘 다 수백만 동시 요청 처리 가능)
- **처리량**: 거의 동일 (WebFlux가 약간 더 효율적일 수 있으나 차이 미미)
- **메모리**: 거의 동일 (둘 다 효율적)

**차이점:**
- **코드 복잡도**: Spring MVC가 훨씬 낮음
- **학습 곡선**: Spring MVC가 훨씬 낮음
- **생태계**: Spring MVC가 더 넓음 (기존 라이브러리 모두 사용 가능)

**결론:**
- **성능상 차이는 거의 없음**
- **개발 생산성 측면에서 Spring MVC가 더 유리**
- **WebFlux는 완전한 리액티브 스택일 때만 약간의 이점**

**설정:**
```properties
# application.properties
spring.threads.virtual.enabled=true
```

---

## 3. Co-Talk 프로젝트 선택

### 3.1 최종 결정: Spring MVC + Virtual Threads

**선택 이유:**

1. **Java 25 Virtual Threads 활용**
   - Virtual Threads로 WebFlux 수준의 동시성 확보
   - 기존 코드 패턴 유지 가능

2. **개발 생산성**
   - 명령형 코드 (기존 방식)
   - 학습 곡선 낮음
   - 디버깅 용이

3. **생태계 호환성**
   - Spring Data JPA (블로킹) 사용 가능
   - 기존 라이브러리 모두 호환
   - WebFlux는 리액티브 라이브러리 필요

4. **실제 성능**
   - Virtual Threads로 충분한 성능
   - 대부분의 경우 WebFlux와 유사한 처리량

### 3.2 WebFlux를 고려할 경우

**언제 WebFlux가 유리한가:**

1. **완전한 리액티브 스택**
   - R2DBC (리액티브 데이터베이스 드라이버)
   - 리액티브 Redis 클라이언트
   - 모든 라이브러리가 리액티브

2. **초고성능이 필수**
   - WebFlux가 약간 더 효율적 (하지만 차이 미미)

3. **기존 리액티브 코드베이스**
   - 이미 리액티브로 구축된 경우

**단점:**
- 학습 곡선 높음
- 코드 복잡도 증가
- 디버깅 어려움
- 모든 라이브러리 리액티브 필요

---

## 4. 성능 비교 (대규모 트래픽)

### 4.1 시대별 동시 연결 처리 능력

#### Virtual Threads 이전 (Java 17 이하)

| 방식 | 동시 연결 | 메모리 사용 | 코드 복잡도 | 평가 |
|------|----------|------------|------------|------|
| Spring MVC (전통) | ~500 | 높음 | 낮음 | ❌ 제약 많음 |
| Spring WebFlux | 수십만 | 매우 낮음 | 높음 | ✅ 유리 |

**결론: WebFlux가 확실히 유리했음**

#### Virtual Threads 이후 (Java 21+)

| 방식 | 동시 연결 | 메모리 사용 | 코드 복잡도 | 평가 |
|------|----------|------------|------------|------|
| Spring MVC + Virtual Threads | 수백만 | 낮음 | 낮음 | ✅ 권장 |
| Spring WebFlux | 수백만 | 매우 낮음 | 높음 | ⚠️ 선택적 |

**결론: 성능 차이가 거의 없어짐**

### 4.2 실제 벤치마크 (참고)

**10,000 TPS 처리:**
- Spring MVC + Virtual Threads: ✅ 충분 (성능 우수)
- Spring WebFlux: ✅ 충분 (성능 우수)
- Spring MVC (전통, Java 17 이하): ❌ 부족

**성능 차이:**
- Spring MVC + Virtual Threads vs WebFlux: **거의 동일 (차이 미미)**
- WebFlux가 약간 더 효율적일 수 있으나 **실용적 차이는 거의 없음**

**결론:**
- Virtual Threads가 있으면 Spring MVC로도 충분
- 성능상 차이는 거의 없음
- 개발 생산성 측면에서 Spring MVC가 더 유리

---

## 5. 구현 전략

### 5.1 Spring MVC + Virtual Threads 설정

**의존성:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**설정:**
```properties
# application.properties
spring.threads.virtual.enabled=true
```

**코드:**
```java
@RestController
public class ChatController {
    @Autowired
    private ChatService chatService;
    
    @GetMapping("/chats")
    public List<ChatRoom> getChatRooms() {
        // 기존 코드 그대로 사용
        // Virtual Thread에서 자동 실행
        return chatService.findAll();
    }
}
```

### 5.2 WebSocket은 별도 처리

**WebSocket 서버:**
- Netty 기반 독립 서버 (최고 성능)
- 또는 Spring WebSocket (Spring MVC와 호환)

**이유:**
- WebSocket은 실시간 통신이므로 최적화 필요
- REST API는 Spring MVC + Virtual Threads로 충분

---

## 6. 마이그레이션 전략

### 6.1 초기 단계
- Spring MVC + Virtual Threads로 시작
- 기존 코드 패턴 유지

### 6.2 성능 문제 발생 시
1. **먼저 확인:**
   - Virtual Threads 활성화 확인
   - 데이터베이스 쿼리 최적화
   - 캐싱 전략

2. **그래도 부족하면:**
   - WebFlux 고려
   - 또는 Netty 직접 사용

---

## 7. 최종 권장사항

### ✅ Spring MVC + Virtual Threads (강력 권장)

**이유:**
- **성능**: Java 25 Virtual Threads로 WebFlux와 거의 동일한 성능
- **개발 생산성**: 기존 코드 패턴 유지, 학습 곡선 낮음
- **생태계**: 기존 라이브러리 모두 활용 가능
- **대규모 트래픽**: 수백만 동시 요청 처리 가능

**핵심:**
- Virtual Threads 이전에는 WebFlux가 확실히 유리했음
- **지금은 성능상 차이가 거의 없음**
- 개발 생산성 측면에서 Spring MVC가 더 유리

### ⚠️ Spring WebFlux (선택적, 비권장)

**고려 시점:**
- 완전한 리액티브 스택 구축 가능할 때
- 팀이 이미 리액티브 프로그래밍에 익숙할 때
- 성능보다 리액티브 패러다임이 중요한 경우

**단점:**
- 성능상 큰 이점 없음 (Virtual Threads로 충분)
- 코드 복잡도 증가
- 학습 곡선 높음

---

## 8. 결론

### 핵심 요약

**Virtual Threads 이전 (Java 17 이하):**
- ✅ **WebFlux가 확실히 유리했음**
- Spring MVC는 동시성에 큰 제약

**Virtual Threads 이후 (Java 21+):**
- ✅ **성능상 차이가 거의 없어짐**
- Spring MVC + Virtual Threads = WebFlux 수준의 성능
- 개발 생산성 측면에서 Spring MVC가 더 유리

### Co-Talk 프로젝트 선택

**REST API**: Spring MVC + Virtual Threads ✅
**WebSocket**: Netty 기반 독립 서버 ✅

**이유:**
- Virtual Threads로 WebFlux와 거의 동일한 성능
- 개발 생산성과 유지보수성 우수
- 대규모 트래픽 처리 가능
- 기존 생태계 활용 가능
---

**문서 작성일**: 2024년
**최종 수정일**: 2024년


