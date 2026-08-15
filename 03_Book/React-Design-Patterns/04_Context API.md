---
date: 2025-09-29
tags:
  - book
  - react
  - design-patterns
  - frontend
  - general
---
예시:
현재 시간을 기반으로 다크/라이트 테마를 보여주는 애플리케이션

`ThemeContextType`를 정의하고, 타입을 다루는 `ThemeContext` 인스턴스 생성
```ts
import React from "react";

export type ThemeContextType = {
	theme: "light" | "dark";
};

export const ThemeContext = React.createContext<ThemeContextType | undefined>(
	undefined
);
```

테마값을 리액트 상태로 관리하는 `ThemeProvider` 컴포넌트 생성
```tsx
import React, { useState } from "react";
import { ThemeContext, ThemeContextType } from "./ThemeContext";

export const ThemeProvider = ({ children }) => {
	const [theme, setTheme] = useState<"light" | "dark">("light");
	const value: ThemeContextType = { theme };
	
	return (
		<ThemeContext.Provider value={value}>
			{children}
		</ThemeContext.Provider>
	)
}
```

애플리케이션에 `ThemeProvider` 컴포넌트 사용
```tsx
import React from "react";
import { ThemeProvider } from "./ThemeProvider";
import App from "./App";

const Root = () => {
	return (
		<ThemeProvider>
			<App />
		</ThemeProvider>
	)
}
```

이제 애플리케이션의 모든 컴포넌트에서 현재 테마값에 접근 가능
```tsx
import React, { useContext } from "react";
import { ThemeContext } from "./ThemeContext";

const ThemedComponent = () => {
	const context = useContext(ThemeContext);
	const { theme } = context;
	
	return <div className={theme}>Current Theme: {theme}</div>
}
```

`ThemeContext` 인스턴스를 조금 수정해보자
```ts
type Theme = {
	theme: "light" | "dark";
	toggleTheme: () => void;
};

const ThemeContext = React.createContext<Theme>({
	theme: "light",
	toggleTheme: () => {},
})
```

