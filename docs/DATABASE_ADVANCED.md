# 💾 데이터베이스 고급 가이드

> 성능 최적화, 보안, 모니터링까지 - 운영 환경 준비

## 📋 목차
1. [고급 테이블 설계](#고급-테이블-설계)
2. [인덱스 최적화](#인덱스-최적화)
3. [트리거 및 함수](#트리거-및-함수)
4. [성능 모니터링](#성능-모니터링)
5. [백업 및 복구](#백업-및-복구)
6. [보안 설정](#보안-설정)

---

## 🏗️ 고급 테이블 설계

### 로그 테이블 추가

#### 다운로드 로그 테이블
```sql
CREATE TABLE download_logs (
    id SERIAL PRIMARY KEY,
    material_id INTEGER REFERENCES materials(id) ON DELETE CASCADE,
    user_ip INET, -- IP 주소 타입
    user_agent TEXT,
    download_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    referer_url TEXT,
    session_id VARCHAR(100)
);

-- 파티셔닝 (월별)
CREATE TABLE download_logs_2024_01 PARTITION OF download_logs
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
```

#### 조회 로그 테이블
```sql
CREATE TABLE view_logs (
    id SERIAL PRIMARY KEY,
    material_id INTEGER REFERENCES materials(id) ON DELETE CASCADE,
    user_ip INET,
    user_agent TEXT,
    viewed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    session_id VARCHAR(100),
    view_duration INTEGER -- 페이지 체류 시간 (초)
);
```

### 확장 테이블 설계

#### 태그 테이블 (정규화)
```sql
CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    color VARCHAR(7), -- 색상 코드
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE material_tags (
    material_id INTEGER REFERENCES materials(id) ON DELETE CASCADE,
    tag_id INTEGER REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (material_id, tag_id)
);
```

#### 사용자 권한 테이블
```sql
CREATE TABLE permissions (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT
);

CREATE TABLE user_permissions (
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    permission_id INTEGER REFERENCES permissions(id) ON DELETE CASCADE,
    granted_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    granted_by INTEGER REFERENCES users(id),
    PRIMARY KEY (user_id, permission_id)
);
```

---

## 🚀 인덱스 최적화

### 복합 인덱스 생성
```sql
-- 자주 사용되는 검색 조건에 대한 복합 인덱스
CREATE INDEX idx_materials_category_active_created 
ON materials(category_id, is_active, created_at DESC);

-- 전문 검색을 위한 GIN 인덱스
CREATE INDEX idx_materials_fulltext 
ON materials USING gin(to_tsvector('korean', title || ' ' || COALESCE(description, '')));

-- 부분 인덱스 (활성 자료만)
CREATE INDEX idx_materials_active_title 
ON materials(title) 
WHERE is_active = true;
```

### 인덱스 사용률 모니터링
```sql
-- 인덱스 사용 통계 조회
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans,
    idx_tup_read as tuples_read,
    idx_tup_fetch as tuples_fetched
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan DESC;

-- 사용되지 않는 인덱스 찾기
SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) as index_size
FROM pg_stat_user_indexes
WHERE idx_scan < 50 AND schemaname = 'public'
ORDER BY pg_relation_size(indexrelid) DESC;
```

---

## ⚙️ 트리거 및 함수

### 자동 업데이트 트리거
```sql
-- updated_at 자동 갱신 함수
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

-- 트리거 적용
CREATE TRIGGER update_materials_updated_at 
    BEFORE UPDATE ON materials 
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

### 통계 자동 업데이트
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

### 데이터 검증 트리거
```sql
-- 파일 크기 검증 함수
CREATE OR REPLACE FUNCTION validate_file_size()
RETURNS TRIGGER AS $$
BEGIN
    -- 100MB 초과 시 에러
    IF NEW.file_size > 104857600 THEN
        RAISE EXCEPTION 'File size cannot exceed 100MB';
    END IF;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER validate_material_file_size
    BEFORE INSERT OR UPDATE ON materials
    FOR EACH ROW EXECUTE FUNCTION validate_file_size();
```

---

## 📊 성능 모니터링

### 느린 쿼리 분석
```sql
-- 현재 실행 중인 쿼리 확인
SELECT 
    pid,
    now() - pg_stat_activity.query_start AS duration,
    query,
    state
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes';

-- 테이블별 접근 통계
SELECT 
    schemaname,
    tablename,
    seq_scan, -- 순차 스캔 횟수
    seq_tup_read, -- 순차 스캔으로 읽은 행 수
    idx_scan, -- 인덱스 스캔 횟수
    idx_tup_fetch, -- 인덱스로 가져온 행 수
    n_tup_ins, -- 삽입된 행 수
    n_tup_upd, -- 업데이트된 행 수
    n_tup_del  -- 삭제된 행 수
FROM pg_stat_user_tables
WHERE schemaname = 'public';
```

### 데이터베이스 크기 모니터링
```sql
-- 데이터베이스 전체 크기
SELECT pg_size_pretty(pg_database_size('hr_education_system'));

-- 테이블별 크기
SELECT 
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables 
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- 인덱스 크기
SELECT 
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexname::regclass)) as index_size
FROM pg_indexes 
WHERE schemaname = 'public'
ORDER BY pg_relation_size(indexname::regclass) DESC;
```

---

## 💾 백업 및 복구

### 자동 백업 스크립트
```bash
#!/bin/bash
# backup_script.sh

DB_NAME="hr_education_system"
DB_USER="postgres"
BACKUP_DIR="/var/backups/postgresql"
DATE=$(date +%Y%m%d_%H%M%S)

# 백업 디렉터리 생성
mkdir -p $BACKUP_DIR

# 전체 데이터베이스 백업
pg_dump -h localhost -U $DB_USER -d $DB_NAME > $BACKUP_DIR/backup_$DATE.sql

# 압축
gzip $BACKUP_DIR/backup_$DATE.sql

# 7일 이상 된 백업 파일 삭제
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +7 -delete

echo "Backup completed: backup_$DATE.sql.gz"
```

### 선택적 백업
```bash
# 특정 테이블만 백업
pg_dump -h localhost -U postgres -d hr_education_system -t materials > materials_backup.sql

# 스키마만 백업 (데이터 제외)
pg_dump -h localhost -U postgres -d hr_education_system -s > schema_backup.sql

# 데이터만 백업 (스키마 제외)
pg_dump -h localhost -U postgres -d hr_education_system -a > data_backup.sql
```

### 복구 방법
```bash
# 전체 복구
psql -h localhost -U postgres -d hr_education_system < backup_20240125_143000.sql

# 특정 테이블 복구
psql -h localhost -U postgres -d hr_education_system < materials_backup.sql
```

---

## 🔒 보안 설정

### 사용자 권한 관리
```sql
-- 읽기 전용 사용자 생성
CREATE USER readonly_user WITH PASSWORD 'secure_password';

-- 읽기 권한만 부여
GRANT CONNECT ON DATABASE hr_education_system TO readonly_user;
GRANT USAGE ON SCHEMA public TO readonly_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA public TO readonly_user;

-- 애플리케이션 전용 사용자
CREATE USER app_user WITH PASSWORD 'app_secure_password';
GRANT CONNECT ON DATABASE hr_education_system TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO app_user;
```

### 접근 제어 (pg_hba.conf)
```
# 로컬 연결은 신뢰 모드
local   all             postgres                                peer

# 애플리케이션 서버에서만 접근 허용
host    hr_education_system    app_user        192.168.1.0/24     md5

# 관리자는 특정 IP에서만 접근
host    hr_education_system    postgres        192.168.1.100/32   md5

# 다른 모든 연결 거부
host    all             all             0.0.0.0/0            reject
```

### SSL 설정
```sql
-- SSL 강제 설정
ALTER SYSTEM SET ssl = on;
ALTER SYSTEM SET ssl_cert_file = '/path/to/server.crt';
ALTER SYSTEM SET ssl_key_file = '/path/to/server.key';

-- 설정 다시 로드
SELECT pg_reload_conf();
```

---

## 🔧 고급 쿼리 예제

### 복잡한 통계 쿼리
```sql
-- 월별 다운로드 추이와 증감률
WITH monthly_downloads AS (
    SELECT 
        DATE_TRUNC('month', download_at) as month,
        COUNT(*) as downloads
    FROM download_logs
    GROUP BY DATE_TRUNC('month', download_at)
)
SELECT 
    month,
    downloads,
    LAG(downloads) OVER (ORDER BY month) as prev_month,
    CASE 
        WHEN LAG(downloads) OVER (ORDER BY month) IS NULL THEN NULL
        ELSE ROUND(
            ((downloads - LAG(downloads) OVER (ORDER BY month)) * 100.0 / 
             LAG(downloads) OVER (ORDER BY month)), 2
        )
    END as growth_rate
FROM monthly_downloads
ORDER BY month DESC;
```

### 추천 시스템용 쿼리
```sql
-- 사용자 관심사 기반 추천 (IP 기반)
WITH user_preferences AS (
    SELECT 
        dl.user_ip,
        m.category_id,
        COUNT(*) as download_count
    FROM download_logs dl
    JOIN materials m ON dl.material_id = m.id
    WHERE dl.download_at > NOW() - INTERVAL '30 days'
    GROUP BY dl.user_ip, m.category_id
)
SELECT DISTINCT
    m.id,
    m.title,
    c.name as category_name,
    m.download_count
FROM materials m
JOIN categories c ON m.category_id = c.id
JOIN user_preferences up ON m.category_id = up.category_id
WHERE m.is_active = true
  AND up.user_ip = '192.168.1.100' -- 특정 사용자 IP
ORDER BY m.download_count DESC, m.created_at DESC
LIMIT 5;
```

---

## 🚀 성능 최적화 기법

### 파티셔닝
```sql
-- 날짜별 파티셔닝 (로그 테이블)
CREATE TABLE download_logs_partitioned (
    LIKE download_logs INCLUDING ALL
) PARTITION BY RANGE (download_at);

-- 월별 파티션 생성
CREATE TABLE download_logs_2024_01 PARTITION OF download_logs_partitioned
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE download_logs_2024_02 PARTITION OF download_logs_partitioned
FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
```

### 구체화된 뷰 (Materialized View)
```sql
-- 인기 자료 통계 뷰
CREATE MATERIALIZED VIEW popular_materials AS
SELECT 
    m.id,
    m.title,
    c.name as category_name,
    m.download_count,
    m.view_count,
    (m.download_count + m.view_count * 0.1) as popularity_score
FROM materials m
LEFT JOIN categories c ON m.category_id = c.id
WHERE m.is_active = true
ORDER BY popularity_score DESC;

-- 인덱스 생성
CREATE INDEX idx_popular_materials_score ON popular_materials(popularity_score DESC);

-- 자동 갱신 (매일 새벽 3시)
CREATE OR REPLACE FUNCTION refresh_popular_materials()
RETURNS void AS $
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY popular_materials;
END;
$ LANGUAGE plpgsql;
```

### 연결 풀링 설정
```sql
-- PostgreSQL 설정 최적화
ALTER SYSTEM SET max_connections = 200;
ALTER SYSTEM SET shared_buffers = '256MB';
ALTER SYSTEM SET effective_cache_size = '1GB';
ALTER SYSTEM SET maintenance_work_mem = '64MB';
ALTER SYSTEM SET checkpoint_completion_target = 0.9;
ALTER SYSTEM SET wal_buffers = '16MB';
ALTER SYSTEM SET default_statistics_target = 100;

-- 설정 적용
SELECT pg_reload_conf();
```

---

## 📈 모니터링 및 알림

### 성능 메트릭 수집
```sql
-- 데이터베이스 활동 모니터링 뷰
CREATE VIEW db_activity_summary AS
SELECT 
    'Active Connections' as metric,
    COUNT(*) as value
FROM pg_stat_activity
WHERE state = 'active'
UNION ALL
SELECT 
    'Total Connections' as metric,
    COUNT(*) as value
FROM pg_stat_activity
UNION ALL
SELECT 
    'Long Running Queries' as metric,
    COUNT(*) as value
FROM pg_stat_activity
WHERE state = 'active' 
    AND now() - query_start > interval '5 minutes'
UNION ALL
SELECT 
    'Database Size (MB)' as metric,
    pg_database_size(current_database()) / 1024 / 1024 as value;
```

### 자동 알림 함수
```sql
-- 이상 상황 감지 및 알림 함수
CREATE OR REPLACE FUNCTION check_db_health()
RETURNS TABLE(alert_type TEXT, message TEXT, severity TEXT) AS $
BEGIN
    -- 긴 실행 쿼리 확인
    RETURN QUERY
    SELECT 
        'Long Running Query'::TEXT,
        'Query running for ' || (now() - query_start)::TEXT,
        'WARNING'::TEXT
    FROM pg_stat_activity
    WHERE state = 'active' 
        AND now() - query_start > interval '10 minutes'
        AND query NOT LIKE '%pg_stat_activity%';
    
    -- 디스크 사용량 확인 (예시)
    IF (SELECT pg_database_size(current_database()) / 1024 / 1024 / 1024) > 5 THEN
        RETURN QUERY
        SELECT 
            'High Disk Usage'::TEXT,
            'Database size exceeds 5GB'::TEXT,
            'CRITICAL'::TEXT;
    END IF;
    
    -- 연결 수 확인
    IF (SELECT COUNT(*) FROM pg_stat_activity) > 150 THEN
        RETURN QUERY
        SELECT 
            'High Connection Count'::TEXT,
            'Connection count: ' || (SELECT COUNT(*) FROM pg_stat_activity)::TEXT,
            'WARNING'::TEXT;
    END IF;
    
    RETURN;
END;
$ LANGUAGE plpgsql;
```

---

## 🧪 테스트 데이터 생성

### 대량 테스트 데이터 생성
```sql
-- 대량의 테스트 교육자료 생성
INSERT INTO materials (title, description, category_id, file_path, file_name, original_file_name, file_size, mime_type, uploaded_by, view_count, download_count)
SELECT 
    'Test Material ' || i,
    'Description for test material ' || i,
    (i % 6) + 1, -- 카테고리 1-6 순환
    '/uploads/test_' || i || '.pdf',
    'test_' || i || '.pdf',
    'Test_' || i || '.pdf',
    1000000 + (i * 1000), -- 다양한 파일 크기
    'application/pdf',
    1,
    floor(random() * 1000), -- 랜덤 조회수
    floor(random() * 100) -- 랜덤 다운로드 수
FROM generate_series(1, 10000) as i;

-- 대량의 다운로드 로그 생성
INSERT INTO download_logs (material_id, user_ip, download_at)
SELECT 
    (floor(random() * 10000) + 1)::INTEGER,
    ('192.168.1.' || (floor(random() * 254) + 1)::INTEGER)::INET,
    NOW() - (random() * interval '90 days')
FROM generate_series(1, 50000);
```

### 성능 테스트 쿼리
```sql
-- 복잡한 검색 쿼리 성능 테스트
EXPLAIN ANALYZE
SELECT 
    m.id,
    m.title,
    c.name as category_name,
    m.download_count,
    COUNT(dl.id) as recent_downloads
FROM materials m
LEFT JOIN categories c ON m.category_id = c.id
LEFT JOIN download_logs dl ON m.id = dl.material_id 
    AND dl.download_at > NOW() - INTERVAL '7 days'
WHERE m.is_active = true
    AND (m.title ILIKE '%교육%' OR m.description ILIKE '%교육%')
GROUP BY m.id, m.title, c.name, m.download_count
ORDER BY m.download_count DESC, recent_downloads DESC
LIMIT 20;
```

---

## 🔄 데이터 마이그레이션

### 기존 데이터 정리 및 마이그레이션
```sql
-- 중복 데이터 제거
WITH duplicates AS (
    SELECT id, 
           ROW_NUMBER() OVER (PARTITION BY title, file_name ORDER BY created_at) as rn
    FROM materials
)
DELETE FROM materials 
WHERE id IN (
    SELECT id FROM duplicates WHERE rn > 1
);

-- 데이터 정규화 (태그 분리)
-- 1. 기존 materials 테이블에서 태그 추출
INSERT INTO tags (name)
SELECT DISTINCT unnest(string_to_array(tags_string, ','))
FROM materials_old
WHERE tags_string IS NOT NULL
ON CONFLICT (name) DO NOTHING;

-- 2. 관계 테이블 채우기
INSERT INTO material_tags (material_id, tag_id)
SELECT 
    m.id,
    t.id
FROM materials_old m
CROSS JOIN LATERAL unnest(string_to_array(m.tags_string, ',')) as tag_name
JOIN tags t ON t.name = tag_name
WHERE m.tags_string IS NOT NULL;
```

### 버전 관리
```sql
-- 스키마 버전 관리 테이블
CREATE TABLE schema_versions (
    version VARCHAR(20) PRIMARY KEY,
    applied_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    description TEXT
);

-- 초기 버전 기록
INSERT INTO schema_versions (version, description) 
VALUES ('1.0.0', 'Initial schema creation');
```

---

## 🛡️ 고급 보안 설정

### 행 수준 보안 (Row Level Security)
```sql
-- RLS 활성화
ALTER TABLE materials ENABLE ROW LEVEL SECURITY;

-- 정책 생성: 관리자는 모든 자료 접근 가능
CREATE POLICY admin_all_materials ON materials
    FOR ALL TO admin_role
    USING (true);

-- 정책 생성: 일반 사용자는 활성 자료만 조회 가능
CREATE POLICY user_active_materials ON materials
    FOR SELECT TO public
    USING (is_active = true);
```

### 감사 로그
```sql
-- 감사 로그 테이블
CREATE TABLE audit_log (
    id SERIAL PRIMARY KEY,
    table_name VARCHAR(50) NOT NULL,
    operation VARCHAR(10) NOT NULL, -- INSERT, UPDATE, DELETE
    old_values JSONB,
    new_values JSONB,
    user_name VARCHAR(50),
    changed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 감사 트리거 함수
CREATE OR REPLACE FUNCTION audit_trigger()
RETURNS TRIGGER AS $
BEGIN
    IF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (table_name, operation, old_values, user_name)
        VALUES (TG_TABLE_NAME, TG_OP, to_jsonb(OLD), current_user);
        RETURN OLD;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, operation, old_values, new_values, user_name)
        VALUES (TG_TABLE_NAME, TG_OP, to_jsonb(OLD), to_jsonb(NEW), current_user);
        RETURN NEW;
    ELSIF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (table_name, operation, new_values, user_name)
        VALUES (TG_TABLE_NAME, TG_OP, to_jsonb(NEW), current_user);
        RETURN NEW;
    END IF;
END;
$ LANGUAGE plpgsql;

-- 감사 트리거 적용
CREATE TRIGGER materials_audit_trigger
    AFTER INSERT OR UPDATE OR DELETE ON materials
    FOR EACH ROW EXECUTE FUNCTION audit_trigger();
```

---

## 📊 리포팅 및 분석

### 비즈니스 인텔리전스 뷰
```sql
-- 월별 KPI 대시보드 뷰
CREATE VIEW monthly_kpi AS
SELECT 
    DATE_TRUNC('month', created_at) as month,
    COUNT(*) as materials_uploaded,
    SUM(CASE WHEN is_featured THEN 1 ELSE 0 END) as featured_materials,
    AVG(file_size) as avg_file_size,
    SUM(download_count) as total_downloads,
    COUNT(DISTINCT category_id) as active_categories
FROM materials
WHERE is_active = true
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month DESC;

-- 카테고리별 성과 분석
CREATE VIEW category_performance AS
SELECT 
    c.name as category_name,
    COUNT(m.id) as material_count,
    SUM(m.download_count) as total_downloads,
    AVG(m.download_count) as avg_downloads,
    SUM(m.view_count) as total_views,
    ROUND(
        SUM(m.download_count) * 100.0 / 
        NULLIF((SELECT SUM(download_count) FROM materials WHERE is_active = true), 0),
        2
    ) as download_percentage
FROM categories c
LEFT JOIN materials m ON c.id = m.category_id AND m.is_active = true
WHERE c.is_active = true
GROUP BY c.id, c.name
ORDER BY total_downloads DESC;
```

---

## 🚀 CursorAI 고급 활용

### 복잡한 쿼리 최적화 요청
```
"다음 쿼리가 너무 느려:
[느린 쿼리 붙여넣기]

실행 계획:
[EXPLAIN ANALYZE 결과 붙여넣기]

어떤 인덱스를 추가하고, 쿼리를 어떻게 개선하면 될까?
PostgreSQL 14 사용 중이야."
```

### 스키마 설계 검토 요청
```
"PostgreSQL로 교육자료 관리 시스템 DB를 설계했어.
다음 테이블들을 검토해서 개선점 알려줘:

[테이블 CREATE 문들 붙여넣기]

특히 다음 사항들 체크해줘:
- 정규화 수준이 적절한지
- 인덱스 전략
- 외래키 제약조건
- 성능 최적화 포인트"
```

### 백업/복구 전략 수립
```
"PostgreSQL 데이터베이스의 백업/복구 전략을 수립해줘.

환경:
- 데이터베이스 크기: 약 50GB
- 일일 데이터 증가: 약 500MB
- RTO: 4시간 이내
- RPO: 1시간 이내

고려사항:
- 자동화된 백업
- 포인트인타임 복구
- 백업 검증
- 재해복구 시나리오"
```

---

## 🔗 운영 도구 연동

### Grafana 모니터링
```sql
-- Grafana용 메트릭 쿼리
-- 1. 활성 연결 수
SELECT COUNT(*) as active_connections 
FROM pg_stat_activity 
WHERE state = 'active';

-- 2. 데이터베이스 크기 (MB)
SELECT pg_database_size(current_database()) / 1024 / 1024 as db_size_mb;

-- 3. 시간당 다운로드 수
SELECT 
    DATE_TRUNC('hour', download_at) as hour,
    COUNT(*) as downloads_per_hour
FROM download_logs
WHERE download_at > NOW() - INTERVAL '24 hours'
GROUP BY DATE_TRUNC('hour', download_at)
ORDER BY hour;
```

### 슬랙 알림 연동 (예시)
```bash
#!/bin/bash
# slack_alert.sh
# PostgreSQL 상태 확인 후 슬랙 알림

DB_SIZE=$(psql -h localhost -U postgres -d hr_education_system -t -c "SELECT pg_database_size(current_database()) / 1024 / 1024 / 1024;")
CONNECTIONS=$(psql -h localhost -U postgres -d hr_education_system -t -c "SELECT COUNT(*) FROM pg_stat_activity;")

if (( $(echo "$DB_SIZE > 10" | bc -l) )); then
    curl -X POST -H 'Content-type: application/json' \
        --data '{"text":"⚠️ Database size exceeded 10GB: '$DB_SIZE'GB"}' \
        $SLACK_WEBHOOK_URL
fi

if [ "$CONNECTIONS" -gt 100 ]; then
    curl -X POST -H 'Content-type: application/json' \
        --data '{"text":"⚠️ High connection count: '$CONNECTIONS'"}' \
        $SLACK_WEBHOOK_URL
fi
```

---

## 📚 고급 학습 자료

- [PostgreSQL 성능 튜닝 가이드](https://wiki.postgresql.org/wiki/Performance_Optimization)
- [PostgreSQL 모니터링](https://www.postgresql.org/docs/current/monitoring.html)
- [pg_stat_statements 활용법](https://www.postgresql.org/docs/current/pgstatstatements.html)
- [PostgreSQL 백업 전략](https://www.postgresql.org/docs/current/backup.html)

---

## 📝 체크리스트

운영 환경 준비 완료 체크리스트:

### 성능 ✅
- [ ] 적절한 인덱스 생성 완료
- [ ] 쿼리 성능 테스트 완료
- [ ] 구체화된 뷰 설정 완료

### 보안 ✅  
- [ ] 사용자 권한 분리 완료
- [ ] SSL 설정 완료
- [ ] 감사 로그 설정 완료

### 모니터링 ✅
- [ ] 성능 메트릭 수집 설정 완료
- [ ] 알림 시스템 구축 완료
- [ ] 대시보드 구성 완료

### 백업/복구 ✅
- [ ] 자동 백업 스크립트 설정 완료
- [ ] 복구 테스트 완료
- [ ] 재해복구 계획 수립 완료

---

**🚀 고급 데이터베이스 설정 완료!**  
이제 진짜 운영 환경에서 사용할 수 있는 수준의 데이터베이스가 준비되었습니다!