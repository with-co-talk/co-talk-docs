---
layout: default
title: Frontend Tech Stack
description: Co-Talk 프론트엔드 기술 스택
---

# 프론트엔드 기술 스택

[← 기술 스택 개요](./index)

---

## 목차

- [플랫폼 전략](#플랫폼-전략)
- [웹 프레임워크](#웹-프레임워크)
- [상태 관리](#상태-관리)
- [실시간 통신](#실시간-통신)
- [UI 라이브러리](#ui-라이브러리)
- [모바일 (Flutter)](#모바일-flutter)

---

## 플랫폼 전략

| 플랫폼 | 기술 | 단계 |
|--------|------|------|
| **웹** | React + PWA | MVP |
| **모바일** | Flutter (iOS, Android) | 확장 |
| **데스크톱** | Flutter Desktop 또는 Electron | 선택 |

**Flutter 선택 이유**
- 네이티브에 가까운 성능
- 모든 플랫폼에서 동일한 UI
- iOS, Android, Web, Desktop 모두 지원
- Hot Reload로 빠른 개발

→ [프론트엔드 플랫폼 전략 상세](../decisions/frontend-platform-strategy)
→ [Flutter vs React Native 비교](../decisions/flutter-vs-react-native)

---

## 웹 프레임워크

### ✅ 결정: React 18.3+ with TypeScript

**선택 이유**
- 컴포넌트 기반 개발
- 풍부한 생태계
- 타입 안정성 (TypeScript)
- 백엔드와 타입 공유 가능

**보안 대응**
- 최신 버전 사용 (React 18.3+)
- 정기적 업데이트 및 npm audit
- XSS, CSRF 방지 모범 사례 준수

---

## 상태 관리

### ✅ 결정: Zustand + React Query

| 라이브러리 | 용도 |
|-----------|------|
| **Zustand** | 클라이언트 상태 관리 (가볍고 빠름) |
| **React Query** | 서버 상태 관리 (캐싱, 동기화) |

**대안 비교**
| 기술 | 평가 |
|------|------|
| Redux Toolkit | ⚠️ 복잡도 증가, MVP에는 과함 |

---

## 실시간 통신

### ✅ 결정: Socket.io-client (초기) → Native WebSocket (최적화 시)

**전략**
| 단계 | 기술 | 이유 |
|------|------|------|
| **초기** | Socket.io-client | 빠른 개발, 자동 재연결 |
| **최적화** | Native WebSocket | 더 낮은 지연 |

---

## UI 라이브러리

### ✅ 결정: Tailwind CSS + Headless UI

**선택 이유**
- 유틸리티 기반으로 빠른 개발
- 번들 크기 최소화
- 커스터마이징 용이
- Headless UI로 접근성 확보

**대안 비교**
| 기술 | 평가 |
|------|------|
| Material-UI | ⚠️ 번들 크기 큼, 커스터마이징 어려움 |

### 빌드 도구

**✅ 결정: Vite**
- 빠른 개발 서버
- 빠른 빌드 속도
- 최적화된 번들

---

## 모바일 (Flutter)

### ✅ 결정: Flutter 3.0+

**기술 스택**
| 항목 | 선택 |
|------|------|
| 언어 | Dart |
| 상태 관리 | Riverpod 또는 Bloc |
| 실시간 통신 | socket_io_client |
| UI | Material Design / Cupertino |
| 플랫폼 | iOS, Android, Web, Desktop |

**장점**
- 단일 코드베이스로 모든 플랫폼 지원
- 네이티브에 가까운 성능
- Hot Reload로 빠른 개발
- 풍부한 위젯 라이브러리

---

## 웹 기술 요약

```
React 18.3+ + TypeScript
├── 상태 관리: Zustand + React Query
├── 실시간: Socket.io-client → Native WebSocket
├── 스타일링: Tailwind CSS + Headless UI
└── 빌드: Vite
```

---

## 다음 문서

→ [인프라 기술](./infrastructure)
