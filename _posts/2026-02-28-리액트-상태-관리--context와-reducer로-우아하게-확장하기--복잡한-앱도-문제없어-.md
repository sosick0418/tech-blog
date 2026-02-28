---
layout: post
title: "리액트 상태 관리, Context와 Reducer로 우아하게 확장하기: 복잡한 앱도 문제없어!"
date: 2026-02-28T01:00:52.172Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거입니다. 2026년 2월 28일, 오늘도 빠르게 변화하는 프론트엔드 세계의 흥미로운 소식을 전해드립니다. 오늘은 리액트 개발자라면 누구나 한 번쯤 고민했을 '상태 관리'에 대한 이야기를 해볼까 합니다.

---



프론트엔드 개발에서 상태 관리는 늘 뜨거운 감자입니다. 특히 리액트 프로젝트가 커질수록 `useState`만으로는 복잡한 전역 상태를 효과적으로 다루기 어려워지죠. 이럴 때 리액트 내장 훅인 `Context`와 `useReducer`를 조합하면 외부 라이브러리 없이도 견고하고 확장 가능한 상태 관리 시스템을 구축할 수 있습니다.

## 핵심 개념 설명: Context와 useReducer의 시너지

`Context`는 컴포넌트 트리를 통해 데이터를 전달할 때 '프롭 드릴링(Prop Drilling)'을 피하게 해주는 강력한 도구입니다. 특정 값을 하위 컴포넌트들에게 직접 전달하지 않고도, Context를 통해 원하는 컴포넌트에서 그 값에 접근할 수 있게 해주죠. 하지만 Context 자체는 상태를 '관리'하기보다는 '전달'하는 데 특화되어 있습니다.

여기서 `useReducer`가 등장합니다. `useReducer`는 `useState`의 대안으로, 복잡한 상태 로직을 다룰 때 유용합니다. 상태 업데이트 로직을 리듀서 함수 안에 중앙 집중화하여, 여러 상태 업데이트가 서로 의존하거나 순서가 중요한 경우에도 예측 가능한 방식으로 상태를 변경할 수 있게 합니다.

이 둘을 함께 사용하면, `Context`가 `useReducer`로 관리되는 상태와 디스패치 함수를 컴포넌트 트리에 제공하고, 하위 컴포넌트들은 `useContext`를 통해 필요한 상태와 액션을 받아 상태를 업데이트할 수 있게 됩니다. 마치 Redux와 비슷한 패턴이지만, 리액트 자체 기능만으로 구현할 수 있다는 장점이 있습니다.

## 실제 코드 예시 (React/Next.js/TypeScript 활용)

간단한 투두(Todo) 리스트 앱을 예시로 들어볼까요?

```tsx
// src/contexts/TodoContext.tsx
import React, { createContext, useReducer, useContext, ReactNode } from 'react';

// 1. 상태 타입 정의
interface Todo {
  id: string;
  text: string;
  isCompleted: boolean;
}

interface TodoState {
  todos: Todo[];
}

// 2. 액션 타입 정의
type TodoAction =
  | { type: 'ADD_TODO'; payload: string }
  | { type: 'TOGGLE_TODO'; payload: string }
  | { type: 'REMOVE_TODO'; payload: string };

// 3. 리듀서 함수
const todoReducer = (state: TodoState, action: TodoAction): TodoState => {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        todos: [
          ...state.todos,
          { id: String(Date.now()), text: action.payload, isCompleted: false },
        ],
      };
    case 'TOGGLE_TODO':
      return {
        todos: state.todos.map((todo) =>
          todo.id === action.payload
            ? { ...todo, isCompleted: !todo.isCompleted }
            : todo
        ),
      };
    case 'REMOVE_TODO':
      return {
        todos: state.todos.filter((todo) => todo.id !== action.payload),
      };
    default:
      return state;
  }
};

// 4. 초기 상태
const initialState: TodoState = {
  todos: [],
};

// 5. Context 생성
interface TodoContextType {
  state: TodoState;
  dispatch: React.Dispatch<TodoAction>;
}

const TodoContext = createContext<TodoContextType | undefined>(undefined);

// 6. Provider 컴포넌트
interface TodoProviderProps {
  children: ReactNode;
}

export const TodoProvider = ({ children }: TodoProviderProps) => {
  const [state, dispatch] = useReducer(todoReducer, initialState);

  return (
    <TodoContext.Provider value={{ state, dispatch }}>
      {children}
    </TodoContext.Provider>
  );
};

// 7. Context를 쉽게 사용할 수 있는 커스텀 훅
export const useTodo = () => {
  const context = useContext(TodoContext);
  if (context === undefined) {
    throw new Error('useTodo must be used within a TodoProvider');
  }
  return context;
};
```

이제 이 `TodoProvider`를 앱의 최상단 컴포넌트(예: Next.js의 `_app.tsx` 또는 `layout.tsx`)에 감싸주고, `useTodo` 훅을 사용하여 상태와 액션을 처리할 수 있습니다.

```tsx
// src/app/layout.tsx (Next.js App Router 예시)
import { TodoProvider } from '../contexts/TodoContext';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="ko">
      <body>
        <TodoProvider>
          {children}
        </TodoProvider>
      </body>
    </html>
  );
}

// src/components/TodoList.tsx
'use client'; // 클라이언트 컴포넌트임을 명시 (Next.js)

import React, { useState } from 'react';
import { useTodo } from '../contexts/TodoContext';

const TodoList = () => {
  const { state, dispatch } = useTodo();
  const [newTodoText, setNewTodoText] = useState('');

  const handleAddTodo = () => {
    if (newTodoText.trim()) {
      dispatch({ type: 'ADD_TODO', payload: newTodoText });
      setNewTodoText('');
    }
  };

  return (
    <div>
      <h1>My Todo List</h1>
      <input
        type="text"
        value={newTodoText}
        onChange={(e) => setNewTodoText(e.target.value)}
        placeholder="Add a new todo"
      />
      <button onClick={handleAddTodo}>Add Todo</button>
      <ul>
        {state.todos.map((todo) => (
          <li key={todo.id} style={{ textDecoration: todo.isCompleted ? 'line-through' : 'none' }}>
            <span onClick={() => dispatch({ type: 'TOGGLE_TODO', payload: todo.id })}>
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: 'REMOVE_TODO', payload: todo.id })}>
              Remove
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default TodoList;
```

## 실무 적용 팁

1.  **Context 분리**: 애플리케이션의 상태가 커지면 단일 Context는 관리하기 어려워집니다. 도메인별(예: 인증, 사용자 설정, 장바구니 등)로 Context를 분리하여 모듈성을 높이고 성능 최적화에 유리하게 만드세요.
2.  **`useCallback`과 `React.memo` 활용**: Context 값이 변경되면 해당 Context를 사용하는 모든 컴포넌트가 리렌더링될 수 있습니다. Provider에서 제공하는 `dispatch` 함수는 참조가 변하지 않지만, 다른 상태 기반 함수를 제공한다면 `useCallback`으로 메모이제이션하고, 하위 컴포넌트에는 `React.memo`를 적용하여 불필요한 리렌더링을 방지하세요.
3.  **초기 상태 로딩**: 서버에서 초기 상태를 받아와야 할 경우, `useEffect`로 비동기 데이터를 가져와 `dispatch`를 통해 초기 상태를 설정하거나, Next.js의 서버 컴포넌트, `getServerSideProps` 등으로 초기 데이터를 주입할 수 있습니다.
4.  **테스트 용이성**: 리듀서 함수는 순수 함수이므로 단위 테스트하기 매우 용이합니다. Context Provider를 모킹하여 특정 상태에서 컴포넌트가 어떻게 렌더링되고 액션에 반응하는지 테스트할 수 있습니다.

## 마무리: 견고한 상태 관리의 첫걸음

React Context와 `useReducer`를 함께 사용하는 패턴은 Redux와 같은 외부 상태 관리 라이브러리 없이도 복잡한 애플리케이션의 상태를 효율적이고 예측 가능하게 관리할 수 있는 강력한 대안입니다. 이 패턴은 특히 중대형 프로젝트에서 프롭 드릴링 문제를 해결하고, 상태 로직을 중앙 집중화하여 코드의 가독성과 유지보수성을 크게 향상시킵니다.

물론, 모든 상황에 이 패턴이 최적의 해답은 아닐 수 있습니다. 애플리케이션의 규모와 팀의 선호도에 따라 Zustand, Jotai, Recoil 등 더 가볍거나 특화된 상태 관리 라이브러리를 고려해 볼 수도 있습니다. 하지만 리액트 내장 기능만으로도 충분히 강력한 상태 관리 시스템을 구축할 수 있다는 점을 이해하는 것은 프론트엔드 개발자로서 매우 중요한 역량입니다. 오늘 배운 내용을 바탕으로 여러분의 프로젝트에 우아한 상태 관리를 적용해 보세요!

---

## 참고 자료
- [Escalando el estado en React con Context + Reducer](https://dev.to/kcode/escalando-el-estado-en-react-con-context-reducer-3ioh) by Kevin Curruchich
- [How I Built an AI Course Generator in a Weekend (DuoUniversal)](https://dev.to/danilocaffaro/how-i-built-an-ai-course-generator-in-a-weekend-duouniversal-p9l) by Danilo Caffaro
- [This Week In React #270 : Next.js, React Router | Hermes, React Navigation, CSS Grid, Maestro | Node, Oxfmt](https://dev.to/sebastienlorber/this-week-in-react-270-nextjs-react-router-hermes-react-navigation-css-grid-maestro--106o) by Sebastien Lorber
- [React vs Vue in 2026: What the npm Data Actually Says](https://dev.to/royce_fabbd83cb268312e928/react-vs-vue-in-2026-what-the-npm-data-actually-says-4nll) by Royce
- [Fast search, filter & sort for large client-side collections in JavaScript](https://dev.to/traeop_8fe714e58a84498dc2/fast-search-filter-sort-for-large-client-side-collections-in-javascript-bj5) by traeop