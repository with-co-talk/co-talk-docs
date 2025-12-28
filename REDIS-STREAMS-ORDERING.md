---
layout: default
title: Redis Streams Ordering
description: 메시지 순서 보장 전략
---

# Redis Streams 순서 보장 가이드
## Co-Talk 프로젝트 메시지 큐 선택

---

## 1. Redis Streams 순서 보장 특성

### 1.1 기본 순서 보장

**Redis Streams는 기본적으로 순서를 보장합니다.**

**특징:**
- **추가 순서 보장**: 메시지가 추가된 순서대로 저장됨
- **읽기 순서 보장**: 컨슈머 그룹 내에서 순서대로 읽음
- **트래픽과 무관**: 낮은 트래픽이든 높은 트래픽이든 순서 보장

**예시:**
```
Producer가 메시지를 보낸 순서:
1. Message A
2. Message B  
3. Message C

Consumer가 읽는 순서:
1. Message A ✅
2. Message B ✅
3. Message C ✅
```

### 1.2 컨슈머 그룹 내 순서 보장

**단일 컨슈머 그룹:**
- 같은 컨슈머 그룹 내에서는 **완벽한 순서 보장**
- 각 메시지는 순서대로 처리됨

**예시:**
```java
// 컨슈머 그룹 "message-processors" 생성
XGROUP CREATE chat:messages message-processors 0

// 메시지 읽기 (순서 보장)
XREADGROUP GROUP message-processors consumer-1 COUNT 10 STREAMS chat:messages >
```

**결과:**
- 메시지 A, B, C가 순서대로 처리됨
- 트래픽이 높아도 순서 보장됨

### 1.3 여러 컨슈머가 있을 때

**같은 컨슈머 그룹 내 여러 컨슈머:**
- 각 컨슈머는 다른 메시지를 처리하지만
- **각 컨슈머가 받은 메시지는 순서대로 처리됨**

**예시:**
```
Stream: [A, B, C, D, E, F]

Consumer-1: A, C, E (순서대로 처리)
Consumer-2: B, D, F (순서대로 처리)
```

**주의사항:**
- 전체 순서는 보장되지 않음 (Consumer-1이 A를 처리하는 동안 Consumer-2가 B를 처리할 수 있음)
- 하지만 각 컨슈머 내에서는 순서 보장됨

---

## 2. Co-Talk 프로젝트에서의 활용

### 2.1 메시지 저장 시나리오

**사용 사례:**
- 메시지를 Redis Streams에 저장
- 백그라운드에서 PostgreSQL에 비동기 저장

**순서 보장 필요성:**
- 같은 채팅방의 메시지는 순서대로 저장되어야 함
- 메시지 A가 메시지 B보다 먼저 저장되어야 함

### 2.2 구현 방법

**방법 1: 채팅방별 Stream (권장)**

```java
// 각 채팅방마다 별도 Stream 생성
String streamKey = "chat:messages:" + chatRoomId;

// 메시지 추가 (순서 보장)
redisTemplate.opsForStream().add(streamKey, Map.of(
    "senderId", senderId,
    "content", content,
    "timestamp", System.currentTimeMillis()
));
```

**장점:**
- 채팅방별로 완벽한 순서 보장
- 병렬 처리 가능 (다른 채팅방은 독립적으로 처리)

**방법 2: 단일 Stream + Consumer Group**

```java
// 단일 Stream에 모든 메시지 저장
String streamKey = "chat:messages";

// 메시지 추가 (순서 보장)
redisTemplate.opsForStream().add(streamKey, Map.of(
    "chatRoomId", chatRoomId,
    "senderId", senderId,
    "content", content
));

// 컨슈머 그룹으로 읽기
XReadGroupArgs args = XReadGroupArgs.StreamOffset.from(streamKey, ">");
List<MapRecord<String, Object, Object>> messages = 
    redisTemplate.opsForStream().readGroup(consumerGroup, consumerName, args);
```

**장점:**
- 단일 Stream으로 관리 간단
- 컨슈머 그룹으로 부하 분산

**단점:**
- 같은 채팅방의 메시지도 다른 컨슈머가 처리할 수 있음
- 채팅방별 순서 보장이 어려울 수 있음

### 2.3 권장 구현: 채팅방별 Stream

**이유:**
- 채팅방별로 완벽한 순서 보장
- 병렬 처리 가능
- 확장성 우수

**구현:**
```java
@Service
public class MessageQueueService {
    
    public void enqueueMessage(String chatRoomId, Message message) {
        String streamKey = "chat:messages:" + chatRoomId;
        
        // 메시지 추가 (순서 보장)
        redisTemplate.opsForStream().add(streamKey, Map.of(
            "id", message.getId(),
            "senderId", message.getSenderId(),
            "content", message.getContent(),
            "timestamp", message.getTimestamp()
        ));
    }
    
    @Scheduled(fixedDelay = 1000)
    public void processMessages() {
        // 각 채팅방의 Stream에서 메시지 읽기
        // 순서대로 처리됨
    }
}
```

---

## 3. 트래픽 수준별 순서 보장

### 3.1 낮은 트래픽 (초기)

**특징:**
- 초당 메시지: ~100개
- 동시 채팅방: ~1,000개

**순서 보장:**
- ✅ **완벽하게 보장됨**
- Redis Streams의 기본 동작으로 충분
- 추가 설정 불필요

### 3.2 중간 트래픽

**특징:**
- 초당 메시지: ~1,000개
- 동시 채팅방: ~10,000개

**순서 보장:**
- ✅ **여전히 완벽하게 보장됨**
- 채팅방별 Stream 사용 시 문제 없음
- 컨슈머 그룹으로 부하 분산 가능

### 3.3 높은 트래픽

**특징:**
- 초당 메시지: ~10,000개
- 동시 채팅방: ~100,000개

**순서 보장:**
- ✅ **여전히 보장됨**
- 채팅방별 Stream 사용 시 각 채팅방 내 순서 보장
- 여러 컨슈머가 있어도 각 채팅방은 단일 컨슈머가 처리

**최적화:**
- 채팅방별 Stream 사용 (권장)
- 컨슈머 그룹으로 부하 분산
- 배치 처리로 성능 향상

---

## 4. Redis Streams vs Kafka 순서 보장 비교

### 4.1 순서 보장

| 특성 | Redis Streams | Kafka |
|------|-------------|-------|
| 기본 순서 보장 | ✅ 보장 | ✅ 보장 |
| 트래픽과 무관 | ✅ 보장 | ✅ 보장 |
| 파티션 내 순서 | ✅ 보장 | ✅ 보장 |
| 여러 파티션 | ⚠️ 주의 필요 | ✅ 파티션 키로 보장 |

### 4.2 차이점

**Redis Streams:**
- 단일 Stream: 순서 보장
- 여러 컨슈머: 각 컨슈머 내 순서 보장
- **채팅방별 Stream 사용 시 완벽한 순서 보장**

**Kafka:**
- 파티션 내 순서 보장
- 파티션 키로 같은 키는 같은 파티션으로
- **파티션 키 = 채팅방 ID로 설정 시 완벽한 순서 보장**

### 4.3 Co-Talk 프로젝트에서

**Redis Streams로 충분한 이유:**
- 채팅방별 Stream 사용 시 순서 보장됨
- 트래픽이 낮을 때뿐만 아니라 높을 때도 보장됨
- Kafka보다 설정 간단

**Kafka로 전환할 때:**
- 일일 메시지 10억+ 도달 시
- 더 강력한 내구성 필요 시
- 더 복잡한 스트림 처리 필요 시

---

## 5. 실제 구현 예시

### 5.1 메시지 저장 (순서 보장)

```java
@Service
public class MessageService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    public void sendMessage(String chatRoomId, Message message) {
        // 1. WebSocket으로 즉시 전달
        websocketService.sendToRoom(chatRoomId, message);
        
        // 2. Redis Streams에 저장 (순서 보장)
        String streamKey = "chat:messages:" + chatRoomId;
        redisTemplate.opsForStream().add(streamKey, Map.of(
            "id", message.getId(),
            "senderId", message.getSenderId(),
            "content", message.getContent(),
            "timestamp", message.getTimestamp()
        ));
    }
}
```

### 5.2 메시지 처리 (순서 보장)

```java
@Service
public class MessageProcessor {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private MessageRepository messageRepository;
    
    @Scheduled(fixedDelay = 1000)
    public void processMessages() {
        // 모든 채팅방의 Stream에서 메시지 읽기
        Set<String> chatRoomIds = getActiveChatRooms();
        
        for (String chatRoomId : chatRoomIds) {
            String streamKey = "chat:messages:" + chatRoomId;
            String consumerGroup = "message-processors";
            String consumerName = "consumer-1";
            
            // 메시지 읽기 (순서 보장)
            List<MapRecord<String, Object, Object>> messages = 
                redisTemplate.opsForStream().readGroup(
                    consumerGroup, 
                    consumerName,
                    XReadGroupArgs.StreamOffset.from(streamKey, ">")
                );
            
            // 순서대로 처리
            for (MapRecord<String, Object, Object> record : messages) {
                processMessage(record);
                // ACK 처리
                redisTemplate.opsForStream().acknowledge(
                    consumerGroup, 
                    streamKey, 
                    record.getId()
                );
            }
        }
    }
    
    private void processMessage(MapRecord<String, Object, Object> record) {
        // PostgreSQL에 저장 (순서대로)
        Message message = convertToMessage(record);
        messageRepository.save(message);
    }
}
```

---

## 6. 주의사항 및 최적화

### 6.1 주의사항

**여러 컨슈머가 있을 때:**
- 같은 채팅방의 메시지는 같은 컨슈머가 처리해야 함
- 채팅방별 Stream 사용 시 자동으로 해결됨

**컨슈머 그룹 설정:**
- 컨슈머 그룹을 올바르게 설정해야 순서 보장됨
- PENDING 메시지 처리 필요

### 6.2 최적화

**배치 처리:**
```java
// 여러 메시지를 한 번에 읽기
XReadGroupArgs args = XReadGroupArgs.StreamOffset.from(streamKey, ">")
    .count(100); // 최대 100개씩 읽기
```

**ACK 처리:**
```java
// 메시지 처리 후 즉시 ACK
redisTemplate.opsForStream().acknowledge(
    consumerGroup, 
    streamKey, 
    record.getId()
);
```

**PENDING 메시지 처리:**
```java
// 처리되지 않은 메시지 재처리
List<MapRecord<String, Object, Object>> pendingMessages = 
    redisTemplate.opsForStream().pending(
        consumerGroup, 
        streamKey, 
        consumerName
    );
```

---

## 7. 결론

### ✅ Redis Streams 순서 보장

**핵심:**
- **트래픽이 낮을 때뿐만 아니라 높을 때도 순서 보장됨**
- 채팅방별 Stream 사용 시 완벽한 순서 보장
- 컨슈머 그룹 내에서 순서대로 처리됨

**Co-Talk 프로젝트:**
- Redis Streams로 충분함
- 채팅방별 Stream 사용 권장
- Kafka 전환은 대용량 도달 시 고려

**최종 권장:**
- 초기: Redis Streams (채팅방별 Stream)
- 확장 시: 필요하면 Kafka로 전환 (하지만 Redis Streams도 충분할 수 있음)
