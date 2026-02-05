---
name: FE-page
description: 프론트엔드 페이지를 작업하여 Next.js 페이지와 컴포넌트를 생성합니다. 새로운 화면이나 UI 컴포넌트가 필요할 때 사용하세요.
context: fork
agent: fe-agent
---

# FE Page 가이드

## 개요
Next.js App Router로 페이지와 컴포넌트를 생성합니다.

## 기술 스택
- **Framework**: Next.js 14 (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS

## 프로젝트 구조
```
frontend/src/
├── app/
│   ├── layout.tsx       # 공통 레이아웃
│   ├── page.tsx         # 홈 (/)
│   └── [도메인]/
│       └── page.tsx     # 도메인 페이지
└── components/          # 재사용 컴포넌트
```

## 참조 문서
- [page-template.md](references/page-template.md) - 페이지 템플릿
- [component-template.md](references/component-template.md) - 컴포넌트 템플릿
- [tailwind.md](references/tailwind.md) - Tailwind 클래스

## 작성 원칙
- ✅ App Router 규칙 준수 (page.tsx, layout.tsx)
- ✅ TypeScript 인터페이스 정의
- ✅ Tailwind 유틸리티 클래스 사용
- ❌ pages/ 디렉토리 사용 금지 (App Router만)
- ❌ CSS 파일 추가 생성 금지
