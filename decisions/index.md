---
layout: default
title: Architecture Decision Records
description: Co-Talk 기술 결정 문서 (ADR)
permalink: /decisions/
---

# 기술 결정 문서 (ADR)

Co-Talk 프로젝트의 주요 기술 결정과 그 배경을 기록한 문서입니다.

---

## 아키텍처 결정

| 문서 | 결정 | 설명 |
|------|------|------|
| [MSA vs Monolith](./msa-vs-monolith) | **모듈형 모놀리식** | 초기 개발 속도 우선, 점진적 MSA 전환 |

---

## 백엔드 결정

| 문서 | 결정 | 설명 |
|------|------|------|
| [Database Selection](./database-selection) | **PostgreSQL** | 관계형 데이터, ACID, JSONB 지원 |
| [Spring MVC vs WebFlux](./spring-mvc-vs-webflux) | **Spring MVC** | Virtual Threads로 충분한 성능, 개발 생산성 |
| [Redis Streams Ordering](./redis-streams-ordering) | **Redis Streams** | 메시지 순서 보장, 초기 단순함 |

---

## 프론트엔드 결정

| 문서 | 결정 | 설명 |
|------|------|------|
| [Frontend Platform Strategy](./frontend-platform-strategy) | **React + Flutter** | 웹 MVP, 모바일 확장 |
| [Flutter vs React Native](./flutter-vs-react-native) | **Flutter** | 성능, UI 일관성, 크로스 플랫폼 |

---

## ADR 작성 원칙

각 ADR은 다음 구조를 따릅니다:

1. **컨텍스트**: 결정이 필요한 상황
2. **고려한 옵션**: 검토한 대안들
3. **결정**: 최종 선택
4. **근거**: 선택 이유
5. **결과**: 예상되는 영향
