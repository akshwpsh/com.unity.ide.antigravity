# Antigravity Git 업로드 가이드

## 현재 상태
- 로컬 저장소: `c:\Users\akshw\Downloads\com.unity.ide.cursor`
- 현재 origin: `https://github.com/boxqkrtm/com.unity.ide.cursor.git`
- 변경사항: Cursor → Antigravity 변환 완료

## 업로드 방법

### 옵션 1: GitHub에서 Fork 후 업로드 (권장)

#### 1. GitHub에서 Fork 생성
1. 브라우저에서 https://github.com/boxqkrtm/com.unity.ide.cursor 접속
2. 우측 상단 **"Fork"** 버튼 클릭
3. Repository name을 `com.unity.ide.antigravity`로 변경
4. **"Create fork"** 클릭
5. Fork된 저장소 URL 복사 (예: `https://github.com/YOUR_USERNAME/com.unity.ide.antigravity.git`)

#### 2. 로컬에서 Remote 변경
```powershell
# 현재 디렉토리로 이동
cd c:\Users\akshw\Downloads\com.unity.ide.cursor

# 기존 origin 제거
git remote remove origin

# 새로운 origin 추가 (YOUR_USERNAME을 실제 GitHub 사용자명으로 변경)
git remote add origin https://github.com/YOUR_USERNAME/com.unity.ide.antigravity.git

# Remote 확인
git remote -v
```

#### 3. 변경사항 커밋
```powershell
# 모든 변경사항 스테이징
git add .

# 커밋
git commit -m "Convert Cursor integration to Antigravity support

- Changed package name to com.unity.ide.antigravity
- Renamed VisualStudioCursorInstallation to VisualStudioAntigravityInstallation
- Updated all Cursor references to Antigravity
- Modified installation paths and process names
- Updated workspace storage paths
- Added comprehensive migration guide
- Updated README and CHANGELOG"

# 커밋 확인
git log -1
```

#### 4. GitHub에 푸시
```powershell
# master 브랜치를 origin으로 푸시
git push -u origin master

# 또는 main 브랜치를 사용하는 경우
# git branch -M main
# git push -u origin main
```

---

### 옵션 2: 새 저장소 생성 후 업로드

#### 1. GitHub에서 새 저장소 생성
1. GitHub 접속 후 **"New repository"** 클릭
2. Repository name: `com.unity.ide.antigravity`
3. Description: `Unity editor integration for Antigravity`
4. Public/Private 선택
5. **"Create repository"** 클릭 (README, .gitignore 등은 추가하지 않음)

#### 2. 로컬에서 Remote 변경 및 푸시
```powershell
cd c:\Users\akshw\Downloads\com.unity.ide.cursor

# 기존 origin 제거
git remote remove origin

# 새 저장소를 origin으로 추가
git remote add origin https://github.com/YOUR_USERNAME/com.unity.ide.antigravity.git

# 변경사항 커밋
git add .
git commit -m "Initial Antigravity Unity integration based on Cursor v2.0.27"

# 푸시
git push -u origin master
```

---

## 푸시 후 확인사항

### 1. GitHub 저장소 설정
- **Settings → General**에서 저장소 설명 추가
- **About** 섹션에 태그 추가: `unity`, `antigravity`, `editor-integration`, `csharp`

### 2. README 확인
- GitHub에서 README.md가 올바르게 표시되는지 확인
- 설치 URL이 올바른지 확인

### 3. Release 생성 (선택사항)
```powershell
# 태그 생성
git tag -a v2.0.27 -m "Initial Antigravity support based on Cursor v2.0.27"

# 태그 푸시
git push origin v2.0.27
```

그 후 GitHub에서:
1. **Releases** → **"Create a new release"**
2. Tag: `v2.0.27` 선택
3. Release title: `v2.0.27 - Initial Antigravity Support`
4. Description에 CHANGELOG 내용 추가
5. **"Publish release"** 클릭

---

## 문제 해결

### 푸시 권한 오류
```powershell
# Personal Access Token 사용 필요
# GitHub Settings → Developer settings → Personal access tokens → Generate new token
# repo 권한 선택 후 생성
# 푸시 시 비밀번호 대신 토큰 사용
```

### 브랜치 이름 변경 필요시
```powershell
# master를 main으로 변경
git branch -M main
git push -u origin main
```

### 대용량 파일 경고
```powershell
# .gitignore에 추가 후 다시 커밋
echo "*.log" >> .gitignore
echo "Temp/" >> .gitignore
git add .gitignore
git commit --amend
```

---

## Unity 패키지 매니저에서 사용

푸시 완료 후 Unity에서 설치:
```
Window → Package Manager → + → Add package from git URL
https://github.com/YOUR_USERNAME/com.unity.ide.antigravity.git
```

---

## 원본 저장소 크레딧 유지

README.md에 이미 크레딧이 포함되어 있습니다:
```markdown
## Credits
Based on the official Unity Visual Studio Code integration package and Cursor integration by boxqkrtm.
```

필요시 추가 크레딧:
- LICENSE 파일 유지
- CHANGELOG에 원본 버전 히스토리 유지 (이미 완료)
