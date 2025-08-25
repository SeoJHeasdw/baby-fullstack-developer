# 🐍 백엔드 개발 가이드 (기본편)

> FastAPI를 이용한 HR 교육자료 관리 API 개발 기본 가이드

## 📋 목차
1. [개발 환경 설정](#개발-환경-설정)
2. [프로젝트 구조 생성](#프로젝트-구조-생성)
3. [데이터베이스 연결](#데이터베이스-연결)
4. [기본 모델 정의](#기본-모델-정의)
5. [핵심 API 구현](#핵심-api-구현)
6. [파일 업로드 기본 구현](#파일-업로드-기본-구현)
7. [CursorAI 활용 팁](#cursorai-활용-팁)
8. [기본 테스트](#기본-테스트)

---

## 🛠️ 개발 환경 설정

### Python 및 가상환경 설정
```bash
# Python 3.9 이상 확인
python --version

# 가상환경 생성
cd hr-education-system
mkdir backend && cd backend
python -m venv venv

# 가상환경 활성화
# Windows
venv\Scripts\activate
# macOS/Linux  
source venv/bin/activate
```

### 기본 패키지 설치
```bash
# 핵심 패키지만 설치 (일단 동작하게 만들기)
pip install fastapi uvicorn[standard]
pip install sqlalchemy psycopg2-binary
pip install pydantic
pip install python-multipart  # 파일 업로드용

# requirements.txt 생성
pip freeze > requirements.txt
```

---

## 📁 기본 프로젝트 구조

### 간단한 디렉터리 구조
```bash
backend/
├── app/
│   ├── __init__.py
│   ├── main.py           # FastAPI 앱
│   ├── database.py       # DB 연결
│   ├── models.py         # 모든 모델을 한 파일에
│   ├── schemas.py        # 모든 스키마를 한 파일에
│   └── api.py           # 모든 API를 한 파일에
├── uploads/             # 업로드 파일 저장소
└── requirements.txt
```

### 필요한 파일들 생성
```bash
# 파일 생성
touch app/__init__.py app/main.py app/database.py app/models.py app/schemas.py app/api.py
mkdir uploads
```

---

## 🔌 데이터베이스 연결

### 데이터베이스 설정 (app/database.py)
```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# 데이터베이스 URL (실제 환경에서는 환경변수 사용)
DATABASE_URL = "postgresql://postgres:your_password@localhost/hr_education_system"

# 엔진 생성
engine = create_engine(DATABASE_URL)

# 세션 팩토리
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# Base 클래스
Base = declarative_base()

# 의존성 주입용 함수
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

---

## 🗃️ 기본 모델 정의

### 모든 모델을 한 파일에 (app/models.py)
```python
from sqlalchemy import Column, Integer, String, Text, BigInteger, Boolean, DateTime, ForeignKey
from sqlalchemy.sql import func
from sqlalchemy.orm import relationship
from .database import Base

# 카테고리 모델
class Category(Base):
    __tablename__ = "categories"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), unique=True, nullable=False)
    description = Column(Text)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    
    # 관계 설정
    materials = relationship("Material", back_populates="category")

# 사용자 모델 (간단하게)
class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(50), unique=True, nullable=False)
    email = Column(String(100), unique=True, nullable=False)
    full_name = Column(String(100), nullable=False)
    is_admin = Column(Boolean, default=False)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    
    # 관계 설정
    uploaded_materials = relationship("Material", back_populates="uploader")

# 교육자료 모델
class Material(Base):
    __tablename__ = "materials"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False)
    description = Column(Text)
    
    # 카테고리 연결
    category_id = Column(Integer, ForeignKey("categories.id"))
    
    # 파일 정보
    file_path = Column(String(500), nullable=False)
    file_name = Column(String(255), nullable=False)
    original_file_name = Column(String(255), nullable=False)
    file_size = Column(BigInteger, nullable=False)
    mime_type = Column(String(100), nullable=False)
    
    # 기본 통계
    view_count = Column(Integer, default=0)
    download_count = Column(Integer, default=0)
    
    # 관리 정보
    uploaded_by = Column(Integer, ForeignKey("users.id"))
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    
    # 관계 설정
    category = relationship("Category", back_populates="materials")
    uploader = relationship("User", back_populates="uploaded_materials")
```

---

## 📝 기본 스키마 정의

### 모든 스키마를 한 파일에 (app/schemas.py)
```python
from pydantic import BaseModel
from typing import Optional
from datetime import datetime

# 카테고리 스키마
class CategoryBase(BaseModel):
    name: str
    description: Optional[str] = None

class CategoryCreate(CategoryBase):
    pass

class Category(CategoryBase):
    id: int
    is_active: bool
    created_at: datetime
    
    class Config:
        from_attributes = True

# 교육자료 스키마
class MaterialBase(BaseModel):
    title: str
    description: Optional[str] = None
    category_id: Optional[int] = None

class MaterialCreate(MaterialBase):
    pass

class Material(MaterialBase):
    id: int
    file_name: str
    original_file_name: str
    file_size: int
    mime_type: str
    view_count: int
    download_count: int
    is_active: bool
    created_at: datetime
    
    class Config:
        from_attributes = True

# 사용자 스키마
class UserBase(BaseModel):
    username: str
    email: str
    full_name: str

class UserCreate(UserBase):
    is_admin: bool = False

class User(UserBase):
    id: int
    is_admin: bool
    is_active: bool
    created_at: datetime
    
    class Config:
        from_attributes = True
```

---

## 🚀 핵심 API 구현

### 모든 API를 한 파일에 (app/api.py)
```python
from fastapi import APIRouter, Depends, HTTPException, UploadFile, File, Form
from sqlalchemy.orm import Session
from typing import List
import os
import uuid
from . import models, schemas
from .database import get_db

router = APIRouter()

# 카테고리 API
@router.get("/categories", response_model=List[schemas.Category])
def get_categories(db: Session = Depends(get_db)):
    """카테고리 목록 조회"""
    categories = db.query(models.Category).filter(
        models.Category.is_active == True
    ).all()
    return categories

@router.post("/categories", response_model=schemas.Category)
def create_category(
    category: schemas.CategoryCreate, 
    db: Session = Depends(get_db)
):
    """카테고리 생성 (관리자만 - 일단 제한 없이)"""
    db_category = models.Category(**category.dict())
    db.add(db_category)
    db.commit()
    db.refresh(db_category)
    return db_category

# 교육자료 API
@router.get("/materials", response_model=List[schemas.Material])
def get_materials(
    skip: int = 0,
    limit: int = 20,
    search: str = None,
    category_id: int = None,
    db: Session = Depends(get_db)
):
    """교육자료 목록 조회 (로그인 불필요)"""
    query = db.query(models.Material).filter(models.Material.is_active == True)
    
    # 검색 필터
    if search:
        query = query.filter(models.Material.title.contains(search))
    
    # 카테고리 필터
    if category_id:
        query = query.filter(models.Material.category_id == category_id)
    
    materials = query.order_by(
        models.Material.created_at.desc()
    ).offset(skip).limit(limit).all()
    
    return materials

@router.get("/materials/{material_id}", response_model=schemas.Material)
def get_material(material_id: int, db: Session = Depends(get_db)):
    """교육자료 상세 조회"""
    material = db.query(models.Material).filter(
        models.Material.id == material_id,
        models.Material.is_active == True
    ).first()
    
    if not material:
        raise HTTPException(status_code=404, detail="자료를 찾을 수 없습니다")
    
    # 조회수 증가
    material.view_count += 1
    db.commit()
    
    return material

@router.post("/materials", response_model=schemas.Material)
def create_material(
    title: str = Form(...),
    description: str = Form(None),
    category_id: int = Form(None),
    file: UploadFile = File(...),
    db: Session = Depends(get_db)
):
    """교육자료 업로드 (일단 인증 없이)"""
    
    # 파일 크기 체크 (10MB 제한)
    if file.size > 10 * 1024 * 1024:
        raise HTTPException(status_code=400, detail="파일 크기가 10MB를 초과합니다")
    
    # 허용된 파일 확장자 체크
    allowed_extensions = ['pdf', 'doc', 'docx', 'ppt', 'pptx', 'txt']
    file_extension = file.filename.split(".")[-1].lower()
    
    if file_extension not in allowed_extensions:
        raise HTTPException(
            status_code=400, 
            detail=f"지원하지 않는 파일 형식입니다. 허용: {', '.join(allowed_extensions)}"
        )
    
    # 고유한 파일명 생성
    unique_filename = f"{uuid.uuid4()}.{file_extension}"
    file_path = os.path.join("uploads", unique_filename)
    
    # 파일 저장
    try:
        with open(file_path, "wb") as buffer:
            content = file.file.read()
            buffer.write(content)
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"파일 저장 실패: {str(e)}")
    
    # 데이터베이스에 저장
    db_material = models.Material(
        title=title,
        description=description,
        category_id=category_id,
        file_path=file_path,
        file_name=unique_filename,
        original_file_name=file.filename,
        file_size=file.size,
        mime_type=file.content_type,
        uploaded_by=1  # 일단 관리자 ID로 고정
    )
    
    db.add(db_material)
    db.commit()
    db.refresh(db_material)
    
    return db_material

@router.get("/materials/{material_id}/download")
def download_material(material_id: int, db: Session = Depends(get_db)):
    """교육자료 다운로드"""
    material = db.query(models.Material).filter(
        models.Material.id == material_id,
        models.Material.is_active == True
    ).first()
    
    if not material:
        raise HTTPException(status_code=404, detail="자료를 찾을 수 없습니다")
    
    # 파일 존재 확인
    if not os.path.exists(material.file_path):
        raise HTTPException(status_code=404, detail="파일을 찾을 수 없습니다")
    
    # 다운로드 수 증가
    material.download_count += 1
    db.commit()
    
    # 파일 응답
    from fastapi.responses import FileResponse
    return FileResponse(
        path=material.file_path,
        filename=material.original_file_name,
        media_type=material.mime_type
    )

@router.delete("/materials/{material_id}")
def delete_material(material_id: int, db: Session = Depends(get_db)):
    """교육자료 삭제 (일단 인증 없이)"""
    material = db.query(models.Material).filter(
        models.Material.id == material_id
    ).first()
    
    if not material:
        raise HTTPException(status_code=404, detail="자료를 찾을 수 없습니다")
    
    # 소프트 삭제 (is_active = False)
    material.is_active = False
    db.commit()
    
    return {"message": "자료가 삭제되었습니다"}
```

---

## 🚀 메인 애플리케이션

### FastAPI 앱 생성 (app/main.py)
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.staticfiles import StaticFiles
from .database import engine, Base
from .api import router
from . import models

# 데이터베이스 테이블 생성
Base.metadata.create_all(bind=engine)

# FastAPI 앱 생성
app = FastAPI(
    title="HR 교육자료 관리 API",
    description="HR팀을 위한 교육자료 관리 API (기본편)",
    version="1.0.0"
)

# CORS 설정 (개발용 - 모든 도메인 허용)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # 운영환경에서는 구체적인 도메인 지정
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 정적 파일 서빙
app.mount("/uploads", StaticFiles(directory="uploads"), name="uploads")

# 라우터 등록
app.include_router(router, prefix="/api")

@app.get("/")
def root():
    return {
        "message": "HR 교육자료 관리 시스템 API (기본편)",
        "docs": "/docs"
    }

@app.get("/health")
def health_check():
    return {"status": "healthy"}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run("app.main:app", host="0.0.0.0", port=8000, reload=True)
```

---

## 🔧 초기 데이터 생성

### 기본 데이터 생성 스크립트 (create_initial_data.py)
```python
from app.database import SessionLocal
from app import models

def create_initial_data():
    db = SessionLocal()
    
    try:
        # 관리자 사용자 생성
        admin_user = db.query(models.User).filter(
            models.User.username == "admin"
        ).first()
        
        if not admin_user:
            admin_user = models.User(
                username="admin",
                email="admin@company.com", 
                full_name="HR 관리자",
                is_admin=True
            )
            db.add(admin_user)
            db.commit()
            print("관리자 계정이 생성되었습니다.")
        
        # 기본 카테고리 생성
        categories = [
            {"name": "신입사원 교육", "description": "신입사원을 위한 기본 교육 자료"},
            {"name": "리더십", "description": "관리자 및 팀리더를 위한 교육"},
            {"name": "컴플라이언스", "description": "법규 준수 및 윤리 교육"},
            {"name": "IT 기술", "description": "IT 및 기술 관련 교육"},
            {"name": "비즈니스 스킬", "description": "업무 효율성 향상 교육"}
        ]
        
        for cat_data in categories:
            existing = db.query(models.Category).filter(
                models.Category.name == cat_data["name"]
            ).first()
            
            if not existing:
                category = models.Category(**cat_data)
                db.add(category)
        
        db.commit()
        print("기본 카테고리가 생성되었습니다.")
        
    finally:
        db.close()

if __name__ == "__main__":
    create_initial_data()
```

---

## 🚀 서버 실행 및 테스트

### 서버 실행
```bash
# 가상환경 활성화 후
cd backend

# 초기 데이터 생성
python create_initial_data.py

# 서버 실행
uvicorn app.main:app --reload

# 또는
python -m app.main
```

### API 테스트
```bash
# 1. 헬스 체크
curl http://localhost:8000/health

# 2. 카테고리 목록 조회
curl http://localhost:8000/api/categories

# 3. 교육자료 목록 조회
curl http://localhost:8000/api/materials

# 4. API 문서 확인
# 브라우저에서 http://localhost:8000/docs
```

---

## 🤖 CursorAI 활용 팁

### 기본 구현 단계에서 유용한 프롬프트

#### API 엔드포인트 구현
```
"FastAPI로 교육자료 목록 API를 만들어줘.

기능:
- 제목으로 검색 가능
- 카테고리별 필터링
- 페이지네이션 (skip, limit)
- 로그인 없이 접근 가능

간단하게 일단 동작하게만 만들어줘."
```

#### 에러 해결
```
"FastAPI 서버 실행시 다음 에러가 나와:
[에러 메시지 붙여넣기]

어떤 문제이고 어떻게 해결해야 할까?
간단한 해결책부터 알려줘."
```

#### 파일 업로드 구현
```
"FastAPI에서 파일 업로드 API를 만들어줘.

요구사항:
- 파일 크기 제한 10MB
- PDF, DOC, PPT 파일만 허용
- 제목, 설명도 함께 받기
- 업로드된 파일은 uploads 폴더에 저장

일단 기본 기능만 동작하게 만들어줘."
```

#### 데이터베이스 연결 문제
```
"SQLAlchemy로 PostgreSQL에 연결하려는데 다음 에러가 나와:
[에러 메시지]

연결 설정을 어떻게 해야 할까?
초보자도 이해할 수 있게 단계별로 알려줘."
```

---

## 🧪 기본 테스트

### 수동 테스트 체크리스트

#### 1. API 기본 동작 확인
```bash
# 서버 실행 확인
curl http://localhost:8000/health
# 응답: {"status": "healthy"}

# 카테고리 목록 확인
curl http://localhost:8000/api/categories
# 응답: JSON 배열

# 교육자료 목록 확인
curl http://localhost:8000/api/materials
# 응답: JSON 배열
```

#### 2. 파일 업로드 테스트
```bash
# 간단한 텍스트 파일로 테스트
echo "테스트 내용" > test.txt

# 파일 업로드 (Postman이나 curl 사용)
curl -X POST "http://localhost:8000/api/materials" \
  -H "Content-Type: multipart/form-data" \
  -F "title=테스트 자료" \
  -F "description=테스트용 파일입니다" \
  -F "category_id=1" \
  -F "file=@test.txt"
```

#### 3. 다운로드 테스트
```bash
# 업로드한 자료 ID로 다운로드
curl "http://localhost:8000/api/materials/1/download" --output downloaded_file.txt
```

### 간단한 Python 테스트 스크립트
```python
# test_basic.py
import requests
import json

BASE_URL = "http://localhost:8000/api"

def test_health():
    response = requests.get("http://localhost:8000/health")
    print(f"Health check: {response.json()}")
    assert response.status_code == 200

def test_categories():
    response = requests.get(f"{BASE_URL}/categories")
    print(f"Categories: {len(response.json())} items")
    assert response.status_code == 200

def test_materials():
    response = requests.get(f"{BASE_URL}/materials")
    print(f"Materials: {len(response.json())} items")
    assert response.status_code == 200

def test_search():
    response = requests.get(f"{BASE_URL}/materials?search=교육")
    print(f"Search results: {len(response.json())} items")
    assert response.status_code == 200

if __name__ == "__main__":
    test_health()
    test_categories()
    test_materials() 
    test_search()
    print("✅ 모든 기본 테스트 통과!")
```

---

## 🚨 기본 문제 해결

### 자주 발생하는 문제들

#### 1. 데이터베이스 연결 실패
```python
# database.py에서 연결 문자열 확인
DATABASE_URL = "postgresql://사용자명:비밀번호@localhost/데이터베이스명"

# 연결 테스트
from sqlalchemy import create_engine
engine = create_engine(DATABASE_URL)
try:
    connection = engine.connect()
    print("✅ 데이터베이스 연결 성공")
    connection.close()
except Exception as e:
    print(f"❌ 연결 실패: {e}")
```

#### 2. 테이블이 생성되지 않음
```python
# main.py에서 테이블 생성 확인
from .database import engine, Base
from . import models  # 중요: models import 필요

Base.metadata.create_all(bind=engine)
```

#### 3. 파일 업로드 폴더 없음
```python
import os

# uploads 폴더 자동 생성
UPLOAD_DIR = "uploads"
if not os.path.exists(UPLOAD_DIR):
    os.makedirs(UPLOAD_DIR)
    print(f"✅ {UPLOAD_DIR} 폴더가 생성되었습니다")
```

#### 4. CORS 에러 (프론트엔드 연결 시)
```python
# main.py에서 CORS 설정 확인
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "http://localhost:5173"],  # 프론트엔드 주소
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## 📝 다음 단계

기본 백엔드 구현이 완료되면:

### ✅ 완료 확인 체크리스트
- [ ] 서버가 정상 실행됨 (`uvicorn app.main:app --reload`)
- [ ] API 문서 접근 가능 (`http://localhost:8000/docs`)
- [ ] 카테고리 목록 조회 가능
- [ ] 교육자료 목록 조회 가능
- [ ] 파일 업로드 가능
- [ ] 파일 다운로드 가능

### 🔄 다음 가이드
- **프론트엔드 개발**: [FRONTEND_GUIDE.md](FRONTEND_GUIDE.md)
- **고급 백엔드 기능**: [BACKEND_ADVANCED.md](BACKEND_ADVANCED.md) (선택)

### 💡 고급 기능 추가 고려사항
기본 구현이 완료되면 다음 기능들을 순차적으로 추가할 수 있습니다:
- 관리자 인증 시스템
- 더 정교한 에러 처리
- 로깅 시스템
- 캐싱
- API 율 제한

---

## 🔗 학습 자료

- [FastAPI 공식 문서](https://fastapi.tiangolo.com/)
- [SQLAlchemy 기본 튜토리얼](https://docs.sqlalchemy.org/en/14/tutorial/)
- [Pydantic 사용법](https://docs.pydantic.dev/)

---

**🎉 기본 백엔드 완성!**  
일단 동작하는 API가 완성되었습니다! 이제 프론트엔드와 연결해보세요.