# 🎓 HR 교육자료 관리 시스템

> **CursorAI와 함께하는 풀스택 웹 개발 프로젝트**  
> HR팀을 위한 교육자료 관리 및 공유 플랫폼

## 📋 프로젝트 개요

### 🎯 프로젝트 목적
- HR팀 관리자가 교육자료를 업로드하고 관리할 수 있는 시스템
- 전사 직원들이 교육자료를 자유롭게 열람하고 학습할 수 있는 플랫폼
- 비전공자도 CursorAI를 활용하여 실제 동작하는 웹사이트 구현

### 👥 대상 사용자
- **관리자**: HR팀 교육담당자 (자료 업로드, 수정, 삭제 권한)
- **일반 사용자**: 전사 직원 (자료 열람 및 다운로드, 로그인 불필요)

### ✨ 주요 기능
- 🔓 **오픈 액세스**: 로그인 없이 모든 교육자료 열람 가능
- 🔐 **관리자 인증**: 업로드/수정/삭제는 관리자만 (SSO 연동 예정)
- 📚 **자료 관리**: 교육자료 업로드, 카테고리 분류, 검색 기능
- 👀 **자료 열람**: 카테고리별 자료 조회, 전체 검색, 다운로드
- 📊 **대시보드**: 자료 현황, 다운로드 통계 (관리자용)
- 📱 **반응형 디자인**: 데스크톱, 태블릿, 모바일 지원

---

## 🏗️ 기술 스택

### Frontend
- **Vue.js 3** - 프론트엔드 프레임워크
- **TypeScript** - 타입 안정성
- **Tailwind CSS** - 유틸리티 우선 CSS 프레임워크
- **Vite** - 빠른 개발 서버 및 빌드 도구

### Backend
- **Python 3.9+** - 프로그래밍 언어
- **FastAPI** - 현대적이고 빠른 웹 API 프레임워크
- **SQLAlchemy** - ORM (Object-Relational Mapping)
- **Pydantic** - 데이터 검증 및 시리얼라이제이션

### Database
- **PostgreSQL** - 관계형 데이터베이스
- **DBeaver** - 데이터베이스 관리 도구

### Development Tools
- **CursorAI** - AI 기반 코드 작성 도구
- **Git** - 버전 관리
- **VS Code** - 코드 에디터 (CursorAI 대안)

---

## 🚀 프로젝트 구조

```
hr-education-system/
├── frontend/                 # Vue.js 프론트엔드
│   ├── src/
│   │   ├── components/      # 재사용 가능한 컴포넌트
│   │   ├── views/          # 페이지 컴포넌트
│   │   ├── router/         # 라우팅 설정
│   │   ├── stores/         # 상태 관리 (Pinia)
│   │   ├── types/          # TypeScript 타입 정의
│   │   └── utils/          # 유틸리티 함수
│   ├── public/             # 정적 파일
│   └── package.json
├── backend/                  # FastAPI 백엔드
│   ├── app/
│   │   ├── models/         # 데이터베이스 모델
│   │   ├── schemas/        # Pydantic 스키마
│   │   ├── routers/        # API 라우터
│   │   ├── services/       # 비즈니스 로직
│   │   ├── database.py     # 데이터베이스 설정
│   │   └── main.py         # FastAPI 애플리케이션
│   ├── requirements.txt    # Python 의존성
│   └── Dockerfile          # 도커 설정
├── database/                # 데이터베이스 스크립트
│   ├── init.sql           # 초기 데이터베이스 스키마
│   └── sample_data.sql    # 샘플 데이터
├── docs/                    # 프로젝트 문서
│   ├── FRONTEND_GUIDE.md   # 프론트엔드 개발 가이드
│   ├── BACKEND_GUIDE.md    # 백엔드 개발 가이드
│   ├── DATABASE_GUIDE.md   # 데이터베이스 설정 가이드
│   └── DEPLOYMENT.md       # 배포 가이드
├── .gitignore
├── README.md
└── docker-compose.yml      # 개발 환경 설정
```

---

## 🎯 학습 목표

이 프로젝트를 통해 다음을 학습할 수 있습니다:

### 기술적 학습
- **풀스택 웹 개발의 전체 흐름** 이해
- **프론트엔드와 백엔드의 분리** 및 API 통신
- **데이터베이스 설계** 및 관리
- **파일 업로드/다운로드** 시스템 구현
- **반응형 웹 디자인** 적용

### AI 도구 활용
- **CursorAI 활용법**: 코드 자동 완성, 리팩토링, 디버깅
- **프롬프트 엔지니어링**: AI와 효과적으로 소통하는 방법
- **AI 기반 개발 워크플로우**: 기존 개발 과정의 혁신

### 실무 경험
- **Git을 이용한 버전 관리**
- **프로젝트 구조화** 및 코드 조직화
- **API 설계** 및 문서화
- **테스트 및 디버깅** 기법

---

## 🏃‍♀️ 빠른 시작

### 1단계: 개발 환경 준비
```bash
# 프로젝트 클론
git clone <repository-url>
cd hr-education-system
```

### 2단계: 데이터베이스 설정
```bash
# PostgreSQL 설치 및 데이터베이스 생성
# 자세한 내용은 docs/DATABASE_GUIDE.md 참조
```

### 3단계: 백엔드 실행
```bash
cd backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

### 4단계: 프론트엔드 실행
```bash
cd frontend
npm install
npm run dev
```

---

## 📚 학습 로드맵

### Week 1: 환경 설정 및 기초 이해
- [ ] 개발 도구 설치 (CursorAI, PostgreSQL, DBeaver)
- [ ] 기본 프로젝트 구조 생성
- [ ] 간단한 "Hello World" 애플리케이션 실행
- [ ] **가이드**: `docs/DATABASE_GUIDE.md` 완료

### Week 2-3: 백엔드 개발
- [ ] FastAPI 기본 개념 학습
- [ ] 데이터베이스 모델 설계
- [ ] 기본 CRUD API 구현
- [ ] 파일 업로드/다운로드 API 구현
- [ ] **가이드**: `docs/BACKEND_GUIDE.md` 완료

### Week 4-5: 프론트엔드 개발
- [ ] Vue.js + TypeScript 기초 학습
- [ ] Tailwind CSS로 UI 구성
- [ ] 컴포넌트 설계 및 구현
- [ ] 라우팅 및 상태 관리
- [ ] **가이드**: `docs/FRONTEND_GUIDE.md` 완료

### Week 6: 통합 및 완성
- [ ] 프론트엔드-백엔드 연결
- [ ] 관리자 기능 구현
- [ ] UI/UX 개선
- [ ] 테스트 및 버그 수정

---

## 🤖 CursorAI 활용 전략

### 효과적인 개발 접근법
1. **단계별 구현**: 큰 기능을 작은 단위로 나누어서 구현
2. **AI와 협업**: 각 단계마다 CursorAI에게 명확한 요구사항 전달
3. **코드 리뷰**: 완성된 코드에 대해 AI에게 개선점 문의
4. **문제 해결**: 에러 발생 시 AI에게 디버깅 도움 요청

### 권장 프롬프트 패턴
```
# 기능 구현 요청
"Vue 3 + TypeScript로 교육자료 카드 컴포넌트를 만들어줘. 
제목, 설명, 파일 크기, 업로드 날짜, 다운로드 버튼이 포함되어야 해."

# 에러 해결 요청
"다음 에러가 발생했어: [에러 메시지]
코드는: [관련 코드]
어떻게 해결할까?"

# 코드 개선 요청
"이 컴포넌트의 성능과 가독성을 개선할 방법 알려줘: [코드]"
```

---

## 🎯 향후 확장 아이디어

### Phase 2: 기본 기능 확장
- **알림 시스템**: 새로운 자료 업로드 알림
- **즐겨찾기**: 개인별 즐겨찾는 자료 관리
- **댓글 시스템**: 자료별 질문과 답변
- **통계 대시보드**: 자료별 다운로드 통계

### Phase 3: 고급 기능
- **SSO 연동**: Active Directory, Google Workspace 등
- **동영상 스트리밍**: 비디오 교육자료 재생
- **퀴즈 시스템**: 학습 후 이해도 체크
- **모바일 앱**: 모바일 전용 앱 개발

---

## 📞 지원 및 문의

### 학습 리소스
- [FastAPI 공식 문서](https://fastapi.tiangolo.com/)
- [Vue.js 공식 가이드](https://vuejs.org/guide/)
- [TypeScript 핸드북](https://www.typescriptlang.org/docs/)
- [Tailwind CSS 문서](https://tailwindcss.com/docs)

### 개발 가이드
- **데이터베이스 설정**: [DATABASE_GUIDE.md](docs/DATABASE_GUIDE.md)
- **백엔드 개발**: [BACKEND_GUIDE.md](docs/BACKEND_GUIDE.md)
- **프론트엔드 개발**: [FRONTEND_GUIDE.md](docs/FRONTEND_GUIDE.md)
- **배포**: [DEPLOYMENT.md](docs/DEPLOYMENT.md)

---

## 📝 다음 단계

1. ✅ **README.md** 확인 완료
2. 🔄 **[DATABASE_GUIDE.md](docs/DATABASE_GUIDE.md)** 읽고 데이터베이스 설정
3. 🔄 **[BACKEND_GUIDE.md](docs/BACKEND_GUIDE.md)** 따라서 백엔드 구현
4. 🔄 **[FRONTEND_GUIDE.md](docs/FRONTEND_GUIDE.md)** 따라서 프론트엔드 구현

---

**🎉 성균관대 빅데이터학과 진학을 축하드립니다!**  
완전 축하 ^_^
이 프로젝트가 개발자로서의 첫 걸음에 도움이 되길 바랍니다.

---

*Made with ❤️ and CursorAI*
