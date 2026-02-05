---
name: DB-model
description: SQLAlchemy 모델을 생성하거나 수정합니다
context: fork
agent: be-agent
---

# DB-model Skill

## 목적
데이터베이스 테이블에 대응하는 ORM 모델을 생성합니다.

## 입력
- 테이블 이름
- 컬럼 정의
- 관계 정의

## 출력
- `app/models.py` 업데이트

## 템플릿

### 기본 모델
```python
from sqlalchemy import Column, Integer, String, Boolean, DateTime, ForeignKey, Text
from sqlalchemy.orm import relationship
from datetime import datetime
from .database import Base

class Todo(Base):
    """TODO 항목 모델"""
    __tablename__ = "todos"
    
    # Primary Key
    id = Column(Integer, primary_key=True, index=True)
    
    # Foreign Keys
    user_id = Column(
        Integer,
        ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
        index=True,
        comment="작성자 ID"
    )
    
    # 데이터 컬럼
    title = Column(String(255), nullable=False, comment="제목")
    description = Column(Text, nullable=True, comment="상세 설명")
    completed = Column(Boolean, default=False, index=True, comment="완료 여부")
    priority = Column(Integer, default=0, comment="우선순위 (0-5)")
    
    # 타임스탬프 (필수)
    created_at = Column(
        DateTime,
        default=datetime.utcnow,
        nullable=False,
        comment="생성 시각"
    )
    updated_at = Column(
        DateTime,
        default=datetime.utcnow,
        onupdate=datetime.utcnow,
        nullable=False,
        comment="수정 시각"
    )
    
    # 관계
    user = relationship("User", back_populates="todos")
    
    def __repr__(self):
        return f"<Todo(id={self.id}, title='{self.title}', completed={self.completed})>"
```

## 관계 패턴

### One-to-Many (1:N)
```python
class User(Base):
    """사용자 모델"""
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String(255), unique=True, nullable=False)
    name = Column(String(100), nullable=False)
    
    # 관계: 한 사용자는 여러 TODO를 가짐
    todos = relationship(
        "Todo",
        back_populates="user",
        cascade="all, delete-orphan",  # 사용자 삭제 시 TODO도 삭제
        lazy="dynamic"  # 쿼리로만 로드
    )

class Todo(Base):
    """TODO 모델"""
    __tablename__ = "todos"
    
    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey("users.id", ondelete="CASCADE"))
    title = Column(String(255), nullable=False)
    
    # 관계: TODO는 한 사용자에게 속함
    user = relationship("User", back_populates="todos")
```

### Many-to-Many (N:M)
```python
from sqlalchemy import Table

# 연결 테이블
post_tags = Table(
    'post_tags',
    Base.metadata,
    Column('post_id', Integer, ForeignKey('posts.id', ondelete='CASCADE')),
    Column('tag_id', Integer, ForeignKey('tags.id', ondelete='CASCADE')),
    UniqueConstraint('post_id', 'tag_id', name='uix_post_tag')
)

class Post(Base):
    """게시글 모델"""
    __tablename__ = "posts"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String(255), nullable=False)
    
    # 관계: 게시글은 여러 태그를 가짐
    tags = relationship(
        "Tag",
        secondary=post_tags,
        back_populates="posts"
    )

class Tag(Base):
    """태그 모델"""
    __tablename__ = "tags"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(50), unique=True, nullable=False)
    
    # 관계: 태그는 여러 게시글에 사용됨
    posts = relationship(
        "Post",
        secondary=post_tags,
        back_populates="tags"
    )
```

### Self-Referencing (자기 참조)
```python
class Category(Base):
    """카테고리 모델 (트리 구조)"""
    __tablename__ = "categories"
    
    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    parent_id = Column(Integer, ForeignKey("categories.id"), nullable=True)
    
    # 관계
    parent = relationship(
        "Category",
        remote_side=[id],  # 부모 쪽 지정
        back_populates="children"
    )
    children = relationship(
        "Category",
        back_populates="parent",
        cascade="all, delete-orphan"
    )
```

## 컬럼 타입 참고

```python
from sqlalchemy import (
    Integer,      # 정수
    String,       # 문자열 (길이 지정 필수)
    Text,         # 긴 텍스트
    Boolean,      # True/False
    DateTime,     # 날짜/시간
    Date,         # 날짜
    Time,         # 시간
    Float,        # 실수
    Numeric,      # 정밀한 숫자 (금액 등)
    JSON,         # JSON 데이터
    Enum,         # 열거형
)

# 예시
class Product(Base):
    __tablename__ = "products"
    
    id = Column(Integer, primary_key=True)
    name = Column(String(200), nullable=False)
    description = Column(Text)
    price = Column(Numeric(10, 2), nullable=False)  # 10자리, 소수점 2자리
    stock = Column(Integer, default=0)
    is_active = Column(Boolean, default=True)
    metadata_json = Column(JSON, nullable=True)
    created_at = Column(DateTime, default=datetime.utcnow)
```
