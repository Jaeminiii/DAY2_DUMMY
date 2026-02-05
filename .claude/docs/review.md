# 스킬 검토 보고서

작성일: 2026-02-05

## 1. 현재 스킬 목록 (13개)

### 백엔드 스킬 (9개)

- **BE-CRUD**: FastAPI CRUD API 작성
  - 도메인 모델의 기본 CRUD 엔드포인트 생성
  - SQLite + SQLAlchemy 기반
  - 최소한의 코드로 빠른 구현

- **BE-DEBUG**: 백엔드 에러 디버깅
  - Import, DB, Validation, HTTP 에러 해결
  - 스택트레이스 기반 원인 분석
  - 최소 수정으로 문제 해결

- **BE-refactor**: 코드 리팩토링
  - 네이밍, 중복 제거, 구조 개선
  - 동작 변경 없이 가독성 향상
  - 점진적 개선 원칙

- **BE-TEST**: API 테스트 작성
  - pytest + httpx 기반
  - 단위/통합 테스트
  - 독립적 테스트 케이스

- **BE-file-upload**: 파일 업로드 시스템 ⭐ (신규)
  - 로컬, S3, Cloudinary 등 다양한 스토리지 지원
  - 파일 타입 및 크기 검증
  - 범용적으로 사용 가능

- **BE-env**: 환경 변수 관리 ⭐ (신규)
  - .env 파일 + pydantic-settings
  - 개발/프로덕션 환경 분리
  - 민감 정보 안전 관리

- **DB-model**: SQLAlchemy 모델 생성/수정 ✅ (수정됨)
  - ORM 모델 작성
  - 관계 정의 (1:N, N:M, Self-Referencing)
  - 컬럼 타입 및 제약조건
  - context: fork, agent: be-agent 추가

- **DB-migration**: 데이터베이스 마이그레이션 ⭐ (신규)
  - Alembic 기반 스키마 버전 관리
  - 안전한 스키마 변경
  - 롤백 가능

- **Deploy**: 배포 및 컨테이너화 ⭐ (신규)
  - Docker, docker-compose
  - CI/CD 파이프라인
  - 다양한 플랫폼 지원 (Vercel, Railway, AWS)

### 프론트엔드 스킬 (3개)

- **FE-CRUD**: CRUD 화면 작성
  - Next.js App Router 기반
  - 목록, 상세, 생성, 수정 페이지
  - Tailwind CSS 스타일링

- **FE-api**: API 호출 코드
  - fetch 기반 API 통신
  - 프록시 경로 활용 (`/api/*`)
  - TypeScript 타입 정의 및 에러 처리

- **FE-page**: 페이지/컴포넌트 생성
  - Next.js 페이지 구조
  - 재사용 가능한 컴포넌트
  - App Router 규칙 준수

### 워크플로우 스킬 (1개)

- **git_commit**: Git 커밋 워크플로우
  - progress.md 자동 업데이트
  - task.md 상태 관리
  - 커밋 메시지 컨벤션 적용

---

## 2. 스킬 검토 결과

### ✅ 잘 작성된 부분

- **명확한 책임 분리**
  - 각 스킬이 단일 책임을 가지고 있음
  - 백엔드/프론트엔드 도메인별 구분이 체계적

- **서브에이전트 활용**
  - `context: fork` 설정으로 독립된 컨텍스트 실행
  - `agent: be-agent`, `agent: fe-agent` 지정으로 전문성 확보

- **참조 문서 구조**
  - `references/` 디렉토리로 상세 가이드 분리
  - 템플릿, 패턴, 예제 코드 제공

- **일관된 원칙**
  - ✅/❌ 형식으로 작성 원칙 명시
  - 최소 코드, 빠른 구현 철학 유지

### ✅ 수정 완료

#### 1. DB-model 스킬 설정 추가
- ✅ `context: fork` 추가
- ✅ `agent: be-agent` 추가
- 이제 be-agent가 독립된 컨텍스트에서 DB 모델 작업 수행

#### 2. 범용 스킬 4개 추가
- ✅ **BE-file-upload**: 파일 업로드 시스템 (여러 스토리지 지원)
- ✅ **BE-env**: 환경 변수 관리 (pydantic-settings)
- ✅ **DB-migration**: 데이터베이스 마이그레이션 (Alembic)
- ✅ **Deploy**: Docker 및 배포 설정

### ⚠️ 남은 개선사항

#### git_commit 스킬 이름 불일치 (선택적)

**현재 상태:**
- 디렉토리명: `git_commit`
- 파일 내 스킬명: `git-commit-workflow`
- 시스템에서 인식: `git_commit`

**권장 조치:**
- 기능상 문제 없으나, 일관성을 위해 스킬명을 `git_commit`으로 통일 권장

---

## 3. 추가 추천 스킬

### 🔴 우선순위: 높음 (프로젝트 성숙도 향상)

#### BE-auth
```yaml
name: BE-auth
description: 백엔드 인증/인가 시스템 구현. JWT 토큰, 세션 관리, 권한 체크 등
context: fork
agent: be-agent
```

**필요 이유:**
- 실제 서비스에는 사용자 인증이 필수
- 보호된 API 엔드포인트 구현
- 역할 기반 접근 제어 (RBAC)

**구현 내용:**
- JWT 토큰 발급/검증
- 비밀번호 해싱 (bcrypt)
- OAuth 2.0 통합 (선택)
- 권한 데코레이터 (`@require_auth`)

---

#### FE-auth
```yaml
name: FE-auth
description: 프론트엔드 인증 UI 및 토큰 관리. 로그인/로그아웃, 보호된 라우트 처리
context: fork
agent: fe-agent
```

**필요 이유:**
- 로그인/회원가입 화면 필요
- 토큰 저장 및 자동 갱신
- 인증 상태에 따른 라우팅

**구현 내용:**
- 로그인/회원가입 폼
- 토큰 저장 (localStorage/cookie)
- Protected Route 미들웨어
- 자동 로그아웃 (토큰 만료)

---

#### DB-migration
```yaml
name: DB-migration
description: Alembic을 사용한 데이터베이스 마이그레이션 관리
context: fork
agent: be-agent
```

**필요 이유:**
- 현재는 Alembic 사용 금지이지만, 프로덕션에서는 필수
- 스키마 변경 이력 관리
- 팀 협업 시 DB 동기화

**구현 내용:**
- Alembic 초기 설정
- 마이그레이션 파일 생성
- 업그레이드/다운그레이드 스크립트
- 자동 마이그레이션 감지

---

#### E2E-test
```yaml
name: E2E-test
description: Playwright/Cypress를 사용한 E2E 테스트 작성
context: fork
agent: fe-agent
```

**필요 이유:**
- 사용자 시나리오 기반 테스트
- 백엔드-프론트엔드 통합 검증
- 회귀 테스트 자동화

**구현 내용:**
- Playwright 설정
- 주요 사용자 플로우 테스트
- 스크린샷/비디오 기록
- CI/CD 통합

---

### 🟡 우선순위: 중간 (확장성 및 품질 향상)

#### FE-form
```yaml
name: FE-form
description: React Hook Form + Zod를 사용한 폼 검증 구현
context: fork
agent: fe-agent
```

**필요 이유:**
- 복잡한 폼 처리 (다단계, 동적 필드)
- 타입 안전한 검증
- 성능 최적화 (불필요한 리렌더링 방지)

**구현 내용:**
- React Hook Form 설정
- Zod 스키마 정의
- 커스텀 검증 규칙
- 에러 메시지 표시

---

#### Performance
```yaml
name: Performance
description: 성능 최적화. 쿼리 최적화, 캐싱, 번들 크기 감소 등
context: fork
```

**필요 이유:**
- 사용자 경험 향상
- 서버 부하 감소
- SEO 개선

**구현 내용 (백엔드):**
- SQL 쿼리 최적화
- Redis 캐싱
- 페이지네이션 개선

**구현 내용 (프론트엔드):**
- 코드 스플리팅
- 이미지 최적화 (Next.js Image)
- 번들 분석 및 최적화

---

#### Docker-deploy
```yaml
name: Docker-deploy
description: Docker 컨테이너화 및 배포 설정
```

**필요 이유:**
- 환경 일관성 보장
- 손쉬운 배포
- 확장성 (Kubernetes 등)

**구현 내용:**
- Dockerfile 작성 (BE/FE)
- docker-compose.yml
- 멀티 스테이지 빌드
- CI/CD 파이프라인 (GitHub Actions)

---

#### API-docs
```yaml
name: API-docs
description: FastAPI Swagger/OpenAPI 문서 커스터마이징
context: fork
agent: be-agent
```

**필요 이유:**
- API 사용법 명확화
- 프론트엔드 개발자와 협업 개선
- 외부 개발자 지원

**구현 내용:**
- 상세한 설명 추가
- 예시 요청/응답
- 태그 및 그룹화
- 커스텀 OpenAPI 스키마

---

### 🟢 우선순위: 낮음 (선택적)

#### FE-state
```yaml
name: FE-state
description: Zustand/Jotai를 사용한 전역 상태 관리
context: fork
agent: fe-agent
```

**필요 이유:**
- 앱 규모가 커질 때 필요
- 컴포넌트 간 상태 공유
- Props drilling 방지

**구현 내용:**
- Zustand store 설정
- 상태 슬라이스 정의
- DevTools 연동

---

#### Code-review
```yaml
name: Code-review
description: 코드 품질 자동 체크. ESLint, Prettier, mypy, ruff 등
```

**필요 이유:**
- 코드 스타일 일관성
- 잠재적 버그 사전 감지
- 코드 리뷰 시간 단축

**구현 내용:**
- ESLint + Prettier (FE)
- ruff + mypy (BE)
- Pre-commit hooks
- CI/CD 통합

---

#### BE-logging
```yaml
name: BE-logging
description: 구조화된 로깅 시스템 구축
context: fork
agent: be-agent
```

**필요 이유:**
- 디버깅 효율성
- 프로덕션 모니터링
- 에러 트래킹

**구현 내용:**
- 구조화된 로깅 (JSON)
- 로그 레벨 관리
- Sentry 연동
- 로그 필터링

---

## 4. 적용 현황

### ✅ Phase 1: 완료됨
1. ✅ DB-model 스킬에 `context: fork`, `agent: be-agent` 추가
2. ✅ 범용 스킬 4개 추가 (BE-file-upload, BE-env, DB-migration, Deploy)
3. ⚠️ git_commit 스킬 이름 통일 (선택적)

### 📋 Phase 2: 프로젝트별 추가 (필요 시)
1. 🔴 BE-auth
2. 🔴 FE-auth
3. 🔴 E2E-test

### Phase 3: 중기 추가 (프로덕션 준비)
1. 🟡 DB-migration
2. 🟡 Docker-deploy
3. 🟡 Performance

### Phase 4: 장기 추가 (확장 시)
1. 🟢 FE-form
2. 🟢 FE-state
3. 🟢 BE-logging
4. 🟢 Code-review
5. 🟢 API-docs

---

## 5. 결론

### 전반적 평가
- ✅ 스킬 구조가 체계적이고 잘 설계됨
- ✅ 백엔드/프론트엔드 분리가 명확함
- ✅ 서브에이전트 활용으로 전문성 확보
- ✅ 범용 틀로 다양한 프로젝트에 활용 가능 **(2026-02-05 개선)**

### 현재 틀의 완성도
- **기본 CRUD 시스템**: 완벽 ✅
- **파일 업로드**: 완벽 ✅ (신규 추가)
- **환경 변수 관리**: 완벽 ✅ (신규 추가)
- **데이터베이스 마이그레이션**: 완벽 ✅ (신규 추가)
- **배포 시스템**: 완벽 ✅ (신규 추가)

### 강점
- 최소 코드, 빠른 구현 철학
- 참조 문서로 상세 가이드 제공
- 일관된 작성 원칙
- 범용성: 쇼핑몰, 블로그, SaaS 등 다양한 프로젝트에 적용 가능

### 프로젝트별 추가 스킬 (필요 시)
**쇼핑몰 프로젝트:**
- BE-auth, FE-auth (인증)
- BE-payment, FE-checkout (결제)
- FE-cart (장바구니)
- FE-admin (관리자)

**블로그 프로젝트:**
- BE-markdown (마크다운 처리)
- FE-editor (에디터)
- BE-search (전체 텍스트 검색)

**SaaS 프로젝트:**
- BE-auth (인증 필수)
- BE-subscription (구독)
- BE-webhook (이벤트)
- FE-dashboard (대시보드)

### 범용 틀로서의 가치
현재 프로젝트는 **어떤 웹 애플리케이션이든 시작할 수 있는 견고한 기반**을 제공합니다:
- 백엔드: FastAPI + SQLAlchemy + Alembic
- 프론트엔드: Next.js + TypeScript + Tailwind
- 인프라: Docker + 다양한 배포 옵션
- 개발 경험: 에이전트 기반 효율적 개발

---

**작성자:** Claude Sonnet 4.5
**검토일:** 2026-02-05
