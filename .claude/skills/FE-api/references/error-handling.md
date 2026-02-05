# 에러 처리

## 기본 에러 처리
```tsx
try {
  const data = await fetchApi<User[]>('/users');
  setUsers(data);
} catch (error) {
  console.error('API 에러:', error);
  setError('데이터를 불러오지 못했습니다');
}
```

## 상태 코드별 처리
```tsx
export async function fetchApi<T>(endpoint: string): Promise<T> {
  const res = await fetch(`/api${endpoint}`);

  if (!res.ok) {
    if (res.status === 404) throw new Error('데이터를 찾을 수 없습니다');
    if (res.status === 401) throw new Error('로그인이 필요합니다');
    if (res.status === 500) throw new Error('서버 오류가 발생했습니다');
    throw new Error('API 호출 실패');
  }

  return res.json();
}
```

## 컴포넌트에서 에러 표시
```tsx
'use client';
import { useState, useEffect } from 'react';

export default function UsersPage() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetch('/api/users')
      .then(res => {
        if (!res.ok) throw new Error('API 호출 실패');
        return res.json();
      })
      .then(setUsers)
      .catch(e => setError(e.message))
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <div className="p-4">로딩중...</div>;
  if (error) return <div className="p-4 text-red-500">에러: {error}</div>;

  return <ul>{/* ... */}</ul>;
}
```

## 재시도 패턴
```tsx
const [retryCount, setRetryCount] = useState(0);

const retry = () => {
  setError(null);
  setRetryCount(c => c + 1);
};

useEffect(() => {
  fetchData();
}, [retryCount]);

// 에러 시
if (error) return (
  <div>
    <p>에러: {error}</p>
    <button onClick={retry}>다시 시도</button>
  </div>
);
```
