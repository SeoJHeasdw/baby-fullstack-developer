# 🐍 백엔드 개발 가이드 (Python + FastAPI)

> FastAPI를 이용한 HR 교육자료 관리 API 개발 완전 가이드

## 📋 목차
1. [개발 환경 설정](#개발-환경-설정)
2. [프로젝트 구조 생성](#프로젝트-구조-생성)
3. [데이터베이스 연결](#데이터베이스-연결)
4. [모델 정의](#모델-정의)
5. [API 라우터 구현](#api-라우터-구현)
6. [파일 업로드/다운로드](#파일-업로드다운로드)
7. [관리자 인증](#관리자-인증)
8. [CursorAI 활용 팁](#cursorai-활용-팁)
9. [테스트 및 디버깅](#테스트-및-디버깅)

---

## 🛠️ 개발 환경 설정

### Python 버전 확인
```bash
# Python 3.9 이상 필요
python --version
# 또는
python3 --version

# pip 업그레이드
python -m pip install --upgrade pip
```

### 가상환경 생성
```bash
# 프로젝트 루트 디렉터리에서
mkdir hr-education-system
cd hr-education-system
mkdir backend
cd backend

# 가상환경 생성
python -m venv venv

# 가상환경 활성화
# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate

# 가상환경 활성화 확인 (터미널 앞에 (venv) 표시)
```

### 필수 패키지 설치
```bash
# 핵심 패키지들
pip install fastapi uvicorn[standard]
pip install sqlalchemy psycopg2-binary alembic
pip install pydantic pydantic-settings
pip install python-multipart  # 파일 업로드용

# 인증 및 보안
pip install python-jose[cryptography] passlib[bcrypt]
pip install python-decouple

# 유틸리티
pip install aiofiles  # 비동기 파일 처리
pip install python-magic  # MIME 타입 감지 (선택사항)

# 개발 도구
pip install pytest httpx pytest-asyncio
pip install black isort flake8  # 코드 포매팅

# requirements.txt 생성
pip freeze > requirements.txt
```

---

## 📁 프로젝트 구조 생성

### 디렉터리 구조 만들기
```bash
# backend 폴더 안에서 실행
mkdir app
mkdir app/models app/schemas app/routers app/services app/core app/utils
mkdir uploads  # 파일 업로드 저장소
mkdir tests

# 필요한 파일들 생성
touch app/__init__.py
touch app/main.py app/database.py
touch app/core/__init__.py app/core/config.py app/core/security.py
touch app/models/__init__.py app/models/user.py app/models/material.py app/models/category.py
touch app/schemas/__init__.py app/schemas/user.py app/schemas/material.py app/schemas/category.py
touch app/routers/__init__.py app/routers/materials.py app/routers/categories.py app/routers/admin.py
touch app/services/__init__.py app/services/file_service.py
touch app/utils/__init__.py app/utils/deps.py
touch .env
```

### 최종 프로젝트 구조
```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI 앱 초기화
│   ├── database.py            # DB 연결 설정
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py          # 환경 설정
│   │   └── security.py        # 인증 관련
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py           # 사용자 모델
│   │   ├── material.py       # 교육자료 모델
│   │   └── category.py       # 카테고리 모델
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user.py           # 사용자 스키마
│   │   ├── material.py       # 교육자료 스키마
│   │   └── category.py       # 카테고리 스키마
│   ├── routers/
│   │   ├── __init__.py
│   │   ├── materials.py      # 교육자료 API
│   │   ├── categories.py     # 카테고리 API
│   │   └── admin.py          # 관리자 API
│   ├── services/
│   │   ├── __init__.py
│   │   └── file_service.py   # 파일 처리 서비스
│   └── utils/
│       ├── __init__.py
│       └── deps.py           # 의존성 주입
├── uploads/                   # 업로드 파일 저장소
├── tests/                     # 테스트 파일들
├── requirements.txt
├── .env
└── alembic.ini               # DB 마이그레이션 설정
```

---

## ⚙️ 핵심 설정 파일들

### 1. 환경 설정 (.env)
```env
# .env 파일
DATABASE_URL=postgresql://postgres:your_password@localhost/hr_education_system
SECRET_KEY=your-super-secret-key-here-make-it-very-complex-and-random
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=1440

# 파일 업로드 설정
MAX_FILE_SIZE=50  # MB
ALLOWED_FILE_TYPES=pdf,doc,docx,ppt,pptx,xls,xlsx,txt,mp4,mov,avi
UPLOAD_DIR=./uploads

# 관리자 설정 (초기 계정)
ADMIN_USERNAME=admin
ADMIN_EMAIL=admin@company.com
ADMIN_FULL_NAME=HR 관리자
```

### 2. 설정 클래스 (app/core/config.py)
```python
from pydantic_settings import BaseSettings
from typing import List
import os

class Settings(BaseSettings):
    # 데이터베이스
    database_url: str
    
    # 인증
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 1440  # 24시간
    
    # 파일 업로드
    max_file_size: int = 50  # MB
    allowed_file_types: List[str] = ["pdf", "doc", "docx", "ppt", "pptx", "xls", "xlsx", "txt", "mp4", "mov", "avi"]
    upload_dir: str = "./uploads"
    
    # 관리자 설정
    admin_username: str = "admin"
    admin_email: str = "admin@company.com"
    admin_full_name: str = "HR 관리자"
    
    # CORS 설정
    backend_cors_origins: List[str] = ["http://localhost:3000", "http://localhost:5173", "http://localhost:8080"]
    
    class Config:
        env_file = ".env"

# 전역 설정 인스턴스
settings = Settings()

# 업로드 디렉터리 생성
os.makedirs(settings.upload_dir, exist_ok=True)
```

### 3. 데이터베이스 연결 (app/database.py)
```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from .core.config import settings

# 데이터베이스 엔진 생성
engine = create_engine(
    settings.database_url,
    # 연결 풀 설정
    pool_size=20,
    max_overflow=0,
    pool_pre_ping=True,  # 연결 유효성 검사
    echo=False  # 개발시 True로 설정하면 SQL 로그 출력
)

# 세션 팩토리 생성
SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine
)

# Base 클래스 생성 (모든 모델이 상속)
Base = declarative_base()

# 데이터베이스 의존성 주입용 함수
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

---

## 🗃️ 모델 정의

### 1. 카테고리 모델 (app/models/category.py)
```python
from sqlalchemy import Column, Integer, String, Text, Boolean, DateTime
from sqlalchemy.sql import func
from sqlalchemy.orm import relationship
from ..database import Base

class Category(Base):
    __tablename__ = "categories"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), unique=True, nullable=False, index=True)
    description = Column(Text)
    icon = Column(String(50))  # 아이콘 클래스명
    color = Column(String(20))  # 카테고리 색상
    sort_order = Column(Integer, default=0, index=True)
    is_active = Column(Boolean, default=True, index=True)
    
    # 타임스탬프
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    # 관계 설정 (카테고리 -> 교육자료들)
    materials = relationship("Material", back_populates="category")
```

### 2. 사용자 모델 (app/models/user.py)
```python
from sqlalchemy import Column, Integer, String, Boolean, DateTime
from sqlalchemy.sql import func
from sqlalchemy.orm import relationship
from ..database import Base

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String(50), unique=True, index=True, nullable=False)
    email = Column(String(100), unique=True, index=True, nullable=False)
    full_name = Column(String(100), nullable=False)
    department = Column(String(100))
    position = Column(String(100))
    
    # 권한 관리
    is_admin = Column(Boolean, default=False, index=True)
    is_active = Column(Boolean, default=True, index=True)
    
    # 로그인 기록
    last_login = Column(DateTime(timezone=True))
    
    # 타임스탬프
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    # 관계 설정 (사용자 -> 업로드한 자료들)
    uploaded_materials = relationship("Material", back_populates="uploader")
```

### 3. 교육자료 모델 (app/models/material.py)
```python
from sqlalchemy import Column, Integer, String, Text, BigInteger, Boolean, DateTime, ForeignKey, ARRAY
from sqlalchemy.sql import func
from sqlalchemy.orm import relationship
from ..database import Base

class Material(Base):
    __tablename__ = "materials"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(200), nullable=False, index=True)
    description = Column(Text)
    content = Column(Text)  # 자료 내용
    
    # 카테고리 관계
    category_id = Column(Integer, ForeignKey("categories.id"), index=True)
    
    # 파일 정보
    file_path = Column(String(500), nullable=False)
    file_name = Column(String(255), nullable=False)
    original_file_name = Column(String(255), nullable=False)
    file_size = Column(BigInteger, nullable=False)
    mime_type = Column(String(100), nullable=False)
    
    # 메타데이터
    tags = Column(ARRAY(String))  # PostgreSQL 배열 타입
    difficulty_level = Column(Integer)  # 1-5
    estimated_time = Column(Integer)  # 예상 소요시간 (분)
    
    # 업로드 관리
    uploaded_by = Column(Integer, ForeignKey("users.id"))
    
    # 상태 및 통계
    is_featured = Column(Boolean, default=False, index=True)
    is_active = Column(Boolean, default=True, index=True)
    view_count = Column(Integer, default=0)
    download_count = Column(Integer, default=0)
    
    # 타임스탬프
    created_at = Column(DateTime(timezone=True), server_default=func.now(), index=True)
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    
    # 관계 설정
    category = relationship("Category", back_populates="materials")
    uploader = relationship("User", back_populates="uploaded_materials")


### 4. 로그 모델들 (app/models/log.py)
```python
from sqlalchemy import Column, Integer, String, Text, DateTime, ForeignKey, INET
from sqlalchemy.sql import func
from ..database import Base

class DownloadLog(Base):
    __tablename__ = "download_logs"
    
    id = Column(Integer, primary_key=True, index=True)
    material_id = Column(Integer, ForeignKey("materials.id"), index=True)
    user_ip = Column(INET)  # IP 주소
    user_agent = Column(Text)
    session_id = Column(String(100))
    referer_url = Column(Text)
    download_at = Column(DateTime(timezone=True), server_default=func.now(), index=True)

class ViewLog(Base):
    __tablename__ = "view_logs"
    
    id = Column(Integer, primary_key=True, index=True)
    material_id = Column(Integer, ForeignKey("materials.id"), index=True)
    user_ip = Column(INET)
    user_agent = Column(Text)
    session_id = Column(String(100))
    view_duration = Column(Integer)  # 초 단위
    viewed_at = Column(DateTime(timezone=True), server_default=func.now(), index=True)
```

---

## 📝 스키마 정의 (Pydantic)

### 1. 카테고리 스키마 (app/schemas/category.py)
```python
from pydantic import BaseModel
from typing import Optional, List
from datetime import datetime

# 기본 카테고리 스키마
class CategoryBase(BaseModel):
    name: str
    description: Optional[str] = None
    icon: Optional[str] = None
    color: Optional[str] = None
    sort_order: int = 0
    is_active: bool = True

# 카테고리 생성용
class CategoryCreate(CategoryBase):
    pass

# 카테고리 수정용
class CategoryUpdate(BaseModel):
    name: Optional[str] = None
    description: Optional[str] = None
    icon: Optional[str] = None
    color: Optional[str] = None
    sort_order: Optional[int] = None
    is_active: Optional[bool] = None

# 응답용 카테고리 스키마
class Category(CategoryBase):
    id: int
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    class Config:
        from_attributes = True

# 카테고리별 통계 포함
class CategoryWithStats(Category):
    material_count: int = 0
    total_downloads: int = 0
```

### 2. 교육자료 스키마 (app/schemas/material.py)
```python
from pydantic import BaseModel, validator
from typing import Optional, List
from datetime import datetime

# 기본 교육자료 스키마
class MaterialBase(BaseModel):
    title: str
    description: Optional[str] = None
    content: Optional[str] = None
    category_id: Optional[int] = None
    tags: Optional[List[str]] = []
    difficulty_level: Optional[int] = None
    estimated_time: Optional[int] = None
    is_featured: bool = False
    is_active: bool = True
    
    @validator('difficulty_level')
    def validate_difficulty(cls, v):
        if v is not None and (v < 1 or v > 5):
            raise ValueError('난이도는 1-5 사이의 값이어야 합니다.')
        return v
    
    @validator('tags')
    def validate_tags(cls, v):
        if v and len(v) > 10:
            raise ValueError('태그는 최대 10개까지 가능합니다.')
        return v

# 교육자료 생성용 (파일 정보는 별도 처리)
class MaterialCreate(MaterialBase):
    pass

# 교육자료 수정용
class MaterialUpdate(BaseModel):
    title: Optional[str] = None
    description: Optional[str] = None
    content: Optional[str] = None
    category_id: Optional[int] = None
    tags: Optional[List[str]] = None
    difficulty_level: Optional[int] = None
    estimated_time: Optional[int] = None
    is_featured: Optional[bool] = None
    is_active: Optional[bool] = None

# 응답용 교육자료 스키마
class Material(MaterialBase):
    id: int
    file_name: str
    original_file_name: str
    file_size: int
    mime_type: str
    view_count: int = 0
    download_count: int = 0
    uploaded_by: Optional[int] = None
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    class Config:
        from_attributes = True

# 카테고리 정보 포함된 교육자료
class MaterialWithCategory(Material):
    category: Optional[dict] = None

# 파일 업로드 응답
class FileUploadResponse(BaseModel):
    filename: str
    size: int
    mime_type: str
    message: str

# 검색 및 필터링용
class MaterialFilter(BaseModel):
    search: Optional[str] = None
    category_id: Optional[int] = None
    tags: Optional[List[str]] = []
    difficulty_level: Optional[int] = None
    is_featured: Optional[bool] = None
    skip: int = 0
    limit: int = 20
```

### 3. 사용자 스키마 (app/schemas/user.py)
```python
from pydantic import BaseModel, EmailStr
from typing import Optional
from datetime import datetime

# 기본 사용자 스키마
class UserBase(BaseModel):
    username: str
    email: EmailStr
    full_name: str
    department: Optional[str] = None
    position: Optional[str] = None
    is_active: bool = True

# 사용자 생성용 (관리자만)
class UserCreate(UserBase):
    is_admin: bool = False

# 사용자 수정용
class UserUpdate(BaseModel):
    username: Optional[str] = None
    email: Optional[EmailStr] = None
    full_name: Optional[str] = None
    department: Optional[str] = None
    position: Optional[str] = None
    is_admin: Optional[bool] = None
    is_active: Optional[bool] = None

# 응답용 사용자 스키마
class User(UserBase):
    id: int
    is_admin: bool
    last_login: Optional[datetime] = None
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    class Config:
        from_attributes = True

# 관리자 인증용 토큰
class Token(BaseModel):
    access_token: str
    token_type: str = "bearer"
    user: User

class TokenData(BaseModel):
    username: Optional[str] = None
```

---

## 🛡️ 보안 및 인증 (app/core/security.py)

```python
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import HTTPException, status
from .config import settings

# 비밀번호 해싱
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """비밀번호 검증"""
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    """비밀번호 해싱"""
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    """JWT 토큰 생성"""
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=settings.access_token_expire_minutes)
    
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, settings.secret_key, algorithm=settings.algorithm)
    return encoded_jwt

def verify_token(token: str) -> dict:
    """JWT 토큰 검증"""
    try:
        payload = jwt.decode(token, settings.secret_key, algorithms=[settings.algorithm])
        username: str = payload.get("sub")
        if username is None:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="토큰이 유효하지 않습니다",
                headers={"WWW-Authenticate": "Bearer"},
            )
        return payload
    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="토큰이 유효하지 않습니다",
            headers={"WWW-Authenticate": "Bearer"},
        )
```

---

## 🔌 API 라우터 구현

### 1. 교육자료 API (app/routers/materials.py)
```python
from fastapi import APIRouter, Depends, HTTPException, status, UploadFile, File, Form, Request
from sqlalchemy.orm import Session
from typing import List, Optional
import os
import uuid
import aiofiles
from ..database import get_db
from ..models.material import Material
from ..models.category import Category
from ..models.log import DownloadLog, ViewLog
from ..schemas.material import (
    Material as MaterialSchema, 
    MaterialCreate, 
    MaterialUpdate,
    MaterialFilter,
    FileUploadResponse
)
from ..core.config import settings
from ..utils.deps import get_current_admin_user

router = APIRouter()

@router.get("/", response_model=List[MaterialSchema])
async def get_materials(
    request: Request,
    skip: int = 0,
    limit: int = 20,
    search: Optional[str] = None,
    category_id: Optional[int] = None,
    is_featured: Optional[bool] = None,
    db: Session = Depends(get_db)
):
    """교육자료 목록 조회 (로그인 불필요)"""
    query = db.query(Material).filter(Material.is_active == True)
    
    # 검색 필터
    if search:
        query = query.filter(Material.title.ilike(f"%{search}%"))
    
    # 카테고리 필터
    if category_id:
        query = query.filter(Material.category_id == category_id)
    
    # 추천 자료 필터
    if is_featured is not None:
        query = query.filter(Material.is_featured == is_featured)
    
    # 최신순 정렬
    materials = query.order_by(Material.created_at.desc()).offset(skip).limit(limit).all()
    
    return materials

@router.get("/{material_id}", response_model=MaterialSchema)
async def get_material(
    material_id: int,
    request: Request,
    db: Session = Depends(get_db)
):
    """교육자료 상세 조회"""
    material = db.query(Material).filter(
        Material.id == material_id,
        Material.is_active == True
    ).first()
    
    if not material:
        raise HTTPException(status_code=404, detail="자료를 찾을 수 없습니다")
    
    # 조회 로그 기록
    try:
        client_ip = request.client.host
        user_agent = request.headers.get("user-agent", "")
        
        view_log = ViewLog(
            material_id=material_id,
            user_ip=client_ip,
            user_agent=user_agent
        )
        db.add(view_log)
        
        # 조회수 증가
        material.view_count += 1
        db.commit()
        
    except Exception as e:
        # 로깅 실패해도 자료 조회는 계속 진행
        db.rollback()
        print(f"View log error: {e}")
    
    return material

@router.post("/", response_model=MaterialSchema)
async def create_material(
    title: str = Form(...),
    description: str = Form(None),
    category_id: int = Form(None),
    tags: str = Form(""),  # 쉼표로 구분된 문자열
    difficulty_level: int = Form(None),
    estimated_time: int = Form(None),
    is_featured: bool = Form(False),
    file: UploadFile = File(...),
    db: Session = Depends(get_db),
    current_user = Depends(get_current_admin_user)
):
    """교육자료 업로드 (관리자만)"""
    
    # 파일 크기 검증
    if file.size > settings.max_file_size * 1024 * 1024:
        raise HTTPException(
            status_code=400, 
            detail=f"파일 크기가 {settings.max_file_size}MB를 초과합니다"
        )
    
    # 파일 확장자 검증
    file_extension = file.filename.split(".")[-1].lower()
    if file_extension not in settings.allowed_file_types:
        raise HTTPException(
            status_code=400,
            detail=f"지원하지 않는 파일 형식입니다. 허용된 형식: {', '.join(settings.allowed_file_types)}"
        )
    
    # 고유한 파일명 생성
    unique_filename = f"{uuid.uuid4()}.{file_extension}"
    file_path = os.path.join(settings.upload_dir, unique_filename)
    
    # 파일 저장
    try:
        async with aiofiles.open(file_path, 'wb') as f:
            content = await file.read()
            await f.write(content)
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"파일 저장 실패: {str(e)}")
    
    # 태그 처리
    tag_list = [tag.strip() for tag in tags.split(",") if tag.strip()] if tags else []
    
    # 데이터베이스에 저장
    db_material = Material(
        title=title,
        description=description,
        category_id=category_id,
        file_path=file_path,
        file_name=unique_filename,
        original_file_name=file.filename,
        file_size=file.size,
        mime_type=file.content_type,
        tags=tag_list,
        difficulty_level=difficulty_level,
        estimated_time=estimated_time,
        is_featured=is_featured,
        uploaded_by=current_user.id
    )
    
    db.add(db_material)
    db.commit()
    db.refresh(db_material)
    
    return db_material

@router.get("/{material_id}/download")
async def download_material(
    material_id: int,
    request: Request,
    db: Session = Depends(get_db)
):
    """교육자료 다운로드"""
    material = db.query(Material).filter(
        Material.id == material_id,
        Material.is_active == True
    ).first()
    
    if not material:
        raise HTTPException(status_code=404, detail="자료를 찾을 수 없습니다")
    
    # 파일 존재 확인
    if not os.path.exists(material.file_path):
        raise HTTPException(status_code=404, detail="파일을 찾을 수 없습니다")
    
    # 다운로드 로그 기록
    try:
        client_ip = request.client.host
        user_agent = request.headers.get("user-agent", "")
        
        download_log = DownloadLog(
            material_id=material_id,
            user_ip=client_ip,
            user_agent=user_agent
        )
        db.add(download_log)
        db.commit()
        
    except Exception as e:
        print(f"Download log error: {e}")
    
    # 파일 응답
    from fastapi.responses import FileResponse
    return FileResponse(
        path=material.file_path,
        filename=material.original_file_name,
        media_type=material.mime_type
    )

@router.put("/{material_id}", response_model=MaterialSchema)
async def update_material(
    material_id: int,
    material_update: MaterialUpdate,
    db: Session = Depends(get_db),
    current_user = Depends(get_current_admin_user)
):
    """교육자료 수정 (관리자만)"""
    material = db.query(Material).filter(Material.id == material_id).first()
    
    if not material:
        raise HTTPException(status_code=404, detail="자료를 찾을 수 없습니다")
    
    # 필드 업데이트
    update_data = material_update.dict(exclude_unset=True)
    for field, value in update_data.items():
        setattr(material, field, value)
    
    db.commit()
    db.refresh(material)
    
    return material

@router.delete("/{material_id}")
async def delete_material(
    material_id: int,
    db: Session = Depends(get_db),
    current_user = Depends(get_current_admin_user)
):
    """교육자료 삭제 (관리자만)"""
    material = db.query(Material).filter(Material.id == material_id).first()
    
    if not material:
        raise HTTPException(status_code=404, detail="자료를 찾을 수 없습니다")
    
    # 실제 파일 삭제
    try:
        if os.path.exists(material.file_path):
            os.remove(material.file_path)
    except Exception as e:
        print(f"File deletion error: {e}")
    
    # 데이터베이스에서 삭제 (또는 비활성화)
    db.delete(material)  # 실제 삭제
    # material.is_active = False  # 소프트 삭제 원할 경우
    
    db.commit()
    
    return {"message": "자료가 삭제되었습니다"}
```

### 2. 카테고리 API (app/routers/categories.py)
```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import List
from ..database import get_db
from ..models.category import Category
from ..schemas.category import Category as CategorySchema, CategoryCreate, CategoryUpdate, CategoryWithStats
from ..utils.deps import get_current_admin_user

router = APIRouter()

@router.get("/", response_model=List[CategoryWithStats])
async def get_categories(
    include_inactive: bool = False,
    db: Session = Depends(get_db)
):
    """카테고리 목록 조회 (통계 포함)"""
    query = db.query(Category)
    
    if not include_inactive:
        query = query.filter(Category.is_active == True)
    
    categories = query.order_by(Category.sort_order, Category.name).all()
    
    # 각 카테고리별 통계 추가
    result = []
    for category in categories:
        category_dict = {
            "id": category.id,
            "name": category.name,
            "description": category.description,
            "icon": category.icon,
            "color": category.color,
            "sort_order": category.sort_order,
            "is_active": category.is_active,
            "created_at": category.created_at,
            "updated_at": category.updated_at,
            "material_count": len([m for m in category.materials if m.is_active]),
            "total_downloads": sum(m.download_count for m in category.materials if m.is_active)
        }
        result.append(category_dict)
    
    return result

@router.post("/", response_model=CategorySchema)
async def create_category(
    category: CategoryCreate,
    db: Session = Depends(get_db),
    current_user = Depends(get_current_admin_user)
):
    """카테고리 생성 (관리자만)"""
    
    # 이름 중복 확인
    existing = db.query(Category).filter(Category.name == category.name).first()
    if existing:
        raise HTTPException(status_code=400, detail="이미 존재하는 카테고리명입니다")
    
    db_category = Category(**category.dict())
    db.add(db_category)
    db.commit()
    db.refresh(db_category)
    
    return db_category

@router.put("/{category_id}", response_model=CategorySchema)
async def update_category(
    category_id: int,
    category_update: CategoryUpdate,
    db: Session = Depends(get_db),
    current_user = Depends(get_current_admin_user)
):
    """카테고리 수정 (관리자만)"""
    category = db.query(Category).filter(Category.id == category_id).first()
    
    if not category:
        raise HTTPException(status_code=404, detail="카테고리를 찾을 수 없습니다")
    
    update_data = category_update.dict(exclude_unset=True)
    for field, value in update_data.items():
        setattr(category, field, value)
    
    db.commit()
    db.refresh(category)
    
    return category
```

### 3. 관리자 API (app/routers/admin.py)
```python
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.orm import Session
from datetime import datetime, timedelta
from typing import List
from ..database import get_db
from ..models.user import User
from ..models.material import Material
from ..models.log import DownloadLog, ViewLog
from ..schemas.user import User as UserSchema, Token
from ..core.security import create_access_token
from ..core.config import settings
from ..utils.deps import get_current_admin_user

router = APIRouter()
security = HTTPBearer()

@router.post("/login", response_model=Token)
async def admin_login(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: Session = Depends(get_db)
):
    """관리자 로그인 (향후 SSO 연동 대비)"""
    # 현재는 간단한 토큰 기반 인증
    # 실제 환경에서는 SSO 연동으로 교체 예정
    
    # 하드코딩된 관리자 확인 (임시)
    if credentials.credentials != "admin-secret-token":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="유효하지 않은 인증 정보입니다"
        )
    
    # 관리자 사용자 조회 또는 생성
    admin_user = db.query(User).filter(
        User.username == settings.admin_username
    ).first()
    
    if not admin_user:
        # 관리자 계정이 없으면 생성
        admin_user = User(
            username=settings.admin_username,
            email=settings.admin_email,
            full_name=settings.admin_full_name,
            is_admin=True,
            is_active=True
        )
        db.add(admin_user)
        db.commit()
        db.refresh(admin_user)
    
    # 로그인 시간 업데이트
    admin_user.last_login = datetime.utcnow()
    db.commit()
    
    # JWT 토큰 생성
    access_token_expires = timedelta(minutes=settings.access_token_expire_minutes)
    access_token = create_access_token(
        data={"sub": admin_user.username},
        expires_delta=access_token_expires
    )
    
    return {
        "access_token": access_token,
        "token_type": "bearer",
        "user": admin_user
    }

@router.get("/dashboard")
async def get_dashboard_stats(
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_admin_user)
):
    """관리자 대시보드 통계"""
    
    # 기본 통계
    total_materials = db.query(Material).filter(Material.is_active == True).count()
    total_downloads = db.query(DownloadLog).count()
    total_views = db.query(ViewLog).count()
    
    # 최근 30일 다운로드 추이
    thirty_days_ago = datetime.utcnow() - timedelta(days=30)
    recent_downloads = db.query(DownloadLog).filter(
        DownloadLog.download_at >= thirty_days_ago
    ).count()
    
    # 인기 자료 TOP 5
    popular_materials = db.query(Material).filter(
        Material.is_active == True
    ).order_by(
        Material.download_count.desc()
    ).limit(5).all()
    
    return {
        "total_materials": total_materials,
        "total_downloads": total_downloads,
        "total_views": total_views,
        "recent_downloads": recent_downloads,
        "popular_materials": [
            {
                "id": m.id,
                "title": m.title,
                "download_count": m.download_count,
                "view_count": m.view_count
            }
            for m in popular_materials
        ]
    }

@router.get("/logs/downloads")
async def get_download_logs(
    skip: int = 0,
    limit: int = 100,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_admin_user)
):
    """다운로드 로그 조회 (관리자만)"""
    logs = db.query(DownloadLog).order_by(
        DownloadLog.download_at.desc()
    ).offset(skip).limit(limit).all()
    
    return logs
```

---

## 🔧 의존성 주입 (app/utils/deps.py)
```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.orm import Session
from ..database import get_db
from ..models.user import User
from ..core.security import verify_token

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: Session = Depends(get_db)
) -> User:
    """현재 사용자 조회 (JWT 토큰 기반)"""
    
    # 토큰 검증
    payload = verify_token(credentials.credentials)
    username = payload.get("sub")
    
    # 사용자 조회
    user = db.query(User).filter(User.username == username).first()
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="사용자를 찾을 수 없습니다"
        )
    
    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="비활성화된 사용자입니다"
        )
    
    return user

async def get_current_admin_user(
    current_user: User = Depends(get_current_user)
) -> User:
    """관리자 권한 확인"""
    if not current_user.is_admin:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="관리자 권한이 필요합니다"
        )
    return current_user
```

---

## 🚀 메인 애플리케이션 (app/main.py)
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.staticfiles import StaticFiles
from .core.config import settings
from .database import engine, Base
from .routers import materials, categories, admin

# 데이터베이스 테이블 생성 (개발환경에서만)
# 운영환경에서는 Alembic 사용 권장
Base.metadata.create_all(bind=engine)

# FastAPI 앱 생성
app = FastAPI(
    title="HR 교육자료 관리 시스템 API",
    description="HR팀을 위한 교육자료 업로드 및 관리 API",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

# CORS 설정
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.backend_cors_origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 정적 파일 서빙 (업로드된 파일들)
app.mount("/uploads", StaticFiles(directory="uploads"), name="uploads")

# 라우터 등록
app.include_router(
    materials.router,
    prefix="/api/materials",
    tags=["교육자료"]
)

app.include_router(
    categories.router,
    prefix="/api/categories",
    tags=["카테고리"]
)

app.include_router(
    admin.router,
    prefix="/api/admin",
    tags=["관리자"]
)

@app.get("/")
async def root():
    return {
        "message": "HR 교육자료 관리 시스템 API",
        "version": "1.0.0",
        "docs": "/docs"
    }

@app.get("/api/health")
async def health_check():
    """시스템 상태 확인"""
    return {
        "status": "healthy",
        "timestamp": datetime.utcnow().isoformat()
    }

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(
        "app.main:app",
        host="0.0.0.0",
        port=8000,
        reload=True  # 개발환경에서만
    )
```

---

## 🤖 CursorAI 활용 팁

### 백엔드 개발 시 유용한 프롬프트

#### 1. API 엔드포인트 구현 요청
```
"FastAPI에서 교육자료 검색 API를 만들어줘.
- 제목, 설명, 태그로 검색 가능
- 카테고리별 필터링
- 페이지네이션 포함
- 로그인 없이 접근 가능
- 검색 로그도 기록"
```

#### 2. 데이터베이스 쿼리 최적화
```
"다음 SQLAlchemy 쿼리를 최적화해줘:
[느린 쿼리 코드]

N+1 쿼리 문제를 해결하고 인덱스 활용도 개선해줘."
```

#### 3. 파일 업로드 기능 구현
```
"FastAPI에서 대용량 파일 업로드를 처리하는 안전한 방법 알려줘.
- 파일 크기 제한
- 확장자 검증
- 바이러스 스캔 (선택)
- 진행률 표시 가능한 구조
- 중복 파일명 처리"
```

#### 4. 에러 처리 및 로깅
```
"FastAPI에서 전역 예외 처리기를 만들어줘.
- 사용자 친화적인 에러 메시지
- 상세한 로그 기록
- 에러 타입별 다른 응답
- 개발/운영 환경 구분"
```

#### 5. 성능 최적화
```
"FastAPI 앱의 성능을 개선하는 방법들 알려줘:
- 데이터베이스 연결 풀링
- 캐싱 전략
- 비동기 처리
- 메모리 사용량 최적화"
```

---

## 🧪 테스트 및 디버깅

### 개발 서버 실행
```bash
# 가상환경 활성화 후
cd backend

# 개발 서버 실행 (자동 재시작)
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# 또는 Python 직접 실행
python -m app.main
```

### API 테스트

#### 1. 브라우저에서 테스트
```
# API 문서 확인
http://localhost:8000/docs

# 또는 ReDoc
http://localhost:8000/redoc

# 헬스체크
http://localhost:8000/api/health
```

#### 2. cURL로 테스트
```bash
# 카테고리 목록 조회
curl -X GET "http://localhost:8000/api/categories"

# 교육자료 목록 조회
curl -X GET "http://localhost:8000/api/materials"

# 교육자료 검색
curl -X GET "http://localhost:8000/api/materials?search=신입사원&limit=5"
```

#### 3. Python으로 테스트
```python
import requests

# 기본 테스트
response = requests.get("http://localhost:8000/api/materials")
print(response.json())

# 관리자 로그인 테스트
headers = {"Authorization": "Bearer admin-secret-token"}
response = requests.post(
    "http://localhost:8000/api/admin/login",
    headers=headers
)
print(response.json())
```

### 단위 테스트 (tests/test_main.py)
```python
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.main import app
from app.database import get_db, Base

# 테스트용 인메모리 데이터베이스
SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base.metadata.create_all(bind=engine)

def override_get_db():
    try:
        db = TestingSessionLocal()
        yield db
    finally:
        db.close()

app.dependency_overrides[get_db] = override_get_db
client = TestClient(app)

def test_read_main():
    """기본 엔드포인트 테스트"""
    response = client.get("/")
    assert response.status_code == 200
    assert "HR 교육자료 관리 시스템" in response.json()["message"]

def test_health_check():
    """헬스체크 테스트"""
    response = client.get("/api/health")
    assert response.status_code == 200
    assert response.json()["status"] == "healthy"

def test_get_categories():
    """카테고리 목록 조회 테스트"""
    response = client.get("/api/categories")
    assert response.status_code == 200
    assert isinstance(response.json(), list)

def test_get_materials():
    """교육자료 목록 조회 테스트"""
    response = client.get("/api/materials")
    assert response.status_code == 200
    assert isinstance(response.json(), list)
```

### 테스트 실행
```bash
# pytest 설치
pip install pytest pytest-asyncio httpx

# 테스트 실행
pytest

# 상세 출력과 함께 테스트
pytest -v

# 특정 테스트 파일 실행
pytest tests/test_main.py

# 커버리지와 함께 테스트
pip install pytest-cov
pytest --cov=app
```

---

## 🚨 일반적인 문제 해결

### 1. 데이터베이스 연결 오류
```python
# 연결 확인
from app.database import engine
from sqlalchemy import text

try:
    with engine.connect() as conn:
        result = conn.execute(text("SELECT 1"))
        print("데이터베이스 연결 성공")
except Exception as e:
    print(f"데이터베이스 연결 실패: {e}")
```

### 2. 모델 및 테이블 동기화 문제
```bash
# Alembic으로 마이그레이션 관리
pip install alembic

# Alembic 초기화
alembic init alembic

# 마이그레이션 생성
alembic revision --autogenerate -m "Initial migration"

# 마이그레이션 적용
alembic upgrade head
```

### 3. 파일 업로드 오류
```python
# 업로드 디렉터리 확인
import os
from app.core.config import settings

if not os.path.exists(settings.upload_dir):
    os.makedirs(settings.upload_dir)
    print(f"업로드 디렉터리 생성: {settings.upload_dir}")

# 권한 확인 (Linux/macOS)
os.chmod(settings.upload_dir, 0o755)
```

### 4. CORS 오류
```python
# main.py에서 CORS 설정 확인
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "http://localhost:5173"],  # 프론트엔드 주소
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### 5. 메모리 사용량 최적화
```python
# 대용량 파일 처리시 스트리밍
from fastapi.responses import StreamingResponse
import aiofiles

async def stream_file(file_path: str):
    async with aiofiles.open(file_path, 'rb') as file:
        while True:
            chunk = await file.read(8192)  # 8KB씩 읽기
            if not chunk:
                break
            yield chunk

@router.get("/stream/{material_id}")
async def stream_material(material_id: int):
    # 파일 스트리밍 응답
    return StreamingResponse(
        stream_file(material.file_path),
        media_type=material.mime_type
    )
```

---

## 📈 성능 모니터링

### 로깅 설정
```python
# app/core/logging.py
import logging
import sys
from pathlib import Path

# 로그 디렉터리 생성
log_dir = Path("logs")
log_dir.mkdir(exist_ok=True)

# 로거 설정
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
    handlers=[
        logging.FileHandler("logs/app.log"),
        logging.StreamHandler(sys.stdout)
    ]
)

logger = logging.getLogger("hr_education_system")
```

### API 응답 시간 미들웨어
```python
# app/middleware/timing.py
import time
from fastapi import Request, Response
from fastapi.middleware.base import BaseHTTPMiddleware

class TimingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start_time = time.time()
        response = await call_next(request)
        process_time = time.time() - start_time
        
        response.headers["X-Process-Time"] = str(process_time)
        
        # 느린 요청 로깅 (2초 이상)
        if process_time > 2.0:
            logger.warning(f"Slow request: {request.url} took {process_time:.2f}s")
        
        return response

# main.py에서 추가
app.add_middleware(TimingMiddleware)
```

---

## 🚀 배포 준비

### Docker 설정 (Dockerfile)
```dockerfile
FROM python:3.9-slim

# 작업 디렉터리 설정
WORKDIR /app

# 시스템 패키지 설치
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Python 패키지 설치
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 앱 코드 복사
COPY . .

# 업로드 디렉터리 생성
RUN mkdir -p uploads

# 포트 노출
EXPOSE 8000

# 헬스체크
HEALTHCHECK --interval=30s --timeout=30s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/api/health || exit 1

# 애플리케이션 실행
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 환경 변수 관리 (.env.production)
```env
DATABASE_URL=postgresql://user:password@db:5432/hr_education_system
SECRET_KEY=super-secure-production-key-here
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=1440

MAX_FILE_SIZE=100
UPLOAD_DIR=/app/uploads

# 운영환경 설정
DEBUG=False
LOG_LEVEL=INFO
```

---

## 📝 다음 단계

백엔드 개발이 완료되었다면:

1. ✅ **API 테스트 완료**
   ```bash
   # 모든 엔드포인트 테스트
   pytest -v
   
   # API 문서 확인
   http://localhost:8000/docs
   ```

2. ✅ **데이터베이스 연결 확인**
   ```bash
   # 데이터 조회 테스트
   curl http://localhost:8000/api/categories
   curl http://localhost:8000/api/materials
   ```

3. 🔄 **다음 가이드로 이동**
   - **프론트엔드 개발**: [FRONTEND_GUIDE.md](FRONTEND_GUIDE.md)

---

## 🔗 추가 리소스

- [FastAPI 공식 문서](https://fastapi.tiangolo.com/)
- [SQLAlchemy 문서](https://docs.sqlalchemy.org/)
- [Pydantic 문서](https://docs.pydantic.dev/)
- [Uvicorn 문서](https://www.uvicorn.org/)
- [Python 비동기 프로그래밍](https://docs.python.org/3/library/asyncio.html)

---

**백엔드 API 개발 완료! 🎉**  
이제 프론트엔드를 개발하여 사용자 인터페이스를 만들어보세요!
