# SQLAlchemy 모델

```python
# app/models/todo.py
from sqlalchemy import Column, Integer, String, Boolean
from app.database import Base

class Todo(Base):
    __tablename__ = "todos"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    completed = Column(Boolean, default=False)
```

**컬럼 옵션**: `unique=True`, `nullable=True`, `default=값`