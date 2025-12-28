# 문서 레포지토리 분리 실행 가이드

이 가이드는 `co-talk` 레포지토리에서 `co-talk-docs` 레포지토리로 문서를 분리하는 단계별 가이드입니다.

## ✅ 준비 완료

다음 파일들이 준비되었습니다:
- ✅ `docs/_config.yml` - baseurl 제거됨
- ✅ `.github/workflows/pages-docs.yml` - 새 레포지토리용 워크플로우
- ✅ `docs/MIGRATION-GUIDE.md` - 상세 가이드
- ✅ `docs/README.md` - 문서 레포지토리용 README

## 🚀 분리 단계

### 1단계: GitHub에서 새 레포지토리 생성

1. GitHub에 로그인
2. https://github.com/organizations/with-co-talk/repositories/new 접속
3. 레포지토리 정보 입력:
   - **Repository name**: `co-talk-docs`
   - **Description**: `Co-Talk 프로젝트 문서 레포지토리`
   - **Visibility**: Public (또는 Private)
   - **Initialize this repository with**: 체크하지 않음 (기존 파일 사용)
4. "Create repository" 클릭

### 2단계: 로컬에서 새 레포지토리 준비

PowerShell에서 실행:

```powershell
# 상위 디렉토리로 이동
cd ..

# 새 디렉토리 생성
mkdir co-talk-docs
cd co-talk-docs

# Git 초기화
git init

# 원격 레포지토리 추가
git remote add origin https://github.com/with-co-talk/co-talk-docs.git
```

### 3단계: 문서 파일 복사

```powershell
# co-talk 레포지토리의 docs 폴더 내용을 새 레포지토리로 복사
Copy-Item -Path ..\co-talk\docs\* -Destination . -Recurse -Force

# .github 폴더도 복사 (워크플로우 포함)
Copy-Item -Path ..\co-talk\.github\workflows\pages-docs.yml -Destination .github\workflows\pages-docs.yml -Force
```

### 4단계: 커밋 및 푸시

```powershell
# 파일 추가
git add .

# 커밋
git commit -m "docs: 문서 레포지토리 분리

- co-talk 레포지토리에서 문서 분리
- GitHub Pages 루트 배포 설정
- baseurl 제거 (루트에서 서빙)"

# 메인 브랜치로 설정 및 푸시
git branch -M main
git push -u origin main
```

### 5단계: GitHub Pages 설정

1. https://github.com/with-co-talk/co-talk-docs 접속
2. Settings → Pages
3. Source: "GitHub Actions" 선택
4. 저장

### 6단계: 배포 확인

1-2분 후 다음 URL에서 확인:
- https://with-co-talk.github.io/

### 7단계: 기존 레포지토리 정리 (선택)

`co-talk` 레포지토리에서:

```powershell
cd ..\co-talk

# docs 폴더 삭제 (또는 백업 후 삭제)
# 주의: 삭제 전 백업 권장
Remove-Item -Path docs -Recurse -Force

# 기존 워크플로우 제거 또는 비활성화
# .github/workflows/pages.yml 파일 삭제 또는 수정
```

## ✅ 완료 체크리스트

- [ ] 새 레포지토리 `co-talk-docs` 생성 완료
- [ ] 문서 파일 복사 완료
- [ ] GitHub Pages 설정 완료
- [ ] 배포 확인: https://with-co-talk.github.io/
- [ ] 기존 레포지토리 README 업데이트 확인
- [ ] 기존 docs 폴더 정리 (선택)

## 📝 참고

- 상세 가이드: [docs/MIGRATION-GUIDE.md](./docs/MIGRATION-GUIDE.md)
- 레포지토리 전략: [docs/REPOSITORY-STRATEGY.md](./docs/REPOSITORY-STRATEGY.md)

