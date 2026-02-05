# API 호출

## 기본 fetch 함수
```tsx
// lib/api.ts
const API_BASE = '/api';

export async function fetchApi<T>(endpoint: string): Promise<T> {
  const res = await fetch(`${API_BASE}${endpoint}`);
  if (!res.ok) throw new Error('API 호출 실패');
  return res.json();
}

export async function postApi<T>(endpoint: string, data: object): Promise<T> {
  const res = await fetch(`${API_BASE}${endpoint}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  });
  if (!res.ok) throw new Error('API 호출 실패');
  return res.json();
}

export async function putApi<T>(endpoint: string, data: object): Promise<T> {
  const res = await fetch(`${API_BASE}${endpoint}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data),
  });
  if (!res.ok) throw new Error('API 호출 실패');
  return res.json();
}

export async function deleteApi(endpoint: string): Promise<void> {
  const res = await fetch(`${API_BASE}${endpoint}`, { method: 'DELETE' });
  if (!res.ok) throw new Error('API 호출 실패');
}
```

## 사용 예시
```tsx
// CRUD 호출
const todos = await fetchApi<Todo[]>('/todos');
const newTodo = await postApi<Todo>('/todos', { title: '새 할일' });
const updated = await putApi<Todo>('/todos/1', { title: '수정', completed: true });
await deleteApi('/todos/1');
```

## 프록시 설정
`/api/*` 요청은 `next.config.js`의 rewrites로 백엔드(localhost:8000)로 프록시됩니다.
