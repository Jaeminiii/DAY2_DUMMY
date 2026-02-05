---
name: BE-CRUD
description: 백엔드 초기 코드를 작성합니다. Python FastAPI로 구현하며 필수적인 로직만 최대한 간단하게 작성합니다. Todo, User, Product 등 도메인 모델의 CRUD API가 필요할 때 사용합니다.
context: fork
agent: be-agent
---

# BE CRUD 작성 가이드

## 개요
FastAPI로 기본 CRUD API를 생성합니다. 최소한의 코드로 빠르게 동작하는 백엔드 구축이 목표입니다.

## 기술 스택
- **Framework**: FastAPI
- **Language**: Python 3.12+
- **Database**: SQLite (별도 설치 불필요)
- **ORM**: SQLAlchemy

## 프로젝트 구조
```
backend/
├── app/
│   ├── database.py      # DB 연결, 세션
│   ├── main.py          # FastAPI 앱 진입점
│   ├── models/          # SQLAlchemy 모델
│   ├── schemas/         # Pydantic 스키마
│   └── routers/         # API 엔드포인트
└── app.db               # SQLite DB (자동 생성)
```

## 파일별 가이드
- [database.md](references/database.md) - DB 설정
- [models.md](references/models.md) - 모델 작성
- [schemas.md](references/schemas.md) - 스키마 작성
- [endpoints.md](references/endpoints.md) - 엔드포인트 작성

## 실행
```bash
cd backend && .venv\Scripts\activate && uvicorn app.main:app --reload
```

## 작성 원칙
- ✅ 단일 파일로 시작 가능하도록 최소화
- ✅ SQLite 사용 (별도 DB 설치 불필요)
- ✅ CORS 기본 포함
- ❌ 복잡한 폴더 구조 금지
- ❌ 인증/인가 초기 단계 제외
- ❌ Alembic 사용 금지