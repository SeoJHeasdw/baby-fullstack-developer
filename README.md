🎓 HR 교육자료 관리 시스템

CursorAI와 함께하는 풀스택 웹 개발 프로젝트
HR팀을 위한 교육자료 관리 및 공유 플랫폼 + 실무 Git 워크플로우 마스터

🚀 30초 요약
**"개발 완전 초보자"**가 **"실무 가능한 개발자"**가 되는 여정 🌟

✅ 실제 동작하는 웹사이트 구축 (2주면 완성!)
🤖 CursorAI 마스터 - AI와 함께하는 스마트 개발
💼 실무 스킬 - Git, 배포, 팀 협업까지
🎯 포트폴리오 - 취업/이직에 바로 활용 가능

📊 완주하면 이런 것들을 할 수 있어요:

Vue.js + FastAPI + PostgreSQL 풀스택 개발
GitHub 기반 협업 및 코드 관리
Docker 배포 및 CI/CD 파이프라인 구축
CursorAI로 개발 생산성 10배 향상

## 📋 프로젝트 개요

### 🎯 프로젝트 목적
- HR팀 관리자가 교육자료를 업로드하고 관리할 수 있는 시스템
- 전사 직원들이 교육자료를 자유롭게 열람하고 학습할 수 있는 플랫폼
- **CursorAI를 활용하여** 비전공자도 실제 동작하는 웹사이트 구현

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
- **CursorAI** - AI 기반 코드 작성 도구 ⭐
- **Git** - 버전 관리
- **Docker** - 컨테이너화

---

## 📚 학습 가이드 (단계별)

### 🟢 **1단계: 기본 구현 (필수)**
```
Week 1-2: 기본 환경 구축 및 동작하는 시스템 만들기
```

| 순서 | 가이드 문서 | 소요 시간 | 설명 |
|------|-------------|----------|------|
| 1 | **[🤖 CursorAI 실전 가이드](docs/CURSOR_AI_GUIDE.md)** | 2시간 | AI 도구 활용법 마스터 |
| 2 | **[💾 데이터베이스 기본](docs/DATABASE_GUIDE.md)** | 3시간 | PostgreSQL 설치 및 기본 테이블 생성 |
| 3 | **[🐍 백엔드 기본](docs/BACKEND_GUIDE.md)** | 6시간 | FastAPI로 기본 API 구현 |
| 4 | **[🖥️ 프론트엔드 개발](docs/FRONTEND_GUIDE.md)** | 8시간 | Vue.js로 사용자 인터페이스 구현 |
| 5 | **[🖥️ GIT 소스 관리](docs/GIT_GUIDE.md)** | 꾸준히 | git을 이용하여 소스의 버전&형상 관리 |

### 🟡 **2단계: 고급 기능 (권장)**
```
Week 3-4: 운영 환경을 위한 고급 기능 추가
```

| 순서 | 가이드 문서 | 소요 시간 | 설명 |
|------|-------------|----------|------|
| 5 | **[💾 데이터베이스 고급](docs/DATABASE_ADVANCED.md)** | 4시간 | 성능 최적화, 백업, 모니터링 |
| 6 | **[🐍 백엔드 고급](docs/BACKEND_ADVANCED.md)** | 8시간 | 인증, 보안, 캐싱, 테스트 자동화 |
| 7 | **[🚀 배포 가이드](docs/DEPLOYMENT.md)** | 3시간 | Docker, CI/CD, 운영 환경 배포 |

---

## 🎯 학습 목표별 로드맵

### 🚀 **빠른 체험 (1주일)**
```bash
# 목표: 일단 동작하는 시스템 만들어보기
1. CursorAI 설치 및 기본 사용법 (30분)
2. 데이터베이스 기본 설정 (1시간)  
3. 백엔드 기본 API 구현 (3시간)
4. 프론트엔드 기본 화면 구현 (4시간)
5. 통합 테스트 (30분)
```

### 💼 **실무 준비 (3주일)**
```bash
# 목표: 실제 회사에서 사용 가능한 수준
1-7. 위의 모든 가이드 완주
8. 성능 최적화 및 보안 강화
9. 자동화된 테스트 및 배포 파이프라인 구축
10. 모니터링 및 운영 관리 시스템 구축
```

### 🎓 **포트폴리오용 (1개월)**
```bash
# 목표: 취업/이직용 완성도 높은 프로젝트
1-10. 위의 모든 과정 + 추가 기능
- SSO 연동 (Google, Microsoft)
- 고급 검색 기능 (전문 검색)
- 실시간 알림 시스템
- 모바일 앱 연동 준비
- 상세한 문서화 및 발표 자료
```

---

## 🚀 빠른 시작

### 1단계: 학습 가이드 레포지토리 클론
```bash
# 학습 가이드 다운로드
git clone https://github.com/your-username/baby-fullstack-developer.git
cd baby-fullstack-developer

# 가이드 문서 확인
ls docs/
```

### 2단계: 실제 프로젝트 레포지토리 생성
```bash
# 새 프로젝트 레포지토리 생성
mkdir hr-education-system
cd hr-education-system
git init

# 기본 구조 생성 (CursorAI와 함께)
mkdir -p frontend backend uploads
touch README.md docker-compose.yml
```

### 3단계: CursorAI 활용법 학습 ⭐
```bash
# 🤖 CursorAI 실전 가이드 먼저 읽기 (중요!)
# baby-fullstack-developer/docs/CURSOR_AI_GUIDE.md

# AI와 함께 개발하는 방법을 익히고 나서 다음 단계로 진행
```

### 4단계: 프론트엔드 개발
```bash
# 🖥️ 프론트엔드 가이드 따라하기
# baby-fullstack-developer/docs/FRONTEND_GUIDE.md

# hr-education-system 레포지토리에서 구현
cd hr-education-system/frontend
npm install
npm run dev
```

### 5단계: 데이터베이스 설정
```bash
# 💾 데이터베이스 가이드 따라하기
# baby-fullstack-developer/docs/DATABASE_GUIDE.md

# PostgreSQL 설치 → DBeaver 연결 → 테이블 생성
```

### 6단계: 백엔드 개발
```bash
# 🐍 백엔드 가이드 따라하기  
# baby-fullstack-developer/docs/BACKEND_GUIDE.md

# hr-education-system 레포지토리에서 구현
cd hr-education-system/backend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
uvicorn app.main:app --reload
```

---

## 🤖 CursorAI 특화 기능

이 프로젝트는 **CursorAI의 강력한 기능**을 최대한 활용하도록 설계되었습니다:

### ✨ **AI 페어 프로그래밍**
- 단계별 구현 가이드에 **실제 프롬프트 예시** 포함
- 에러 해결 시 AI와 협업하는 방법
- 코드 리뷰 및 최적화 요청 패턴

### 🎯 **스마트 개발 워크플로우**
```bash
# 예시: 컴포넌트 구현 시
1. 요구사항을 명확히 정의
2. CursorAI에게 구체적인 프롬프트 제공
3. 생성된 코드 리뷰 및 이해
4. 테스트 및 개선 반복
```

### 📚 **학습 가속화**
- **초보자**: AI가 설명해주는 코드로 빠른 학습
- **경험자**: 반복 작업 자동화로 핵심 로직에 집중
- **모든 레벨**: 최신 베스트 프랙티스 자동 적용

---

## 🎨 프로젝트 구조

### 📚 **학습 가이드 레포지토리** (현재 위치)
```
baby-fullstack-developer/
├── 📋 README.md                 # 전체 가이드 및 로드맵
├── 📚 docs/                     # 단계별 학습 문서
│   ├── CURSOR_AI_GUIDE.md       # 🆕 CursorAI 실전 활용법
│   ├── DATABASE_GUIDE.md        # 데이터베이스 기본 (초보자용)
│   ├── DATABASE_ADVANCED.md     # 데이터베이스 고급 (운영환경용)
│   ├── BACKEND_GUIDE.md         # 백엔드 기본 (초보자용)
│   ├── BACKEND_ADVANCED.md      # 백엔드 고급 (엔터프라이즈용)
│   ├── FRONTEND_GUIDE.md        # 프론트엔드 개발
│   ├── GIT_GUIDE.md             # Git 사용법 기본&고급
│   └── DEPLOYMENT.md            # 배포 및 운영(아직 생성 X)
└── 🎯 templates/                # 초기 템플릿 파일들 (선택사항)(아직 생성 X)
    ├── .env.example
    ├── docker-compose.template.yml
    └── package.template.json
```

### 🏗️ **실제 프로젝트 레포지토리** (별도 생성)
```
hr-education-system/
├── 🖥️ frontend/                 # Vue.js 프론트엔드
│   ├── src/
│   │   ├── components/          # 재사용 가능한 컴포넌트
│   │   ├── views/              # 페이지 컴포넌트
│   │   ├── stores/             # 상태 관리 (Pinia)
│   │   └── services/           # API 통신
│   └── package.json
├── 🐍 backend/                   # FastAPI 백엔드
│   ├── app/
│   │   ├── models/             # 데이터베이스 모델
│   │   ├── schemas/            # API 스키마
│   │   ├── routers/            # API 라우터
│   │   └── core/               # 핵심 기능 (보안, 설정)
│   └── requirements.txt
├── 📁 uploads/                   # 업로드된 파일 저장소
├── 🐳 docker-compose.yml         # 개발 환경 설정
├── 🐳 docker-compose.prod.yml    # 운영 환경 설정
├── ⚙️ .github/workflows/        # CI/CD 파이프라인
│   └── ci-cd.yml
└── 📝 README.md                 # 프로젝트별 README
```

### 🔗 **두 레포지토리 연결**
1. **학습 단계**: `baby-fullstack-developer` 가이드 문서 순서대로 학습
2. **실제 구현**: `hr-education-system` 레포지토리에서 코딩 실습
3. **CursorAI 활용**: 각 레포지토리의 컨텍스트에 집중하여 AI 도움 최적화

---

## 🎯 학습 성과

이 프로젝트를 완주하면 다음을 습득할 수 있습니다:

### 🛠️ **기술적 역량**
- **소스 관리**: 가장 빈번하게 사용되는 GIT을 잘 사용할 수 있음
- **풀스택 웹 개발**: 프론트엔드 + 백엔드 + 데이터베이스
- **모던 개발 스택**: Vue 3, FastAPI, PostgreSQL
- **API 설계**: RESTful API 설계 및 구현
- **데이터베이스 설계**: 정규화, 인덱싱, 성능 최적화
- **보안**: 인증, 인가, 파일 보안, API 보안

### 🤖 **AI 시대 개발 역량**
- **CursorAI 마스터**: AI 기반 개발 워크플로우
- **프롬프트 엔지니어링**: AI와 효과적으로 소통하는 방법
- **AI 페어 프로그래밍**: 개발 생산성 10배 향상

### 💼 **실무 경험**
- **Git 워크플로우**: 브랜치 전략, PR 리뷰
- **테스트 자동화**: 단위 테스트, 통합 테스트
- **CI/CD**: 자동화된 빌드 및 배포
- **모니터링**: 로깅, 메트릭, 헬스체크

---

## 🏃‍♀️ 학습 진행 가이드

### 📅 **일정별 체크리스트**

#### Week 1: AI 도구 + 기본 환경 구축
- [ ] **Day 1**: CursorAI 설치 및 기본 사용법 익히기
- [ ] **Day 2**: PostgreSQL 설치 및 데이터베이스 설정
- [ ] **Day 3-4**: FastAPI 백엔드 기본 구현
- [ ] **Day 5-7**: Vue.js 프론트엔드 기본 구현

#### Week 2: 기능 완성 및 통합
- [ ] **Day 8-10**: 파일 업로드/다운로드 기능 구현
- [ ] **Day 11-12**: 검색 및 카테고리 기능 구현  
- [ ] **Day 13-14**: 프론트엔드-백엔드 통합 테스트

#### Week 3-4: 고급 기능 (선택)
- [ ] **Week 3**: 인증 시스템, 보안 강화, 성능 최적화
- [ ] **Week 4**: 테스트 자동화, CI/CD, 배포

### 🎯 **마일스톤**
1. **🟢 MVP 완성**: 기본 기능 모두 동작
2. **🟡 베타 버전**: 실제 사용 가능한 수준
3. **🔴 프로덕션**: 운영 환경 배포 가능

---

## 🆘 도움이 필요할 때

### 🤖 **CursorAI 활용하기**
```bash
# 막혔을 때 CursorAI에게 물어보기
"다음 에러가 발생했어: [에러 메시지]
코드는: [관련 코드]
어떻게 해결할까?"
```

### 📚 **추가 학습 리소스**
- [FastAPI 공식 문서](https://fastapi.tiangolo.com/)
- [Vue.js 공식 가이드](https://vuejs.org/guide/)
- [PostgreSQL 튜토리얼](https://www.postgresql.org/docs/current/tutorial.html)
- [Tailwind CSS 문서](https://tailwindcss.com/docs)

### 💬 **커뮤니티**
- GitHub Issues를 통한 질문/답변
- 개발 진행 상황 공유
- 코드 리뷰 요청

---

## 🔮 향후 확장 아이디어

### 🚀 **Phase 2: 고급 기능**
- **실시간 알림**: WebSocket 기반 실시간 업데이트
- **고급 검색**: ElasticSearch 연동, 전문 검색
- **소셜 기능**: 댓글, 좋아요, 즐겨찾기
- **분석 대시보드**: 사용자 행동 분석, 통계

### 🌟 **Phase 3: 확장**
- **모바일 앱**: React Native 또는 Flutter
- **마이크로서비스**: 아키텍처 분리 및 확장
- **AI 추천**: 개인화된 교육자료 추천
- **다국어 지원**: i18n 국제화

---

## 📞 Contact & Support

### 🙋‍♂️ **프로젝트 관리자**
- **GitHub**: [@your-username](https://github.com/your-username)
- **Email**: your-email@example.com

### 🤝 **기여하기**
1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 라이센스

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**🎉 성균관대 빅데이터학과 진학을 축하드립니다!**  

이 프로젝트가 개발자로서의 첫 걸음에 도움이 되길 바랍니다.

---

*Made with ❤️ and CursorAI*