# HTTP 상태 코드 에러

## 404 Not Found
```json
{"detail": "Not Found"}
```
**원인 1**: URL 경로 오류
**확인**: 요청 URL과 라우터 경로 일치 여부

**원인 2**: 라우터 미등록
**해결**: `main.py`에 라우터 등록
```python
app.include_router(todos.router)
```

**원인 3**: DB에서 데이터 없음
**확인**: 해당 ID의 데이터 존재 여부

## 405 Method Not Allowed
```json
{"detail": "Method Not Allowed"}
```
**원인**: 잘못된 HTTP 메서드
**확인**: GET/POST/PUT/DELETE 일치 여부

## 422 Unprocessable Entity
```json
{"detail": [{"loc": [...], "msg": "...", "type": "..."}]}
```
**원인**: 요청 데이터 검증 실패
**확인**: [validation-errors.md](validation-errors.md) 참조

## 500 Internal Server Error
```json
{"detail": "Internal Server Error"}
```
**원인**: 서버 코드 에러
**확인**: 터미널 로그에서 스택트레이스 확인
```bash
uvicorn app.main:app --reload  # 실시간 에러 확인
```
