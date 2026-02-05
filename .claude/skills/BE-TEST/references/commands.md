# 테스트 실행 명령어

## 기본 실행
```bash
cd backend
pytest
```

## 옵션
```bash
pytest -v                      # 상세 출력
pytest -s                      # print() 출력
pytest test/test_items.py      # 특정 파일
pytest -k "test_create"        # 이름 필터
```

## 커버리지
```bash
pip install pytest-cov
pytest --cov=app               # 커버리지 측정
pytest --cov=app --cov-report=html  # HTML 리포트
```

## 실패 시 디버깅
```bash
pytest -x                      # 첫 실패 시 중단
pytest --pdb                   # 실패 시 디버거 진입
pytest -v --tb=short           # 짧은 트레이스백
```
