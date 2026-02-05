# 네이밍 컨벤션

## Python 기본 규칙

| 대상 | 스타일 | 예시 |
|------|--------|------|
| 변수 | snake_case | `user_name`, `total_count` |
| 함수 | snake_case | `get_user()`, `create_item()` |
| 클래스 | PascalCase | `User`, `TodoItem` |
| 상수 | UPPER_SNAKE | `MAX_LIMIT`, `DEFAULT_PAGE` |

## FastAPI 엔드포인트 네이밍
```python
# ✅ Good
@app.get("/users")
def get_users(): ...

@app.get("/users/{user_id}")
def get_user(user_id: int): ...

@app.post("/users")
def create_user(): ...

@app.put("/users/{user_id}")
def update_user(user_id: int): ...

@app.delete("/users/{user_id}")
def delete_user(user_id: int): ...

# ❌ Bad
@app.get("/getUsers")      # 동사 중복
@app.get("/user_list")     # 언더스코어 사용
@app.post("/createUser")   # camelCase URL
```

## SQLAlchemy 모델 네이밍
```python
# ✅ Good
class User(Base):
    __tablename__ = "users"  # 복수형
    
    id = Column(Integer, primary_key=True)
    email = Column(String)
    created_at = Column(DateTime)

# ❌ Bad
class user(Base):           # 소문자 클래스명
    __tablename__ = "User"  # 대문자 테이블명
```

## Pydantic 스키마 네이밍
```python
# ✅ Good - 용도별 구분
class UserBase(BaseModel): ...
class UserCreate(UserBase): ...
class UserUpdate(UserBase): ...
class UserResponse(UserBase): ...

# ❌ Bad
class UserDTO(BaseModel): ...      # 모호함
class UserModel(BaseModel): ...    # SQLAlchemy와 혼동
```

## 변수명 개선 예시
```python
# ❌ Bad
d = db.query(User).all()
x = item.model_dump()
res = {"status": "ok"}

# ✅ Good
users = db.query(User).all()
item_dict = item.model_dump()
response = {"status": "ok"}
```