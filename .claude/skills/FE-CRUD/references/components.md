# CRUD 컴포넌트

## 목록 아이템
```tsx
// components/TodoItem.tsx
interface TodoItemProps {
  todo: { id: number; title: string; completed: boolean };
  onDelete: (id: number) => void;
}

export default function TodoItem({ todo, onDelete }: TodoItemProps) {
  return (
    <div className="flex justify-between items-center p-3 border rounded">
      <span className={todo.completed ? 'line-through text-gray-400' : ''}>
        {todo.title}
      </span>
      <button
        onClick={() => onDelete(todo.id)}
        className="text-red-500 hover:text-red-700"
      >
        삭제
      </button>
    </div>
  );
}
```

## 버튼
```tsx
// components/Button.tsx
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'danger';
  type?: 'button' | 'submit';
}

export default function Button({
  children,
  onClick,
  variant = 'primary',
  type = 'button'
}: ButtonProps) {
  const styles = {
    primary: 'bg-blue-500 hover:bg-blue-600 text-white',
    danger: 'bg-red-500 hover:bg-red-600 text-white',
  };

  return (
    <button
      type={type}
      onClick={onClick}
      className={`px-4 py-2 rounded ${styles[variant]}`}
    >
      {children}
    </button>
  );
}
```

## 로딩 스피너
```tsx
// components/Loading.tsx
export default function Loading() {
  return (
    <div className="flex justify-center py-8">
      <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-500" />
    </div>
  );
}
```
