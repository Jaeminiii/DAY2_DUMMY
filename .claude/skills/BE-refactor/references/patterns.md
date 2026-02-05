# FastAPI 패턴

## 의존성 주입 활용

### 공통 쿼리 파라미터
```python
from fastapi import Query

def pagination_params(
    skip: int = Query(0, ge=0),
    limit: int = Query(20, ge=1, le=100)
):
    return {"skip": skip, "limit": limit}

@app.get("/users")
def get_users(
    pagination: dict = Depends(pagination_params),
    db: Session = Depends(get_db)
):
    return db.query(User).offset(pagination["skip"]).limit(pagination["limit"]).all()
```

## 응답 모델 명시

### Before ❌
```python
@app.get("/users")
def get_users(db: Session = Depends(get_db)):
    return db.query(User).all()  # 어떤 형태로 반환되는지 불명확
```

### After ✅
```python
@app.get("/users", response_model=List[UserResponse])
def get_users(db: Session = Depends(get_db)) -> List[User]:
    return db.query(User).all()
```

## 에러 응답 통일
```python
from fastapi import HTTPException

# 공통 에러 함수
def not_found(resource: str = "Resource"):
    raise HTTPException(status_code=404, detail=f"{resource} not found")

def bad_request(message: str):
    raise HTTPException(status_code=400, detail=message)

# 사용
@app.get("/users/{user_id}")
def get_user(user_id: int, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        not_found("User")
    return user
```

## 타입 힌트 추가

### Before ❌
```python
def create_item(item, db):
    db_item = Item(**item.model_dump())
    db.add(db_item)
    db.commit()
    return db_item
```

### After ✅
```python
def create_item(item: ItemCreate, db: Session) -> Item:
    db_item = Item(**item.model_dump())
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item
```

## 환경 변수 관리 (심화)
```python
# 초기 단계에서는 권장하지 않음
# 프로젝트가 커지면 적용

from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str = "sqlite:///./app.db"
    debug: bool = True
    
    class Config:
        env_file = ".env"

settings = Settings()
```