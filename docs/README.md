# Co-Talk Documentation

Co-Talk 프로젝트의 모든 문서를 관리하는 레포지토리입니다.

## 📖 문서 사이트

**문서 사이트**: https://with-co-talk.github.io/

GitHub Pages를 통해 자동으로 배포됩니다.

## 📁 문서 구조

- **제품 문서**: PRD, 사용자 가이드 등
- **기술 문서**: Tech Stack, Architecture 등
- **기술 결정 문서 (ADR)**: 각종 기술 선택 근거

## 🚀 로컬에서 실행

### Jekyll 설치

```bash
# Ruby 설치 (이미 설치되어 있다면 생략)
# Windows: https://rubyinstaller.org/

# Jekyll 및 의존성 설치
gem install bundler
bundle install
```

### 로컬 서버 실행

```bash
bundle exec jekyll serve
```

브라우저에서 http://localhost:4000 접속

## 📝 문서 작성 가이드

1. 마크다운 파일 작성
2. Jekyll front matter 추가:
   ```yaml
   ---
   layout: default
   title: 문서 제목
   description: 문서 설명
   ---
   ```
3. 커밋 및 푸시
4. GitHub Pages 자동 배포

## 🎨 다이어그램 지원

- **Mermaid**: 플로우차트, 시퀀스 다이어그램 등
- **PlantUML**: UML 표준 다이어그램 (클래스, 컴포넌트, 배포 등)
- **KaTeX**: 수학 공식

자세한 사용법은 [다이어그램 가이드](./DIAGRAM-GUIDE.md) 참조

## 📚 주요 문서

- [PRD](./PRD.md) - 제품 요구사항 문서
- [Tech Stack](./TECH-STACK.md) - 최종 기술 스택 결정서 ⭐
- [Architecture](./ARCHITECTURE.md) - 아키텍처 설계 문서
- [Architecture Scale](./ARCHITECTURE-SCALE.md) - 대규모 트래픽 아키텍처
- [MSA vs 모놀리식](./MSA-VS-MONOLITH.md) - 아키텍처 패턴 선택
- [레포지토리 전략](./REPOSITORY-STRATEGY.md) - 레포지토리 구조 결정

## 🔗 관련 레포지토리

- **프로젝트 레포지토리**: [co-talk](https://github.com/with-co-talk/co-talk)
