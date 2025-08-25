# 💾 데이터베이스 설정 가이드 (기본편)

> PostgreSQL + DBeaver를 이용한 데이터베이스 구축 기본 가이드

## 📋 목차
1. [PostgreSQL 설치](#postgresql-설치)
2. [DBeaver 설치 및 연결](#dbeaver-설치-및-연결)
3. [기본 데이터베이스 및 테이블 생성](#기본-데이터베이스-및-테이블-생성)
4. [샘플 데이터 삽입](#샘플-데이터-삽입)
5. [기본 CRUD 쿼리](#기본-crud-쿼리)
6. [CursorAI 활용 팁](#cursorai-활용-팁)

---

## 🐘 PostgreSQL 설치

### Windows 설치

#### 1단계: PostgreSQL 다운로드
```bash
# 공식 사이트에서 다운로드
https://www.postgresql.org/download/windows/
```

#### 2단계: 설치 진행
```
1. 다운로드한 설치파일 실행
2. 설치 경로: 기본값 사용
3. 구성요소: 모든 항목 체크 (기본값)
4. 비밀번호 설정: postgres 사용자의 비밀번호 입력 (꼭 기억하세요!)
5. 포트: 5432 (기본값)
6. 로케일: 기본값
```

#### 3단계: 설치 확인
```bash
# 명령 프롬프트에서 확인
psql --version
```

### macOS 설치 (Homebrew)
```bash
# PostgreSQL 설치
brew install postgresql@15

# 서비스 시작
brew services start postgresql@15
```

### Linux (Ubuntu) 설치
```bash
# PostgreSQL 설치
sudo apt update
sudo apt install postgresql postgresql-contrib

# 서비스 시작
sudo systemctl start postgresql
```

---

## 🦫 DBeaver 설치 및 연결

### DBeaver 설치
1. https://dbeaver.io/download/ 에서 Community Edition 다운로드
2. 설치 후 실행

### PostgreSQL 연결 설정
```
1. "새 데이터베이스 연결" 클릭
2. PostgreSQL 선택
3. 연결 정보 입력:
   - 서버 호스트: localhost
   - 포트: 5432
   - 데이터베이스: postgres
   - 사용자명: postgres
   - 비밀번호: [설치 시 설정한 비밀번호]
4. "연결 테스트" 후 완료
```

---

## 🏗️ 기본 데이터베이스 및 테이블 생성

### 1. 프로젝트 데이터베이스 생성
```sql
-- hr_education_system 데이터베이스 생성
CREATE DATABASE hr_education_system
    WITH 
    OWNER = postgres
    ENCODING = 'UTF8';
```

### 2. 새 데이터베이스에 연결
DBeaver에서 새 연결을 만들되, 데이터베이스 이름을 `hr_education_system`으로 설정

### 3. 기본 테이블 생성

#### 카테고리 테이블
```sql
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

#### 사용자 테이블 (관리자용)
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    is_admin BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

#### 교육자료 테이블
```sql
CREATE TABLE materials (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    category_id INTEGER REFERENCES categories(id),
    
    -- 파일 정보
    file_path VARCHAR(500) NOT NULL,
    file_name VARCHAR(255) NOT NULL,
    original_file_name VARCHAR(255) NOT NULL,
    file_size BIGINT NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    
    -- 기본 통계
    view_count INTEGER DEFAULT 0,
    download_count INTEGER DEFAULT 0,
    
    -- 관리 정보
    uploaded_by INTEGER REFERENCES users(id),
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🗃️ 샘플 데이터 삽입

### 관리자 계정 생성
```sql
INSERT INTO users (username, email, full_name, is_admin) VALUES
('admin', 'admin@company.com', 'HR 관리자', TRUE);
```

### 카테고리 생성
```sql
INSERT INTO categories (name, description) VALUES
('신입사원 교육', '신입사원을 위한 기본 교육 자료'),
('리더십', '관리자 및 팀리더를 위한 리더십 교육'),
('컴플라이언스', '법규 준수 및 윤리 교육 자료'),
('IT 기술', 'IT 및 소프트웨어 기술 교육'),
('비즈니스 스킬', '업무 효율성 향상 교육'),
('안전 교육', '산업 안전 및 건강 관련 교육');
```

### 샘플 교육자료
```sql
INSERT INTO materials (
    title, description, category_id, 
    file_path, file_name, original_file_name, 
    file_size, mime_type, uploaded_by
) VALUES
(
    '2024년 신입사원 오리엔테이션', 
    '회사 소개 및 기본 업무 프로세스',
    1, 
    '/uploads/orientation_2024.pdf', 'orientation_2024.pdf', '신입사원_오리엔테이션.pdf',
    2560000, 'application/pdf', 1
),
(
    '효과적인 팀 관리 전략',
    '팀리더를 위한 관리 방법론',
    2,
    '/uploads/team_management.pptx', 'team_management.pptx', '팀_관리_전략.pptx',
    4250000, 'application/vnd.openxmlformats-officedocument.presentationml.presentation', 1
),
(
    'Git 기초 사용법',
    'Git 버전 관리 시스템 기본 사용법',
    4,
    '/uploads/git_basics.pdf', 'git_basics.pdf', 'Git_기초.pdf',
    1890000, 'application/pdf', 1
);
```

---

## 📝 기본 CRUD 쿼리

### 조회 (Read)
```sql
-- 모든 교육자료 조회
SELECT * FROM materials WHERE is_active = true;

-- 카테고리별 자료 조회
SELECT m.title, m.description, c.name as category_name
FROM materials m
JOIN categories c ON m.category_id = c.id
WHERE m.is_active = true;

-- 특정 자료 상세 조회
SELECT * FROM materials WHERE id = 1;
```

### 생성 (Create)
```sql
-- 새 카테고리 추가
INSERT INTO categories (name, description) 
VALUES ('새로운 카테고리', '카테고리 설명');

-- 새 교육자료 추가
INSERT INTO materials (title, description, category_id, file_path, file_name, original_file_name, file_size, mime_type, uploaded_by)
VALUES ('새 교육자료', '자료 설명', 1, '/uploads/new_file.pdf', 'new_file.pdf', '새_자료.pdf', 1000000, 'application/pdf', 1);
```

### 수정 (Update)
```sql
-- 교육자료 제목 수정
UPDATE materials 
SET title = '수정된 제목' 
WHERE id = 1;

-- 다운로드 횟수 증가
UPDATE materials 
SET download_count = download_count + 1 
WHERE id = 1;
```

### 삭제 (Delete)
```sql
-- 교육자료 비활성화 (소프트 삭제)
UPDATE materials 
SET is_active = false 
WHERE id = 1;

-- 실제 삭제 (주의!)
-- DELETE FROM materials WHERE id = 1;
```

---

## 🤖 CursorAI 활용 팁

### 테이블 생성 시
```
"PostgreSQL로 교육자료 관리 테이블을 만들어줘.
필요한 필드:
- id (자동 증가)
- 제목, 설명
- 파일 정보 (경로, 이름, 크기)
- 카테고리 연결
- 생성일시

기본적인 테이블만 일단 만들어줘."
```

### 샘플 데이터 생성 시
```
"방금 만든 테이블들에 넣을 샘플 데이터 만들어줘.
카테고리 5개, 각 카테고리당 교육자료 2-3개씩.
실제 회사에서 사용할 만한 현실적인 내용으로 해줘."
```

### 쿼리 작성 시
```
"다음 기능을 위한 SQL 쿼리 만들어줘:
1. 카테고리별 자료 개수 조회
2. 인기 자료 TOP 5 (다운로드 수 기준)
3. 최근 업로드된 자료 10개

초보자도 이해하기 쉽게 주석도 달아줘."
```

---

## 🔍 데이터 확인하기

### 테이블 목록 확인
```sql
-- 데이터베이스의 모든 테이블 보기
SELECT table_name FROM information_schema.tables 
WHERE table_schema = 'public';
```

### 데이터 개수 확인
```sql
-- 각 테이블의 데이터 개수
SELECT 
    'categories' as table_name, COUNT(*) as count FROM categories
UNION ALL
SELECT 
    'materials' as table_name, COUNT(*) as count FROM materials
UNION ALL
SELECT 
    'users' as table_name, COUNT(*) as count FROM users;
```

### 관계 확인
```sql
-- 카테고리별 자료 개수
SELECT 
    c.name as category_name,
    COUNT(m.id) as material_count
FROM categories c
LEFT JOIN materials m ON c.id = m.category_id AND m.is_active = true
GROUP BY c.id, c.name
ORDER BY material_count DESC;
```

---

## 🚨 기본 문제 해결

### PostgreSQL 연결 안 될 때
```bash
# Windows에서 서비스 확인
services.msc 에서 PostgreSQL 서비스 확인

# 서비스 시작
net start postgresql-x64-15
```

### DBeaver 연결 실패 시
```
1. 호스트: localhost (127.0.0.1로 시도)
2. 포트: 5432 확인
3. 비밀번호 다시 확인
4. 방화벽 설정 확인
```

### 테이블 생성 오류
```sql
-- 테이블이 이미 존재하는 경우
DROP TABLE IF EXISTS materials CASCADE;
DROP TABLE IF EXISTS categories CASCADE;
DROP TABLE IF EXISTS users CASCADE;

-- 다시 생성
```

---

## 📝 다음 단계

기본 데이터베이스 설정이 완료되었다면:

1. ✅ **데이터 확인**
   ```sql
   SELECT * FROM categories;
   SELECT * FROM materials;
   SELECT * FROM users;
   ```

2. ✅ **기본 쿼리 연습**
   - 각 CRUD 쿼리를 직접 실행해보기
   - 다양한 조건으로 데이터 조회해보기

3. 🔄 **다음 가이드**
   - **백엔드 개발**: [BACKEND_GUIDE.md](BACKEND_GUIDE.md)
   - **고급 DB 기능**: [DATABASE_ADVANCED.md](DATABASE_ADVANCED.md) (선택)

---

## 🔗 추가 학습 자료

- [PostgreSQL 기본 문법](https://www.postgresql.org/docs/current/tutorial.html)
- [DBeaver 사용법](https://dbeaver.io/docs/)
- [SQL 기초 튜토리얼](https://www.w3schools.com/sql/)

---

**🎉 기본 데이터베이스 완성!**  
이제 백엔드 API를 만들어서 데이터베이스와 연결해보세요!