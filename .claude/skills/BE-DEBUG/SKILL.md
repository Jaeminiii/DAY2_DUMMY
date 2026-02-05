---
name: BE-DEBUG
description: FastAPI 백엔드 에러를 분석하고 해결합니다. 에러메세지, 버그 수정 등의 요청시에 활용합니다.
context: fork
agent: be-agent
---

# BE DEBUG 가이드

## 개요
FastAPI 백엔드 에러를 분석하고 해결합니다. 에러 메시지 기반으로 원인을 파악하고 수정합니다.

## 디버깅 절차
1. 에러 메시지/스택트레이스 확인
2. 관련 파일 위치 파악
3. 원인 분석
4. 수정 및 검증

## 에러 유형별 가이드
- [import-errors.md](references/import-errors.md) - Import/Module 에러
- [db-errors.md](references/db-errors.md) - DB/SQLAlchemy 에러
- [validation-errors.md](references/validation-errors.md) - Pydantic 검증 에러
- [http-errors.md](references/http-errors.md) - HTTP 상태 코드 에러

## 디버깅 원칙
- ✅ 에러 메시지 전체 확인
- ✅ 스택트레이스에서 파일:라인 번호 추적
- ✅ 최소 수정으로 해결
- ❌ 관련 없는 코드 변경 금지
- ❌ 추측으로 수정 금지 (원인 파악 우선)

## 실행 및 테스트
```bash
# 서버 실행
cd backend && .venv\Scripts\activate && uvicorn app.main:app --reload

# API 테스트: http://localhost:8000/docs
```

