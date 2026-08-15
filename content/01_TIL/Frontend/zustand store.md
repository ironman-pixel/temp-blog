---
date: 2025-07-02
tags:
  - til
  - frontend
  - react
  - zustand
---
# ❓ Information
* defining zustand store

---
# ❗ Relevant data
## 📦 Information Resources
> [heropy.dev: zustand 설명](https://www.heropy.dev/p/n74Tgc)
> 
> [velog: zustand useAuthStore](https://velog.io/@juwon98/Reac-Zustand-useAuthStore)

# 🔰 Content ->  

## Store 생성
```ts
create<타입>()
```

```ts
import { create } from 'zustand'

export const useCountStore = create<{
  count: number
  increase: () => void
  decrease: () => void
}>(set => ({
  count: 1,
  increase: () => set(state => ({ count: state.count + 1 })),
  decrease: () => set(state => ({ count: state.count - 1 }))
}))
```

## Store 사용

### 액션 사용 방식
`useCountStore(...)` 방식이 있고, `useCountStore.getState()` 방식이 있다.
둘 다 store 의 기능을 정상적으로 사용하는 것이다.

그런데 컴포넌트 내부에서는 `useCountStore(...)`를 사용한다.
이유는 컴포넌트의 리렌더링을 위해서이다.

`useUserStore.getState()` 는 **현재 상태 객체를 즉시 가져오기 때문**에 값이 바뀌어도 리렌더링이 안된다
따라서 **비UI 로직** 에서는 `useCountStore.getState()` 를 사용하는 것이다.

**부가적 포인트**
- zustand는 내부적으로 **vanilla store**를 제공하고, `useUserStore`는 그 위에 React Hook을 입힌 것
- 그래서 **렌더링 연결 여부**에 따라 두 가지 방식 중 선택하는 거지, 기능적으로는 같은 store를 바라보고 있음

### 액션 하나씩 가져오기
store 를 사용하는 모든 컴포넌트의 불필요한 리렌더링을 방지하기 위해, 컴포넌트에서는 한 번에 하나씩만 상태(액션)을 가져와야함
```ts
import { useCountStore } from './store/count'

export default function App() {
  const count = useCountStore(state => state.count)
  const increase = useCountStore(state => state.increase)
  const decrease = useCountStore(state => state.decrease)
  return (
    <>
      <h2>{count}</h2>
      <button onClick={increase}>+1</button>
      <button onClick={decrease}>-1</button>
    </>
  )
}
```

### 다중 상태 선택(useShallow)
`useShallow`훅을 사용하면 여러 상태(액션)을 한 번에 객체나 배열로 가져올 수 있다
```ts
import { useShallow } from 'zustand/shallow'
import { useCountStore } from './store/count'

export default function App() {
  // 객체
  // const { count, increase, decrease } = useCountStore(state => ({
  const countState = useCountStore(
    useShallow(state => ({
      count: state.count,
      increase: state.increase,
      decrease: state.decrease
    }))
  )
  
  return (
    <>
      <h2>{countState.count}</h2>
      <button onClick={countState.increase}>+1</button>
      <button onClick={countState.decrease}>-1</button>
    </>
  )
}
```

```ts
import { useShallow } from 'zustand/shallow'
import { useCountStore } from './store/count'

export default function App() {
  // 배열
  // const [count, increase, decrease] = useCountStore(
  const countState = useCountStore(
    useShallow(state => [state.count, state.increase, state.decrease])
  )
  
  return (
    <>
      <h2>{countState[0]}</h2>
      <button onClick={countState[1]}>+1</button>
      <button onClick={countState[2]}>-1</button>
    </>
  )
}
```

### 액션 분리
여러 컴포넌트에서 단일 스토어의 액션을 많이 사용한다면, 액션을 분리해 관리하는 패턴을 활용할 수 있음.
다음과 같이 `actions` 객체 안에서 모든 액션을 관리하면, 각 컴포넌트에서 필요한 액션만 가져오기 쉬움
```ts
import { create } from 'zustand'

export const useCountStore = create<{
  count: number
  actions: {
    increase: () => void
    decrease: () => void
  }
}>(set => ({
  count: 1,
  actions: {
    increase: () => set(state => ({ count: state.count + 1 })),
    decrease: () => set(state => ({ count: state.count - 1 }))
  }
}))
```

```ts
import { useCountStore } from './store/count'

export default function App() {
  const count = useCountStore(state => state.count)
  const { increase, decrease } = useCountStore(state => state.actions)
  return (
    <>
      <h2>{count}</h2>
      <button onClick={increase}>+1</button>
      <button onClick={decrease}>-1</button>
    </>
  )
}
```


zustand has middleware `devtools`, `persist`

## devtools

### What it does
From Chrome extention `redux-devtools`  Zustand store can be shown

### How to use
1. Wrap the create function with devtools as the first parameter.
2. Set the store name as the second parameter of devtools.
```ts
import create from 'zustand';
import { devtools } from 'zustand/middleware';

const useStore = create(
	devtools(
		(set) => ({
			// ...
		}), { name: "CounterStore" }
	)
);
```

## persist

### What it does
Saves state inside **browser storage** (ex: localStorege, sessionStorage, IndexedDB). 
So page Refreshing, app restarting won't wipe the data.

### How to use
1. Wrap the `create` function with `persist`
2. Set the store name, storage to use, etc as the second parameter of devtools.
```ts
import create from 'zustand';
import { persist } from 'zustand/middleware';

const useState = create(
	persist(
		(set) => ({
			count: 0,
			increment: () => set((state) => ({ count: state.count + 1 })),
		}),
		{
			name: 'count-storage', // 저장소에 저장될 키 이름
			storage: createJSONStorage(() => localStorage), // 사용할 저장소
			partialize: (store) => ({ count: store.count }). // 로컬 스토리지에 저장할 상태만 선택
		}
	)
);
```

