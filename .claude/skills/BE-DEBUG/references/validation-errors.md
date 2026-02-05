# Pydantic 검증 에러

## field required
```json
{"detail": [{"loc": ["body", "title"], "msg": "field required", "type": "value_error.missing"}]}
```
**원인**: 필수 필드 누락
**해결**: 요청 body에 필드 추가 또는 스키마에서 Optional 처리
```python
title: Optional[str] = None
```

## value is not a valid integer
```json
{"detail": [{"loc": ["body", "price"], "msg": "value is not a valid integer"}]}
```
**원인**: 타입 불일치 (문자열 → 정수)
**해결**: 요청 데이터 타입 확인

## extra fields not permitted
```json
{"detail": [{"loc": ["body", "extra_field"], "msg": "extra fields not permitted"}]}
```
**원인**: 스키마에 없는 필드 전송
**해결**: 요청 body에서 불필요한 필드 제거 또는 스키마에 필드 추가

## Config 에러
```
ConfigError: unable to infer type for attribute
```
**원인**: Pydantic 모델 필드에 타입 힌트 누락
**해결**: 모든 필드에 타입 명시
```python
class Todo(BaseModel):
    title: str  # 타입 필수
```
