---
name: FE-api
description: 프론트엔드에서 백엔드 API 호출 코드를 작성합니다. 데이터 fetching이나 상태관리가 필요할 때 사용하세요.
context: fork
agent: fe-agent
---

# FE API 가이드

## 개요
프론트엔드에서 백엔드 API를 호출하는 코드를 작성합니다.

## 프록시 설정
`/api/*` 요청은 `next.config.js`의 rewrites로 백엔드(localhost:8000)로 프록시됩니다.
CORS 문제 없이 상대경로로 API 호출 가능합니다.

## 파일 위치
```
frontend/src/
└── lib/
    └── api.ts           # API 호출 함수
```

## 참조 문서
- [fetch.md](references/fetch.md) - fetch 함수
- [hooks.md](references/hooks.md) - 커스텀 훅
- [error-handling.md](references/error-handling.md) - 에러 처리

## 작성 원칙
- ✅ 프록시 경로 사용 (`/api/...`)
- ✅ TypeScript 타입 정의
- ✅ 에러 처리 포함
- ❌ 직접 localhost:8000 호출 금지
- ❌ axios 등 외부 라이브러리 금지 (초기 단계)
