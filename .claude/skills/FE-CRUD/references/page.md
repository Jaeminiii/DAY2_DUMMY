# 페이지 구조

## 목록 페이지
```tsx
// app/todos/page.tsx
'use client';
import { useEffect, useState } from 'react';

interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

export default function TodosPage() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/todos')
      .then(res => res.json())
      .then(setTodos)
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <div>로딩중...</div>;

  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold mb-4">Todo 목록</h1>
      <ul className="space-y-2">
        {todos.map(todo => (
          <li key={todo.id} className="p-2 border rounded">
            {todo.title}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## 상세 페이지
```tsx
// app/todos/[id]/page.tsx
'use client';
import { useEffect, useState } from 'react';
import { useParams } from 'next/navigation';

export default function TodoDetailPage() {
  const { id } = useParams();
  const [todo, setTodo] = useState<Todo | null>(null);

  useEffect(() => {
    fetch(`/api/todos/${id}`)
      .then(res => res.json())
      .then(setTodo);
  }, [id]);

  if (!todo) return <div>로딩중...</div>;

  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold">{todo.title}</h1>
      <p>완료: {todo.completed ? '예' : '아니오'}</p>
    </div>
  );
}
```

## App Router 규칙
- `page.tsx`: 해당 경로의 페이지
- `layout.tsx`: 공통 레이아웃
- `[param]`: 동적 라우트
