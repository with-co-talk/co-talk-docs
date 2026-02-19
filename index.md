---
layout: default
title: Co-Talk Documentation
description: 대화에 집중한 커뮤니케이션 플랫폼 문서
---

<div class="home-header">
  <h1>Co-Talk Documentation</h1>
  <p>대화에 집중한 커뮤니케이션 플랫폼 문서 사이트입니다.</p>
</div>

<div class="getting-started">
  <h3>시작하기</h3>
  <ol>
    <li><a href="{{ site.baseurl }}/PRD">PRD</a>에서 제품 요구사항을 확인하세요</li>
    <li><a href="{{ site.baseurl }}/tech-stack/">Tech Stack</a>에서 기술 스택을 파악하세요</li>
    <li><a href="{{ site.baseurl }}/architecture/">Architecture</a>에서 시스템 구조를 이해하세요</li>
  </ol>
</div>

<div class="doc-section">
  <h2>제품 문서</h2>
  <ul class="doc-list">
    <li>
      <a href="{{ site.baseurl }}/PRD">
        <span class="doc-title">PRD</span>
        <span class="doc-desc">제품 요구사항 문서</span>
      </a>
    </li>
  </ul>
</div>

<div class="doc-section">
  <h2>API Reference</h2>
  <ul class="doc-list">
    <li>
      <a href="{{ site.baseurl }}/api/">
        <span class="doc-title">API Overview</span>
        <span class="doc-desc">REST API 문서 (68개 엔드포인트)</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/api/API_AUTH">
        <span class="doc-title">└ Auth</span>
        <span class="doc-desc">인증, OAuth, 비밀번호, 계정</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/api/API_USER">
        <span class="doc-title">└ User</span>
        <span class="doc-desc">사용자 정보, 검색, 프로필</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/api/API_FRIEND">
        <span class="doc-title">└ Friend</span>
        <span class="doc-desc">친구 요청, 친구 관리, 차단</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/api/API_CHATROOM">
        <span class="doc-title">└ ChatRoom</span>
        <span class="doc-desc">채팅방 생성, 관리, 멤버</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/api/API_MESSAGE">
        <span class="doc-title">└ Message</span>
        <span class="doc-desc">메시지, 반응, 검색</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/api/API_NOTIFICATION">
        <span class="doc-title">└ Notification</span>
        <span class="doc-desc">알림, 디바이스, 파일 업로드</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/api/API_ADMIN">
        <span class="doc-title">└ Admin</span>
        <span class="doc-desc">관리자, 신고, 통계</span>
      </a>
    </li>
  </ul>
</div>

<div class="doc-section">
  <h2>아키텍처</h2>
  <ul class="doc-list">
    <li>
      <a href="{{ site.baseurl }}/architecture/">
        <span class="doc-title">Architecture</span>
        <span class="doc-desc">MVP 시스템 아키텍처</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/architecture/backend">
        <span class="doc-title">└ Backend</span>
        <span class="doc-desc">서버 구조, 컴포넌트</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/architecture/frontend">
        <span class="doc-title">└ Frontend</span>
        <span class="doc-desc">React 앱 구조</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/architecture/database">
        <span class="doc-title">└ Database</span>
        <span class="doc-desc">ERD, 테이블 설계</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/architecture/api">
        <span class="doc-title">└ API</span>
        <span class="doc-desc">REST API, WebSocket</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/architecture/infrastructure">
        <span class="doc-title">└ Infrastructure</span>
        <span class="doc-desc">배포, 보안, 모니터링</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/ARCHITECTURE-SCALE">
        <span class="doc-title">Scale Architecture</span>
        <span class="doc-desc">대규모 트래픽 아키텍처</span>
      </a>
    </li>
  </ul>
</div>

<div class="doc-section">
  <h2>기술 스택</h2>
  <ul class="doc-list">
    <li>
      <a href="{{ site.baseurl }}/tech-stack/">
        <span class="doc-title">Tech Stack</span>
        <span class="doc-desc">최종 기술 스택 결정서</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/tech-stack/backend">
        <span class="doc-title">└ Backend</span>
        <span class="doc-desc">Java, Spring Boot, Netty</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/tech-stack/frontend">
        <span class="doc-title">└ Frontend</span>
        <span class="doc-desc">React, Flutter</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/tech-stack/infrastructure">
        <span class="doc-title">└ Infrastructure</span>
        <span class="doc-desc">Kubernetes, Redis</span>
      </a>
    </li>
  </ul>
</div>

<div class="doc-section">
  <h2>기술 결정 (ADR)</h2>
  <ul class="doc-list">
    <li>
      <a href="{{ site.baseurl }}/decisions/">
        <span class="doc-title">All Decisions</span>
        <span class="doc-desc">기술 결정 문서 목록</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/decisions/msa-vs-monolith">
        <span class="doc-title">└ MSA vs Monolith</span>
        <span class="doc-desc">아키텍처 패턴 선택</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/decisions/database-selection">
        <span class="doc-title">└ Database Selection</span>
        <span class="doc-desc">PostgreSQL 선택 근거</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/decisions/spring-mvc-vs-webflux">
        <span class="doc-title">└ Spring MVC vs WebFlux</span>
        <span class="doc-desc">프레임워크 선택</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/decisions/redis-streams-ordering">
        <span class="doc-title">└ Redis Streams</span>
        <span class="doc-desc">메시지 순서 보장</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/decisions/frontend-platform-strategy">
        <span class="doc-title">└ Platform Strategy</span>
        <span class="doc-desc">웹/모바일 플랫폼</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/decisions/flutter-vs-react-native">
        <span class="doc-title">└ Flutter vs RN</span>
        <span class="doc-desc">모바일 프레임워크</span>
      </a>
    </li>
  </ul>
</div>

<div class="doc-section">
  <h2>가이드</h2>
  <ul class="doc-list">
    <li>
      <a href="{{ site.baseurl }}/guides/">
        <span class="doc-title">Guides</span>
        <span class="doc-desc">가이드 문서 목록</span>
      </a>
    </li>
    <li>
      <a href="{{ site.baseurl }}/guides/diagram-guide">
        <span class="doc-title">└ Diagram Guide</span>
        <span class="doc-desc">Mermaid, KaTeX 사용법</span>
      </a>
    </li>
  </ul>
</div>
