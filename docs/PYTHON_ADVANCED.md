# 🐍 파이썬 심화 가이드 (빅데이터학과 비전공자용)

> 성균관대학원 빅데이터학과 진학자를 위한 실전 파이썬 마스터

## 📋 목차
1. [가상환경과 패키지 관리](#가상환경과-패키지-관리)
2. [객체지향 프로그래밍 심화](#객체지향-프로그래밍-심화)
3. [비동기 프로그래밍](#비동기-프로그래밍)
4. [데코레이터와 제너레이터](#데코레이터와-제너레이터)
5. [데이터 분석 핵심 라이브러리](#데이터-분석-핵심-라이브러리)
6. [FastAPI와 데이터베이스](#fastapi와-데이터베이스)
7. [배치 처리와 자동화](#배치-처리와-자동화)
8. [테스팅과 로깅](#테스팅과-로깅)
9. [성능 최적화](#성능-최적화)
10. [실전 활용](#실전-활용)

---

## 📦 가상환경과 패키지 관리

### 가상환경이 왜 필요한가?

#### 문제 상황
```bash
# 프로젝트 A에서는 pandas 1.5.0이 필요
# 프로젝트 B에서는 pandas 2.0.0이 필요
# → 충돌 발생! 😱
```

#### 해결책: 가상환경
```bash
프로젝트 A 환경: pandas==1.5.0
프로젝트 B 환경: pandas==2.0.0
→ 서로 독립적! ✅
```

### 가상환경 생성과 관리

#### 1. venv 사용 (파이썬 내장)
```bash
# 가상환경 생성
python -m venv 빅데이터_프로젝트

# 가상환경 활성화 (Mac/Linux)
source 빅데이터_프로젝트/bin/activate

# 가상환경 활성화 (Windows)
빅데이터_프로젝트\Scripts\activate

# 비활성화
deactivate
```

#### 2. conda 사용 (아나콘다)
```bash
# 가상환경 생성 (Python 3.9)
conda create -n 빅데이터_프로젝트 python=3.9

# 활성화
conda activate 빅데이터_프로젝트

# 비활성화  
conda deactivate

# 환경 목록 확인
conda env list
```

### 패키지 관리 모범사례

#### requirements.txt 작성
```bash
# 현재 환경의 패키지 목록 저장
pip freeze > requirements.txt

# 다른 환경에서 같은 패키지들 설치
pip install -r requirements.txt
```

#### requirements.txt 예시
```text
# 데이터 분석
pandas==2.1.0
numpy==1.24.0
matplotlib==3.7.0
seaborn==0.12.0

# 머신러닝
scikit-learn==1.3.0
xgboost==1.7.0

# 웹 개발
fastapi==0.103.0
uvicorn==0.23.0

# 데이터베이스
sqlalchemy==2.0.0
psycopg2-binary==2.9.0
```

#### poetry 사용 (고급)
```bash
# poetry 설치
pip install poetry

# 프로젝트 초기화
poetry init

# 패키지 추가
poetry add pandas numpy

# 개발용 패키지 추가
poetry add --group dev pytest black

# 가상환경 활성화
poetry shell

# 의존성 설치
poetry install
```

---

## 🏗️ 객체지향 프로그래밍 심화

### 추상 클래스와 인터페이스
```python
from abc import ABC, abstractmethod

class 데이터분석기(ABC):
    """추상 클래스 - 구현 강제"""
    
    @abstractmethod
    def 데이터로드(self, 파일경로: str):
        pass
    
    @abstractmethod
    def 전처리(self):
        pass
    
    @abstractmethod  
    def 분석(self):
        pass

class CSV분석기(데이터분석기):
    def __init__(self):
        self.데이터 = None
    
    def 데이터로드(self, 파일경로: str):
        import pandas as pd
        self.데이터 = pd.read_csv(파일경로)
        print(f"{파일경로} 로드 완료")
    
    def 전처리(self):
        # 결측치 제거
        self.데이터 = self.데이터.dropna()
        print("전처리 완료")
    
    def 분석(self):
        return self.데이터.describe()

# 사용
분석기 = CSV분석기()
분석기.데이터로드("data.csv")
분석기.전처리()
결과 = 분석기.분석()
```

### 프로퍼티와 데코레이터
```python
class 학생:
    def __init__(self, 이름: str):
        self._이름 = 이름
        self._점수들 = []
    
    @property
    def 이름(self):
        """읽기 전용 프로퍼티"""
        return self._이름
    
    @property
    def 평균(self):
        """계산된 프로퍼티"""
        if not self._점수들:
            return 0
        return sum(self._점수들) / len(self._점수들)
    
    @property
    def 학점(self):
        평균 = self.평균
        if 평균 >= 90: return 'A'
        elif 평균 >= 80: return 'B'
        elif 평균 >= 70: return 'C'
        else: return 'F'
    
    def 점수추가(self, 점수: float):
        if 0 <= 점수 <= 100:
            self._점수들.append(점수)
        else:
            raise ValueError("점수는 0-100 사이여야 합니다")

# 사용
학생 = 학생("홍길동")
학생.점수추가(85)
학생.점수추가(90)
print(f"{학생.이름} 평균: {학생.평균}, 학점: {학생.학점}")
```

---

## 🔄 비동기 프로그래밍

### 동기 vs 비동기 이해하기

#### 동기 프로그래밍 (순차적)
```python
import time
import requests

def 동기방식_데이터수집():
    urls = [
        'https://httpbin.org/delay/1',
        'https://httpbin.org/delay/1', 
        'https://httpbin.org/delay/1'
    ]
    
    start = time.time()
    결과들 = []
    
    for url in urls:
        response = requests.get(url)
        결과들.append(response.status_code)
    
    print(f"동기 방식: {time.time() - start:.2f}초 소요")  # 약 3초
    return 결과들
```

#### 비동기 프로그래밍 (병렬처리)
```python
import asyncio
import aiohttp
import time

async def fetch_data(session, url):
    async with session.get(url) as response:
        return response.status

async def 비동기방식_데이터수집():
    urls = [
        'https://httpbin.org/delay/1',
        'https://httpbin.org/delay/1',
        'https://httpbin.org/delay/1'
    ]
    
    start = time.time()
    
    async with aiohttp.ClientSession() as session:
        # 모든 요청을 동시에 실행
        tasks = [fetch_data(session, url) for url in urls]
        결과들 = await asyncio.gather(*tasks)
    
    print(f"비동기 방식: {time.time() - start:.2f}초 소요")  # 약 1초
    return 결과들

# 실행
# asyncio.run(비동기방식_데이터수집())
```

### 비동기 파일 처리
```python
import aiofiles
import asyncio

async def 대용량파일처리(파일경로):
    """비동기로 대용량 파일 처리"""
    
    async with aiofiles.open(파일경로, 'r', encoding='utf-8') as 파일:
        줄번호 = 0
        async for 줄 in 파일:
            줄번호 += 1
            
            # 비동기적으로 각 줄 처리
            await 줄처리(줄, 줄번호)
            
            # 1000줄마다 진행상황 출력
            if 줄번호 % 1000 == 0:
                print(f"{줄번호}줄 처리 완료")

async def 줄처리(줄, 번호):
    """각 줄을 비동기적으로 처리"""
    # 시뮬레이션: 약간의 지연
    await asyncio.sleep(0.001)  
    
    # 실제 처리 로직 (예: 데이터 정제, API 호출 등)
    처리된_데이터 = 줄.strip().upper()
    return 처리된_데이터
```

---

## 🎭 데코레이터와 제너레이터

### 데코레이터 기초
```python
import time
from functools import wraps

def 실행시간측정(func):
    """함수 실행 시간을 측정하는 데코레이터"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} 실행시간: {end-start:.4f}초")
        return result
    return wrapper

def 캐시(func):
    """함수 결과를 캐시하는 데코레이터"""
    cache = {}
    
    @wraps(func)
    def wrapper(*args, **kwargs):
        key = str(args) + str(kwargs)
        if key in cache:
            print(f"캐시에서 반환: {func.__name__}")
            return cache[key]
        
        result = func(*args, **kwargs)
        cache[key] = result
        return result
    return wrapper

# 사용 예시
@실행시간측정
@캐시
def 피보나치(n):
    if n <= 1:
        return n
    return 피보나치(n-1) + 피보나치(n-2)

# 실행
print(피보나치(10))  # 첫 실행: 계산
print(피보나치(10))  # 두 번째: 캐시에서 반환
```

### 클래스 데코레이터
```python
def API엔드포인트(경로: str):
    """API 엔드포인트를 등록하는 클래스 데코레이터"""
    def decorator(cls):
        cls._api_path = 경로
        cls._endpoints = {}
        
        # 클래스의 모든 메서드를 검사
        for 이름, 메서드 in cls.__dict__.items():
            if hasattr(메서드, '_http_method'):
                cls._endpoints[이름] = {
                    'method': 메서드._http_method,
                    'path': 메서드._path
                }
        
        return cls
    return decorator

def GET(경로: str):
    """GET 메서드 데코레이터"""
    def decorator(func):
        func._http_method = 'GET'
        func._path = 경로
        return func
    return decorator

@API엔드포인트('/api/users')
class 사용자API:
    @GET('/list')
    def 사용자목록조회(self):
        return {"users": ["홍길동", "김철수"]}
    
    @GET('/{user_id}')
    def 사용자상세조회(self, user_id: int):
        return {"user_id": user_id, "name": "홍길동"}

# 사용
api = 사용자API()
print(f"API 경로: {api._api_path}")
print(f"엔드포인트들: {api._endpoints}")
```

### 제너레이터 (Generator)
```python
def 대용량데이터생성기(크기: int):
    """메모리 효율적인 데이터 생성"""
    for i in range(크기):
        # 복잡한 계산 시뮬레이션
        데이터 = {
            'id': i,
            'value': i ** 2,
            'status': 'active' if i % 2 == 0 else 'inactive'
        }
        yield 데이터  # 한 번에 하나씩 반환

def 파일라인생성기(파일경로: str):
    """메모리 효율적인 파일 읽기"""
    with open(파일경로, 'r', encoding='utf-8') as file:
        for 줄번호, 줄 in enumerate(file, 1):
            yield {
                '줄번호': 줄번호,
                '내용': 줄.strip(),
                '길이': len(줄.strip())
            }

# 사용 예시
print("=== 대용량 데이터 처리 ===")
데이터생성기 = 대용량데이터생성기(1000000)  # 100만 개
for i, 데이터 in enumerate(데이터생성기):
    if i < 5:  # 처음 5개만 출력
        print(데이터)
    elif i >= 10:  # 11개만 처리하고 중단
        break

# 제너레이터 표현식
제곱수들 = (x**2 for x in range(1000) if x % 2 == 0)
print(f"첫 5개 짝수의 제곱: {list(next(제곱수들) for _ in range(5))}")
```

---

## ⚡ FastAPI와 데이터베이스

### FastAPI 기본 설정
```python
from fastapi import FastAPI, HTTPException, Depends
from sqlalchemy import create_engine, Column, Integer, String, Float
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, Session
from pydantic import BaseModel
from typing import List, Optional

# 데이터베이스 설정
SQLALCHEMY_DATABASE_URL = "sqlite:///./빅데이터.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()

# 데이터베이스 모델
class 학생DB(Base):
    __tablename__ = "학생들"
    
    id = Column(Integer, primary_key=True, index=True)
    이름 = Column(String, index=True)
    학번 = Column(String, unique=True, index=True)
    전공 = Column(String)
    학점 = Column(Float)

# Pydantic 모델 (API 스키마)
class 학생생성(BaseModel):
    이름: str
    학번: str
    전공: str
    학점: float

class 학생응답(BaseModel):
    id: int
    이름: str
    학번: str
    전공: str
    학점: float
    
    class Config:
        orm_mode = True

# FastAPI 앱
app = FastAPI(title="빅데이터학과 학생 관리 시스템", version="1.0.0")

# 데이터베이스 종속성
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

# API 엔드포인트들
@app.post("/학생/", response_model=학생응답)
def 학생생성(학생: 학생생성, db: Session = Depends(get_db)):
    db_학생 = 학생DB(**학생.dict())
    db.add(db_학생)
    db.commit()
    db.refresh(db_학생)
    return db_학생

@app.get("/학생/", response_model=List[학생응답])
def 학생목록조회(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    학생들 = db.query(학생DB).offset(skip).limit(limit).all()
    return 학생들

@app.get("/학생/{학생_id}", response_model=학생응답)
def 학생상세조회(학생_id: int, db: Session = Depends(get_db)):
    학생 = db.query(학생DB).filter(학생DB.id == 학생_id).first()
    if 학생 is None:
        raise HTTPException(status_code=404, detail="학생을 찾을 수 없습니다")
    return 학생

@app.get("/통계/전공별")
def 전공별통계(db: Session = Depends(get_db)):
    from sqlalchemy import func
    결과 = db.query(
        학생DB.전공,
        func.count(학생DB.id).label('학생수'),
        func.avg(학생DB.학점).label('평균학점')
    ).group_by(학생DB.전공).all()
    
    return [
        {"전공": row.전공, "학생수": row.학생수, "평균학점": round(row.평균학점, 2)}
        for row in 결과
    ]

# 실행: uvicorn main:app --reload
```

---

## 🔄 배치 처리와 자동화

### Cron Job과 스케줄링
```python
import schedule
import time
import logging
from datetime import datetime
import pandas as pd
import smtplib
from email.mime.text import MIMEText
from pathlib import Path

# 로깅 설정
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('배치작업.log'),
        logging.StreamHandler()
    ]
)

class 데이터배치처리:
    def __init__(self, 데이터경로: str):
        self.데이터경로 = Path(데이터경로)
        self.결과경로 = Path("결과")
        self.결과경로.mkdir(exist_ok=True)
    
    def 일일데이터처리(self):
        """매일 실행되는 데이터 처리 작업"""
        try:
            logging.info("일일 데이터 처리 작업 시작")
            
            # 1. 데이터 로드
            if not self.데이터경로.exists():
                logging.warning(f"데이터 파일이 없습니다: {self.데이터경로}")
                return
            
            df = pd.read_csv(self.데이터경로)
            logging.info(f"데이터 로드 완료: {len(df)}행")
            
            # 2. 데이터 정제
            원본크기 = len(df)
            df = df.dropna()  # 결측치 제거
            df = df.drop_duplicates()  # 중복 제거
            logging.info(f"데이터 정제 완료: {원본크기} → {len(df)}행")
            
            # 3. 통계 계산
            통계 = {
                '처리날짜': datetime.now().strftime('%Y-%m-%d %H:%M:%S'),
                '총_데이터수': len(df),
                '수치컬럼_평균': df.select_dtypes(include=['number']).mean().to_dict(),
                '범주컬럼_분포': {col: df[col].value_counts().head().to_dict() 
                              for col in df.select_dtypes(include=['object']).columns}
            }
            
            # 4. 결과 저장
            오늘 = datetime.now().strftime('%Y%m%d')
            결과파일 = self.결과경로 / f"처리결과_{오늘}.csv"
            통계파일 = self.결과경로 / f"통계_{오늘}.json"
            
            df.to_csv(결과파일, index=False, encoding='utf-8')
            
            import json
            with open(통계파일, 'w', encoding='utf-8') as f:
                json.dump(통계, f, ensure_ascii=False, indent=2)
            
            logging.info(f"결과 저장 완료: {결과파일}, {통계파일}")
            
            # 5. 이메일 알림 (선택사항)
            self.이메일알림(f"일일 데이터 처리 완료 ({len(df)}건)")
            
        except Exception as e:
            logging.error(f"배치 작업 중 오류: {e}")
            self.이메일알림(f"배치 작업 실패: {e}")
    
    def 주간리포트생성(self):
        """매주 실행되는 리포트 생성"""
        try:
            logging.info("주간 리포트 생성 시작")
            
            # 최근 7일간의 처리 결과들 수집
            리포트데이터 = []
            for i in range(7):
                날짜 = (datetime.now() - pd.Timedelta(days=i)).strftime('%Y%m%d')
                파일 = self.결과경로 / f"처리결과_{날짜}.csv"
                if 파일.exists():
                    df = pd.read_csv(파일)
                    리포트데이터.append({
                        '날짜': 날짜,
                        '데이터수': len(df)
                    })
            
            if 리포트데이터:
                리포트df = pd.DataFrame(리포트데이터)
                리포트파일 = self.결과경로 / f"주간리포트_{datetime.now().strftime('%Y%m%d')}.csv"
                리포트df.to_csv(리포트파일, index=False)
                logging.info(f"주간 리포트 생성 완료: {리포트파일}")
                
        except Exception as e:
            logging.error(f"주간 리포트 생성 중 오류: {e}")
    
    def 이메일알림(self, 메시지: str):
        """이메일 알림 발송 (설정 필요)"""
        try:
            # 실제 환경에서는 환경변수로 설정
            SMTP_SERVER = "smtp.gmail.com"
            SMTP_PORT = 587
            EMAIL = "your-email@gmail.com"
            PASSWORD = "your-app-password"
            
            msg = MIMEText(메시지, 'plain', 'utf-8')
            msg['Subject'] = '데이터 배치 처리 알림'
            msg['From'] = EMAIL
            msg['To'] = EMAIL
            
            # 실제 발송은 주석 처리 (테스트용)
            # with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
            #     server.starttls()
            #     server.login(EMAIL, PASSWORD)
            #     server.send_message(msg)
            
            logging.info(f"이메일 알림 (시뮬레이션): {메시지}")
            
        except Exception as e:
            logging.error(f"이메일 발송 실패: {e}")

# 배치 작업 스케줄링
def 배치작업설정():
    배치처리기 = 데이터배치처리("데이터/원본데이터.csv")
    
    # 매일 오전 6시에 데이터 처리
    schedule.every().day.at("06:00").do(배치처리기.일일데이터처리)
    
    # 매주 월요일 오전 9시에 주간 리포트
    schedule.every().monday.at("09:00").do(배치처리기.주간리포트생성)
    
    # 매시간마다 간단한 상태 체크
    schedule.every().hour.do(lambda: logging.info("시스템 정상 동작 중"))
    
    logging.info("배치 작업 스케줄 설정 완료")
    
    # 스케줄러 실행
    while True:
        schedule.run_pending()
        time.sleep(60)  # 1분마다 체크

# 실행
if __name__ == "__main__":
    배치작업설정()
```

### Celery를 이용한 비동기 작업 처리
```python
from celery import Celery
import pandas as pd
import time
from typing import List

# Celery 앱 설정
app = Celery(
    '데이터처리',
    broker='redis://localhost:6379/0',  # Redis 브로커
    backend='redis://localhost:6379/0'  # 결과 저장소
)

@app.task
def 대용량데이터처리(파일경로: str, 청크크기: int = 1000):
    """대용량 데이터를 청크 단위로 처리하는 비동기 작업"""
    try:
        총_처리수 = 0
        처리결과 = []
        
        # 파일을 청크 단위로 읽기
        for chunk_df in pd.read_csv(파일경로, chunksize=청크크기):
            # 각 청크 처리
            처리된_청크 = chunk_df.dropna().drop_duplicates()
            
            # 통계 계산
            청크통계 = {
                '청크크기': len(처리된_청크),
                '평균값': 처리된_청크.select_dtypes(include=['number']).mean().to_dict()
            }
            
            처리결과.append(청크통계)
            총_처리수 += len(처리된_청크)
            
            # 진행상황 업데이트
            현재진행률 = (총_처리수 / 10000) * 100  # 가정: 총 10000건
            대용량데이터처리.update_state(
                state='PROGRESS',
                meta={'진행률': min(현재진행률, 100), '처리건수': 총_처리수}
            )
            
            time.sleep(0.1)  # CPU 부하 방지
        
        return {
            'status': '완료',
            '총_처리건수': 총_처리수,
            '청크_통계': 처리결과
        }
        
    except Exception as e:
        return {'status': '실패', 'error': str(e)}

@app.task
def 머신러닝_모델학습(데이터경로: str, 모델타입: str = 'random_forest'):
    """머신러닝 모델 학습을 비동기로 처리"""
    from sklearn.ensemble import RandomForestClassifier
    from sklearn.model_selection import train_test_split
    from sklearn.metrics import accuracy_score
    import joblib
    
    try:
        # 데이터 로드
        df = pd.read_csv(데이터경로)
        X = df.drop('target', axis=1)  # 타겟 컬럼 제외
        y = df['target']
        
        # 데이터 분할
        X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
        
        # 모델 학습
        if 모델타입 == 'random_forest':
            model = RandomForestClassifier(n_estimators=100)
        else:
            raise ValueError(f"지원하지 않는 모델 타입: {모델타입}")
        
        model.fit(X_train, y_train)
        
        # 성능 평가
        y_pred = model.predict(X_test)
        accuracy = accuracy_score(y_test, y_pred)
        
        # 모델 저장
        모델파일 = f"model_{모델타입}_{int(time.time())}.joblib"
        joblib.dump(model, 모델파일)
        
        return {
            'status': '완료',
            '정확도': accuracy,
            '모델파일': 모델파일,
            '학습데이터크기': len(X_train),
            '테스트데이터크기': len(X_test)
        }
        
    except Exception as e:
        return {'status': '실패', 'error': str(e)}

# 작업 실행 예시
def 작업실행예시():
    # 비동기 작업 시작
    task1 = 대용량데이터처리.delay("big_data.csv", 청크크기=500)
    task2 = 머신러닝_모델학습.delay("training_data.csv", "random_forest")
    
    print(f"데이터 처리 작업 ID: {task1.id}")
    print(f"모델 학습 작업 ID: {task2.id}")
    
    # 작업 상태 확인
    while not task1.ready():
        상태 = task1.state
        if 상태 == 'PROGRESS':
            정보 = task1.info
            print(f"진행률: {정보.get('진행률', 0):.1f}%, 처리건수: {정보.get('처리건수', 0)}")
        time.sleep(2)
    
    # 결과 확인
    결과1 = task1.get()
    결과2 = task2.get()
    
    print("=== 데이터 처리 결과 ===")
    print(결과1)
    print("=== 모델 학습 결과 ===")
    print(결과2)

# 실행 (Celery worker 먼저 시작: celery -A tasks worker --loglevel=info)
```

---

## 🧪 테스팅과 로깅

### pytest를 이용한 테스팅
```python
import pytest
import pandas as pd
from unittest.mock import patch, mock_open
import tempfile
import os

# 테스트할 클래스
class 데이터분석기:
    def __init__(self, 데이터경로: str):
        self.데이터경로 = 데이터경로
        self.데이터 = None
    
    def 데이터로드(self):
        if not os.path.exists(self.데이터경로):
            raise FileNotFoundError(f"파일이 없습니다: {self.데이터경로}")
        self.데이터 = pd.read_csv(self.데이터경로)
        return len(self.데이터)
    
    def 평균계산(self, 컬럼명: str):
        if self.데이터 is None:
            raise ValueError("데이터를 먼저 로드하세요")
        if 컬럼명 not in self.데이터.columns:
            raise KeyError(f"컬럼이 없습니다: {컬럼명}")
        return self.데이터[컬럼명].mean()

# 테스트 코드
class Test데이터분석기:
    
    @pytest.fixture
    def 샘플데이터(self):
        """테스트용 샘플 데이터 생성"""
        with tempfile.NamedTemporaryFile(mode='w', suffix='.csv', delete=False) as f:
            f.write("이름,나이,점수\n홍길동,25,85\n김철수,30,90\n이영희,28,88\n")
            return f.name
    
    def test_데이터로드_성공(self, 샘플데이터):
        """정상적인 데이터 로드 테스트"""
        분석기 = 데이터분석기(샘플데이터)
        결과 = 분석기.데이터로드()
        
        assert 결과 == 3
        assert 분석기.데이터 is not None
        assert len(분석기.데이터.columns) == 3
        
        # 임시 파일 정리
        os.unlink(샘플데이터)
    
    def test_데이터로드_파일없음(self):
        """파일이 없을 때 예외 처리 테스트"""
        분석기 = 데이터분석기("없는파일.csv")
        
        with pytest.raises(FileNotFoundError):
            분석기.데이터로드()
    
    def test_평균계산_정상(self, 샘플데이터):
        """평균 계산 정상 동작 테스트"""
        분석기 = 데이터분석기(샘플데이터)
        분석기.데이터로드()
        
        평균 = 분석기.평균계산('점수')
        expected = (85 + 90 + 88) / 3
        
        assert 평균 == expected
        os.unlink(샘플데이터)
    
    def test_평균계산_데이터없음(self):
        """데이터 로드 전 평균 계산 시도 테스트"""
        분석기 = 데이터분석기("dummy.csv")
        
        with pytest.raises(ValueError, match="데이터를 먼저 로드하세요"):
            분석기.평균계산('점수')
    
    @pytest.mark.parametrize("잘못된_컬럼", ["존재하지않는컬럼", "age", "score"])
    def test_평균계산_잘못된컬럼(self, 샘플데이터, 잘못된_컬럼):
        """잘못된 컬럼명으로 평균 계산 시도 테스트"""
        분석기 = 데이터분석기(샘플데이터)
        분석기.데이터로드()
        
        with pytest.raises(KeyError):
            분석기.평균계산(잘못된_컬럼)
        
        os.unlink(샘플데이터)

# 실행: pytest test_data_analyzer.py -v
```

### 로깅 시스템
```python
import logging
from logging.handlers import RotatingFileHandler
from datetime import datetime
import functools
import traceback

# 고급 로깅 설정
def 로깅설정(로그파일: str = "빅데이터_분석.log"):
    """구조적 로깅 시스템 설정"""
    
    # 포맷터 설정
    포맷터 = logging.Formatter(
        '%(asctime)s | %(levelname)-8s | %(name)s | %(funcName)s:%(lineno)d | %(message)s'
    )
    
    # 로거 생성
    logger = logging.getLogger('빅데이터분석')
    logger.setLevel(logging.DEBUG)
    
    # 콘솔 핸들러
    콘솔_핸들러 = logging.StreamHandler()
    콘솔_핸들러.setLevel(logging.INFO)
    콘솔_핸들러.setFormatter(포맷터)
    
    # 파일 핸들러 (로테이션 적용)
    파일_핸들러 = RotatingFileHandler(
        로그파일, 
        maxBytes=10*1024*1024,  # 10MB
        backupCount=5
    )
    파일_핸들러.setLevel(logging.DEBUG)
    파일_핸들러.setFormatter(포맷터)
    
    # 핸들러 추가
    logger.addHandler(콘솔_핸들러)
    logger.addHandler(파일_핸들러)
    
    return logger

# 로깅 데코레이터
def 로그기록(logger=None):
    """함수 실행을 자동으로 로깅하는 데코레이터"""
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            log = logger or logging.getLogger('빅데이터분석')
            
            # 함수 시작 로그
            log.info(f"함수 시작: {func.__name__}")
            log.debug(f"매개변수 - args: {args}, kwargs: {kwargs}")
            
            try:
                start_time = datetime.now()
                결과 = func(*args, **kwargs)
                실행시간 = (datetime.now() - start_time).total_seconds()
                
                log.info(f"함수 완료: {func.__name__} ({실행시간:.3f}초)")
                return 결과
                
            except Exception as e:
                log.error(f"함수 오류: {func.__name__} - {str(e)}")
                log.debug(f"에러 상세:\n{traceback.format_exc()}")
                raise
        
        return wrapper
    return decorator

# 사용 예시
logger = 로깅설정()

@로그기록(logger)
def 복잡한데이터처리(데이터경로: str, 설정: dict):
    """복잡한 데이터 처리 함수 (로깅 적용)"""
    logger.info("데이터 처리 시작")
    
    try:
        # 1단계: 데이터 로드
        logger.debug(f"데이터 로드 시작: {데이터경로}")
        df = pd.read_csv(데이터경로)
        logger.info(f"데이터 로드 완료: {len(df)}행")
        
        # 2단계: 전처리
        logger.debug("데이터 전처리 시작")
        if 설정.get('결측치제거', True):
            원본크기 = len(df)
            df = df.dropna()
            logger.info(f"결측치 제거: {원본크기} → {len(df)}행")
        
        # 3단계: 분석
        logger.debug("데이터 분석 시작")
        결과 = {
            '총_행수': len(df),
            '컬럼수': len(df.columns),
            '수치컬럼_통계': df.select_dtypes(include=['number']).describe().to_dict()
        }
        
        logger.info("데이터 처리 완료")
        return 결과
        
    except FileNotFoundError:
        logger.error(f"파일을 찾을 수 없습니다: {데이터경로}")
        raise
    except pd.errors.EmptyDataError:
        logger.error("빈 데이터 파일입니다")
        raise
    except Exception as e:
        logger.error(f"예상치 못한 오류: {str(e)}")
        raise

# 실행 예시
설정 = {'결측치제거': True}
결과 = 복잡한데이터처리("데이터.csv", 설정)
```

---

## ⚡ 성능 최적화

### 메모리 효율적인 코드
```python
import sys
from memory_profiler import profile  # pip install memory-profiler

# 메모리 비효율적인 코드
def 비효율적_데이터처리():
    """메모리를 많이 사용하는 방식"""
    큰리스트 = list(range(1000000))  # 1백만 개 리스트
    제곱리스트 = [x**2 for x in 큰리스트]  # 또 다른 1백만 개 리스트
    짝수리스트 = [x for x in 제곱리스트 if x % 2 == 0]
    return sum(짝수리스트)

# 메모리 효율적인 코드  
def 효율적_데이터처리():
    """제너레이터를 사용한 메모리 효율적 방식"""
    return sum(x**2 for x in range(1000000) if (x**2) % 2 == 0)

# 성능 비교
import time

def 성능비교():
    # 비효율적 방식
    start = time.time()
    결과1 = 비효율적_데이터처리()
    시간1 = time.time() - start
    
    # 효율적 방식
    start = time.time()  
    결과2 = 효율적_데이터처리()
    시간2 = time.time() - start
    
    print(f"비효율적 방식: {시간1:.3f}초, 결과: {결과1}")
    print(f"효율적 방식: {시간2:.3f}초, 결과: {결과2}")
    print(f"성능 개선: {(시간1/시간2):.1f}배 빠름")

성능비교()
```

### NumPy 벡터화 최적화
```python
import numpy as np
import time

# 느린 방식: 반복문 사용
def 파이썬반복문(배열):
    결과 = []
    for x in 배열:
        결과.append(x**2 + 2*x + 1)
    return 결과

# 빠른 방식: NumPy 벡터화
def 넘파이벡터화(배열):
    return 배열**2 + 2*배열 + 1

# 성능 비교
데이터 = list(range(100000))
넘파이배열 = np.array(데이터)

# Python 반복문
start = time.time()
결과1 = 파이썬반복문(데이터)
시간1 = time.time() - start

# NumPy 벡터화
start = time.time()
결과2 = 넘파이벡터화(넘파이배열)
시간2 = time.time() - start

print(f"Python 반복문: {시간1:.4f}초")
print(f"NumPy 벡터화: {시간2:.4f}초")
print(f"속도 개선: {시간1/시간2:.0f}배 빠름")
```

---

## 🎯 실전 활용 팁

### 환경변수와 설정 관리
```python
import os
from dotenv import load_dotenv  # pip install python-dotenv
from dataclasses import dataclass
from typing import Optional

# .env 파일 로드
load_dotenv()

@dataclass
class 프로젝트설정:
    데이터베이스_URL: str
    API_키: str
    로그레벨: str = "INFO"
    배치크기: int = 1000
    디버그모드: bool = False
    
    @classmethod
    def from_env(cls):
        return cls(
            데이터베이스_URL=os.getenv("DATABASE_URL", "sqlite:///default.db"),
            API_키=os.getenv("API_KEY", ""),
            로그레벨=os.getenv("LOG_LEVEL", "INFO"),
            배치크기=int(os.getenv("BATCH_SIZE", "1000")),
            디버그모드=os.getenv("DEBUG", "false").lower() == "true"
        )

# 사용
설정 = 프로젝트설정.from_env()
print(f"데이터베이스: {설정.데이터베이스_URL}")
```

### 타입 힌팅 (Type Hints)
```python
from typing import List, Dict, Optional, Union, Callable
from dataclasses import dataclass
import pandas as pd

@dataclass
class 학생정보:
    이름: str
    나이: int
    성적: List[float]
    메타데이터: Optional[Dict[str, str]] = None

def 데이터분석함수(
    데이터: pd.DataFrame,
    그룹컬럼: str,
    집계함수: Callable[[pd.Series], float] = lambda x: x.mean(),
    필터조건: Optional[Dict[str, Union[str, int, float]]] = None
) -> Dict[str, float]:
    """타입 힌팅이 적용된 데이터 분석 함수"""
    
    # 필터 적용
    if 필터조건:
        for 컬럼, 값 in 필터조건.items():
            데이터 = 데이터[데이터[컬럼] == 값]
    
    # 그룹별 집계
    결과 = 데이터.groupby(그룹컬럼).apply(집계함수)
    return 결과.to_dict()

# 사용 (IDE에서 자동완성과 타입 체크 지원)
학생들: List[학생정보] = [
    학생정보("홍길동", 25, [85.0, 90.0, 88.0]),
    학생정보("김철수", 24, [92.0, 87.0, 90.0])
]
```

---

**🎉 Python 심화 과정 완료!**

이제 여러분은:
- 🏗️ 고급 객체지향 프로그래밍 구현
- 🔄 비동기 프로그래밍으로 성능 최적화  
- 🎭 데코레이터와 제너레이터 활용
- ⚡ FastAPI로 RESTful API 개발
- 🔄 배치 처리 시스템 구축
- 🧪 체계적인 테스팅과 로깅
- ⚡ 성능 최적화 기법 적용

**빅데이터학과에서 활용할 수 있는 실무 Python 스킬을 모두 갖추었습니다!** 🚀

---

## 📚 추가 학습 자료

### 온라인 학습 자료
- **Python 공식 문서**: https://docs.python.org/3/
- **Real Python**: https://realpython.com/
- **Python Enhancement Proposals (PEPs)**: https://peps.python.org/

### 빅데이터/AI 관련 라이브러리
- **NumPy**: 수치 계산 라이브러리
- **Pandas**: 데이터 분석 라이브러리  
- **Scikit-learn**: 머신러닝 라이브러리
- **TensorFlow/PyTorch**: 딥러닝 프레임워크
- **Apache Spark (PySpark)**: 빅데이터 처리

### 실습 환경
- **Jupyter Notebook**: 대화형 개발 환경
- **Google Colab**: 클라우드 기반 무료 환경
- **Anaconda**: 데이터 사이언스 통합 환경

**🎯 다음 단계: 실제 프로젝트에 적용하며 경험을 쌓아보세요!**

