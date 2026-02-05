# Fetch 함수

## 기본 API 함수
```tsx
// lib/api.ts
export async function fetchApi<T>(endpoint: string): Promise<T> {
  const res = await fetch(`/api${endpoint}`);
  if (!res.ok) throw new Error('API 호출 실패');
  return res.json();
}

export async function postApi<T>(endpoint: string, data: object): Promise<T> {
  const res = await fetch(`/api${endpoint}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  });
  if (!res.ok) throw new Error('API 호출 실패');
  return res.json();
}

export async function putApi<T>(endpoint: string, data: object): Promise<T> {
  const res = await fetch(`/api${endpoint}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  });
  if (!res.ok) throw new Error('API 호출 실패');
  return res.json();
}

export async function deleteApi(endpoint: string): Promise<void> {
  const res = await fetch(`/api${endpoint}`, { method: 'DELETE' });
  if (!res.ok) throw new Error('API 호출 실패');
}
```

## 사용 예시
```tsx
// GET
const users = await fetchApi<User[]>('/users');

// POST
const newUser = await postApi<User>('/users', { name: '홍길동', email: 'hong@test.com' });

// PUT
const updated = await putApi<User>('/users/1', { name: '김철수' });

// DELETE
await deleteApi('/users/1');
```
