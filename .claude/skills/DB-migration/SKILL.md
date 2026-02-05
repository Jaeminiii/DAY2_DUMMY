---
name: DB-migration
description: Alembic을 사용한 데이터베이스 스키마 마이그레이션 관리
context: fork
agent: be-agent
---

# DB 마이그레이션 가이드

## 개요
Alembic을 사용하여 데이터베이스 스키마 변경을 버전 관리합니다.

## 필요성
- 스키마 변경 이력 추적
- 팀 협업 시 DB 동기화
- 롤백 가능
- 프로덕션 배포 시 안전한 스키마 업데이트

## 설치
```bash
pip install alembic
```

## 초기 설정

### 1. Alembic 초기화
```bash
cd backend
alembic init alembic
```

**생성되는 구조:**
```
backend/
├── alembic/
│   ├── versions/         # 마이그레이션 파일들
│   ├── env.py           # Alembic 환경 설정
│   └── script.py.mako   # 마이그레이션 템플릿
├── alembic.ini          # Alembic 설정 파일
└── app/
    ├── models/
    └── database.py
```

### 2. alembic.ini 수정
```ini
# alembic.ini
[alembic]
script_location = alembic

# 데이터베이스 URL (환경 변수 사용)
# sqlalchemy.url = driver://user:pass@localhost/dbname
# ↓ 주석 처리하고 env.py에서 설정
```

### 3. alembic/env.py 수정
```python
# alembic/env.py
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context

# 모델과 설정 import
from app.database import Base
from app.config import settings
import app.models  # 모든 모델 import

# Alembic Config 객체
config = context.config

# 환경 변수에서 DATABASE_URL 가져오기
config.set_main_option("sqlalchemy.url", settings.DATABASE_URL)

# 로깅 설정
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

# 메타데이터 설정
target_metadata = Base.metadata

def run_migrations_offline() -> None:
    """오프라인 모드"""
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()

def run_migrations_online() -> None:
    """온라인 모드"""
    connectable = engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata
        )

        with context.begin_transaction():
            context.run_migrations()

if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

## 마이그레이션 명령어

### 1. 마이그레이션 파일 생성
```bash
# 자동 생성 (모델 변경 감지)
alembic revision --autogenerate -m "설명"

# 예시
alembic revision --autogenerate -m "Add product table"
alembic revision --autogenerate -m "Add user email column"
```

### 2. 마이그레이션 적용
```bash
# 최신 버전으로 업그레이드
alembic upgrade head

# 특정 버전으로 업그레이드
alembic upgrade <revision_id>

# 한 단계 업그레이드
alembic upgrade +1
```

### 3. 마이그레이션 롤백
```bash
# 한 단계 다운그레이드
alembic downgrade -1

# 특정 버전으로 다운그레이드
alembic downgrade <revision_id>

# 모두 롤백 (초기화)
alembic downgrade base
```

### 4. 현재 상태 확인
```bash
# 현재 마이그레이션 버전 확인
alembic current

# 마이그레이션 히스토리 확인
alembic history

# 대기 중인 마이그레이션 확인
alembic heads
```

## 마이그레이션 파일 예시

### 생성된 파일 (versions/xxx_add_product_table.py)
```python
"""Add product table

Revision ID: abc123
Revises:
Create Date: 2026-02-05 10:00:00
"""
from alembic import op
import sqlalchemy as sa

# revision identifiers
revision = 'abc123'
down_revision = None
branch_labels = None
depends_on = None

def upgrade() -> None:
    """마이그레이션 적용"""
    op.create_table(
        'products',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('name', sa.String(length=200), nullable=False),
        sa.Column('price', sa.Numeric(precision=10, scale=2), nullable=False),
        sa.Column('stock', sa.Integer(), nullable=True),
        sa.Column('created_at', sa.DateTime(), nullable=False),
        sa.PrimaryKeyConstraint('id')
    )
    op.create_index(op.f('ix_products_id'), 'products', ['id'], unique=False)

def downgrade() -> None:
    """마이그레이션 롤백"""
    op.drop_index(op.f('ix_products_id'), table_name='products')
    op.drop_table('products')
```

## 일반적인 마이그레이션 작업

### 컬럼 추가
```python
def upgrade():
    op.add_column('products', sa.Column('description', sa.Text(), nullable=True))

def downgrade():
    op.drop_column('products', 'description')
```

### 컬럼 수정
```python
def upgrade():
    op.alter_column('products', 'price',
                    existing_type=sa.Numeric(10, 2),
                    type_=sa.Numeric(12, 2),
                    nullable=False)

def downgrade():
    op.alter_column('products', 'price',
                    existing_type=sa.Numeric(12, 2),
                    type_=sa.Numeric(10, 2),
                    nullable=False)
```

### 컬럼 삭제
```python
def upgrade():
    op.drop_column('products', 'old_column')

def downgrade():
    op.add_column('products', sa.Column('old_column', sa.String(100)))
```

### 인덱스 추가
```python
def upgrade():
    op.create_index('ix_products_name', 'products', ['name'])

def downgrade():
    op.drop_index('ix_products_name', table_name='products')
```

### 외래 키 추가
```python
def upgrade():
    op.create_foreign_key(
        'fk_products_category',
        'products', 'categories',
        ['category_id'], ['id'],
        ondelete='CASCADE'
    )

def downgrade():
    op.drop_constraint('fk_products_category', 'products', type_='foreignkey')
```

## 워크플로우

### 1. 모델 변경
```python
# app/models/product.py
class Product(Base):
    __tablename__ = "products"

    id = Column(Integer, primary_key=True)
    name = Column(String(200), nullable=False)
    # 새로운 컬럼 추가
    description = Column(Text, nullable=True)
```

### 2. 마이그레이션 생성
```bash
alembic revision --autogenerate -m "Add product description"
```

### 3. 마이그레이션 파일 확인
```python
# alembic/versions/xxx_add_product_description.py 확인
# upgrade(), downgrade() 함수 검토
```

### 4. 마이그레이션 적용
```bash
alembic upgrade head
```

### 5. 확인
```bash
alembic current
```

## 주의사항

### ✅ 해야 할 것
- 마이그레이션 파일은 git에 커밋
- 프로덕션 배포 전 테스트 환경에서 먼저 테스트
- downgrade() 함수도 작성 (롤백 가능하도록)
- 마이그레이션 파일 생성 후 내용 검토

### ❌ 하지 말아야 할 것
- 이미 적용된 마이그레이션 파일 수정 금지
- 데이터 손실 가능성 있는 작업은 백업 후 진행
- 프로덕션 DB에 직접 마이그레이션 금지 (스테이징 테스트 후)

## 데이터 마이그레이션

### 데이터 변환 예시
```python
def upgrade():
    # 1. 컬럼 추가
    op.add_column('products', sa.Column('slug', sa.String(200)))

    # 2. 데이터 변환
    connection = op.get_bind()
    products = connection.execute("SELECT id, name FROM products").fetchall()

    for product_id, name in products:
        slug = name.lower().replace(' ', '-')
        connection.execute(
            f"UPDATE products SET slug = '{slug}' WHERE id = {product_id}"
        )

    # 3. NOT NULL 제약 추가
    op.alter_column('products', 'slug', nullable=False)

def downgrade():
    op.drop_column('products', 'slug')
```

## 팀 협업

### 충돌 해결
1. 팀원의 마이그레이션 pull
2. 자신의 마이그레이션 생성
3. 순서 확인 (down_revision)
4. 충돌 시 수동으로 down_revision 수정

### 브랜치 병합
```bash
# 여러 head가 있는 경우 병합
alembic merge heads -m "Merge migrations"
```

## 프로덕션 배포

### 안전한 배포 절차
```bash
# 1. 백업
pg_dump dbname > backup.sql

# 2. 마이그레이션 적용 (dry-run)
alembic upgrade head --sql > migration.sql
# migration.sql 파일 검토

# 3. 마이그레이션 적용
alembic upgrade head

# 4. 확인
alembic current

# 5. 문제 발생 시 롤백
alembic downgrade -1
```

## 참조 문서
- [setup.md](references/setup.md) - 초기 설정 상세
- [commands.md](references/commands.md) - 명령어 참고
- [best-practices.md](references/best-practices.md) - 모범 사례
