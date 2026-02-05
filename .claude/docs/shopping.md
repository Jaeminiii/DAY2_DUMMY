# 쇼핑몰 사이트 구현 계획

작성일: 2026-02-05

---

## 1. 현재 틀로 구현 가능한 수준

### ✅ 즉시 구현 가능 (현재 스킬 활용)

#### 기본 기능
- **상품 관리 (Product CRUD)**
  - 상품 목록 조회, 상세 페이지
  - 상품 등록, 수정, 삭제 (관리자용)
  - BE-CRUD + FE-CRUD 스킬 활용

- **카테고리 관리**
  - 카테고리별 상품 분류
  - 계층형 카테고리 (부모-자식 관계)
  - DB-model 스킬로 Self-Referencing 구현

- **사용자 관리**
  - 회원 정보 CRUD
  - 프로필 페이지
  - 단순 회원 가입 폼

- **기본 UI**
  - 상품 목록 그리드
  - 상품 상세 페이지
  - 간단한 검색/필터
  - Tailwind CSS 기반 반응형 디자인

#### 개발 환경
- **로컬 개발 서버**
  - FastAPI (localhost:8000)
  - Next.js (localhost:3000)
  - 즉시 실행 가능

- **API 문서**
  - Swagger UI 자동 생성
  - 엔드포인트 테스트 가능

### ⚠️ 제한적 구현 가능 (추가 작업 필요)

#### 준비된 구조 + 추가 개발
- **간단한 장바구니**
  - 로컬 스토리지 기반 (비회원)
  - DB 연동 (회원)
  - 수량 조절, 삭제 기능

- **주문 내역**
  - 주문 정보 저장 (결제 제외)
  - 주문 상태 관리 (접수, 배송 중, 완료)
  - 주문 조회 페이지

- **기본 검색**
  - 상품명 기반 검색
  - 카테고리 필터링
  - 가격 범위 필터

### ❌ 현재 틀로는 구현 불가 (외부 서비스/추가 인프라 필요)

#### 외부 연동 필수
- **결제 시스템**
  - PG사 연동 (Stripe, Toss Payments, KG이니시스 등)
  - 결제 SDK 통합 필요

- **이미지 호스팅**
  - 상품 이미지 업로드/저장
  - CDN 필요 (AWS S3, Cloudinary, Vercel Blob 등)

- **이메일/SMS 알림**
  - 주문 확인 메일
  - 배송 알림
  - SMTP 서버 또는 SendGrid, AWS SES 필요

- **실시간 재고 동기화**
  - WebSocket 또는 Server-Sent Events
  - Redis 캐싱

---

## 2. 변경점 추천

### 🔴 필수 변경사항

#### 1. 데이터베이스 변경
**현재:** SQLite (파일 기반)

**변경 후:** PostgreSQL

**이유:**
- SQLite는 동시 쓰기 제한 (멀티유저 환경 부적합)
- 트랜잭션 격리 수준 낮음
- 프로덕션 배포 시 데이터 손실 위험

**적용 방법:**
```python
# backend/app/database.py
DATABASE_URL = "postgresql://user:password@localhost:5432/shopping_db"
```

---

#### 2. 인증/인가 시스템 추가
**현재:** 없음

**변경 후:** JWT 기반 인증

**필수 기능:**
- 회원가입/로그인
- 비밀번호 해싱 (bcrypt)
- Access Token + Refresh Token
- 권한 구분 (일반 사용자 / 관리자)

**새로운 스킬 필요:**
- BE-auth
- FE-auth

---

#### 3. 파일 업로드 시스템
**현재:** 없음

**변경 후:** 클라우드 스토리지

**옵션:**
- **AWS S3** (가장 범용적)
- **Cloudinary** (이미지 최적화 자동)
- **Vercel Blob** (Next.js 통합 간편)

**구현:**
```python
# 백엔드: 이미지 업로드 엔드포인트
@router.post("/products/{id}/images")
async def upload_image(file: UploadFile):
    # S3 또는 Cloudinary에 업로드
    url = await upload_to_storage(file)
    return {"image_url": url}
```

---

#### 4. 환경 변수 관리
**현재:** 하드코딩

**변경 후:** `.env` 파일 + pydantic-settings

**설정 항목:**
- 데이터베이스 연결 정보
- JWT 시크릿 키
- S3/Cloudinary API 키
- PG사 API 키

```bash
# backend/.env
DATABASE_URL=postgresql://user:pass@localhost/db
SECRET_KEY=your-secret-key
AWS_ACCESS_KEY_ID=...
STRIPE_API_KEY=...
```

---

### 🟡 권장 변경사항

#### 5. CORS 설정 강화
**현재:** 모든 origin 허용

**변경 후:** 프로덕션 도메인만 허용

```python
# backend/app/main.py
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "http://localhost:3000",  # 개발
        "https://yourdomain.com",  # 프로덕션
    ],
    allow_credentials=True,
)
```

---

#### 6. 페이지네이션 표준화
**현재:** 없음

**변경 후:** Offset/Limit 방식 또는 Cursor 방식

```python
@router.get("/products")
def list_products(skip: int = 0, limit: int = 20):
    products = db.query(Product).offset(skip).limit(limit).all()
    return products
```

---

#### 7. 에러 응답 표준화
**현재:** 기본 FastAPI 에러

**변경 후:** 일관된 JSON 형식

```python
{
  "error": {
    "code": "PRODUCT_NOT_FOUND",
    "message": "상품을 찾을 수 없습니다.",
    "details": {...}
  }
}
```

---

## 3. 추가되어야 할 기능

### 🛒 쇼핑몰 핵심 기능

#### 1. 상품 관리
- [x] 상품 CRUD (기본 가능)
- [ ] **상품 이미지 업로드** (여러 장)
- [ ] **재고 관리** (수량 추적, 품절 표시)
- [ ] **상품 옵션** (색상, 사이즈 등)
- [ ] **할인가 관리** (정가, 할인율, 판매가)
- [ ] **상품 상태** (판매중, 품절, 단종)

**DB 모델 예시:**
```python
class Product(Base):
    id = Column(Integer, primary_key=True)
    name = Column(String(200), nullable=False)
    description = Column(Text)
    price = Column(Numeric(10, 2), nullable=False)
    discount_rate = Column(Integer, default=0)  # 0-100
    stock = Column(Integer, default=0)
    status = Column(Enum("active", "sold_out", "discontinued"))
    category_id = Column(Integer, ForeignKey("categories.id"))

    # 관계
    images = relationship("ProductImage", back_populates="product")
    options = relationship("ProductOption", back_populates="product")

class ProductImage(Base):
    id = Column(Integer, primary_key=True)
    product_id = Column(Integer, ForeignKey("products.id"))
    url = Column(String(500), nullable=False)
    order = Column(Integer, default=0)  # 이미지 순서
    is_primary = Column(Boolean, default=False)  # 대표 이미지
```

---

#### 2. 장바구니 시스템
- [ ] **장바구니 추가/삭제**
- [ ] **수량 조절**
- [ ] **선택 삭제** (체크박스)
- [ ] **전체 선택/해제**
- [ ] **합계 금액 계산**
- [ ] **품절 상품 처리**

**DB 모델:**
```python
class Cart(Base):
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    product_id = Column(Integer, ForeignKey("products.id"))
    quantity = Column(Integer, default=1)
    option_id = Column(Integer, ForeignKey("product_options.id"))
    created_at = Column(DateTime, default=datetime.utcnow)
```

**프론트엔드 상태 관리:**
- 비회원: localStorage
- 회원: DB + Zustand/Jotai (전역 상태)

---

#### 3. 주문/결제 시스템
- [ ] **주문서 작성** (배송지, 요청사항)
- [ ] **결제 수단 선택** (카드, 계좌이체, 간편결제)
- [ ] **PG사 연동** (Toss Payments, Stripe 등)
- [ ] **주문 확인 페이지**
- [ ] **결제 성공/실패 처리**
- [ ] **재고 차감** (트랜잭션)

**주문 플로우:**
```
1. 장바구니 → 주문서 작성
2. 배송지 입력
3. 결제 수단 선택
4. PG사 결제 페이지 이동
5. 결제 완료 후 콜백
6. 주문 완료 페이지
```

**DB 모델:**
```python
class Order(Base):
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    order_number = Column(String(50), unique=True)  # ORD-20260205-1234
    status = Column(Enum("pending", "paid", "shipped", "delivered", "cancelled"))

    # 배송 정보
    recipient_name = Column(String(100))
    phone = Column(String(20))
    address = Column(String(500))
    postal_code = Column(String(10))
    request = Column(Text)  # 배송 요청사항

    # 결제 정보
    payment_method = Column(String(50))  # card, bank, kakao, etc
    total_amount = Column(Numeric(10, 2))
    paid_at = Column(DateTime)

    # 관계
    items = relationship("OrderItem", back_populates="order")

class OrderItem(Base):
    id = Column(Integer, primary_key=True)
    order_id = Column(Integer, ForeignKey("orders.id"))
    product_id = Column(Integer, ForeignKey("products.id"))
    quantity = Column(Integer)
    price = Column(Numeric(10, 2))  # 주문 당시 가격 저장
    option_name = Column(String(100))
```

---

#### 4. 사용자 인증/권한
- [ ] **회원가입** (이메일 인증)
- [ ] **로그인/로그아웃**
- [ ] **JWT 토큰 발급**
- [ ] **비밀번호 재설정**
- [ ] **소셜 로그인** (Google, Kakao)
- [ ] **권한 구분** (User, Admin)

**사용자 모델:**
```python
class User(Base):
    id = Column(Integer, primary_key=True)
    email = Column(String(255), unique=True, nullable=False)
    password_hash = Column(String(255), nullable=False)
    name = Column(String(100))
    phone = Column(String(20))
    role = Column(Enum("user", "admin"), default="user")
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)

    # 관계
    orders = relationship("Order", back_populates="user")
    cart_items = relationship("Cart", back_populates="user")
    addresses = relationship("Address", back_populates="user")
```

---

#### 5. 검색 및 필터링
- [ ] **상품명 검색** (전체 텍스트 검색)
- [ ] **카테고리 필터**
- [ ] **가격 범위 필터**
- [ ] **정렬** (최신순, 가격 낮은순/높은순, 인기순)
- [ ] **검색 자동완성** (Optional)

**API 엔드포인트:**
```python
@router.get("/products/search")
def search_products(
    q: str = None,  # 검색어
    category_id: int = None,
    min_price: float = None,
    max_price: float = None,
    sort_by: str = "created_at",
    order: str = "desc",
    skip: int = 0,
    limit: int = 20,
):
    query = db.query(Product)

    if q:
        query = query.filter(Product.name.ilike(f"%{q}%"))
    if category_id:
        query = query.filter(Product.category_id == category_id)
    if min_price:
        query = query.filter(Product.price >= min_price)
    if max_price:
        query = query.filter(Product.price <= max_price)

    # 정렬
    if sort_by == "price":
        query = query.order_by(Product.price.desc() if order == "desc" else Product.price)

    return query.offset(skip).limit(limit).all()
```

---

#### 6. 리뷰/평점 시스템
- [ ] **상품 리뷰 작성** (구매자만)
- [ ] **별점 (1-5점)**
- [ ] **리뷰 이미지 첨부**
- [ ] **도움이 됐어요** 기능
- [ ] **관리자 답변**

**DB 모델:**
```python
class Review(Base):
    id = Column(Integer, primary_key=True)
    product_id = Column(Integer, ForeignKey("products.id"))
    user_id = Column(Integer, ForeignKey("users.id"))
    order_id = Column(Integer, ForeignKey("orders.id"))  # 구매 검증
    rating = Column(Integer)  # 1-5
    content = Column(Text)
    images = Column(JSON)  # ["url1", "url2"]
    helpful_count = Column(Integer, default=0)
    created_at = Column(DateTime, default=datetime.utcnow)
```

---

#### 7. 관리자 페이지
- [ ] **대시보드** (매출, 주문 통계)
- [ ] **상품 관리** (등록, 수정, 삭제)
- [ ] **주문 관리** (상태 변경, 배송 처리)
- [ ] **회원 관리** (목록, 권한 수정)
- [ ] **리뷰 관리** (승인, 삭제)
- [ ] **통계** (일별 매출, 인기 상품)

**프론트엔드 구조:**
```
frontend/src/app/
├── admin/
│   ├── layout.tsx           # 관리자 레이아웃 (사이드바)
│   ├── dashboard/page.tsx   # 대시보드
│   ├── products/page.tsx    # 상품 관리
│   ├── orders/page.tsx      # 주문 관리
│   └── users/page.tsx       # 회원 관리
```

---

#### 8. 배송 관리
- [ ] **배송지 저장** (여러 개)
- [ ] **기본 배송지 설정**
- [ ] **배송 상태 추적**
- [ ] **운송장 번호 입력** (관리자)

**DB 모델:**
```python
class Address(Base):
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    name = Column(String(100))  # 배송지명 (집, 회사 등)
    recipient = Column(String(100))
    phone = Column(String(20))
    address = Column(String(500))
    postal_code = Column(String(10))
    is_default = Column(Boolean, default=False)
```

---

#### 9. 위시리스트 (찜)
- [ ] **상품 찜하기/해제**
- [ ] **찜 목록 조회**
- [ ] **찜한 상품 장바구니 담기**

**DB 모델:**
```python
class Wishlist(Base):
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey("users.id"))
    product_id = Column(Integer, ForeignKey("products.id"))
    created_at = Column(DateTime, default=datetime.utcnow)
```

---

#### 10. 쿠폰/프로모션
- [ ] **쿠폰 발급**
- [ ] **쿠폰 적용**
- [ ] **할인 계산**
- [ ] **사용 내역 관리**

**DB 모델:**
```python
class Coupon(Base):
    id = Column(Integer, primary_key=True)
    code = Column(String(50), unique=True)
    discount_type = Column(Enum("percentage", "fixed"))  # 퍼센트 or 고정 금액
    discount_value = Column(Numeric(10, 2))
    min_purchase_amount = Column(Numeric(10, 2))  # 최소 구매 금액
    max_discount_amount = Column(Numeric(10, 2))  # 최대 할인 금액
    valid_from = Column(DateTime)
    valid_until = Column(DateTime)
    usage_limit = Column(Integer)  # 사용 횟수 제한
    used_count = Column(Integer, default=0)
```

---

### 🔧 기술적 개선 사항

#### 11. 성능 최적화
- [ ] **이미지 레이지 로딩**
- [ ] **무한 스크롤** (상품 목록)
- [ ] **Redis 캐싱** (인기 상품, 카테고리)
- [ ] **DB 인덱싱** (검색 성능)
- [ ] **CDN 활용** (정적 파일)

---

#### 12. 보안
- [ ] **HTTPS 적용** (필수)
- [ ] **SQL Injection 방어** (ORM 사용으로 기본 방어)
- [ ] **XSS 방어** (입력 검증, sanitize)
- [ ] **CSRF 토큰**
- [ ] **Rate Limiting** (API 호출 제한)
- [ ] **비밀번호 정책** (8자 이상, 특수문자 포함)

---

#### 13. 알림 시스템
- [ ] **주문 확인 이메일**
- [ ] **배송 시작 알림**
- [ ] **배송 완료 알림**
- [ ] **재입고 알림**
- [ ] **프로모션 알림**

**연동 서비스:**
- SendGrid (이메일)
- AWS SES (이메일)
- Twilio (SMS)
- Firebase Cloud Messaging (푸시 알림)

---

#### 14. 분석/통계
- [ ] **방문자 추적** (Google Analytics)
- [ ] **전환율 분석**
- [ ] **장바구니 이탈률**
- [ ] **매출 통계 대시보드**
- [ ] **인기 상품 순위**

---

## 4. 구현 로드맵

### Phase 1: MVP (최소 기능 제품) - 2-3주

**목표:** 기본적인 쇼핑 플로우 구현

- [x] 프로젝트 초기 설정
- [ ] DB 변경 (SQLite → PostgreSQL)
- [ ] 상품 CRUD (이미지 제외)
- [ ] 카테고리 관리
- [ ] 간단한 장바구니 (localStorage)
- [ ] 사용자 인증 (회원가입/로그인)
- [ ] 주문 정보 저장 (결제 제외)

**결과물:**
- 상품 등록/조회 가능
- 로그인 후 장바구니 담기 가능
- 주문서 작성 가능 (결제 미연동)

---

### Phase 2: 핵심 기능 - 3-4주

**목표:** 실제 구매 가능한 쇼핑몰

- [ ] 이미지 업로드 (S3/Cloudinary)
- [ ] 결제 연동 (Toss Payments 추천)
- [ ] 주문 상태 관리
- [ ] 재고 관리
- [ ] 검색/필터링
- [ ] 관리자 페이지 (기본)

**결과물:**
- 실제 결제 가능
- 관리자가 주문 관리 가능
- 재고 자동 차감

---

### Phase 3: 사용자 경험 개선 - 2-3주

**목표:** 편의성 및 신뢰성 향상

- [ ] 리뷰/평점 시스템
- [ ] 위시리스트
- [ ] 배송지 관리
- [ ] 주문 내역 조회
- [ ] 이메일 알림
- [ ] 쿠폰 시스템

**결과물:**
- 사용자 만족도 향상
- 재구매 유도 기능

---

### Phase 4: 고도화 - 3-4주

**목표:** 프로덕션 준비 및 스케일링

- [ ] 성능 최적화 (Redis, CDN)
- [ ] E2E 테스트
- [ ] Docker 배포
- [ ] CI/CD 파이프라인
- [ ] 모니터링 (Sentry, 로그)
- [ ] 관리자 대시보드 (통계)

**결과물:**
- 프로덕션 배포 가능
- 안정적인 운영 환경

---

## 5. 필요한 스킬 추가

### 새로운 스킬 목록

```yaml
# 1. BE-auth (필수)
name: BE-auth
description: JWT 인증, 회원가입, 로그인, 권한 관리
context: fork
agent: be-agent
```

```yaml
# 2. FE-auth (필수)
name: FE-auth
description: 로그인 UI, 토큰 관리, Protected Route
context: fork
agent: fe-agent
```

```yaml
# 3. BE-payment (필수)
name: BE-payment
description: 결제 게이트웨이 연동 (Toss, Stripe)
context: fork
agent: be-agent
```

```yaml
# 4. FE-cart (필수)
name: FE-cart
description: 장바구니 UI 및 상태 관리
context: fork
agent: fe-agent
```

```yaml
# 5. BE-file-upload (필수)
name: BE-file-upload
description: 이미지 업로드 (S3, Cloudinary)
context: fork
agent: be-agent
```

```yaml
# 6. FE-admin (필수)
name: FE-admin
description: 관리자 페이지 (상품/주문/회원 관리)
context: fork
agent: fe-agent
```

```yaml
# 7. BE-search (권장)
name: BE-search
description: 전체 텍스트 검색, 필터링, 정렬
context: fork
agent: be-agent
```

```yaml
# 8. FE-checkout (필수)
name: FE-checkout
description: 주문서 작성, 결제 플로우
context: fork
agent: fe-agent
```

---

## 6. 기술 스택 최종 권장

### 백엔드
- **프레임워크:** FastAPI
- **데이터베이스:** PostgreSQL (SQLite 대체)
- **ORM:** SQLAlchemy
- **인증:** JWT (python-jose)
- **비밀번호:** bcrypt
- **파일 업로드:** AWS S3 또는 Cloudinary
- **결제:** Toss Payments (한국) / Stripe (글로벌)
- **이메일:** SendGrid 또는 AWS SES
- **캐싱:** Redis (선택)

### 프론트엔드
- **프레임워크:** Next.js 14 (App Router)
- **언어:** TypeScript
- **스타일:** Tailwind CSS
- **상태 관리:** Zustand (장바구니, 인증 상태)
- **폼 검증:** React Hook Form + Zod
- **결제 UI:** Toss Payments SDK
- **이미지 최적화:** Next.js Image

### 인프라
- **호스팅 (BE):** AWS EC2, Fly.io, Railway
- **호스팅 (FE):** Vercel, Netlify
- **데이터베이스:** AWS RDS (PostgreSQL), Supabase
- **파일 스토리지:** AWS S3, Cloudinary
- **도메인/SSL:** Route53 + ACM, Cloudflare

---

## 7. 예상 비용 (MVP 기준)

### 개발 비용
- **개발 기간:** 2-3개월 (1인 풀타임 기준)
- **Phase 1-2 완료:** 최소 구매 가능

### 운영 비용 (월간, 소규모 트래픽 기준)
- **Vercel (프론트엔드):** $0 (Hobby) ~ $20 (Pro)
- **Railway/Fly.io (백엔드):** $5 ~ $20
- **AWS RDS (PostgreSQL):** $15 ~ $50
- **AWS S3 (이미지 저장):** $5 ~ $10
- **SendGrid (이메일):** $0 (100통/일) ~ $20
- **도메인:** $10/년
- **SSL:** $0 (Let's Encrypt)

**총 예상 비용:** $25 ~ $100/월

---

## 8. 결론

### 현재 틀의 강점
- ✅ 빠른 프로토타이핑 가능
- ✅ 기본 CRUD 구조 완성
- ✅ 백엔드/프론트엔드 분리 아키텍처
- ✅ 에이전트 시스템으로 효율적 개발

### 쇼핑몰 구현을 위한 핵심 추가 사항
1. **PostgreSQL 전환** (SQLite 대체)
2. **인증/인가 시스템** (BE-auth, FE-auth)
3. **결제 연동** (BE-payment)
4. **파일 업로드** (BE-file-upload)
5. **장바구니 상태 관리** (FE-cart)
6. **관리자 페이지** (FE-admin)

### 추천 개발 순서
```
1. DB 전환 (PostgreSQL)
2. 인증 시스템 구축
3. 상품 관리 (이미지 포함)
4. 장바구니 기능
5. 주문서 작성
6. 결제 연동
7. 관리자 페이지
8. 리뷰/통계 등 부가 기능
```

**최소 구매 가능 버전(MVP)까지:** 2-3주
**완성도 높은 쇼핑몰:** 2-3개월

---

---

## 9. 쇼핑몰 핵심 기능만 추리기 (MVP 우선순위)

### ⭐⭐⭐ Phase 1: 필수 기능 (2-3주)

#### 1. 상품 관리
- 상품 CRUD
- 이미지 업로드 (1-3장)
- 재고 수량 표시
- 카테고리 분류

#### 2. 사용자 인증
- 회원가입/로그인
- JWT 토큰 발급
- 비밀번호 해싱
- 권한 구분 (일반/관리자)

#### 3. 장바구니
- 상품 담기/삭제
- 수량 조절
- 합계 금액 계산
- 회원: DB 저장 / 비회원: localStorage

#### 4. 주문/결제
- 주문서 작성
- 배송지 입력
- 결제 연동 (Toss Payments)
- 주문 확인 페이지

#### 5. 관리자 페이지
- 상품 등록/수정
- 주문 관리 (상태 변경)
- 간단한 통계 (총 매출, 주문 수)

---

### ⭐⭐ Phase 2: 개선 기능 (나중에)

- 검색/필터 (상품명, 카테고리, 가격)
- 상품 정렬 (최신순, 가격순)

---

### ⭐ Phase 3: 선택 기능 (여유 있을 때)

- 리뷰/평점 시스템
- 위시리스트
- 쿠폰/프로모션
- 이메일 알림

---

## 10. 이미지 호스팅 방법 상세 비교

### 1️⃣ 로컬 저장 (개발/테스트용)

**장점:**
- 무료
- 외부 서비스 불필요
- 빠른 구현

**단점:**
- 서버 재시작 시 손실 가능
- 스케일링 불가
- CDN 없음
- 프로덕션 부적합

**구현 코드:**

```python
# backend/app/routers/upload.py
from fastapi import APIRouter, UploadFile, File, HTTPException
from pathlib import Path
import shutil
import uuid

router = APIRouter()
UPLOAD_DIR = Path("uploads/products")
UPLOAD_DIR.mkdir(parents=True, exist_ok=True)

@router.post("/upload")
async def upload_image(file: UploadFile = File(...)):
    # 파일 확장자 검증
    allowed_extensions = {".jpg", ".jpeg", ".png", ".webp"}
    file_ext = Path(file.filename).suffix.lower()

    if file_ext not in allowed_extensions:
        raise HTTPException(400, "이미지 파일만 업로드 가능합니다")

    # 고유 파일명 생성
    unique_filename = f"{uuid.uuid4()}{file_ext}"
    file_path = UPLOAD_DIR / unique_filename

    # 파일 저장
    with open(file_path, "wb") as buffer:
        shutil.copyfileobj(file.file, buffer)

    return {"url": f"/uploads/products/{unique_filename}"}

# main.py에서 static files 설정
from fastapi.staticfiles import StaticFiles
app.mount("/uploads", StaticFiles(directory="uploads"), name="uploads")
```

**사용 시기:** 초기 개발/테스트 단계만

---

### 2️⃣ Cloudinary (⭐ 추천)

**장점:**
- 무료 플랜 제공 (25GB 저장, 25GB 대역폭/월)
- 자동 이미지 최적화 (WebP 변환, 압축)
- 리사이징/크롭 자동
- CDN 포함 (전 세계 빠른 속도)
- 간단한 API

**단점:**
- 무료 플랜 초과 시 비용 발생

**가격:**
- Free: $0 (25GB 저장, 25GB 대역폭)
- Plus: $99/월 (100GB)
- Advanced: $249/월 (500GB)

**설치:**
```bash
pip install cloudinary
```

**구현 코드:**

```python
# backend/app/config.py
import cloudinary
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    CLOUDINARY_CLOUD_NAME: str
    CLOUDINARY_API_KEY: str
    CLOUDINARY_API_SECRET: str

settings = Settings()

cloudinary.config(
    cloud_name=settings.CLOUDINARY_CLOUD_NAME,
    api_key=settings.CLOUDINARY_API_KEY,
    api_secret=settings.CLOUDINARY_API_SECRET
)

# backend/app/routers/upload.py
from cloudinary.uploader import upload, destroy

@router.post("/products/upload")
async def upload_product_image(file: UploadFile):
    try:
        # Cloudinary에 업로드
        result = upload(
            file.file,
            folder="products",
            transformation=[
                {"width": 1000, "crop": "limit"},  # 최대 1000px
                {"quality": "auto:good"}  # 자동 최적화
            ]
        )

        return {
            "url": result["secure_url"],
            "public_id": result["public_id"],
            "width": result["width"],
            "height": result["height"]
        }
    except Exception as e:
        raise HTTPException(500, f"업로드 실패: {str(e)}")

@router.delete("/products/image/{public_id}")
async def delete_image(public_id: str):
    """이미지 삭제"""
    result = destroy(public_id)
    return {"result": result}
```

**환경 변수 (.env):**
```bash
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
```

**프론트엔드 사용:**
```tsx
// 이미지 자동 최적화 URL
<img
  src={productImageUrl}
  alt="상품 이미지"
  loading="lazy"
/>

// 리사이징 URL 생성
const thumbnailUrl = productImageUrl.replace(
  '/upload/',
  '/upload/w_300,h_300,c_fill/'
);
```

**회원가입:** https://cloudinary.com/users/register/free

**사용 시기:** MVP부터 프로덕션까지 전 단계

---

### 3️⃣ Vercel Blob (Next.js 사용 시)

**장점:**
- Vercel과 완벽한 통합
- 엣지 네트워크 (빠름)
- 간단한 API

**단점:**
- 무료 플랜 없음
- Vercel 종속

**가격:**
- $0.15/GB 저장
- $0.30/GB 대역폭

**설치:**
```bash
npm install @vercel/blob
```

**구현 (Next.js API Route):**
```typescript
// frontend/src/app/api/upload/route.ts
import { put } from '@vercel/blob';
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  const form = await request.formData();
  const file = form.get('file') as File;

  if (!file) {
    return NextResponse.json({ error: 'No file' }, { status: 400 });
  }

  const blob = await put(file.name, file, {
    access: 'public',
    token: process.env.BLOB_READ_WRITE_TOKEN,
  });

  return NextResponse.json({ url: blob.url });
}

// 사용 예시
async function uploadImage(file: File) {
  const formData = new FormData();
  formData.append('file', file);

  const res = await fetch('/api/upload', {
    method: 'POST',
    body: formData,
  });

  const { url } = await res.json();
  return url;
}
```

**사용 시기:** Vercel에 배포하고 Next.js 중심 개발 시

---

### 4️⃣ AWS S3 (프로덕션급, 대규모)

**장점:**
- 무제한 확장성
- 저렴한 비용 ($0.023/GB)
- 높은 안정성 (99.999999999%)
- CloudFront CDN 연동 가능

**단점:**
- 설정 복잡 (IAM, Bucket Policy)
- 이미지 최적화 별도 작업 필요

**가격:**
- 스토리지: $0.023/GB/월
- 데이터 전송: $0.09/GB (처음 10TB)
- PUT 요청: $0.005/1000건

**설치:**
```bash
pip install boto3
```

**구현 코드:**

```python
# backend/app/services/s3.py
import boto3
from botocore.exceptions import ClientError
from app.config import settings

s3_client = boto3.client(
    's3',
    aws_access_key_id=settings.AWS_ACCESS_KEY_ID,
    aws_secret_access_key=settings.AWS_SECRET_ACCESS_KEY,
    region_name=settings.AWS_REGION
)

async def upload_to_s3(file: UploadFile, folder: str = "products"):
    try:
        # 고유 파일명 생성
        file_ext = Path(file.filename).suffix
        unique_filename = f"{folder}/{uuid.uuid4()}{file_ext}"

        # S3 업로드
        s3_client.upload_fileobj(
            file.file,
            settings.S3_BUCKET_NAME,
            unique_filename,
            ExtraArgs={
                'ACL': 'public-read',
                'ContentType': file.content_type
            }
        )

        # URL 생성
        url = f"https://{settings.S3_BUCKET_NAME}.s3.{settings.AWS_REGION}.amazonaws.com/{unique_filename}"
        return url

    except ClientError as e:
        raise HTTPException(500, f"S3 업로드 실패: {str(e)}")

# 이미지 삭제
async def delete_from_s3(file_key: str):
    s3_client.delete_object(
        Bucket=settings.S3_BUCKET_NAME,
        Key=file_key
    )
```

**환경 변수:**
```bash
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=ap-northeast-2
S3_BUCKET_NAME=your-bucket-name
```

**S3 Bucket 설정:**
1. AWS 콘솔에서 S3 버킷 생성
2. 퍼블릭 액세스 허용 설정
3. CORS 설정:
```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "POST", "PUT", "DELETE"],
    "AllowedOrigins": ["*"],
    "ExposeHeaders": []
  }
]
```

**사용 시기:** 트래픽 많은 프로덕션, 비용 최적화 필요 시

---

### 5️⃣ Supabase Storage

**장점:**
- Supabase 사용 시 DB와 통합
- 무료 플랜 1GB
- 간단한 API

**단점:**
- Supabase 종속
- 이미지 최적화 없음

**가격:**
- Free: 1GB
- Pro: $25/월 (100GB)

**설치:**
```bash
pip install supabase
```

**구현:**
```python
from supabase import create_client

supabase = create_client(
    settings.SUPABASE_URL,
    settings.SUPABASE_KEY
)

@router.post("/upload")
async def upload_image(file: UploadFile):
    file_bytes = await file.read()
    unique_filename = f"{uuid.uuid4()}{Path(file.filename).suffix}"

    # 업로드
    supabase.storage.from_('products').upload(
        unique_filename,
        file_bytes
    )

    # Public URL 생성
    url = supabase.storage.from_('products').get_public_url(unique_filename)
    return {"url": url}
```

**사용 시기:** Supabase를 DB로 사용하는 경우

---

## 11. 이미지 호스팅 최종 추천

### 개발/테스트 단계
→ **로컬 저장** (무료, 빠른 구현)

### MVP 출시
→ **Cloudinary 무료 플랜** (25GB면 충분, 자동 최적화)

### 프로덕션 (트래픽 적음)
→ **Cloudinary 유료 플랜** ($99/월, 관리 편함)

### 프로덕션 (트래픽 많음, 비용 최적화)
→ **AWS S3 + CloudFront** (저렴, 확장성 높음)

### Next.js + Vercel 배포
→ **Vercel Blob** (통합 간편)

### PostgreSQL + Supabase
→ **Supabase Storage** (DB와 통합)

---

## 12. Cloudinary 시작 가이드 (추천)

### Step 1: 회원가입
1. https://cloudinary.com/users/register/free 접속
2. 이메일 회원가입 (Google 연동 가능)
3. Dashboard에서 정보 확인:
   - Cloud Name
   - API Key
   - API Secret

### Step 2: 백엔드 설정
```bash
# 설치
pip install cloudinary

# .env 파일 생성
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
```

### Step 3: 코드 적용
위의 Cloudinary 구현 코드 참고

### Step 4: 테스트
Swagger UI (localhost:8000/docs)에서 이미지 업로드 테스트

### Step 5: 프론트엔드 연동
```tsx
const handleImageUpload = async (file: File) => {
  const formData = new FormData();
  formData.append('file', file);

  const res = await fetch('/api/products/upload', {
    method: 'POST',
    body: formData,
  });

  const { url } = await res.json();
  console.log('업로드된 이미지 URL:', url);
};
```

---

**작성자:** Claude Sonnet 4.5
**작성일:** 2026-02-05
**최종 수정:** 2026-02-05
