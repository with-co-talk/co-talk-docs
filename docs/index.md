---
layout: default
title: Co-Talk Documentation
description: 대화에 집중한 커뮤니케이션 플랫폼 문서
---

<div style="text-align: center; margin: 3rem 0;">
  <h1 style="border: none; padding: 0; margin: 0;">
    <i class="fas fa-comments" style="color: #6366f1; font-size: 3rem;"></i>
  </h1>
  <h1 style="border: none; padding: 0; margin: 1rem 0;">
    Co-Talk Documentation
  </h1>
  <p style="font-size: 1.2rem; color: var(--text-light); max-width: 600px; margin: 0 auto;">
    대화에 집중한 커뮤니케이션 플랫폼 문서 사이트입니다.
  </p>
</div>

---

## 📚 문서 목록

### 제품 문서

<div class="doc-card">
  <h3><i class="fas fa-file-alt"></i> <a href="{{ site.baseurl }}/PRD">PRD (Product Requirements Document)</a></h3>
  <p>제품 요구사항 문서 - Co-Talk의 목표, 기능, 사용자 스토리를 정의합니다.</p>
</div>

### 기술 문서

#### 기술 스택

<div class="doc-card">
  <h3><i class="fas fa-layer-group"></i> <a href="{{ site.baseurl }}/TECH-STACK">Tech Stack <span class="badge">⭐</span></a></h3>
  <p>최종 기술 스택 결정서 - 성능 목표 기반으로 선택된 기술 스택을 상세히 설명합니다.</p>
</div>

#### 아키텍처

<div class="doc-card">
  <h3><i class="fas fa-sitemap"></i> <a href="{{ site.baseurl }}/ARCHITECTURE">Architecture</a></h3>
  <p>기술 스택 및 아키텍처 설계 문서 - MVP 단계의 시스템 구조를 설명합니다.</p>
</div>

<div class="doc-card">
  <h3><i class="fas fa-server"></i> <a href="{{ site.baseurl }}/ARCHITECTURE-SCALE">Architecture Scale</a></h3>
  <p>대규모 트래픽 아키텍처 문서 - 확장 가능한 시스템 설계를 다룹니다.</p>
</div>

#### 가이드

<div class="doc-card">
  <h3><i class="fas fa-chart-line"></i> <a href="{{ site.baseurl }}/DIAGRAM-GUIDE">다이어그램 & 수식 가이드 <span class="badge">🎨 NEW</span></a></h3>
  <p>Mermaid 다이어그램과 수학 공식 작성법 - 플로우차트, 시퀀스, ER 다이어그램 등을 문서에 추가할 수 있습니다.</p>
</div>

### 기술 결정 문서 (ADR)

#### 아키텍처

<div class="doc-card">
  <h3><i class="fas fa-sitemap"></i> <a href="{{ site.baseurl }}/MSA-VS-MONOLITH">MSA vs 모놀리식 아키텍처</a></h3>
  <p>아키텍처 패턴 선택 - 모듈형 모놀리식에서 점진적 MSA 전환 전략을 설명합니다.</p>
</div>

#### 백엔드

<div class="doc-card">
  <h3><i class="fas fa-database"></i> <a href="{{ site.baseurl }}/DATABASE-SELECTION">Database Selection</a></h3>
  <p>PostgreSQL 선택 이유 - 다른 데이터베이스와의 비교 및 선택 근거를 설명합니다.</p>
</div>

<div class="doc-card">
  <h3><i class="fas fa-code"></i> <a href="{{ site.baseurl }}/SPRING-MVC-VS-WEBFLUX">Spring MVC vs WebFlux</a></h3>
  <p>Spring MVC 선택 이유 - Virtual Threads를 고려한 프레임워크 선택 근거입니다.</p>
</div>

<div class="doc-card">
  <h3><i class="fas fa-stream"></i> <a href="{{ site.baseurl }}/REDIS-STREAMS-ORDERING">Redis Streams Ordering</a></h3>
  <p>Redis Streams 순서 보장 가이드 - 메시지 순서 보장 전략을 설명합니다.</p>
</div>

#### 프론트엔드

<div class="doc-card">
  <h3><i class="fas fa-mobile-alt"></i> <a href="{{ site.baseurl }}/FRONTEND-PLATFORM-STRATEGY">Frontend Platform Strategy</a></h3>
  <p>프론트엔드 플랫폼 전략 - 웹, 모바일, 데스크톱 플랫폼 전략을 설명합니다.</p>
</div>

<div class="doc-card">
  <h3><i class="fab fa-flutter"></i> <a href="{{ site.baseurl }}/FLUTTER-VS-REACT-NATIVE">Flutter vs React Native</a></h3>
  <p>Flutter 선택 이유 - React Native와의 상세 비교 및 선택 근거입니다.</p>
</div>

---

## 🚀 빠른 시작

<div class="alert alert-info">
  <strong><i class="fas fa-lightbulb"></i> 시작하기 전에:</strong>
  <ol style="margin: 0.5rem 0 0 1.5rem; padding: 0;">
    <li><strong>제품 이해</strong>: <a href="{{ site.baseurl }}/PRD">PRD</a>부터 시작하세요</li>
    <li><strong>기술 스택 확인</strong>: <a href="{{ site.baseurl }}/TECH-STACK">Tech Stack</a>에서 전체 기술 스택을 확인하세요</li>
    <li><strong>아키텍처 이해</strong>: <a href="{{ site.baseurl }}/ARCHITECTURE">Architecture</a>에서 시스템 구조를 파악하세요</li>
  </ol>
</div>

---

## 📝 문서 작성 가이드

<div class="doc-nav">
  <h3><i class="fas fa-book"></i> 문서 작성 규칙</h3>
  <ul>
    <li>✅ 모든 문서는 <strong>Markdown</strong> 형식으로 작성됩니다</li>
    <li>✅ 기술 결정은 <strong>ADR (Architecture Decision Records)</strong> 형식을 따릅니다</li>
    <li>✅ 문서는 <strong>Git</strong>을 통해 버전 관리됩니다</li>
    <li>✅ 변경사항은 <strong>Pull Request</strong>를 통해 검토됩니다</li>
  </ul>
</div>

---

<div style="text-align: center; margin: 4rem 0; padding: 2rem; background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%); border-radius: 12px; color: white;">
  <h2 style="color: white; border: none; margin: 0;">
    <i class="fas fa-heart"></i> Co-Talk
  </h2>
  <p style="font-size: 1.1rem; margin: 1rem 0 0 0; opacity: 0.9;">
    Meaningful conversations, meaningful connections.
  </p>
</div>
