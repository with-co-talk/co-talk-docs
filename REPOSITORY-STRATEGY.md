---
layout: default
title: 레포지토리 전략
description: Co-Talk 프로젝트 레포지토리 구조 전략
---

# 레포지토리 전략
## Co-Talk 프로젝트 - 레포지토리 구조 결정

**결정일**: 2024년  
**상태**: 승인됨  
**결정자**: [결정자명]

---

## 1. 결정 배경

Co-Talk 프로젝트는 모듈형 모놀리식에서 점진적 MSA 전환을 계획하고 있습니다. 레포지토리 구조는 아키텍처 전환과 함께 고려해야 합니다.

---

## 2. 레포지토리 전략 옵션

### 2.1 Monorepo (단일 레포지토리)

#### 구조
```
co-talk/
├── services/
│   ├── auth-service/
│   ├── user-service/
│   ├── friend-service/
│   ├── chat-service/
│   ├── message-service/
│   └── websocket-service/
├── docs/              # 문서
├── shared/            # 공통 라이브러리
│   ├── common/
│   └── domain/
└── infrastructure/     # 인프라 코드
```

#### 장점
1. **코드 공유 용이**
   - 공통 도메인 모델 공유
   - 공통 유틸리티 라이브러리
   - 타입 정의 공유

2. **전체 빌드/테스트**
   - 모든 서비스 통합 테스트
   - 변경사항 영향 범위 파악 용이
   - CI/CD 파이프라인 단순

3. **버전 관리 단순**
   - 모든 서비스가 같은 버전
   - 의존성 관리 용이

4. **리팩토링 용이**
   - 서비스 간 코드 이동 쉬움
   - 대규모 리팩토링 지원

#### 단점
1. **레포지토리 크기 증가**
   - 클론 시간 증가
   - IDE 성능 저하 가능

2. **권한 관리 복잡**
   - 서비스별 접근 권한 제어 어려움
   - 모든 개발자가 전체 코드 접근

3. **배포 복잡도**
   - 변경된 서비스만 배포하는 로직 필요
   - 빌드 시간 증가

---

### 2.2 Multi-repo (서비스별 레포지토리)

#### 구조
```
co-talk-auth-service/
co-talk-user-service/
co-talk-friend-service/
co-talk-chat-service/
co-talk-message-service/
co-talk-websocket-service/
co-talk-docs/          # 문서 전용
co-talk-shared/        # 공통 라이브러리 (선택)
```

#### 장점
1. **서비스별 독립 관리**
   - 서비스별 독립적인 버전 관리
   - 서비스별 독립 배포
   - 서비스별 CI/CD 파이프라인

2. **권한 분리 용이**
   - 서비스별 접근 권한 제어
   - 팀별 레포지토리 소유권

3. **레포지토리 크기 작음**
   - 빠른 클론
   - IDE 성능 우수

4. **기술 스택 다양성**
   - 서비스별 다른 기술 스택 사용 가능
   - 서비스별 독립적 업그레이드

#### 단점
1. **코드 공유 어려움**
   - 공통 코드를 패키지로 배포 필요
   - 버전 관리 복잡

2. **버전 관리 복잡**
   - 서비스 간 의존성 관리 어려움
   - 호환성 문제 가능

3. **통합 테스트 어려움**
   - 여러 레포지토리 동기화 필요
   - 로컬 개발 환경 구성 복잡

---

## 3. Co-Talk 프로젝트 분석

### 3.1 프로젝트 단계별 요구사항

#### Phase 1-2: 모듈형 모놀리식 (1-6개월)
- **구조**: 단일 애플리케이션
- **레포지토리**: 단일 레포지토리
- **팀 규모**: 소규모 (2-5명)

#### Phase 3: 하이브리드 (6-12개월)
- **구조**: WebSocket 독립, 나머지 모놀리식
- **레포지토리**: 2개 레포지토리 (모놀리식 + WebSocket)
- **팀 규모**: 중규모 (5-10명)

#### Phase 4: MSA (12개월+)
- **구조**: 완전한 마이크로서비스
- **레포지토리**: Multi-repo 또는 Monorepo
- **팀 규모**: 대규모 (10명+)

---

## 4. 최종 결정

### ✅ **결정: 단계별 레포지토리 전략**

### 4.1 Phase 1-2: Monorepo (단일 레포지토리)

**구조:**
```
co-talk/
├── backend/           # Spring Boot 모놀리식
│   ├── auth/
│   ├── user/
│   ├── friend/
│   ├── chat/
│   ├── message/
│   └── websocket/
├── frontend/          # React 웹앱
├── docs/              # 문서 (현재)
└── infrastructure/     # Docker, K8s 설정
```

**선택 이유:**
1. 초기 단계에서 빠른 개발
2. 코드 공유 용이
3. 통합 테스트 용이
4. 단일 배포 단위

### 4.2 Phase 3: 하이브리드 레포지토리

**구조:**
```
co-talk/               # 모놀리식 (Auth, User, Friend, Chat, Message)
├── backend/
└── frontend/

co-talk-websocket/     # WebSocket 서버 (독립)
└── websocket-server/

co-talk-docs/          # 문서 (분리 권장)
└── docs/
```

**선택 이유:**
1. WebSocket 서버는 독립 확장 필요
2. 문서는 모든 서비스에 공통이므로 분리
3. 점진적 분리로 리스크 최소화

### 4.3 Phase 4: Multi-repo (MSA 전환)

**구조:**
```
co-talk-auth-service/
co-talk-user-service/
co-talk-friend-service/
co-talk-chat-service/
co-talk-message-service/
co-talk-websocket-service/
co-talk-frontend/
co-talk-docs/          # 문서 (필수 분리)
co-talk-shared/        # 공통 라이브러리 (선택)
```

**선택 이유:**
1. 서비스별 독립 배포
2. 팀별 독립 개발
3. 서비스별 기술 스택 선택 가능
4. 권한 관리 용이

---

## 5. 문서 레포지토리 분리 시점

### ✅ **권장: Phase 3 (WebSocket 분리 시점) 또는 초기부터**

**이유:**
1. **문서는 모든 서비스에 공통**
   - MSA 전환과 무관하게 분리 가능
   - 초기부터 분리해도 문제없음

2. **독립적 관리 용이**
   - 문서 팀과 개발 팀 분리 가능
   - 문서 업데이트가 코드 배포와 무관

3. **URL 단순화**
   - `with-co-talk.github.io` (루트)
   - 또는 `docs.co-talk.com` (커스텀 도메인)

**현재 상태:**
- 레포지토리: `co-talk` (프로젝트 레포지토리)
- 문서 경로: `docs/` 폴더
- 배포 URL: `https://with-co-talk.github.io/co-talk/`

**분리 옵션:**
1. **지금 분리**: `co-talk-docs` 레포지토리 생성
2. **Phase 3 분리**: WebSocket 분리 시 함께 분리
3. **현재 유지**: Phase 4 MSA 전환 시 분리

---

## 6. 레포지토리 전환 계획

### 6.1 Monorepo → Multi-repo 전환

#### 전환 단계
1. **WebSocket 서버 분리 (Phase 3)**
   - `co-talk-websocket` 레포지토리 생성
   - WebSocket 모듈 이동
   - 독립 배포 설정

2. **문서 분리 (Phase 3 또는 초기)**
   - `co-talk-docs` 레포지토리 생성
   - `docs/` 폴더 이동
   - GitHub Pages 설정

3. **나머지 서비스 분리 (Phase 4)**
   - 서비스별 레포지토리 생성
   - 모듈별 이동
   - API Gateway 통합

#### 전환 시 주의사항
- **Git 히스토리 보존**
  - `git filter-branch` 또는 `git subtree`
  - 또는 새 레포지토리 생성 후 파일 이동

- **의존성 관리**
  - 공통 라이브러리는 패키지로 배포
  - Maven/Gradle 저장소 활용

- **CI/CD 파이프라인**
  - 각 레포지토리별 파이프라인 설정
  - 통합 테스트는 별도 레포지토리

---

## 7. 문서 레포지토리 분리 권장사항

### ✅ **초기부터 분리 권장**

**이유:**
1. 문서는 코드와 독립적으로 관리 가능
2. 문서 업데이트가 코드 배포와 무관
3. URL 단순화 (`with-co-talk.github.io`)
4. 문서 팀과 개발 팀 분리 용이

**구조:**
```
co-talk-docs/          # 문서 전용 레포지토리
├── docs/              # Jekyll 소스
├── _config.yml
└── README.md

co-talk/               # 프로젝트 레포지토리
├── backend/
├── frontend/
└── README.md          # 문서 링크 포함
```

**장점:**
- 문서 URL: `https://with-co-talk.github.io/` (루트)
- 프로젝트 URL: `https://github.com/with-co-talk/co-talk`
- 명확한 역할 분리

---

## 8. 결론

### 레포지토리 전략 요약

| 단계 | 아키텍처 | 레포지토리 구조 | 문서 위치 |
|------|----------|----------------|-----------|
| Phase 1-2 | 모듈형 모놀리식 | Monorepo (단일) | `co-talk/docs/` 또는 분리 |
| Phase 3 | 하이브리드 | 2-3개 레포지토리 | `co-talk-docs/` (분리 권장) |
| Phase 4 | MSA | Multi-repo | `co-talk-docs/` (필수 분리) |

### 문서 레포지토리 분리 권장

**✅ 초기부터 분리 권장**
- 문서는 모든 서비스에 공통
- 독립적 관리 용이
- URL 단순화
- MSA 전환과 무관

**전환 시점:**
- **옵션 1**: 지금 분리 (권장)
- **옵션 2**: Phase 3 (WebSocket 분리 시)
- **옵션 3**: Phase 4 (MSA 전환 시)

---

**문서 작성일**: 2024년  
**최종 수정일**: 2024년  
**작성자**: [작성자명]  
**승인자**: [승인자명]

