# CRUD 테스트 템플릿

## Create 테스트
```python
def test_create_item(client):
    response = client.post("/items", json={"name": "Test", "price": 100})
    assert response.status_code == 200
    assert response.json()["name"] == "Test"
```

## Read 테스트
```python
def test_get_items(client):
    response = client.get("/items")
    assert response.status_code == 200
    assert isinstance(response.json(), list)

def test_get_item_not_found(client):
    response = client.get("/items/9999")
    assert response.status_code == 404
```

## Update 테스트
```python
def test_update_item(client):
    # 생성
    create_res = client.post("/items", json={"name": "Old", "price": 100})
    item_id = create_res.json()["id"]

    # 수정
    update_res = client.put(f"/items/{item_id}", json={"name": "New", "price": 200})
    assert update_res.status_code == 200
    assert update_res.json()["name"] == "New"
```

## Delete 테스트
```python
def test_delete_item(client):
    # 생성
    create_res = client.post("/items", json={"name": "Delete", "price": 100})
    item_id = create_res.json()["id"]

    # 삭제
    delete_res = client.delete(f"/items/{item_id}")
    assert delete_res.status_code == 200

    # 확인
    get_res = client.get(f"/items/{item_id}")
    assert get_res.status_code == 404
```

## 에러 케이스 테스트
```python
def test_create_invalid_data(client):
    response = client.post("/items", json={"name": ""})  # price 누락
    assert response.status_code == 422
```
