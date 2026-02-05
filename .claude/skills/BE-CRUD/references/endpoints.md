# CRUD 엔드포인트

```python
# app/routers/todos.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import List
from app.database import get_db
from app.models.todo import Todo
from app.schemas.todo import TodoCreate, TodoResponse

router = APIRouter(prefix="/todos", tags=["todos"])

@router.post("", response_model=TodoResponse)
def create(todo: TodoCreate, db: Session = Depends(get_db)):
    db_todo = Todo(**todo.model_dump())
    db.add(db_todo)
    db.commit()
    db.refresh(db_todo)
    return db_todo

@router.get("", response_model=List[TodoResponse])
def read_all(db: Session = Depends(get_db)):
    return db.query(Todo).all()

@router.get("/{id}", response_model=TodoResponse)
def read_one(id: int, db: Session = Depends(get_db)):
    todo = db.query(Todo).filter(Todo.id == id).first()
    if not todo:
        raise HTTPException(404, "Not found")
    return todo

@router.put("/{id}", response_model=TodoResponse)
def update(id: int, todo: TodoCreate, db: Session = Depends(get_db)):
    db_todo = db.query(Todo).filter(Todo.id == id).first()
    if not db_todo:
        raise HTTPException(404, "Not found")
    for k, v in todo.model_dump().items():
        setattr(db_todo, k, v)
    db.commit()
    db.refresh(db_todo)
    return db_todo

@router.delete("/{id}")
def delete(id: int, db: Session = Depends(get_db)):
    db_todo = db.query(Todo).filter(Todo.id == id).first()
    if not db_todo:
        raise HTTPException(404, "Not found")
    db.delete(db_todo)
    db.commit()
    return {"message": "Deleted"}
```

**main.py에 라우터 등록**: `app.include_router(todos.router)`