# 문서 레포지토리 분리 가이드

이 문서는 `co-talk` 레포지토리에서 `co-talk-docs` 레포지토리로 문서를 분리하는 가이드입니다.

## 분리 이유

- 문서는 모든 서비스에 공통
- 독립적 관리 용이
- URL 단순화 (`with-co-talk.github.io/` 루트)
- 문서 팀과 개발 팀 분리 용이

## 분리 절차

### 1. GitHub에서 새 레포지토리 생성

1. GitHub에서 `co-talk-docs` 레포지토리 생성
   - Public 또는 Private 선택
   - README, .gitignore, license는 생성하지 않음 (기존 파일 사용)

### 2. 로컬에서 새 레포지토리 준비

```bash
# 새 디렉토리 생성
cd ..
mkdir co-talk-docs
cd co-talk-docs

# Git 초기화
git init

# 원격 레포지토리 추가
git remote add origin https://github.com/with-co-talk/co-talk-docs.git
```

### 3. 문서 파일 복사

```bash
# co-talk 레포지토리의 docs 폴더 내용을 새 레포지토리로 복사
cp -r ../co-talk/docs/* .
cp -r ../co-talk/docs/.* . 2>/dev/null || true  # 숨김 파일도 복사
```

또는 Windows PowerShell:
```powershell
# co-talk 레포지토리의 docs 폴더 내용을 새 레포지토리로 복사
Copy-Item -Path ..\co-talk\docs\* -Destination . -Recurse -Force
```

### 4. GitHub Pages 워크플로우 생성

`.github/workflows/pages.yml` 파일 생성 (이미 준비됨):

```yaml
name: Deploy GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Pages
        uses: actions/configure-pages@v4
      
      - name: Build with Jekyll
        uses: actions/jekyll-build-pages@v1
        with:
          source: ./
          destination: ./_site
      
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
      
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 5. 커밋 및 푸시

```bash
# 파일 추가
git add .

# 커밋
git commit -m "docs: 문서 레포지토리 분리

- co-talk 레포지토리에서 문서 분리
- GitHub Pages 루트 배포 설정
- baseurl 제거 (루트에서 서빙)"

# 푸시
git branch -M main
git push -u origin main
```

### 6. GitHub Pages 설정

1. GitHub 레포지토리 Settings → Pages
2. Source: "GitHub Actions" 선택
3. 저장

### 7. 기존 레포지토리 업데이트

`co-talk` 레포지토리의 README.md 업데이트:

```markdown
## 📚 문서

> **📖 [문서 사이트 보기](https://with-co-talk.github.io/)** (GitHub Pages)

문서는 별도 레포지토리에서 관리됩니다:
- **문서 레포지토리**: [co-talk-docs](https://github.com/with-co-talk/co-talk-docs)
- **문서 사이트**: https://with-co-talk.github.io/
```

### 8. 기존 docs 폴더 정리 (선택)

`co-talk` 레포지토리에서 `docs/` 폴더를 삭제하거나 README로 대체:

```bash
# co-talk 레포지토리에서
rm -rf docs/
# 또는 docs/ 폴더를 README로 대체
```

## 완료 확인

- [ ] 새 레포지토리 생성 완료
- [ ] 파일 복사 완료
- [ ] GitHub Pages 배포 완료
- [ ] URL 확인: https://with-co-talk.github.io/
- [ ] 기존 레포지토리 README 업데이트
- [ ] 기존 docs 폴더 정리

## 주의사항

1. **Git 히스토리**: 새 레포지토리이므로 히스토리는 이전되지 않습니다. 필요시 `git filter-branch` 사용 가능
2. **링크 업데이트**: 기존 문서 링크가 있다면 업데이트 필요
3. **CI/CD**: 기존 워크플로우는 docs 폴더를 감시하므로 제거 또는 수정 필요

