# 폼 처리

## 생성 폼
```tsx
// components/TodoForm.tsx
'use client';
import { useState } from 'react';

interface TodoFormProps {
  onSubmit: (title: string) => void;
}

export default function TodoForm({ onSubmit }: TodoFormProps) {
  const [title, setTitle] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!title.trim()) return;
    onSubmit(title);
    setTitle('');
  };

  return (
    <form onSubmit={handleSubmit} className="flex gap-2">
      <input
        type="text"
        value={title}
        onChange={e => setTitle(e.target.value)}
        placeholder="할 일 입력"
        className="flex-1 px-3 py-2 border rounded"
      />
      <button
        type="submit"
        className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
      >
        추가
      </button>
    </form>
  );
}
```

## 페이지에서 사용
```tsx
// app/todos/page.tsx
'use client';
import { useState } from 'react';
import TodoForm from '@/components/TodoForm';

export default function TodosPage() {
  const [todos, setTodos] = useState<Todo[]>([]);

  const handleCreate = async (title: string) => {
    const res = await fetch('/api/todos', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ title, completed: false }),
    });
    const newTodo = await res.json();
    setTodos([...todos, newTodo]);
  };

  return (
    <div className="p-4">
      <TodoForm onSubmit={handleCreate} />
      {/* 목록 렌더링 */}
    </div>
  );
}
```

## 입력 필드 스타일
```tsx
// 공통 입력 스타일
className="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
```
