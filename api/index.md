---
layout: default
title: API Reference
nav_order: 4
has_children: true
---

# API Reference

Flutter 클라이언트 개발을 위한 REST API 문서입니다.

## 공통 사항

### Base URL
```
https://api.cotalk.com/api/v1
```

### 인증
대부분의 API는 JWT Bearer 토큰 인증이 필요합니다.
```
Authorization: Bearer {accessToken}
```

### 에러 응답 형식
```json
{
  "status": 400,
  "message": "에러 메시지",
  "code": "ERROR_CODE"
}
```

---

## API 목록

| 문서 | 설명 | 엔드포인트 수 |
|------|------|--------------|
| [API_AUTH](API_AUTH) | 인증, OAuth, 비밀번호, 계정, 약관 | 13개 |
| [API_USER](API_USER) | 사용자 정보, 검색, 프로필, 온라인 상태 | 5개 |
| [API_FRIEND](API_FRIEND) | 친구 요청, 친구 관리, 차단 | 10개 |
| [API_CHATROOM](API_CHATROOM) | 채팅방 생성, 관리, 멤버 관리 | 13개 |
| [API_MESSAGE](API_MESSAGE) | 메시지, 반응, 검색 | 12개 |
| [API_NOTIFICATION](API_NOTIFICATION) | 알림, 디바이스 토큰, 파일 업로드 | 5개 |
| [API_ADMIN](API_ADMIN) | 관리자, 신고, 통계 | 10개 |

**총 68개 엔드포인트**
