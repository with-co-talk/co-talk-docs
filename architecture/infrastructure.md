---
layout: default
title: Infrastructure
description: Co-Talk 인프라 및 배포
---

# 인프라 및 배포

[← 아키텍처 개요](./index)

---

## 목차

- [컨테이너화](#컨테이너화)
- [호스팅](#호스팅)
- [보안](#보안)
- [성능 최적화](#성능-최적화)
- [모니터링](#모니터링)
- [배포 전략](#배포-전략)
- [개발 환경](#개발-환경)

---

## 컨테이너화

### Docker
- 개발 환경 통일
- 배포 간소화

### CI/CD (선택)
- **GitHub Actions** - 자동 빌드 및 배포

---

## 호스팅 (초기)

### 백엔드
- **Railway**, **Render**, **Fly.io** (권장 - 간단한 배포)
- 또는 **AWS EC2**, **DigitalOcean**

### 프론트엔드
- **Vercel**, **Netlify** (권장 - 무료 티어)

### 데이터베이스
- **Supabase** (PostgreSQL 호스팅, 무료 티어)
- 또는 **Railway**, **Render** (PostgreSQL 포함)

---

## 보안

### 인증 및 인가
- JWT 토큰 기반 인증
- 토큰 만료 시간 설정 (예: 7일)
- Refresh Token 고려 (향후)
- Socket.io 연결 시 JWT 토큰 검증

### 데이터 보안
- 비밀번호 bcrypt 해싱 (salt rounds: 10)
- HTTPS/WSS 통신 필수
- SQL Injection 방지 (ORM 사용)
- XSS 방지 (입력 검증 및 이스케이프)
- CSRF 방지 (SameSite 쿠키)

### 입력 검증
- 모든 사용자 입력 검증
- 이메일 형식 검증
- 비밀번호 강도 검증
- 메시지 길이 제한 (1000자)

---

## 성능 최적화

### 데이터베이스
- **인덱스 최적화**
  - Users: email, nickname
  - Messages: (chat_room_id, created_at)
  - Friends: (user_id, friend_id)
- 페이지네이션 (메시지 히스토리)
- 쿼리 최적화 (N+1 문제 방지)

### 실시간 통신
- Socket.io Room 활용 (채팅방별)
- 메시지 큐잉 (고부하 시)
- 연결 풀 관리

### 프론트엔드
- 메시지 가상화 (Virtual Scrolling)
- 이미지 지연 로딩 (향후)
- 코드 스플리팅
- 메모이제이션

---

## 모니터링

### 로깅
- 구조화된 로깅 (JSON 형식)
- 로그 레벨 관리 (DEBUG, INFO, WARN, ERROR)
- 에러 추적 (Sentry 고려)

### 모니터링
- API 응답 시간 모니터링
- WebSocket 연결 수 모니터링
- 데이터베이스 쿼리 성능 모니터링
- 서버 리소스 모니터링

---

## 배포 전략

### 초기 배포
- 단일 서버 구성
- 데이터베이스 백업 (일일 1회)
- 환경 변수 관리 (.env)

### 향후 배포
- Blue-Green 배포
- 무중단 배포
- 자동 스케일링

---

## 개발 환경

### 필수 도구
- Node.js (v20 LTS)
- PostgreSQL (v15+)
- Git
- Docker (선택)

### 코드 품질
- **ESLint** - 코드 린팅
- **Prettier** - 코드 포맷팅
- **Husky** - Git hooks
- **lint-staged** - 커밋 전 검사

### 테스팅 (향후)
- **Jest** - 단위 테스트
- **Supertest** - API 테스트
- **React Testing Library** - 컴포넌트 테스트

### 개발 워크플로우
1. 기능 브랜치 생성
2. 로컬 개발 및 테스트
3. PR 생성 및 코드 리뷰
4. 메인 브랜치 머지
5. 자동 배포 (CI/CD)

---

## 확장성 고려사항

### 수평 확장
- Socket.io Redis 어댑터 (다중 서버 지원)
- 로드 밸런서 (Sticky Session)
- 데이터베이스 읽기 복제본

### 향후 기능 확장
- 그룹 채팅 지원 가능한 구조
- 파일 전송 (멀티파트 업로드)
- 메시지 검색 (Elasticsearch 고려)
- 알림 시스템 (Redis Pub/Sub)

---

## 대규모 트래픽

100만+ 동시 접속자를 위한 확장 아키텍처

→ [대규모 트래픽 아키텍처](../ARCHITECTURE-SCALE)
