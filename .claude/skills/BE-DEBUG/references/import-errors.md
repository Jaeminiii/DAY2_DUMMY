# Import/Module 에러

## ModuleNotFoundError
```
ModuleNotFoundError: No module named 'app'
```
**원인**: 잘못된 디렉토리에서 실행
**해결**: `backend/` 디렉토리에서 실행
```bash
cd backend && uvicorn app.main:app --reload
```

## ImportError: cannot import name
```
ImportError: cannot import name 'Todo' from 'app.models'
```
**원인**: `__init__.py`에 export 누락
**해결**: `app/models/__init__.py`에 추가
```python
from .todo import Todo
```

## circular import
```
ImportError: cannot import name 'X' (most likely due to circular import)
```
**원인**: A가 B를 import, B가 A를 import
**해결**: import를 함수 내부로 이동하거나 구조 변경
