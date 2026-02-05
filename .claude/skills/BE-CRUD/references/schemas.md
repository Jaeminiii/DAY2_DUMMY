# Pydantic 스키마

```python
# app/schemas/todo.py
from pydantic import BaseModel

class TodoBase(BaseModel):
    title: str
    completed: bool = False

class TodoCreate(TodoBase):
    pass

class TodoResponse(TodoBase):
    id: int

    class Config:
        from_attributes = True
```

**패턴**: `Base`(공통) → `Create`(생성용) → `Response`(응답용, id 포함)