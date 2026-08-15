---
date: 2025-11-08
tags:
  - frontend
  - nextjs
---
```bash
$ mkdir next-test
$ cd next-test

$ pnpm create next-app@latest . --typescript --tailwind --app --no-src-dir --import-alias "@/*" -e with-supabase

# upgrade to tailwind v4
$ pnpm add -D tailwindcss@next @tailwindcss/postcss@next

$ pnpm add -D vitest @vitest/ui @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
$ pnpm add prisma @prisma/client
$ pnpm add zustand
$ pnpm add -D prettier eslint-config-prettier prettier-plugin-tailwindcss husky lint-staged
$ pnpm add @tanstack/react-query @tanstack/react-query-devtools

$ pnpm add -D @vitejs/plugin-react

$ pnpm exec husky init
$ echo "pnpm exec lint-staged" > .husky/pre-commit

```

## `@vitejs/plugin-react`가 필요한 이유

### 1. **Vitest = Vite 기반 테스트 러너**

Vitest는 Vite를 사용하여 테스트 파일을 실행합니다. React 컴포넌트를 테스트하려면:

```tsx
// components/Button.test.tsx
import { render, screen } from '@testing-library/react'
import { Button } from './Button'

test('renders button', () => {
  render(<Button>Click me</Button>)  // ← JSX를 변환해야 함!
  expect(screen.getByText('Click me')).toBeInTheDocument()
})
```

### 2. **JSX/TSX 변환**

`@vitejs/plugin-react`가 하는 일:
- ✅ JSX → JavaScript 변환
- ✅ React Fast Refresh 지원 (개발 시)
- ✅ TypeScript + React 타입 체크
- ✅ React의 특수 문법 처리

```jsx
// Before (JSX)
<Button>Click me</Button>

// After (JavaScript)
React.createElement(Button, null, 'Click me')
```

### 3. **없으면 어떻게 되나요?**

```bash
$ pnpm test

❌ Error: Failed to parse source for import analysis because the content contains invalid JS syntax.
If you are using JSX, make sure to name the file with the .jsx or .tsx extension.
```

React 컴포넌트를 인식하지 못하고 테스트가 실패합니다!

### 4. **Next.js와의 관계**

- **Next.js 개발/빌드**: Next.js 자체 컴파일러 사용 (SWC)
- **Vitest 테스트**: Vite + `@vitejs/plugin-react` 사용

둘은 독립적입니다. 테스트 환경에서만 Vite를 사용하므로 플러그인이 필요합니다.

```typescript:vitest.config.ts
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'  // ← React 지원

export default defineConfig({
  plugins: [react()],  // ← 이것이 JSX를 변환해줌
  test: {
    environment: 'jsdom',  // ← 브라우저 환경 시뮬레이션
  },
})
```

---

**요약**: Vitest가 React 컴포넌트(JSX/TSX)를 테스트할 수 있게 해주는 필수 플러그인입니다! 🔧