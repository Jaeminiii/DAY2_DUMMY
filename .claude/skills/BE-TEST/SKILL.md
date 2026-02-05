---
name: BE-TEST
description: 백엔드 API 엔드포인트 테스트를 작성합니다. 백엔드 단위/통합 테스트가 필요할 때 사용하세요.
context: fork
agent: be-agent
---

# BE 테스트 가이드

## 개요
FastAPI 엔드포인트의 단위/통합 테스트를 작성합니다.

## 설치
```bash
pip install pytest httpx
```

## 테스트 파일 위치
```
backend/
└── test/
    ├── conftest.py      # 공통 fixture
    └── test_*.py        # 테스트 파일
```

## 참조 문서
- [setup.md](references/setup.md) - 테스트 환경 설정
- [templates.md](references/templates.md) - CRUD 테스트 템플릿
- [commands.md](references/commands.md) - 실행 명령어

## 테스트 원칙
- ✅ 각 테스트는 독립적으로 실행 가능
- ✅ 테스트 DB 사용 (운영 DB 분리)
- ✅ 명확한 테스트 케이스명
- ❌ 테스트 간 의존성 금지
- ❌ 외부 API 직접 호출 금지
