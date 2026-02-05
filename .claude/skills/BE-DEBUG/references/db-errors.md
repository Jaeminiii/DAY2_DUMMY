# DB/SQLAlchemy 에러

## no such table
```
sqlalchemy.exc.OperationalError: no such table: todos
```
**원인**: 테이블 미생성
**해결**: `main.py`에 추가
```python
Base.metadata.create_all(bind=engine)
```

## UNIQUE constraint failed
```
sqlalchemy.exc.IntegrityError: UNIQUE constraint failed: users.email
```
**원인**: 중복 값 삽입 시도
**해결**: 삽입 전 중복 체크
```python
existing = db.query(User).filter(User.email == email).first()
if existing:
    raise HTTPException(400, "Email already exists")
```

## NOT NULL constraint failed
```
sqlalchemy.exc.IntegrityError: NOT NULL constraint failed: todos.title
```
**원인**: 필수 필드 누락
**해결**: 모델/스키마 필드 확인, nullable=True 또는 default 값 설정

## Object not bound to Session
```
sqlalchemy.orm.exc.DetachedInstanceError
```
**원인**: 세션 종료 후 객체 접근
**해결**: `db.refresh(obj)` 호출 또는 세션 내에서 처리
