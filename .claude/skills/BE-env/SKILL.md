---
name: BE-env
description: 환경 변수 관리 및 설정. .env 파일, pydantic-settings 활용
context: fork
agent: be-agent
---

# BE 환경 변수 관리 가이드

## 개요
환경 변수를 안전하게 관리하고 설정합니다. 개발/프로덕션 환경을 분리하여 관리합니다.

## 필요성
- API 키, 비밀번호 등 민감 정보를 코드에서 분리
- 환경별 (개발/스테이징/프로덕션) 설정 분리
- 타입 안전성 보장 (pydantic)
- 기본값 설정 및 검증

## 설치
```bash
pip install pydantic-settings python-dotenv
```

## 파일 구조
```
backend/
├── .env                  # 환경 변수 (git에 커밋 금지)
├── .env.example          # 환경 변수 템플릿 (git 커밋)
├── app/
│   ├── config.py         # 설정 클래스
│   └── main.py
└── .gitignore            # .env 포함
```

## 참조 문서
- [config-setup.md](references/config-setup.md) - 설정 클래스 작성
- [env-example.md](references/env-example.md) - .env 파일 예시
- [validation.md](references/validation.md) - 환경 변수 검증

## 구현

### 1. .env 파일 생성
```bash
# backend/.env

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/dbname

# Security
SECRET_KEY=your-secret-key-min-32-characters
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30

# CORS
ALLOWED_ORIGINS=http://localhost:3000,https://yourdomain.com

# File Upload (선택)
AWS_ACCESS_KEY_ID=your-aws-access-key
AWS_SECRET_ACCESS_KEY=your-aws-secret-key
AWS_REGION=ap-northeast-2
S3_BUCKET_NAME=your-bucket-name

# Cloudinary (선택)
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret

# Email (선택)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASSWORD=your-app-password

# Environment
ENVIRONMENT=development
DEBUG=True
```

### 2. .env.example 파일 (템플릿)
```bash
# backend/.env.example
# 이 파일을 복사하여 .env 파일을 생성하세요

DATABASE_URL=postgresql://user:password@localhost:5432/dbname
SECRET_KEY=change-this-to-random-secret-key
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
ALLOWED_ORIGINS=http://localhost:3000
ENVIRONMENT=development
DEBUG=True
```

### 3. config.py 작성
```python
# backend/app/config.py
from pydantic_settings import BaseSettings, SettingsConfigDict
from typing import List

class Settings(BaseSettings):
    """애플리케이션 설정"""

    # Database
    DATABASE_URL: str

    # Security
    SECRET_KEY: str
    JWT_ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30

    # CORS
    ALLOWED_ORIGINS: str = "http://localhost:3000"

    @property
    def allowed_origins_list(self) -> List[str]:
        """CORS allowed origins를 리스트로 반환"""
        return [origin.strip() for origin in self.ALLOWED_ORIGINS.split(",")]

    # AWS S3 (선택)
    AWS_ACCESS_KEY_ID: str | None = None
    AWS_SECRET_ACCESS_KEY: str | None = None
    AWS_REGION: str = "ap-northeast-2"
    S3_BUCKET_NAME: str | None = None

    # Cloudinary (선택)
    CLOUDINARY_CLOUD_NAME: str | None = None
    CLOUDINARY_API_KEY: str | None = None
    CLOUDINARY_API_SECRET: str | None = None

    # Email (선택)
    SMTP_HOST: str | None = None
    SMTP_PORT: int = 587
    SMTP_USER: str | None = None
    SMTP_PASSWORD: str | None = None

    # Environment
    ENVIRONMENT: str = "development"
    DEBUG: bool = True

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
        case_sensitive=True
    )

# 싱글톤 인스턴스
settings = Settings()
```

### 4. main.py에서 사용
```python
# backend/app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.config import settings

app = FastAPI(
    title="My API",
    debug=settings.DEBUG
)

# CORS 설정
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.allowed_origins_list,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
def root():
    return {
        "environment": settings.ENVIRONMENT,
        "debug": settings.DEBUG
    }
```

### 5. .gitignore 설정
```bash
# backend/.gitignore
.env
.env.local
.env.*.local

# Python
__pycache__/
*.py[cod]
*.so
.Python
venv/
.venv/

# Database
*.db
*.sqlite3
```

## 환경별 설정

### 개발 환경
```bash
# .env.development
ENVIRONMENT=development
DEBUG=True
DATABASE_URL=sqlite:///./dev.db
ALLOWED_ORIGINS=http://localhost:3000
```

### 프로덕션 환경
```bash
# .env.production
ENVIRONMENT=production
DEBUG=False
DATABASE_URL=postgresql://user:pass@prod-server/db
ALLOWED_ORIGINS=https://yourdomain.com
SECRET_KEY=very-long-random-secret-key
```

## 사용 예시

### 데이터베이스 연결
```python
# backend/app/database.py
from sqlalchemy import create_engine
from app.config import settings

engine = create_engine(
    settings.DATABASE_URL,
    echo=settings.DEBUG  # DEBUG 모드에서만 SQL 로그 출력
)
```

### JWT 토큰 생성
```python
# backend/app/auth.py
from jose import jwt
from datetime import datetime, timedelta
from app.config import settings

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(
        minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES
    )
    to_encode.update({"exp": expire})

    encoded_jwt = jwt.encode(
        to_encode,
        settings.SECRET_KEY,
        algorithm=settings.JWT_ALGORITHM
    )
    return encoded_jwt
```

## 검증 예시

### 필수 환경 변수 검증
```python
from pydantic import field_validator

class Settings(BaseSettings):
    SECRET_KEY: str

    @field_validator("SECRET_KEY")
    def validate_secret_key(cls, v):
        if len(v) < 32:
            raise ValueError("SECRET_KEY는 최소 32자 이상이어야 합니다")
        return v

    DATABASE_URL: str

    @field_validator("DATABASE_URL")
    def validate_database_url(cls, v):
        if not v.startswith(("postgresql://", "sqlite:///")):
            raise ValueError("지원하지 않는 데이터베이스입니다")
        return v
```

## 보안 원칙
- ✅ .env 파일은 절대 git에 커밋하지 않기
- ✅ .env.example 템플릿 제공
- ✅ SECRET_KEY는 충분히 길고 랜덤하게
- ✅ 프로덕션에서는 DEBUG=False
- ✅ 환경 변수로 민감 정보 전달 (인자로 전달 금지)
- ❌ 코드에 하드코딩 금지
- ❌ 로그에 환경 변수 출력 금지

## SECRET_KEY 생성
```python
# 랜덤 SECRET_KEY 생성
import secrets
print(secrets.token_urlsafe(32))
```
