# Co-Talk 문서

이 폴더는 Co-Talk 프로젝트의 모든 문서를 포함합니다.

## 📖 문서 사이트

이 문서들은 GitHub Pages를 통해 웹사이트로 자동 배포됩니다:
**https://with-co-talk.github.io/co-talk/**

## 📁 문서 구조

```
docs/
├── index.md                    # 문서 사이트 홈페이지
├── _config.yml                 # Jekyll 설정 파일
├── PRD.md                      # 제품 요구사항 문서
├── TECH-STACK.md              # 기술 스택 결정서
├── ARCHITECTURE.md            # 아키텍처 설계 (MVP)
├── ARCHITECTURE-SCALE.md      # 대규모 트래픽 아키텍처
├── DATABASE-SELECTION.md      # 데이터베이스 선택 이유
├── SPRING-MVC-VS-WEBFLUX.md   # Spring MVC vs WebFlux 비교
├── FRONTEND-PLATFORM-STRATEGY.md  # 프론트엔드 플랫폼 전략
├── FLUTTER-VS-REACT-NATIVE.md # Flutter vs React Native 비교
└── REDIS-STREAMS-ORDERING.md  # Redis Streams 순서 보장 가이드
```

## 🚀 GitHub Pages 설정

1. GitHub 저장소 설정에서 **Settings > Pages**로 이동
2. **Source**를 `docs` 폴더로 설정
3. 저장 후 몇 분 내에 사이트가 배포됩니다

## 📝 문서 작성 가이드

- 모든 문서는 Markdown 형식으로 작성합니다
- 기술 결정은 ADR (Architecture Decision Records) 형식을 따릅니다
- 문서는 Git을 통해 버전 관리됩니다
- 변경사항을 푸시하면 자동으로 사이트가 업데이트됩니다



