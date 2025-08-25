# 🐍 백엔드 고급 가이드

> 운영 환경을 위한 보안, 성능, 모니터링 고도화

## 📋 목차
1. [인증 및 권한 시스템](#인증-및-권한-시스템)
2. [고급 에러 처리 및 로깅](#고급-에러-처리-및-로깅)
3. [성능 최적화](#성능-최적화)
4. [보안 강화](#보안-강화)
5. [모니터링 및 헬스체크](#모니터링-및-헬스체크)
6. [테스트 자동화](#테스트-자동화)
7. [배포 및 CI/CD](#배포-및-cicd)

---

## 🔐 인증 및 권한 시스템

### JWT 토큰 기반 인증

#### 추가 패키지 설치
```bash
pip install python-jose[cryptography] passlib[bcrypt] python-multipart
```

#### 보안 설정 (app/core/security.py)
```python
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import HTTPException, status

# 환경 변수 (실제로는 .env 파일에서 로드)
SECRET_KEY = "your-super-secret-key-change-this-in-production"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 1440

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

def verify_token(token: str) -> dict:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
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

#### 의존성 주입 시스템 (app/core/deps.py)
```python
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.orm import Session
from .database import get_db
from .models import User
from .core.security import verify_token

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: Session = Depends(get_db)
) -> User:
    payload = verify_token(credentials.credentials)
    username = payload.get("sub")
    
    user = db.query(User).filter(User.username == username).first()
    if not user or not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="사용자를 찾을 수 없거나 비활성 상태입니다"
        )
    return user

async def get_current_admin_user(
    current_user: User = Depends(get_current_user)
) -> User:
    if not current_user.is_admin:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="관리자 권한이 필요합니다"
        )
    return current_user

# 선택적 인증 (로그인 없이도 접근 가능하지만, 로그인시 추가 정보 제공)
async def get_current_user_optional(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: Session = Depends(get_db)
) -> Optional[User]:
    if not credentials:
        return None
    try:
        return await get_current_user(credentials, db)
    except HTTPException:
        return None
```

### SSO 연동 준비 (OAuth2)
```python
# app/core/oauth.py
from fastapi import APIRouter, Depends, HTTPException
from fastapi.security import OAuth2AuthorizationCodeBearer
import httpx

router = APIRouter()

# OAuth2 설정 (예: Google, Microsoft Azure AD)
oauth2_scheme = OAuth2AuthorizationCodeBearer(
    authorizationUrl="https://accounts.google.com/o/oauth2/auth",
    tokenUrl="https://oauth2.googleapis.com/token",
)

@router.get("/auth/google")
async def google_oauth_callback(code: str, db: Session = Depends(get_db)):
    """Google OAuth2 콜백 처리"""
    # Google에서 토큰 교환
    async with httpx.AsyncClient() as client:
        token_response = await client.post(
            "https://oauth2.googleapis.com/token",
            data={
                "client_id": "your-google-client-id",
                "client_secret": "your-google-client-secret",
                "code": code,
                "grant_type": "authorization_code",
                "redirect_uri": "your-redirect-uri"
            }
        )
    
    # 사용자 정보 가져오기 및 계정 생성/업데이트
    # 구현 세부사항은 실제 SSO 제공업체에 따라 달라짐
    
    return {"message": "OAuth2 인증 성공"}
```

---

## 🛡️ 고급 에러 처리 및 로깅

### 전역 예외 처리기
```python
# app/core/exceptions.py
from fastapi import Request, HTTPException
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError
import logging
import traceback

logger = logging.getLogger(__name__)

async def http_exception_handler(request: Request, exc: HTTPException):
    """HTTP 예외 처리"""
    logger.warning(f"HTTP {exc.status_code}: {exc.detail} - {request.url}")
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": True,
            "message": exc.detail,
            "status_code": exc.status_code,
            "path": str(request.url)
        }
    )

async def validation_exception_handler(request: Request, exc: RequestValidationError):
    """검증 예외 처리"""
    logger.warning(f"Validation error: {exc.errors()} - {request.url}")
    return JSONResponse(
        status_code=422,
        content={
            "error": True,
            "message": "입력 데이터가 올바르지 않습니다",
            "details": exc.errors(),
            "path": str(request.url)
        }
    )

async def general_exception_handler(request: Request, exc: Exception):
    """일반 예외 처리"""
    logger.error(f"Unexpected error: {str(exc)} - {request.url}")
    logger.error(traceback.format_exc())
    
    return JSONResponse(
        status_code=500,
        content={
            "error": True,
            "message": "서버 내부 오류가 발생했습니다",
            "path": str(request.url)
        }
    )
```

### 고급 로깅 시스템
```python
# app/core/logging.py
import logging
import sys
from pathlib import Path
from logging.handlers import RotatingFileHandler, TimedRotatingFileHandler

def setup_logging():
    """로깅 시스템 설정"""
    
    # 로그 디렉터리 생성
    log_dir = Path("logs")
    log_dir.mkdir(exist_ok=True)
    
    # 루트 로거 설정
    root_logger = logging.getLogger()
    root_logger.setLevel(logging.INFO)
    
    # 포맷터 생성
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )
    
    # 콘솔 핸들러
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setLevel(logging.INFO)
    console_handler.setFormatter(formatter)
    
    # 파일 핸들러 (일반 로그)
    file_handler = RotatingFileHandler(
        "logs/app.log",
        maxBytes=10*1024*1024,  # 10MB
        backupCount=5
    )
    file_handler.setLevel(logging.INFO)
    file_handler.setFormatter(formatter)
    
    # 에러 로그 핸들러
    error_handler = RotatingFileHandler(
        "logs/error.log",
        maxBytes=10*1024*1024,
        backupCount=5
    )
    error_handler.setLevel(logging.ERROR)
    error_handler.setFormatter(formatter)
    
    # 핸들러 추가
    root_logger.addHandler(console_handler)
    root_logger.addHandler(file_handler)
    root_logger.addHandler(error_handler)
    
    return root_logger

# 애플리케이션별 로거 생성
def get_logger(name: str):
    return logging.getLogger(name)
```

### API 요청/응답 로깅 미들웨어
```python
# app/middleware/logging.py
import time
import json
from fastapi import Request, Response
from fastapi.middleware.base import BaseHTTPMiddleware
import logging

logger = logging.getLogger("api")

class APILoggingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start_time = time.time()
        
        # 요청 로깅
        logger.info(f"📥 {request.method} {request.url} - Client: {request.client.host}")
        
        # 요청 본문 로깅 (민감한 데이터 제외)
        if request.method in ["POST", "PUT", "PATCH"]:
            try:
                body = await request.body()
                if body and len(body) < 1000:  # 1KB 이하만 로깅
                    content_type = request.headers.get("content-type", "")
                    if "application/json" in content_type:
                        try:
                            json_body = json.loads(body)
                            # 비밀번호 등 민감한 필드 마스킹
                            if "password" in json_body:
                                json_body["password"] = "***"
                            logger.info(f"📝 Request body: {json_body}")
                        except json.JSONDecodeError:
                            pass
            except Exception:
                pass
        
        # 요청 처리
        response = await call_next(request)
        
        # 응답 시간 계산
        process_time = time.time() - start_time
        
        # 응답 로깅
        logger.info(
            f"📤 {request.method} {request.url} - "
            f"Status: {response.status_code} - "
            f"Time: {process_time:.3f}s"
        )
        
        # 느린 요청 경고
        if process_time > 2.0:
            logger.warning(f"🐌 Slow request: {request.url} took {process_time:.3f}s")
        
        # 응답 헤더에 처리 시간 추가
        response.headers["X-Process-Time"] = str(process_time)
        
        return response
```

---

## ⚡ 성능 최적화

### 데이터베이스 최적화
```python
# app/core/database.py (개선된 버전)
from sqlalchemy import create_engine, event
from sqlalchemy.pool import QueuePool
import logging

logger = logging.getLogger("database")

# 연결 풀 최적화
engine = create_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=20,          # 기본 연결 수
    max_overflow=30,       # 최대 추가 연결 수
    pool_pre_ping=True,    # 연결 유효성 검사
    pool_recycle=3600,     # 1시간마다 연결 재생성
    echo=False,            # SQL 로깅 (개발시에만 True)
    connect_args={
        "options": "-c timezone=Asia/Seoul"
    }
)

# 데이터베이스 이벤트 리스너 (쿼리 성능 모니터링)
@event.listens_for(engine, "before_cursor_execute")
def receive_before_cursor_execute(conn, cursor, statement, parameters, context, executemany):
    context._query_start_time = time.time()

@event.listens_for(engine, "after_cursor_execute")
def receive_after_cursor_execute(conn, cursor, statement, parameters, context, executemany):
    total = time.time() - context._query_start_time
    if total > 0.1:  # 100ms 이상 걸리는 쿼리 로깅
        logger.warning(f"Slow query ({total:.3f}s): {statement[:200]}...")
```

### 캐싱 시스템
```python
# app/core/cache.py
from functools import wraps
import json
import hashlib
from typing import Optional, Any
import redis
import pickle

# Redis 클라이언트 (선택사항)
redis_client = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

# 메모리 캐시 (간단한 구현)
_memory_cache = {}

class CacheManager:
    def __init__(self, use_redis: bool = False):
        self.use_redis = use_redis and redis_client.ping()
    
    def _generate_key(self, prefix: str, *args, **kwargs) -> str:
        """캐시 키 생성"""
        data = f"{prefix}:{args}:{sorted(kwargs.items())}"
        return hashlib.md5(data.encode()).hexdigest()
    
    def get(self, key: str) -> Optional[Any]:
        """캐시에서 값 가져오기"""
        if self.use_redis:
            try:
                value = redis_client.get(key)
                return pickle.loads(value) if value else None
            except Exception as e:
                logger.warning(f"Redis get error: {e}")
                return None
        else:
            return _memory_cache.get(key)
    
    def set(self, key: str, value: Any, expire: int = 300):
        """캐시에 값 저장"""
        if self.use_redis:
            try:
                redis_client.setex(key, expire, pickle.dumps(value))
            except Exception as e:
                logger.warning(f"Redis set error: {e}")
        else:
            _memory_cache[key] = value
            # 메모리 캐시는 간단하게 구현 (만료 시간 무시)
    
    def delete(self, key: str):
        """캐시에서 값 삭제"""
        if self.use_redis:
            try:
                redis_client.delete(key)
            except Exception:
                pass
        else:
            _memory_cache.pop(key, None)

cache_manager = CacheManager()

def cached(prefix: str, expire: int = 300):
    """캐시 데코레이터"""
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # 캐시 키 생성
            cache_key = cache_manager._generate_key(prefix, *args, **kwargs)
            
            # 캐시에서 조회
            cached_result = cache_manager.get(cache_key)
            if cached_result is not None:
                logger.info(f"Cache hit: {cache_key[:16]}...")
                return cached_result
            
            # 함수 실행
            result = await func(*args, **kwargs)
            
            # 결과 캐시 저장
            cache_manager.set(cache_key, result, expire)
            logger.info(f"Cache set: {cache_key[:16]}...")
            
            return result
        return wrapper
    return decorator
```

### 비동기 처리 및 백그라운드 태스크
```python
# app/core/tasks.py
import asyncio
from typing import Callable, Any
from concurrent.futures import ThreadPoolExecutor
import logging

logger = logging.getLogger("tasks")

# 스레드 풀 생성
thread_pool = ThreadPoolExecutor(max_workers=4)

class BackgroundTasks:
    def __init__(self):
        self._tasks = []
    
    def add_task(self, func: Callable, *args, **kwargs):
        """백그라운드 태스크 추가"""
        self._tasks.append((func, args, kwargs))
    
    async def run_tasks(self):
        """모든 태스크 실행"""
        tasks = []
        for func, args, kwargs in self._tasks:
            if asyncio.iscoroutinefunction(func):
                tasks.append(func(*args, **kwargs))
            else:
                # 동기 함수는 스레드 풀에서 실행
                task = asyncio.get_event_loop().run_in_executor(
                    thread_pool, func, *args, **kwargs
                )
                tasks.append(task)
        
        if tasks:
            await asyncio.gather(*tasks, return_exceptions=True)

# 사용 예시: 파일 처리 백그라운드 태스크
async def process_uploaded_file(file_path: str, material_id: int):
    """업로드된 파일 후처리 (썸네일 생성, 바이러스 검사 등)"""
    try:
        # 파일 처리 로직
        logger.info(f"Processing file: {file_path}")
        
        # 예: PDF 썸네일 생성
        # await generate_pdf_thumbnail(file_path)
        
        # 예: 파일 메타데이터 추출
        # metadata = await extract_file_metadata(file_path)
        
        logger.info(f"File processing completed: {file_path}")
        
    except Exception as e:
        logger.error(f"File processing failed: {file_path}, error: {e}")
```

### 페이지네이션 최적화
```python
# app/core/pagination.py
from typing import Generic, TypeVar, List, Optional
from pydantic import BaseModel
from sqlalchemy.orm import Query
from math import ceil

T = TypeVar('T')

class PaginationParams(BaseModel):
    page: int = 1
    size: int = 20
    max_size: int = 100
    
    def __post_init__(self):
        if self.size > self.max_size:
            self.size = self.max_size
        if self.page < 1:
            self.page = 1

class PaginatedResponse(BaseModel, Generic[T]):
    items: List[T]
    total: int
    page: int
    size: int
    pages: int
    has_next: bool
    has_prev: bool

async def paginate(
    query: Query,
    params: PaginationParams,
    count_query: Optional[Query] = None
) -> PaginatedResponse:
    """페이지네이션 헬퍼"""
    
    # 전체 개수 조회 (최적화된 카운트 쿼리 사용 가능)
    if count_query:
        total = count_query.scalar()
    else:
        total = query.count()
    
    # 페이지 계산
    pages = ceil(total / params.size)
    offset = (params.page - 1) * params.size
    
    # 데이터 조회
    items = query.offset(offset).limit(params.size).all()
    
    return PaginatedResponse(
        items=items,
        total=total,
        page=params.page,
        size=params.size,
        pages=pages,
        has_next=params.page < pages,
        has_prev=params.page > 1
    )
```

---

## 🔒 보안 강화

### 입력 검증 및 새니타이징
```python
# app/core/validators.py
import re
from typing import Optional
from pydantic import validator
import bleach

class SecurityValidators:
    @staticmethod
    def validate_filename(filename: str) -> str:
        """안전한 파일명 검증"""
        # 위험한 문자 제거
        safe_chars = re.compile(r'[^\w\-_\. ]')
        filename = safe_chars.sub('', filename)
        
        # 길이 제한
        if len(filename) > 255:
            filename = filename[:255]
        
        # 예약어 체크 (Windows)
        reserved_names = {'CON', 'PRN', 'AUX', 'NUL', 'COM1', 'COM2', 'COM3', 'COM4', 
                         'COM5', 'COM6', 'COM7', 'COM8', 'COM9', 'LPT1', 'LPT2', 
                         'LPT3', 'LPT4', 'LPT5', 'LPT6', 'LPT7', 'LPT8', 'LPT9'}
        
        name_without_ext = filename.split('.')[0].upper()
        if name_without_ext in reserved_names:
            filename = f"file_{filename}"
        
        return filename
    
    @staticmethod
    def sanitize_html(text: Optional[str]) -> Optional[str]:
        """HTML 새니타이징"""
        if not text:
            return text
        
        # 허용된 태그와 속성
        allowed_tags = ['p', 'br', 'strong', 'em', 'u', 'h1', 'h2', 'h3', 'ul', 'ol', 'li']
        allowed_attributes = {}
        
        return bleach.clean(text, tags=allowed_tags, attributes=allowed_attributes)
    
    @staticmethod
    def validate_search_query(query: str) -> str:
        """검색 쿼리 검증 (SQL 인젝션 방지)"""
        # 기본 길이 제한
        if len(query) > 1000:
            query = query[:1000]
        
        # 위험한 SQL 키워드 제거
        dangerous_patterns = [
            r'\b(DROP|DELETE|INSERT|UPDATE|CREATE|ALTER|EXEC|EXECUTE)\b',
            r'[;\'"\\]',
            r'--',
            r'/\*.*?\*/'
        ]
        
        for pattern in dangerous_patterns:
            query = re.sub(pattern, '', query, flags=re.IGNORECASE)
        
        return query.strip()
```

### 레이트 리미팅
```python
# app/middleware/rate_limit.py
import time
from collections import defaultdict, deque
from fastapi import Request, HTTPException, status
from fastapi.middleware.base import BaseHTTPMiddleware

class RateLimitMiddleware(BaseHTTPMiddleware):
    def __init__(self, app, max_requests: int = 100, window_seconds: int = 60):
        super().__init__(app)
        self.max_requests = max_requests
        self.window_seconds = window_seconds
        self.clients = defaultdict(deque)
    
    async def dispatch(self, request: Request, call_next):
        client_ip = request.client.host
        current_time = time.time()
        
        # 클라이언트의 요청 이력 가져오기
        client_requests = self.clients[client_ip]
        
        # 윈도우 시간 이전 요청들 제거
        while client_requests and client_requests[0] < current_time - self.window_seconds:
            client_requests.popleft()
        
        # 요청 수 확인
        if len(client_requests) >= self.max_requests:
            raise HTTPException(
                status_code=status.HTTP_429_TOO_MANY_REQUESTS,
                detail=f"요청 한도 초과. {self.window_seconds}초 후 다시 시도해주세요."
            )
        
        # 현재 요청 시간 기록
        client_requests.append(current_time)
        
        response = await call_next(request)
        return response

# 특정 엔드포인트용 레이트 리미터
from functools import wraps
from fastapi import Depends

def rate_limit(max_requests: int = 10, window_seconds: int = 60):
    """특정 엔드포인트에 레이트 리미팅 적용"""
    request_history = defaultdict(deque)
    
    def decorator(func):
        @wraps(func)
        async def wrapper(request: Request, *args, **kwargs):
            client_ip = request.client.host
            current_time = time.time()
            
            client_requests = request_history[client_ip]
            
            # 오래된 요청 기록 제거
            while client_requests and client_requests[0] < current_time - window_seconds:
                client_requests.popleft()
            
            if len(client_requests) >= max_requests:
                raise HTTPException(
                    status_code=429,
                    detail=f"API 요청 한도 초과. {window_seconds}초 후 다시 시도해주세요."
                )
            
            client_requests.append(current_time)
            return await func(request, *args, **kwargs)
        
        return wrapper
    return decorator
```

### 파일 보안
```python
# app/core/file_security.py
import os
import magic
import hashlib
from pathlib import Path
from typing import List, Optional

class FileSecurityChecker:
    # 허용된 MIME 타입
    ALLOWED_MIME_TYPES = {
        'application/pdf': ['.pdf'],
        'application/msword': ['.doc'],
        'application/vnd.openxmlformats-officedocument.wordprocessingml.document': ['.docx'],
        'application/vnd.ms-powerpoint': ['.ppt'],
        'application/vnd.openxmlformats-officedocument.presentationml.presentation': ['.pptx'],
        'text/plain': ['.txt'],
        'video/mp4': ['.mp4'],
        'video/quicktime': ['.mov'],
    }
    
    # 위험한 파일 시그니처
    DANGEROUS_SIGNATURES = [
        b'\x4D\x5A',  # PE 실행파일 (MZ)
        b'\x7F\x45\x4C\x46',  # ELF 실행파일
        b'#!',  # 스크립트 파일
    ]
    
    @classmethod
    def check_file_type(cls, file_path: str) -> bool:
        """파일 타입 검증"""
        try:
            # 실제 MIME 타입 확인
            mime = magic.Magic(mime=True)
            detected_mime = mime.from_file(file_path)
            
            # 허용된 타입인지 확인
            if detected_mime not in cls.ALLOWED_MIME_TYPES:
                return False
            
            # 확장자 일치 확인
            file_ext = Path(file_path).suffix.lower()
            allowed_extensions = cls.ALLOWED_MIME_TYPES[detected_mime]
            
            return file_ext in allowed_extensions
            
        except Exception as e:
            logger.error(f"File type check failed: {e}")
            return False
    
    @classmethod
    def scan_for_malware(cls, file_path: str) -> bool:
        """간단한 악성코드 스캔"""
        try:
            with open(file_path, 'rb') as f:
                # 파일 시작 부분 읽기
                header = f.read(1024)
                
                # 위험한 시그니처 확인
                for signature in cls.DANGEROUS_SIGNATURES:
                    if signature in header:
                        return False
                
                # 추가 검사: 파일 크기
                file_size = os.path.getsize(file_path)
                if file_size > 100 * 1024 * 1024:  # 100MB 초과
                    return False
                
            return True
            
        except Exception as e:
            logger.error(f"Malware scan failed: {e}")
            return False
    
    @classmethod
    def calculate_file_hash(cls, file_path: str) -> str:
        """파일 해시 계산 (중복 체크용)"""
        hash_sha256 = hashlib.sha256()
        with open(file_path, 'rb') as f:
            for chunk in iter(lambda: f.read(4096), b""):
                hash_sha256.update(chunk)
        return hash_sha256.hexdigest()
```

---

## 📊 모니터링 및 헬스체크

### 고급 헬스체크
```python
# app/core/health.py
from typing import Dict, Any
from sqlalchemy.orm import Session
from .database import SessionLocal, engine
import psutil
import time

class HealthChecker:
    @staticmethod
    async def check_database() -> Dict[str, Any]:
        """데이터베이스 상태 확인"""
        try:
            db = SessionLocal()
            start_time = time.time()
            
            # 간단한 쿼리 실행
            result = db.execute("SELECT 1").fetchone()
            
            response_time = (time.time() - start_time) * 1000  # ms
            db.close()
            
            return {
                "status": "healthy" if result else "unhealthy",
                "response_time_ms": round(response_time, 2),
                "details": "Database connection successful"
            }
        except Exception as e:
            return {
                "status": "unhealthy",
                "error": str(e),
                "details": "Database connection failed"
            }
    
    @staticmethod
    def check_disk_space() -> Dict[str, Any]:
        """디스크 공간 확인"""
        try:
            disk_usage = psutil.disk_usage('/')
            free_percentage = (disk_usage.free / disk_usage.total) * 100
            
            status = "healthy"
            if free_percentage < 10:
                status = "critical"
            elif free_percentage < 20:
                status = "warning"
            
            return {
                "status": status,
                "free_space_gb": round(disk_usage.free / (1024**3), 2),
                "total_space_gb": round(disk_usage.total / (1024**3), 2),
                "free_percentage": round(free_percentage, 2)
            }
        except Exception as e:
            return {
                "status": "unhealthy",
                "error": str(e)
            }
    
    @staticmethod
    def check_memory() -> Dict[str, Any]:
        """메모리 사용량 확인"""
        try:
            memory = psutil.virtual_memory()
            
            status = "healthy"
            if memory.percent > 90:
                status = "critical"
            elif memory.percent > 80:
                status = "warning"
            
            return {
                "status": status,
                "used_percentage": memory.percent,
                "available_gb": round(memory.available / (1024**3), 2),
                "total_gb": round(memory.total / (1024**3), 2)
            }
        except Exception as e:
            return {
                "status": "unhealthy",
                "error": str(e)
            }

# 헬스체크 엔드포인트
@router.get("/health")
async def health_check():
    """전체 시스템 헬스체크"""
    results = {
        "timestamp": time.time(),
        "status": "healthy",
        "checks": {
            "database": await HealthChecker.check_database(),
            "disk": HealthChecker.check_disk_space(),
            "memory": HealthChecker.check_memory()
        }
    }
    
    # 전체 상태 결정
    for check_name, check_result in results["checks"].items():
        if check_result["status"] in ["unhealthy", "critical"]:
            results["status"] = "unhealthy"
            break
        elif check_result["status"] == "warning":
            results["status"] = "warning"
    
    status_code = 200 if results["status"] in ["healthy", "warning"] else 503
    return JSONResponse(content=results, status_code=status_code)
```

### 메트릭 수집
```python
# app/core/metrics.py
import time
from collections import defaultdict, deque
from typing import Dict, Any

class MetricsCollector:
    def __init__(self):
        self.request_counts = defaultdict(int)
        self.response_times = defaultdict(deque)
        self.error_counts = defaultdict(int)
        self.start_time = time.time()
    
    def record_request(self, endpoint: str, method: str, response_time: float, status_code: int):
        """요청 메트릭 기록"""
        key = f"{method} {endpoint}"
        
        self.request_counts[key] += 1
        self.response_times[key].append(response_time)
        
        # 최근 1000개만 유지
        if len(self.response_times[key]) > 1000:
            self.response_times[key].popleft()
        
        # 에러 카운트
        if status_code >= 400:
            self.error_counts[key] += 1
    
    def get_metrics(self) -> Dict[str, Any]:
        """메트릭 조회"""
        uptime = time.time() - self.start_time
        
        endpoint_stats = {}
        for endpoint in self.request_counts:
            response_times = list(self.response_times[endpoint])
            if response_times:
                avg_response_time = sum(response_times) / len(response_times)
                max_response_time = max(response_times)
                min_response_time = min(response_times)
            else:
                avg_response_time = max_response_time = min_response_time = 0
            
            endpoint_stats[endpoint] = {
                "request_count": self.request_counts[endpoint],
                "error_count": self.error_counts[endpoint],
                "avg_response_time": round(avg_response_time, 3),
                "max_response_time": round(max_response_time, 3),
                "min_response_time": round(min_response_time, 3),
                "error_rate": round(
                    (self.error_counts[endpoint] / self.request_counts[endpoint]) * 100, 2
                ) if self.request_counts[endpoint] > 0 else 0
            }
        
        return {
            "uptime_seconds": round(uptime, 2),
            "total_requests": sum(self.request_counts.values()),
            "total_errors": sum(self.error_counts.values()),
            "endpoints": endpoint_stats
        }

metrics_collector = MetricsCollector()

# 메트릭 수집 미들웨어
class MetricsMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start_time = time.time()
        
        response = await call_next(request)
        
        process_time = time.time() - start_time
        metrics_collector.record_request(
            endpoint=request.url.path,
            method=request.method,
            response_time=process_time,
            status_code=response.status_code
        )
        
        return response
```

---

## 🧪 테스트 자동화

### 단위 테스트
```python
# tests/test_materials.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from app.main import app
from app.database import get_db, Base
from app import models

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

@pytest.fixture
def db_session():
    """테스트용 데이터베이스 세션"""
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()

@pytest.fixture
def sample_category(db_session):
    """테스트용 카테고리 생성"""
    category = models.Category(
        name="테스트 카테고리",
        description="테스트용 카테고리입니다"
    )
    db_session.add(category)
    db_session.commit()
    db_session.refresh(category)
    return category

@pytest.fixture
def sample_material(db_session, sample_category):
    """테스트용 교육자료 생성"""
    material = models.Material(
        title="테스트 자료",
        description="테스트용 교육자료입니다",
        category_id=sample_category.id,
        file_path="/test/path.pdf",
        file_name="test.pdf",
        original_file_name="테스트.pdf",
        file_size=1024,
        mime_type="application/pdf"
    )
    db_session.add(material)
    db_session.commit()
    db_session.refresh(material)
    return material

class TestMaterials:
    def test_get_materials_empty(self):
        """빈 자료 목록 조회 테스트"""
        response = client.get("/api/materials")
        assert response.status_code == 200
        assert response.json() == []
    
    def test_get_materials_with_data(self, sample_material):
        """자료가 있을 때 목록 조회 테스트"""
        response = client.get("/api/materials")
        assert response.status_code == 200
        
        data = response.json()
        assert len(data) == 1
        assert data[0]["title"] == "테스트 자료"
    
    def test_get_material_by_id(self, sample_material):
        """특정 자료 조회 테스트"""
        response = client.get(f"/api/materials/{sample_material.id}")
        assert response.status_code == 200
        
        data = response.json()
        assert data["title"] == "테스트 자료"
        assert data["id"] == sample_material.id
    
    def test_get_nonexistent_material(self):
        """존재하지 않는 자료 조회 테스트"""
        response = client.get("/api/materials/999")
        assert response.status_code == 404
        assert "찾을 수 없습니다" in response.json()["detail"]
    
    def test_search_materials(self, sample_material):
        """자료 검색 테스트"""
        response = client.get("/api/materials?search=테스트")
        assert response.status_code == 200
        
        data = response.json()
        assert len(data) == 1
        assert data[0]["title"] == "테스트 자료"
    
    def test_search_materials_no_results(self, sample_material):
        """검색 결과 없음 테스트"""
        response = client.get("/api/materials?search=존재하지않는검색어")
        assert response.status_code == 200
        assert response.json() == []

class TestCategories:
    def test_get_categories_empty(self):
        response = client.get("/api/categories")
        assert response.status_code == 200
        assert response.json() == []
    
    def test_create_category(self):
        category_data = {
            "name": "새 카테고리",
            "description": "새로운 카테고리입니다"
        }
        response = client.post("/api/categories", json=category_data)
        assert response.status_code == 200
        
        data = response.json()
        assert data["name"] == "새 카테고리"
        assert data["description"] == "새로운 카테고리입니다"
```

### 통합 테스트
```python
# tests/test_integration.py
import pytest
import tempfile
import os
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

class TestFileUploadIntegration:
    def test_full_file_upload_workflow(self):
        """전체 파일 업로드 워크플로우 테스트"""
        
        # 1. 카테고리 생성
        category_data = {
            "name": "통합테스트 카테고리",
            "description": "통합 테스트용"
        }
        category_response = client.post("/api/categories", json=category_data)
        assert category_response.status_code == 200
        category_id = category_response.json()["id"]
        
        # 2. 임시 파일 생성
        with tempfile.NamedTemporaryFile(mode='w', suffix='.txt', delete=False) as f:
            f.write("테스트 파일 내용입니다.")
            temp_file_path = f.name
        
        try:
            # 3. 파일 업로드
            with open(temp_file_path, 'rb') as f:
                upload_response = client.post(
                    "/api/materials",
                    data={
                        "title": "통합테스트 자료",
                        "description": "통합 테스트용 자료",
                        "category_id": category_id
                    },
                    files={"file": ("test.txt", f, "text/plain")}
                )
            
            assert upload_response.status_code == 200
            material_data = upload_response.json()
            material_id = material_data["id"]
            
            # 4. 업로드된 자료 조회
            get_response = client.get(f"/api/materials/{material_id}")
            assert get_response.status_code == 200
            assert get_response.json()["title"] == "통합테스트 자료"
            
            # 5. 다운로드 테스