---
name: FE-CRUD
description: 프론트엔드 CRUD 화면을 작성합니다. Next.js + TypeScript + Tailwind CSS로 목록, 상세, 생성, 수정 화면을 구현합니다.
context: fork
agent: fe-agent
---

# FE CRUD 작성 가이드

## 개요
Next.js App Router로 CRUD 화면을 생성합니다. 최소한의 코드로 빠르게 동작하는 UI 구축이 목표입니다.

## 기술 스택
- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS

## 프로젝트 구조
```
frontend/src/
├── app/
│   ├── layout.tsx           # 공통 레이아웃
│   ├── page.tsx             # 홈 페이지
│   └── [도메인]/
│       ├── page.tsx         # 목록 페이지
│       └── [id]/page.tsx    # 상세 페이지
├── components/              # 재사용 컴포넌트
└── lib/
    └── api.ts               # API 호출 함수
```

## 파일별 가이드
- [page.md](references/page.md) - 페이지 구조
- [components.md](references/components.md) - CRUD 컴포넌트
- [api.md](references/api.md) - API 호출
- [forms.md](references/forms.md) - 폼 처리

## 실행
```bash
cd frontend && npm run dev
```

## 작성 원칙
- ✅ 'use client' 최소화 (필요한 컴포넌트만)
- ✅ Tailwind CSS 유틸리티 사용
- ✅ TypeScript 타입 정의
- ❌ 외부 UI 라이브러리 금지 (초기 단계)
- ❌ 전역 상태관리 금지 (초기 단계)
- ❌ 과도한 컴포넌트 분리 금지
