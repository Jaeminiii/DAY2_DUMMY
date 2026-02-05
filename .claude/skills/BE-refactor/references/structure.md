# 코드 구조 개선

## 중복 코드 제거

### Before ❌
```python
@app.get("/users/{user_id}")
def get_user(user_id: int, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@app.put("/users/{user_id}")
def update_user(user_id: int, data: UserUpdate, db: Session = Depends(get_db)):
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    # 업데이트 로직
```

### After ✅
```python
def get_user_or_404(db: Session, user_id: int) -> User:
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@app.get("/users/{user_id}")
def get_user(user_id: int, db: Session = Depends(get_db)):
    return get_user_or_404(db, user_id)

@app.put("/users/{user_id}")
def update_user(user_id: int, data: UserUpdate, db: Session = Depends(get_db)):
    user = get_user_or_404(db, user_id)
    # 업데이트 로직
```

## 함수 분리

### Before ❌
```python
@app.post("/orders")
def create_order(order: OrderCreate, db: Session = Depends(get_db)):
    # 유효성 검사
    if order.quantity <= 0:
        raise HTTPException(400, "Invalid quantity")
    
    # 재고 확인
    product = db.query(Product).filter(Product.id == order.product_id).first()
    if product.stock < order.quantity:
        raise HTTPException(400, "Not enough stock")
    
    # 주문 생성
    db_order = Order(**order.model_dump())
    db.add(db_order)
    
    # 재고 감소
    product.stock -= order.quantity
    
    db.commit()
    return db_order
```

### After ✅
```python
def validate_order(order: OrderCreate):
    if order.quantity <= 0:
        raise HTTPException(400, "Invalid quantity")

def check_stock(db: Session, product_id: int, quantity: int) -> Product:
    product = db.query(Product).filter(Product.id == product_id).first()
    if product.stock < quantity:
        raise HTTPException(400, "Not enough stock")
    return product

@app.post("/orders")
def create_order(order: OrderCreate, db: Session = Depends(get_db)):
    validate_order(order)
    product = check_stock(db, order.product_id, order.quantity)
    
    db_order = Order(**order.model_dump())
    db.add(db_order)
    product.stock -= order.quantity
    
    db.commit()
    return db_order
```

## 상수 분리

### Before ❌
```python
@app.get("/users")
def get_users(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    if limit > 100:
        limit = 100
    return db.query(User).offset(skip).limit(limit).all()
```

### After ✅
```python
MAX_PAGE_SIZE = 100
DEFAULT_PAGE_SIZE = 20

@app.get("/users")
def get_users(
    skip: int = 0,
    limit: int = DEFAULT_PAGE_SIZE,
    db: Session = Depends(get_db)
):
    limit = min(limit, MAX_PAGE_SIZE)
    return db.query(User).offset(skip).limit(limit).all()
```