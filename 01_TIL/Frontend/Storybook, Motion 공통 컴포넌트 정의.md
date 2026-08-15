---
date: 2025-07-31
tags:
  - til
  - frontend
  - react
  - vite
  - storybook
  - tailwindcss
  - ts
---
# ❓ Information
* Vite + TS + Tailwind 에서 Storybook 을 사용해서 공통 컴포넌트를 정의하는중 알게된 것들

---

# 🔰 Content ->  
## `motion/react` 라이브러리 사용시 주의사항
- Animation Props는 **Tailwind 를 해석할 수 없다.** CSS 속성으로 애니메이션화는 가능하다.
- 태그의 **className은 Tailwind로 작성하되, motion props는 CSS로 정의** 해야함

## Storybook에서 컴포넌트 Props 자동화 방법
### 첫 시도
1. `InputHTMLAttributes<HTMLInputElement>`를 활용해서 등의 표준 HTML 태그의 속성(`id, name, onChange`)들을 자동으로 포함시켰다.
2. 그런데 **`motion/react`를 함께 사용하니까 Type 충돌이 났다.**
3. 알아보니 `motion/react`에서 애니메이션과 제스처 기능을 제공하기 위해 내부적으로 `HTMLMotinProps` 라는 자체적인 타입을 사용한다. 
   이 `HTMLMotionProps` 는 표준 HTML 속성들을 확장하거나, 때로는 **자신들의 로직에 맞게 특정 이벤트 핸들러(onChange, onDrag, onFocus, ...)의 타입 시그니처를 재정의** 한다고 한다.
### 두번째 시도
1. `@/types/motion-props.d.ts` 파일을 만들고 **충돌이 나는 속성을 모두 제거한 Motion태그 Props 타입을 정의했다.**
```ts
import { HTMLMotionProps } from "motion/react";
import {
  HTMLAttributes,
  ButtonHTMLAttributes,
  InputHTMLAttributes,
  AnchorHTMLAttributes,
  ImgHTMLAttributes,
} from "react";

type CommonMotionConfilcts =
  | "onDrag"
  | "onChange"
  | "onFocus"
  | "onBlur"
  | "onKeyDown"
  | "onKeyUp"
  | "onKeyPress"
  | "onPointerDown"
  | "onPointerMove"
  | "onPointerUp"
  | "onPointerCancel"
  | "onPointerEnter"
  | "onPointerLeave"
  | "onPointerOver"
  | "onPointerOut";

export type MotionDivProps = HTMLMotionProps<"div"> &
  Omit<HTMLAttributes<HTMLDivElement>, CommonMotionConfilcts>;

export type MotionButtonProps = HTMLMotionProps<"button"> &
  Omit<ButtonHTMLAttributes<HTMLButtonElement>, CommonMotionConfilcts>;

export type MotionInputProps = HTMLMotionProps<"input"> &
  Omit<InputHTMLAttributes<HTMLInputElement>, CommonMotionConfilcts>;

export type MotionAnchorProps = HTMLMotionProps<"a"> &
  Omit<AnchorHTMLAttributes<HTMLAnchorElement>, CommonMotionConfilcts>;

export type MotionImgProps = HTMLMotionProps<"img"> &
  Omit<ImgHTMLAttributes<HTMLImageElement>, CommonMotionConfilcts>;
```

2. 이제 이렇게 안전하게 사용이 가능해졌다
```tsx
import type { MotionInputProps } from "@/type/motion-props";

type InputProps = MotionInputProps & {
  variant?: "primary" | "secondary";
  fontSize?: "sm" | "md" | "lg";
  motion?: "none" | "inView";
  onChange?: (value: string | number) => void;
};

const Input = ({
  variant = "primary",
  fontSize = "md",
  motion = "none",
  onChange,
  ...props
}: InputProps) => {
  const basicClasses = "border rounded-md p-2";
  
  const variantClasses = {
    primary: "border-blue-500",
    secondary: "border-gray-500",
  };
  
  const fontSizeClasses = {
    sm: "text-sm",
    md: "text-base",
    lg: "text-lg",
  };
  
  const combinedClasses = `
    ${basicClasses}
    ${variantClasses[variant]}
    ${fontSizeClasses[fontSize]}
    ${props.disabled ? "opacity-50" : ""}
    ${props.className || ""}
  `.trim();
  
  return (
    <Motion.input
      initial={motion === "inView" ? { opacity: 0, y: 10 } : {}}
      animate={motion === "inView" ? { opacity: 1, y: 0 } : {}}
      transition={{ duration: 0.3 }}
      className={combinedClasses}
      onChange={(e) => onChange?.(e.target.value)}
      {...props}
    />
  );
};

export default Input;
```