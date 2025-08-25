# 💾 데이터베이스 설정 가이드

> PostgreSQL + DBeaver를 이용한 데이터베이스 구축 완전 가이드

## 📋 목차
1. [PostgreSQL 설치](#postgresql-설치)
2. [DBeaver 설치 및 연결](#dbeaver-설치-및-연결)
3. [데이터베이스 및 테이블 생성](#데이터베이스-및-테이블-생성)
4. [샘플 데이터 삽입](#샘플-데이터-삽입)
5. [CursorAI 활용 팁](#cursorai-활용-팁)
6. [문제 해결](#문제-해결)

---

## 🐘 PostgreSQL 설치

### Windows 설치

#### 1단계: PostgreSQL 다운로드
```bash
# 공식 사이트에서 다운로드
https://www.postgresql.org/download/windows/

# 또는 직접 링크
https://get.enterprisedb.com/postgresql/postgresql-15.4-1-windows-x64.exe
```

#### 2단계: 설치 진행
```
1. 다운로드한 설치파일 실행
2. 설치 경로: C:\Program Files\PostgreSQL\15 (기본값 사용)
3. 구성요소 선택: 모든 항목 체크 (기본값)
4. 데이터 디렉터리: C:\Program Files\PostgreSQL\15\data (기본값)
5. 비밀번호 설정: postgres 사용자의 비밀번호 입력 (꼭 기억하세요!)
6. 포트: 5432 (기본값)
7. 로케일: Korean, Korea (또는 기본값)
```

#### 3단계: 설치 확인
```bash
# 명령 프롬프트(cmd) 실행
# PostgreSQL 버전 확인
psql --version

# 만약 명령어를 찾을 수 없다면 환경변수 추가 필요
# 시스템 환경 변수 > Path에 추가: C:\Program Files\PostgreSQL\15\bin
```

### macOS 설치

#### Homebrew 사용
```bash
# Homebrew 설치 (없을 경우)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# PostgreSQL 설치
brew install postgresql@15

# 서비스 시작
brew services start postgresql@15

# 데이터베이스 초기화
initdb /usr/local/var/postgres
```

### Linux (Ubuntu) 설치

```bash
# 패키지 목록 업데이트
sudo apt update

# PostgreSQL 설치
sudo apt install postgresql postgresql-contrib

# PostgreSQL 서비스 시작
sudo systemctl start postgresql
sudo systemctl enable postgresql

# postgres 사용자로 전환
sudo -u postgres psql
```

---

## 🦫 DBeaver 설치 및 연결

### DBeaver 설치

#### 1단계: DBeaver 다운로드
```bash
# 공식 사이트
https://dbeaver.io/download/

# Community Edition (무료) 다운로드
# Windows: dbeaver-ce-latest-x86_64-setup.exe
# macOS: dbeaver-ce-latest-macos-x86_64.dmg
# Linux: dbeaver-ce_latest_amd64.deb
```

#### 2단계: 설치 및 실행
```
1. 다운로드한 파일 실행
2. 설치 마법사 따라 진행 (기본 설정 사용)
3. DBeaver 실행
4. 첫 실행시 드라이버 다운로드 허용
```

### PostgreSQL 연결 설정

#### 1단계: 새 연결 생성
```
1. DBeaver 실행
2. 좌상단 "새 데이터베이스 연결" 버튼 클릭 (플러그 모양)
3. PostgreSQL 선택 후 "다음"
```

#### 2단계: 연결 정보 입력
```
서버 호스트: localhost
포트: 5432
데이터베이스: postgres (기본 데이터베이스)
사용자명: postgres
비밀번호: [설치 시 설정한 비밀번호]

"연결 테스트" 버튼 클릭하여 연결 확인
성공하면 "완료" 클릭
```

#### 3단계: 연결 확인
```
1. 왼쪽 데이터베이스 탐색기에서 PostgreSQL 연결 확인
2. 연결 클릭하여 데이터베이스 목록 확인
3. "postgres" 데이터베이스 확장하여 스키마 확인
```

---

## 🏗️ 데이터베이스 및 테이블 생성

### 프로젝트 데이터베이스 생성

#### 1단계: 새 데이터베이스 생성
```sql
-- DBeaver SQL 콘솔에서 실행
-- (Ctrl+Enter 또는 오른쪽 클릭 > "스크립트 실행")

CREATE DATABASE hr_education_system
    WITH 
    OWNER = postgres
    ENCODING = 'UTF8'
    LC_COLLATE = 'Korean_Korea.949'
    LC_CTYPE = 'Korean_Korea.949'
    TABLESPACE = pg_default
    CONNECTION LIMIT = -1;
```

#### 2단계: 새 데이터베이스 연결
```
1. DBeaver에서 새 연결 생성 (위 과정 반복)
2. 데이터베이스 이름을 "hr_education_system"으로 변경
3. 나머지 설정은 동일하게 유지
4. 연결 테스트 후 완료
```

### 테이블 스키마 생성

#### 1단계: 카테고리 테이블
```sql
-- categories 테이블 생성
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    icon VARCHAR(50), -- 아이콘 클래스명 저장
    color VARCHAR(20), -- 카테고리 색상
    sort_order INTEGER DEFAULT 0, -- 정렬 순서
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 카테고리 테이블 인덱스
CREATE INDEX idx_categories_active ON categories(is_active);
CREATE INDEX idx_categories_sort_order ON categories(sort_order);
```

#### 2단계: 사용자 테이블 (향후 SSO 연동용)
```sql
-- users 테이블 (현재는 관리자만 사용)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    department VARCHAR(100), -- 부서
    position VARCHAR(100), -- 직급
    is_admin BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    last_login TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 사용자 테이블 인덱스
CREATE INDEX idx_users_active ON users(is_active);
CREATE INDEX idx_users_admin ON users(is_admin);
CREATE INDEX idx_users_email ON users(email);
```

#### 3단계: 교육자료 테이블
```sql
-- materials 테이블
CREATE TABLE materials (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    content TEXT, -- 자료 내용 (선택사항)
    category_id INTEGER REFERENCES categories(id) ON DELETE SET NULL,
    
    -- 파일 정보
    file_path VARCHAR(500) NOT NULL,
    file_name VARCHAR(255) NOT NULL,
    original_file_name VARCHAR(255) NOT NULL, -- 사용자가 업로드한 원본 파일명
    file_size BIGINT NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    
    -- 메타데이터
    tags TEXT[], -- 태그 배열
    difficulty_level INTEGER CHECK (difficulty_level BETWEEN 1 AND 5), -- 난이도 (1-5)
    estimated_time INTEGER, -- 예상 소요시간 (분)
    
    -- 관리 정보
    uploaded_by INTEGER REFERENCES users(id),
    is_featured BOOLEAN DEFAULT FALSE, -- 추천 자료 여부
    is_active BOOLEAN DEFAULT TRUE,
    view_count INTEGER DEFAULT 0, -- 조회수
    download_count INTEGER DEFAULT 0, -- 다운로드 수
    
    -- 시간 정보
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- materials 테이블 인덱스
CREATE INDEX idx_materials_category ON materials(category_id);
CREATE INDEX idx_materials_active ON materials(is_active);
CREATE INDEX idx_materials_featured ON materials(is_featured);
CREATE INDEX idx_materials_title ON materials USING gin(to_tsvector('korean', title));
CREATE INDEX idx_materials_tags ON materials USING gin(tags);
CREATE INDEX idx_materials_created_at ON materials(created_at DESC);
```

#### 4단계: 다운로드 로그 테이블
```sql
-- download_logs 테이블
CREATE TABLE download_logs (
    id SERIAL PRIMARY KEY,
    material_id INTEGER REFERENCES materials(id) ON DELETE CASCADE,
    user_ip INET, -- 사용자 IP (로그인 없이 사용하므로)
    user_agent TEXT, -- 브라우저 정보
    download_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    -- 추가 로그 정보
    referer_url TEXT, -- 참조 페이지
    session_id VARCHAR(100) -- 세션 ID
);

-- download_logs 테이블 인덱스
CREATE INDEX idx_download_logs_material ON download_logs(material_id);
CREATE INDEX idx_download_logs_date ON download_logs(download_at DESC);
CREATE INDEX idx_download_logs_ip ON download_logs(user_ip);
```

#### 5단계: 뷰어 통계 테이블
```sql
-- view_logs 테이블 (페이지 조회 로그)
CREATE TABLE view_logs (
    id SERIAL PRIMARY KEY,
    material_id INTEGER REFERENCES materials(id) ON DELETE CASCADE,
    user_ip INET,
    user_agent TEXT,
    viewed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    session_id VARCHAR(100),
    view_duration INTEGER -- 페이지 체류 시간 (초)
);

-- view_logs 테이블 인덱스
CREATE INDEX idx_view_logs_material ON view_logs(material_id);
CREATE INDEX idx_view_logs_date ON view_logs(viewed_at DESC);
```

### 트리거 생성 (자동 업데이트)

#### updated_at 자동 갱신 함수
```sql
-- updated_at 자동 갱신 함수 생성
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

-- 트리거 적용
CREATE TRIGGER update_categories_updated_at 
    BEFORE UPDATE ON categories 
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_users_updated_at 
    BEFORE UPDATE ON users 
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();

CREATE TRIGGER update_materials_updated_at 
    BEFORE UPDATE ON materials 
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

#### 다운로드 카운터 자동 증가 함수
```sql
-- 다운로드 수 자동 증가 함수
CREATE OR REPLACE FUNCTION increment_download_count()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE materials 
    SET download_count = download_count + 1 
    WHERE id = NEW.material_id;
    RETURN NEW;
END;
$$ language 'plpgsql';

-- 트리거 적용
CREATE TRIGGER increment_material_downloads
    AFTER INSERT ON download_logs
    FOR EACH ROW EXECUTE FUNCTION increment_download_count();
```

---

## 🔧 샘플 데이터 삽입

### 관리자 계정 생성
```sql
-- 관리자 사용자 생성
INSERT INTO users (username, email, full_name, department, position, is_admin) VALUES
('admin', 'admin@company.com', 'HR 관리자', 'Human Resources', '팀장', TRUE),
('hr.manager', 'hr.manager@company.com', '김인사', 'Human Resources', '매니저', TRUE);
```

### 카테고리 데이터
```sql
-- 교육 카테고리 생성
INSERT INTO categories (name, description, icon, color, sort_order) VALUES
('신입사원 교육', '신입사원을 위한 회사 소개 및 기본 교육 자료', 'users', '#3B82F6', 1),
('리더십', '관리자 및 팀리더를 위한 리더십 교육', 'crown', '#10B981', 2),
('컴플라이언스', '법규 준수 및 윤리, 보안 교육 자료', 'shield-check', '#F59E0B', 3),
('IT 기술', 'IT 및 소프트웨어 기술 관련 교육', 'monitor', '#8B5CF6', 4),
('비즈니스 스킬', '업무 효율성 및 커뮤니케이션 향상', 'briefcase', '#EF4444', 5),
('안전 교육', '산업 안전 및 건강 관련 교육', 'hard-hat', '#06B6D4', 6);
```

### 샘플 교육자료 데이터
```sql
-- 샘플 교육자료 (실제 파일이 없으므로 더미 데이터)
INSERT INTO materials (
    title, description, category_id, 
    file_path, file_name, original_file_name, 
    file_size, mime_type, 
    tags, difficulty_level, estimated_time, 
    uploaded_by, is_featured
) VALUES
(
    '2024년 신입사원 오리엔테이션', 
    '회사 소개, 조직도, 업무 프로세스 등 신입사원이 알아야 할 기본 정보',
    1, 
    '/uploads/orientation_2024.pdf', 'orientation_2024.pdf', '2024년_신입사원_오리엔테이션.pdf',
    2560000, 'application/pdf',
    ARRAY['신입사원', '오리엔테이션', '회사소개'], 1, 60,
    1, TRUE
),
(
    '효과적인 팀 관리 전략',
    '팀리더를 위한 팀 관리 및 동기부여 방법론',
    2,
    '/uploads/team_management.pptx', 'team_management.pptx', '효과적인_팀_관리_전략.pptx',
    4250000, 'application/vnd.openxmlformats-officedocument.presentationml.presentation',
    ARRAY['리더십', '팀관리', '동기부여'], 3, 45,
    1, TRUE
),
(
    '개인정보보호법 준수 가이드',
    '개인정보보호법의 주요 내용과 준수사항, 위반 시 제재조치',
    3,
    '/uploads/privacy_guide.pdf', 'privacy_guide.pdf', '개인정보보호법_준수_가이드.pdf',
    1890000, 'application/pdf',
    ARRAY['컴플라이언스', '개인정보보호', '법규'], 2, 30,
    2, FALSE
),
(
    'Git 기초 및 협업 방법',
    'Git 버전 관리 시스템의 기본 사용법과 팀 협업을 위한 워크플로우',
    4,
    '/uploads/git_basics.mp4', 'git_basics.mp4', 'Git_기초_및_협업_방법.mp4',
    125000000, 'video/mp4',
    ARRAY['Git', '버전관리', '협업', 'IT'], 2, 90,
    1, TRUE
),
(
    '효과적인 프레젠테이션 스킬',
    '청중을 사로잡는 프레젠테이션 기법과 실습 가이드',
    5,
    '/uploads/presentation_skills.pdf', 'presentation_skills.pdf', '효과적인_프레젠테이션_스킬.pdf',
    3200000, 'application/pdf',
    ARRAY['프레젠테이션', '커뮤니케이션', '비즈니스'], 2, 40,
    2, FALSE
),
(
    '사무실 화재 대피 요령',
    '화재 발생 시 대피 절차와 소화기 사용법 등 안전 교육',
    6,
    '/uploads/fire_safety.pdf', 'fire_safety.pdf', '사무실_화재_대피_요령.pdf',
    1650000, 'application/pdf',
    ARRAY['안전교육', '화재대피', '응급처치'], 1, 20,
    1, TRUE
);
```

### 샘플 로그 데이터
```sql
-- 샘플 다운로드 로그 (최근 30일)
INSERT INTO download_logs (material_id, user_ip, user_agent, download_at) VALUES
(1, '192.168.1.100', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36', NOW() - INTERVAL '1 day'),
(1, '192.168.1.101', 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36', NOW() - INTERVAL '2 days'),
(2, '192.168.1.102', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36', NOW() - INTERVAL '3 days'),
(4, '192.168.1.103', 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36', NOW() - INTERVAL '1 week'),
(1, '192.168.1.104', 'Mozilla/5.0 (iPhone; CPU iPhone OS 16_0 like Mac OS X)', NOW() - INTERVAL '5 days'),
(3, '192.168.1.105', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36', NOW() - INTERVAL '1 week'),
(6, '192.168.1.106', 'Mozilla/5.0 (iPad; CPU OS 16_0 like Mac OS X) AppleWebKit/605.1.15', NOW() - INTERVAL '2 weeks');

-- 샘플 조회 로그
INSERT INTO view_logs (material_id, user_ip, user_agent, viewed_at, view_duration) VALUES
(1, '192.168.1.100', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36', NOW() - INTERVAL '1 day', 120),
(1, '192.168.1.101', 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36', NOW() - INTERVAL '2 days', 85),
(2, '192.168.1.102', 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36', NOW() - INTERVAL '3 days', 200),
(4, '192.168.1.103', 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36', NOW() - INTERVAL '1 week', 450),
(3, '192.168.1.104', 'Mozilla/5.0 (iPhone; CPU iPhone OS 16_0 like Mac OS X)', NOW() - INTERVAL '5 days', 180);
```

---

## 🤖 CursorAI 활용 팁

### 데이터베이스 설계 시 유용한 프롬프트

#### 테이블 설계 요청
```
"PostgreSQL에서 교육자료 관리 시스템을 위한 테이블을 설계해줘.
- 카테고리별 자료 분류
- 파일 메타데이터 저장
- 다운로드 통계 추적
- 향후 SSO 연동을 위한 사용자 테이블
각 테이블의 관계와 인덱스도 포함해줘."
```

#### 쿼리 최적화 요청
```
"다음 쿼리의 성능을 개선해줘:
[느린 쿼리 붙여넣기]

어떤 인덱스를 추가하면 좋을지도 알려줘."
```

#### 데이터 마이그레이션 스크립트
```
"기존 Excel 파일의 교육자료 목록을 PostgreSQL 테이블로 마이그레이션하는 스크립트 만들어줘.
Excel 컬럼: 제목, 카테고리, 파일명, 업로드날짜, 설명"
```

### 데이터베이스 관리 프롬프트

#### 백업 스크립트 생성
```
"PostgreSQL 데이터베이스 자동 백업 스크립트를 만들어줘.
- 매일 밤 12시에 실행
- 7일간 백업 파일 보관
- 백업 성공/실패 이메일 알림"
```

#### 성능 모니터링 쿼리
```
"PostgreSQL에서 다음을 확인하는 쿼리 만들어줘:
- 가장 많이 조회/다운로드된 자료 TOP 10
- 카테고리별 자료 분포
- 월별 다운로드 통계
- 느린 쿼리 목록"
```

---

## 🔍 유용한 SQL 쿼리 모음

### 관리자용 통계 쿼리

#### 1. 인기 자료 TOP 10
```sql
-- 다운로드 수 기준 인기 자료
SELECT 
    m.title,
    c.name as category_name,
    m.download_count,
    m.view_count,
    m.created_at
FROM materials m
LEFT JOIN categories c ON m.category_id = c.id
WHERE m.is_active = true
ORDER BY m.download_count DESC, m.view_count DESC
LIMIT 10;
```

#### 2. 카테고리별 자료 통계
```sql
-- 카테고리별 자료 수와 다운로드 통계
SELECT 
    c.name as category_name,
    COUNT(m.id) as material_count,
    SUM(m.download_count) as total_downloads,
    AVG(m.download_count) as avg_downloads,
    SUM(m.view_count) as total_views
FROM categories c
LEFT JOIN materials m ON c.id = m.category_id AND m.is_active = true
WHERE c.is_active = true
GROUP BY c.id, c.name, c.sort_order
ORDER BY c.sort_order;
```

#### 3. 월별 업로드/다운로드 추이
```sql
-- 최근 12개월 업로드/다운로드 통계
SELECT 
    DATE_TRUNC('month', created_at) as month,
    COUNT(*) as uploaded_materials,
    SUM(download_count) as total_downloads
FROM materials
WHERE created_at >= NOW() - INTERVAL '12 months'
    AND is_active = true
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month DESC;
```

#### 4. 일별 다운로드 로그 분석
```sql
-- 최근 30일 일별 다운로드 추이
SELECT 
    DATE(download_at) as date,
    COUNT(*) as download_count,
    COUNT(DISTINCT user_ip) as unique_users
FROM download_logs
WHERE download_at >= NOW() - INTERVAL '30 days'
GROUP BY DATE(download_at)
ORDER BY date DESC;
```

### 데이터 정리 쿼리

#### 1. 중복 다운로드 로그 정리
```sql
-- 같은 IP에서 1시간 내 중복 다운로드 제거
DELETE FROM download_logs
WHERE id IN (
    SELECT id FROM (
        SELECT id,
               ROW_NUMBER() OVER (
                   PARTITION BY material_id, user_ip, 
                   DATE_TRUNC('hour', download_at)
                   ORDER BY download_at
               ) as rn
        FROM download_logs
    ) t
    WHERE rn > 1
);
```

#### 2. 오래된 로그 데이터 아카이브
```sql
-- 1년 이상 된 로그 데이터 별도 테이블로 이동
CREATE TABLE download_logs_archive (LIKE download_logs);

INSERT INTO download_logs_archive
SELECT * FROM download_logs
WHERE download_at < NOW() - INTERVAL '1 year';

DELETE FROM download_logs
WHERE download_at < NOW() - INTERVAL '1 year';
```

---

## 🚨 문제 해결

### 일반적인 문제들

#### 1. PostgreSQL 서비스가 시작되지 않는 경우

**Windows:**
```bash
# 서비스 관리자에서 확인
services.msc

# 또는 명령어로 시작
net start postgresql-x64-15
```

**macOS:**
```bash
# Homebrew 서비스 상태 확인
brew services list | grep postgresql

# 서비스 시작
brew services start postgresql@15
```

**Linux:**
```bash
# 서비스 상태 확인
sudo systemctl status postgresql

# 서비스 시작
sudo systemctl start postgresql
```

#### 2. DBeaver 연결 실패

**연결 설정 확인:**
```
호스트: localhost (또는 127.0.0.1)
포트: 5432
데이터베이스: postgres
사용자명: postgres
비밀번호: [설치 시 설정한 비밀번호]
```

**pg_hba.conf 설정 확인 (고급):**
```bash
# 설정 파일 위치 확인
# Windows: C:\Program Files\PostgreSQL\15\data\pg_hba.conf
# macOS: /usr/local/var/postgres/pg_hba.conf
# Linux: /etc/postgresql/15/main/pg_hba.conf

# 로컬 연결 허용 설정 확인
local   all             all                                     trust
host    all             all             127.0.0.1/32            md5
```

#### 3. 권한 오류 해결

```sql
-- 데이터베이스 권한 부여
GRANT ALL PRIVILEGES ON DATABASE hr_education_system TO postgres;

-- 테이블 권한 부여
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO postgres;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO postgres;
```

#### 4. 성능 이슈 해결

**느린 쿼리 분석:**
```sql
-- 느린 쿼리 로깅 활성화 (postgresql.conf)
log_min_duration_statement = 1000  -- 1초 이상 쿼리 로깅

-- 실행 계획 확인
EXPLAIN ANALYZE
SELECT * FROM materials 
WHERE title ILIKE '%검색어%';
```

**인덱스 사용률 확인:**
```sql
-- 인덱스 사용 통계
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans,
    idx_tup_read as index_reads,
    idx_tup_fetch as index_fetches
FROM pg_stat_user_indexes
WHERE idx_scan < 50  -- 사용률이 낮은 인덱스 찾기
ORDER BY idx_scan;
```

### CursorAI 디버깅 프롬프트

```
"PostgreSQL에서 다음 에러가 발생했어:
[에러 메시지]

실행한 쿼리:
[문제가 된 쿼리]

어떤 문제이고 어떻게 해결할까?"
```

```
"DBeaver에서 테이블 생성은 성공했는데 
FastAPI에서 연결할 때 'relation does not exist' 에러가 나와.
어떤 부분을 확인해야 할까?"
```

---

## 📝 다음 단계

데이터베이스 설정이 완료되었다면:

1. ✅ **데이터베이스 연결 테스트**
   ```sql
   -- 모든 테이블이 생성되었는지 확인
   SELECT table_name FROM information_schema.tables 
   WHERE table_schema = 'public';
   ```

2. ✅ **샘플 데이터 확인**
   ```sql
   -- 샘플 데이터 조회
   SELECT * FROM categories;
   SELECT * FROM materials;
   SELECT * FROM users WHERE is_admin = true;
   ```

3. 🔄 **다음 가이드로 이동**
   - **백엔드 개발**: [BACKEND_GUIDE.md](BACKEND_GUIDE.md)
   - **프론트엔드 개발**: [FRONTEND_GUIDE.md](FRONTEND_GUIDE.md)

---

## 🔗 추가 리소스

- [PostgreSQL 공식 문서](https://www.postgresql.org/docs/)
- [DBeaver 사용 가이드](https://dbeaver.io/docs/)
- [SQL 튜토리얼](https://www.w3schools.com/sql/)
- [PostgreSQL 성능 튜닝](https://wiki.postgresql.org/wiki/Performance_Optimization)

---

**데이터베이스 설정 완료! 🎉**  
이제 백엔드 API를 개발하여 데이터베이스와 연결해보세요!
