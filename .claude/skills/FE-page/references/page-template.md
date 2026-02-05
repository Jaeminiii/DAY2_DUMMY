# 페이지 템플릿

## 기본 페이지
```tsx
// app/users/page.tsx
export default function UsersPage() {
  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold">사용자 목록</h1>
    </div>
  );
}
```

## 클라이언트 컴포넌트 페이지
```tsx
// app/dashboard/page.tsx
'use client';
import { useState, useEffect } from 'react';

export default function DashboardPage() {
  const [data, setData] = useState(null);

  useEffect(() => {
    // 데이터 로딩
  }, []);

  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold">대시보드</h1>
    </div>
  );
}
```

## 동적 라우트 페이지
```tsx
// app/users/[id]/page.tsx
'use client';
import { useParams } from 'next/navigation';

export default function UserDetailPage() {
  const { id } = useParams();

  return (
    <div className="p-4">
      <h1>사용자 #{id}</h1>
    </div>
  );
}
```

## 레이아웃
```tsx
// app/admin/layout.tsx
export default function AdminLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex">
      <nav className="w-64 bg-gray-100 p-4">사이드바</nav>
      <main className="flex-1 p-4">{children}</main>
    </div>
  );
}
```
