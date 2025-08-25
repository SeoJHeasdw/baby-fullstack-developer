# 📚 Git 완전 정복 가이드

> **CursorAI와 함께하는 스마트한 Git 워크플로우**  
> 완전 초보자부터 팀 협업까지, 실무에서 바로 쓰는 Git 명령어 모음

## 📋 목차
1. [Git이 뭐고 왜 필요한가요?](#git이-뭐고-왜-필요한가요)
2. [Git 설치 및 초기 설정](#git-설치-및-초기-설정)
3. [혼자 개발할 때 필수 명령어](#혼자-개발할-때-필수-명령어)
4. [GitHub와 연동하기](#github와-연동하기)
5. [팀 협업 워크플로우](#팀-협업-워크플로우)
6. [자주 발생하는 문제 해결](#자주-발생하는-문제-해결)
7. [CursorAI로 Git 마스터하기](#cursorai로-git-마스터하기)

---

## 🤔 Git이 뭐고 왜 필요한가요?

### 🎯 **Git을 안 쓰면 이런 일이...**
```bash
# 😱 이런 경험 있으신가요?
내문서/
├── 프로젝트_최종.zip
├── 프로젝트_최종_진짜최종.zip  
├── 프로젝트_최종_진짜최종_수정.zip
├── 프로젝트_최종_진짜최종_수정_완료.zip
└── 프로젝트_최종_진짜최종_수정_완료_버그수정.zip
```

### ✨ **Git을 쓰면?**
- 📜 **변경 이력 관리**: "언제 누가 무엇을 바꿨는지" 모든 기록
- 🔄 **버전 관리**: 언제든 이전 버전으로 돌아가기 가능
- 👥 **팀 협업**: 여러 명이 동시에 작업해도 충돌 없이 합치기
- 🌐 **백업**: GitHub에 올려두면 컴퓨터가 고장나도 안전
- 🚀 **포트폴리오**: 개발 과정을 보여줄 수 있어 취업에 유리

---

## 🛠️ Git 설치 및 초기 설정

### Windows 설치
```bash
# 1. https://git-scm.com/download/win 에서 다운로드
# 2. 설치 시 모든 옵션 기본값으로 진행
# 3. Git Bash 실행해서 확인
git --version
```

### macOS 설치
```bash
# Homebrew 사용 (추천)
brew install git

# 또는 공식 사이트에서 다운로드
# https://git-scm.com/download/mac
```

### 필수 초기 설정
```bash
# 📝 사용자 정보 설정 (GitHub 계정과 동일하게!)
git config --global user.name "김개발"
git config --global user.email "kim.developer@gmail.com"

# 🎨 기본 브랜치 이름 설정
git config --global init.defaultBranch main

# 📋 설정 확인
git config --list
```

---

## 🚀 혼자 개발할 때 필수 명령어

### 📦 **프로젝트 시작하기**
```bash
# 🆕 새 프로젝트 시작
mkdir my-awesome-project
cd my-awesome-project
git init

# 📁 기존 프로젝트를 Git으로 관리 시작
cd existing-project
git init
```

### 📸 **변경사항 저장하기** (가장 자주 사용!)
```bash
# 🔍 현재 상태 확인
git status

# 📝 변경된 파일들 확인
git diff

# ➕ 저장할 파일 선택 (Staging)
git add .                    # 모든 변경사항 추가
git add src/main.py         # 특정 파일만 추가  
git add src/                # 특정 폴더만 추가

# 💾 변경사항 저장 (Commit)
git commit -m "로그인 기능 구현 완료"

# 🚀 한 번에 add + commit
git commit -am "버그 수정: 로그인 오류 해결"
```

### 📜 **이력 확인하기**
```bash
# 📊 커밋 이력 보기
git log                      # 전체 이력
git log --oneline           # 한 줄로 간략히
git log --graph             # 그래프로 시각화
git log -10                 # 최근 10개만

# 🎨 예쁘게 보기 (추천!)
git log --oneline --graph --decorate --all
```

### 🔄 **되돌리기**
```bash
# 📂 작업 디렉터리의 변경사항 취소
git checkout -- filename.py    # 특정 파일
git checkout .                 # 모든 파일

# ⏪ Staging 영역에서 빼기 (add 취소)
git reset filename.py          # 특정 파일
git reset                     # 모든 파일

# 🚨 마지막 커밋 수정 (푸시하기 전에만!)
git commit --amend -m "수정된 커밋 메시지"

# ⚠️ 특정 커밋으로 돌아가기 (위험!)
git reset --hard abc1234      # 커밋 ID로
git reset --hard HEAD~1       # 바로 이전 커밋으로
```

---

## 🌐 GitHub와 연동하기

### 🔗 **GitHub 저장소 연결**
```bash
# 📡 원격 저장소 등록
git remote add origin https://github.com/username/repo-name.git

# 🚀 처음 푸시 (업로드)
git push -u origin main

# 📤 이후 푸시 (간단히)
git push
```

### 📥 **GitHub에서 코드 가져오기**
```bash
# 🔽 저장소 복제 (Clone)
git clone https://github.com/username/repo-name.git

# 📨 최신 변경사항 가져오기 (Pull)
git pull

# 📦 변경사항만 가져오기 (Fetch)
git fetch
```

### 🔒 **인증 설정 (Personal Access Token)**
```bash
# GitHub에서 Settings > Developer settings > Personal access tokens
# 토큰 생성 후 비밀번호 대신 사용

# 🔐 Windows에서 자격 증명 관리자 사용
# "제어판 > 자격 증명 관리자 > Windows 자격 증명"에서 GitHub 계정 수정
```

---

## 🌿 브랜치 사용하기

### 🌳 **브랜치 기본 개념**
```bash
# "브랜치"는 독립적인 작업 공간
main (기본)     ──●──●──●──●──    
                    │
feature/login      ●──●──●        (새 기능 개발)
```

### 🔀 **브랜치 명령어**
```bash
# 📋 브랜치 목록 보기
git branch                    # 로컬 브랜치
git branch -r                # 원격 브랜치  
git branch -a                # 모든 브랜치

# 🆕 새 브랜치 생성 + 이동
git checkout -b feature/login

# 🔄 브랜치 이동
git checkout main
git checkout feature/login

# 🔀 브랜치 병합 (Merge)
git checkout main
git merge feature/login

# 🗑️ 브랜치 삭제
git branch -d feature/login   # 안전 삭제
git branch -D feature/login   # 강제 삭제
```

---

## 👥 팀 협업 워크플로우

### 🚀 **Feature Branch 워크플로우** (실무 표준)
```bash
# 1️⃣ 최신 코드 가져오기
git checkout main
git pull

# 2️⃣ 새 기능 브랜치 생성
git checkout -b feature/user-profile

# 3️⃣ 개발 작업
# ... 코드 작성 ...
git add .
git commit -m "사용자 프로필 페이지 추가"

# 4️⃣ 원격에 푸시
git push -u origin feature/user-profile

# 5️⃣ GitHub에서 Pull Request 생성
# 6️⃣ 코드 리뷰 후 메인에 병합
# 7️⃣ 로컬에서 정리
git checkout main
git pull
git branch -d feature/user-profile
```

### 🔄 **충돌 해결하기**
```bash
# 😱 충돌 발생 시
git pull
# Auto-merging filename.py
# CONFLICT (content): Merge conflict in filename.py

# 🔧 충돌 파일 수정
# <<<<<<< HEAD
# 내 코드
# =======  
# 다른 사람 코드
# >>>>>>> branch-name

# ✅ 충돌 해결 후
git add filename.py
git commit -m "충돌 해결: filename.py"
```

---

## 🆘 자주 발생하는 문제 해결

### 😅 **"앗, 실수했어요!" 시나리오**

#### 🤦‍♀️ **잘못된 파일을 커밋했어요**
```bash
# 📂 파일을 staging에서 제거 (커밋 전)
git reset filename.py

# 🗑️ 파일을 Git에서 완전히 제거하되 실제 파일은 유지
git rm --cached filename.py

# 🔒 앞으로 무시하고 싶다면
echo "filename.py" >> .gitignore
git add .gitignore
git commit -m "gitignore에 filename.py 추가"
```

#### 🙈 **비밀번호를 실수로 올렸어요**
```bash
# ⚠️ 이미 푸시했다면 GitHub에서 해당 저장소 삭제하고 다시 생성
# 🔐 비밀번호나 API 키는 즉시 변경 필수!

# 📝 .gitignore에 추가해서 재발 방지
echo ".env" >> .gitignore
echo "config.json" >> .gitignore
git add .gitignore
git commit -m "환경 설정 파일 gitignore 추가"
```

#### 💔 **커밋 메시지를 잘못 적었어요**
```bash
# 📝 마지막 커밋 메시지만 수정 (푸시 전)
git commit --amend -m "올바른 커밋 메시지"

# 📚 여러 커밋 수정하려면 (고급)
git rebase -i HEAD~3
```

### 🔧 **자주 쓰는 유틸리티 명령어**
```bash
# 🧹 Git 상태 정리
git clean -fd               # 추적되지 않는 파일/폴더 삭제
git reset --hard HEAD       # 모든 변경사항 취소

# 📊 저장소 통계
git shortlog -sn           # 기여자별 커밋 수
git log --stat             # 파일별 변경 통계

# 🔍 특정 문자열이 포함된 커밋 찾기
git log --grep="버그"
git log -S "function_name" # 코드에서 특정 문자열 검색
```

---

## 🤖 CursorAI로 Git 마스터하기

### ✨ **AI에게 Git 도움받기**

#### 🆘 **에러 해결 요청**
```
"Git에서 다음 에러가 발생했어:
fatal: not a git repository (or any of the parent directories): .git

무슨 문제이고 어떻게 해결해야 할까?"
```

#### 📝 **커밋 메시지 개선 요청**
```
"다음 변경사항들을 좋은 커밋 메시지로 만들어줘:
- 로그인 폼 UI 수정
- 비밀번호 검증 로직 추가  
- 에러 메시지 다국어 지원
- 로그인 실패 시 로그 기록

커밋을 어떻게 나누는 게 좋을까?"
```

#### 🔀 **브랜치 전략 상담**
```
"5명이서 협업하는 프로젝트인데, 어떤 브랜치 전략이 좋을까?
각자 기능을 개발하고, 매주 금요일에 배포할 예정이야.
Git Flow vs GitHub Flow 중에 뭐가 좋을까?"
```

#### 📊 **Git 히스토리 분석**
```
"다음 git log 결과를 보고 뭔가 이상한 점 있는지 알려줘:
[git log --oneline --graph 결과 붙여넣기]

더 깔끔하게 정리하는 방법도 있으면 알려줘."
```

### 🎯 **AI 활용 실전 팁**

#### 🤝 **코드 리뷰 요청**
```
"Pull Request를 올리기 전에 내 코드를 리뷰해줘:
[변경된 코드 붙여넣기]

- 버그 가능성
- 성능 이슈
- 코딩 컨벤션
- 보안 문제
모두 체크해줘."
```

#### 🗂️ **.gitignore 파일 생성**
```
"Vue.js + FastAPI 프로젝트의 .gitignore 파일을 만들어줘.
다음 항목들 포함:
- Python 관련
- Node.js 관련
- IDE 설정 파일
- 환경 변수 파일
- 로그 파일"
```

---

## 📋 Git 치트시트

### ⚡ **매일 쓰는 필수 명령어**
```bash
git status                   # 상태 확인
git add .                    # 모든 변경사항 추가
git commit -m "메시지"        # 커밋
git push                     # 푸시
git pull                     # 풀
git checkout -b feature/name # 새 브랜치 생성+이동
git checkout main           # main 브랜치로 이동
git merge feature/name      # 브랜치 병합
```

### 🛠️ **가끔 쓰는 유용한 명령어**
```bash
git log --oneline           # 간략한 이력
git diff                    # 변경사항 확인
git reset filename          # staging 취소
git checkout -- filename   # 변경사항 취소
git branch -d branch-name   # 브랜치 삭제
git remote -v              # 원격 저장소 확인
git clean -fd              # 추적 안 되는 파일 삭제
```

### 🚨 **위험한 명령어** (조심히 사용!)
```bash
git reset --hard HEAD      # 모든 변경사항 삭제
git push --force           # 강제 푸시
git rebase                 # 히스토리 변경
git reset --hard commit-id # 특정 커밋으로 되돌리기
```

---

## 📚 상황별 워크플로우

### 🏠 **개인 프로젝트**
```bash
# 일일 루틴
git status
git add .
git commit -m "오늘 작업한 내용 설명"
git push
```

### 🏢 **회사 프로젝트**
```bash
# 출근 후
git checkout main
git pull

# 새 작업 시작
git checkout -b feature/new-feature

# 작업 완료 후
git add .
git commit -m "기능 구현 완료"
git push -u origin feature/new-feature

# GitHub에서 PR 생성 → 리뷰 → 병합
# 병합 후 정리
git checkout main
git pull
git branch -d feature/new-feature
```

### 🎓 **학습 프로젝트**
```bash
# 새 기능 실험할 때
git checkout -b experiment/try-something

# 실패했으면
git checkout main
git branch -D experiment/try-something

# 성공했으면
git checkout main
git merge experiment/try-something
```

---

## 🎨 좋은 커밋 메시지 작성법

### ✅ **좋은 예시**
```bash
git commit -m "feat: 사용자 로그인 기능 추가"
git commit -m "fix: 비밀번호 검증 오류 수정"  
git commit -m "docs: README에 설치 가이드 추가"
git commit -m "refactor: 중복 코드 제거 및 함수 분리"
git commit -m "style: 코드 포맷팅 및 들여쓰기 수정"
```

### ❌ **나쁜 예시**
```bash
git commit -m "수정"
git commit -m "버그픽스"
git commit -m "작업함"
git commit -m "ㅁㄴㅇㄹ"
```

### 📝 **커밋 메시지 템플릿**
```bash
# 타입: 간단한 설명 (50자 이내)
#
# 상세 설명 (필요시, 72자로 줄바꿈)
#
# 타입 종류:
# feat: 새 기능
# fix: 버그 수정
# docs: 문서 수정
# style: 코드 스타일 변경
# refactor: 리팩토링
# test: 테스트 추가
# chore: 빌드 설정 등
```

---

## 🔧 고급 Git 기법

### 🍒 **Cherry Pick** (특정 커밋만 가져오기)
```bash
# 다른 브랜치의 특정 커밋만 현재 브랜치에 적용
git cherry-pick abc1234
```

### 📦 **Stash** (임시 저장)
```bash
# 작업 중인 내용을 임시 저장
git stash

# 임시 저장 목록 보기
git stash list

# 임시 저장된 내용 불러오기
git stash pop
```

### 🔍 **Bisect** (버그 발생 지점 찾기)
```bash
# 이진 탐색으로 버그 발생 커밋 찾기
git bisect start
git bisect bad          # 현재는 버그 있음
git bisect good abc123  # 이 커밋은 정상이었음
# Git이 중간 커밋들을 보여주면서 good/bad 판단 요청
```

---

## 🏆 Git 마스터 체크리스트

### 🟢 **Git 초보자 (완료해야 할 것들)**
- [ ] Git 설치 및 초기 설정 완료
- [ ] `git add`, `git commit`, `git push` 이해
- [ ] GitHub 저장소 연결 성공
- [ ] `.gitignore` 파일 사용법 이해
- [ ] 기본적인 되돌리기 (`reset`, `checkout`) 사용 가능

### 🟡 **Git 중급자**
- [ ] 브랜치 생성, 이동, 병합 능숙하게 사용
- [ ] Pull Request 워크플로우 이해
- [ ] 충돌 해결 경험 보유
- [ ] 커밋 메시지 컨벤션 준수
- [ ] `git log`, `git diff` 활용한 이력 분석 가능

### 🔴 **Git 고급자** 
- [ ] Rebase 사용법 이해
- [ ] Cherry-pick, Stash 활용
- [ ] Git hook 사용 경험
- [ ] 복잡한 충돌 상황 해결 가능
- [ ] 팀 Git 전략 수립 및 가이드 가능

---

## 🚨 Git 사용 시 주의사항

### ⚠️ **절대 하면 안 되는 것들**
- 🔐 **비밀번호, API 키 절대 커밋하지 말기**
- 📦 **node_modules, .env 파일 커밋하지 말기**
- 🗑️ **공유 브랜치에서 `git reset --hard` 사용하지 말기**
- 🚀 **다른 사람이 사용 중인 브랜치에 `push --force` 하지 말기**

### 💡 **베스트 프랙티스**
- 📝 **자주, 작은 단위로 커밋하기**
- 🏷️ **의미 있는 커밋 메시지 작성하기**
- 🌿 **기능별로 브랜치 나누어 작업하기**
- 📋 **커밋하기 전에 `git status`로 확인하기**
- 🔄 **정기적으로 `git pull`로 최신 코드 가져오기**

---

## 🔗 추가 학습 자료

### 📚 **공식 문서**
- [Git 공식 문서](https://git-scm.com/docs)
- [GitHub 가이드](https://guides.github.com/)
- [Atlassian Git 튜토리얼](https://www.atlassian.com/git/tutorials)

### 🎮 **인터랙티브 학습**
- [Learn Git Branching](https://learngitbranching.js.org/) - 시각적 Git 학습
- [Git 실습](https://try.github.io/) - GitHub 제공 실습

### 📖 **추천 도서**
- "Pro Git" (무료 온라인 도서)
- "Git 교과서" (한국어)

---

## 🎯 다음 단계

Git 기본기를 익혔다면:
1. **🏢 팀 프로젝트 참여** - 실제 협업 경험 쌓기
2. **🚀 GitHub Actions** - CI/CD 자동화 학습  
3. **🔧 Git Hook** - 커밋/푸시 시 자동 검증
4. **📊 Git 분석 도구** - GitKraken, SourceTree 등 GUI 도구

---

**🎉 축하합니다!** 이제 Git으로 전문적인 개발 워크플로우를 구축할 수 있습니다!

---

*Made with ❤️ and CursorAI*