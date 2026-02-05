# Tailwind 클래스

## 레이아웃
```
flex          - 플렉스 컨테이너
grid          - 그리드 컨테이너
gap-4         - 간격 1rem
p-4           - 패딩 1rem
m-2           - 마진 0.5rem
w-full        - 너비 100%
max-w-md      - 최대 너비 28rem
```

## 텍스트
```
text-xl       - 글자 크기 1.25rem
text-2xl      - 글자 크기 1.5rem
font-bold     - 굵은 글씨
text-gray-600 - 회색 텍스트
text-center   - 가운데 정렬
```

## 배경/테두리
```
bg-white      - 흰색 배경
bg-blue-500   - 파란색 배경
border        - 테두리
rounded       - 둥근 모서리
rounded-lg    - 더 둥근 모서리
shadow-sm     - 작은 그림자
```

## 상태
```
hover:bg-blue-600  - 호버 시 배경색
focus:ring-2       - 포커스 시 링
disabled:opacity-50 - 비활성화 시 투명도
```

## 반응형
```
md:flex       - 768px 이상에서 flex
lg:grid-cols-3 - 1024px 이상에서 3열
```

## 자주 쓰는 조합
```tsx
// 카드
className="border rounded-lg p-4 shadow-sm"

// 버튼
className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"

// 입력
className="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"

// 중앙 정렬 컨테이너
className="min-h-screen flex items-center justify-center"
```
