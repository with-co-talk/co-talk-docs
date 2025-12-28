# GitHub Pages 설정 가이드

Co-Talk 문서를 GitHub Pages로 배포하는 방법입니다.

## 방법 1: GitHub 저장소 설정 (권장)

가장 간단한 방법입니다.

1. GitHub 저장소로 이동: https://github.com/with-co-talk/co-talk
2. **Settings** 탭 클릭
3. 왼쪽 메뉴에서 **Pages** 클릭
4. **Source** 섹션에서:
   - **Deploy from a branch** 선택
   - **Branch**: `main` 선택
   - **Folder**: `/docs` 선택
5. **Save** 클릭
6. 몇 분 후 사이트가 배포됩니다: https://with-co-talk.github.io/co-talk/

## 방법 2: GitHub Actions (자동 배포)

이미 `.github/workflows/pages.yml` 워크플로우가 설정되어 있습니다.

1. GitHub 저장소 설정에서 **Pages**로 이동
2. **Source**를 **GitHub Actions**로 변경
3. `docs/` 폴더의 변경사항이 푸시되면 자동으로 배포됩니다

## 확인

배포가 완료되면 다음 주소에서 문서 사이트를 확인할 수 있습니다:
**https://with-co-talk.github.io/co-talk/**

## 문제 해결

### 사이트가 보이지 않는 경우
- 배포에 몇 분이 걸릴 수 있습니다
- 저장소가 Public인지 확인하세요
- GitHub Actions 탭에서 배포 상태를 확인하세요

### Jekyll 빌드 오류
- `_config.yml` 파일이 루트에 있는지 확인하세요
- Markdown 파일의 문법을 확인하세요

## 커스터마이징

### 테마 변경
`_config.yml` 파일의 `theme` 값을 변경할 수 있습니다:
- `minima` (기본)
- `jekyll-theme-minimal`
- `jekyll-theme-cayman`
- 기타 Jekyll 테마

### 네비게이션 수정
`_config.yml`의 `navigation` 섹션을 수정하여 메뉴를 변경할 수 있습니다.









