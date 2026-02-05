# 커스텀 훅

## useFetch 훅
```tsx
// hooks/useFetch.ts
'use client';
import { useState, useEffect } from 'react';

interface UseFetchResult<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => void;
}

export function useFetch<T>(endpoint: string): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  const fetchData = async () => {
    setLoading(true);
    try {
      const res = await fetch(`/api${endpoint}`);
      if (!res.ok) throw new Error('API 호출 실패');
      setData(await res.json());
      setError(null);
    } catch (e) {
      setError(e instanceof Error ? e.message : '오류 발생');
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    fetchData();
  }, [endpoint]);

  return { data, loading, error, refetch: fetchData };
}
```

## 사용 예시
```tsx
// app/users/page.tsx
'use client';
import { useFetch } from '@/hooks/useFetch';

interface User {
  id: number;
  name: string;
}

export default function UsersPage() {
  const { data: users, loading, error, refetch } = useFetch<User[]>('/users');

  if (loading) return <div>로딩중...</div>;
  if (error) return <div>에러: {error}</div>;

  return (
    <div>
      <button onClick={refetch}>새로고침</button>
      <ul>
        {users?.map(user => <li key={user.id}>{user.name}</li>)}
      </ul>
    </div>
  );
}
```
