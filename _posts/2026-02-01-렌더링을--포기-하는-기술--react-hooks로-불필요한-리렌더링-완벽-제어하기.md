---
layout: post
title: "렌더링을 '포기'하는 기술? React Hooks로 불필요한 리렌더링 완벽 제어하기"
date: 2026-02-01T01:00:58.823Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발의 최전선에서 늘 새로운 기술을 탐구하는 프론트엔드 개발 전문 블로거입니다. 2026년 2월 1일, 오늘도 여러분의 코드를 한층 더 빛나게 할 흥미로운 주제를 들고 왔습니다.

---



리액트 애플리케이션의 성능 최적화는 사용자 경험을 향상시키고 개발 효율성을 높이는 데 있어 핵심적인 요소입니다. 특히, 불필요한 컴포넌트 리렌더링은 애플리케이션의 속도를 저하시키고 자원 소모를 늘리는 주범이 될 수 있죠. 오늘 우리는 React Hooks를 활용하여 이러한 불필요한 리렌더링을 효과적으로 '포기'하고, 컴포넌트의 성능을 극대화하는 방법에 대해 심층적으로 알아보겠습니다.

## 왜 불필요한 리렌더링을 막아야 할까요?

React는 상태(state)나 속성(props)이 변경될 때마다 컴포넌트를 다시 렌더링하여 UI를 업데이트합니다. 이는 선언적인 UI를 구현하는 React의 강력한 장점이지만, 때로는 부모 컴포넌트의 상태가 변경되어 자식 컴포넌트가 불필요하게 다시 렌더링되는 상황이 발생합니다. 예를 들어, 수백 개의 아이템을 렌더링하는 리스트에서 단 하나의 아이템만 변경되었음에도 불구하고 전체 리스트가 다시 렌더링된다면, 이는 명백한 성능 저하로 이어질 것입니다. 이러한 불필요한 작업들을 줄이는 것이 바로 '렌더링 포기(Abandon Rendering)'의 핵심 목표입니다.

## React Hooks를 활용한 렌더링 최적화 핵심 개념

React에서 렌더링을 '포기'한다는 것은, 특정 조건에서 컴포넌트의 재렌더링을 건너뛰도록 하는 최적화 기법을 의미합니다. 이를 위해 주로 `React.memo`, `useCallback`, `useMemo` 세 가지 Hooks를 활용합니다.

1.  **`React.memo`**: 고차 컴포넌트(Higher-Order Component)로, 컴포넌트의 props가 변경되지 않았다면 리렌더링을 건너뛰고 이전에 렌더링된 결과를 재사용하도록 합니다. 함수형 컴포넌트에서 `shouldComponentUpdate`와 유사한 역할을 수행합니다.
2.  **`useCallback`**: 특정 함수를 메모이제이션(memoization)하여, 의존성 배열(dependency array)에 있는 값이 변경되지 않는 한 동일한 함수 인스턴스를 반환하도록 합니다. 이는 자식 컴포넌트에 prop으로 함수를 전달할 때 특히 유용하며, `React.memo`와 함께 사용될 때 시너지를 발휘합니다.
3.  **`useMemo`**: 특정 값(계산 결과)을 메모이제이션하여, 의존성 배열에 있는 값이 변경되지 않는 한 이전에 계산된 값을 재사용하도록 합니다. 복잡하거나 비용이 많이 드는 계산을 캐싱할 때 주로 사용됩니다.

이 세 가지를 통해 우리는 컴포넌트가 언제, 왜 리렌더링되는지 제어하고, 불필요한 작업을 최소화할 수 있습니다.

## 실제 코드 예시: 리렌더링 최적화 적용하기

다음은 부모 컴포넌트의 상태 변화가 자식 컴포넌트에 불필요한 리렌더링을 유발하는 상황과, 이를 `React.memo`와 `useCallback`으로 최적화하는 예시입니다. TypeScript와 React를 기반으로 작성되었습니다.

```tsx
// types.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

// ChildComponent.tsx
import React from 'react';
import { User } from './types';

interface UserItemProps {
  user: User;
  onSelect: (id: number) => void;
}

// 최적화되지 않은 자식 컴포넌트
const UserItem: React.FC<UserItemProps> = ({ user, onSelect }) => {
  console.log(`UserItem ${user.name} 렌더링!`);
  return (
    <div style={{ border: '1px solid #eee', padding: '10px', margin: '5px' }}>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onSelect(user.id)}>선택</button>
    </div>
  );
};

// React.memo를 적용하여 최적화된 자식 컴포넌트
const MemoizedUserItem: React.FC<UserItemProps> = React.memo(({ user, onSelect }) => {
  console.log(`MemoizedUserItem ${user.name} 렌더링!`);
  return (
    <div style={{ border: '1px solid #eee', padding: '10px', margin: '5px' }}>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <button onClick={() => onSelect(user.id)}>선택</button>
    </div>
  );
});

export { UserItem, MemoizedUserItem };


// ParentComponent.tsx
import React, { useState, useCallback, useMemo } from 'react';
import { UserItem, MemoizedUserItem } from './ChildComponent';
import { User } from './types';

const initialUsers: User[] = [
  { id: 1, name: '김철수', email: 'chulsoo@example.com' },
  { id: 2, name: '이영희', email: 'younghee@example.com' },
  { id: 3, name: '박민준', email: 'minjun@example.com' },
];

const UserListContainer: React.FC = () => {
  const [users, setUsers] = useState<User[]>(initialUsers);
  const [count, setCount] = useState(0); // 부모 컴포넌트의 상태

  // 1. 최적화되지 않은 콜백 함수: 매 렌더링마다 새로운 함수 인스턴스 생성
  const handleSelectUserUnoptimized = (id: number) => {
    console.log(`User ${id} selected (Unoptimized)`);
    // 실제 로직 (예: 선택된 사용자 상태 업데이트)
  };

  // 2. useCallback을 사용하여 최적화된 콜백 함수
  // 의존성 배열이 비어있으므로 컴포넌트가 마운트될 때 한 번만 생성됨
  const handleSelectUserOptimized = useCallback((id: number) => {
    console.log(`User ${id} selected (Optimized)`);
    // 실제 로직
  }, []);

  // 3. useMemo를 사용하여 복잡한 계산 결과 메모이제이션
  // users 배열이 변경될 때만 filteredUsers를 다시 계산
  const filteredUsers = useMemo(() => {
    console.log('useMemo: 사용자 필터링 중...');
    return users.filter(user => user.id <= 2); // 예시 필터링
  }, [users]);


  return (
    <div style={{ padding: '20px' }}>
      <h1>사용자 목록</h1>
      <p>부모 컴포넌트 카운트: {count}</p>
      <button onClick={() => setCount(prev => prev + 1)}>카운트 증가 (부모 리렌더링 유발)</button>

      <h2>최적화되지 않은 목록</h2>
      {/* 부모의 count가 변할 때마다 UserItem들이 불필요하게 렌더링됨 */}
      {users.map(user => (
        <UserItem key={user.id} user={user} onSelect={handleSelectUserUnoptimized} />
      ))}

      <hr />

      <h2>React.memo + useCallback으로 최적화된 목록</h2>
      {/* 부모의 count가 변해도 MemoizedUserItem들은 리렌더링되지 않음 (onSelect도 메모이제이션) */}
      {users.map(user => (
        <MemoizedUserItem key={user.id} user={user} onSelect={handleSelectUserOptimized} />
      ))}

      <hr />

      <h2>useMemo로 필터링된 사용자 (예시)</h2>
      {filteredUsers.map(user => (
        <p key={user.id}>필터링된 사용자: {user.name}</p>
      ))}
    </div>
  );
};

export default UserListContainer;
```

위 코드에서 `UserListContainer`의 `count` 상태를 변경하면, `UserListContainer` 컴포넌트가 리렌더링됩니다.
`UserItem` 컴포넌트는 부모가 리렌더링될 때마다 함께 리렌더링되는 것을 콘솔에서 확인할 수 있습니다. 이는 `onSelect` prop으로 전달되는 `handleSelectUserUnoptimized` 함수가 매번 새로운 인스턴스로 생성되기 때문입니다.
하지만 `MemoizedUserItem` 컴포넌트는 `React.memo`로 감싸져 있고, `onSelect` prop으로 전달되는 `handleSelectUserOptimized` 함수가 `useCallback`으로 메모이제이션되어 있기 때문에, `count`가 변경되어도 `MemoizedUserItem`은 리렌더링되지 않습니다.

`useMemo`는 `filteredUsers`와 같이 비용이 많이 드는 계산 결과를 `users` 배열이 변경될 때만 다시 계산하도록 하여 성능을 향상시킵니다.

## 실무 적용 팁

1.  **무조건적인 메모이제이션은 독!**: `React.memo`, `useCallback`, `useMemo`는 오버헤드를 동반합니다. 메모이제이션을 적용하는 비용이 리렌더링을 피하는 이득보다 클 수 있으므로, 항상 성능 프로파일링을 통해 병목 지점을 파악한 후 적용해야 합니다.
2.  **DevTools 활용**: React Developer Tools의 Profiler 탭을 활용하여 어떤 컴포넌트가 얼마나 자주 렌더링되고, 렌더링에 얼마나 많은 시간이 소요되는지 시각적으로 확인하세요. 이는 최적화가 필요한 부분을 찾는 데 큰 도움이 됩니다.
3.  **의존성 배열 관리**: `useCallback`과 `useMemo`의 의존성 배열(`[]`)을 정확히 관리하는 것이 중요합니다. 잘못된 의존성은 예상치 못한 버그를 유발하거나, 오히려 최적화 효과를 상쇄시킬 수 있습니다.
4.  **Context API와 리렌더링**: Context API를 사용할 때, Context 값이 변경되면 해당 Context를 구독하는 모든 컴포넌트가 리렌더링됩니다. `React.memo`와 함께 사용하거나, Context를 세분화하여 불필요한 리렌더링을 줄이는 전략을 고려해야 합니다.
5.  **컴포넌트 분리**: 로직이 복잡하고 자주 변경되는 부분과, 정적이고 변경이 적은 부분을 명확히 분리하여 컴포넌트를 설계하는 것이 리렌더링 최적화의 기본입니다.

## 마무리하며

React Hooks를 활용한 렌더링 최적화는 단순히 코드를 추가하는 것을 넘어, React의 렌더링 메커니즘에 대한 깊은 이해를 요구하는 작업입니다. `React.memo`, `useCallback`, `useMemo`는 강력한 도구이지만, 현명하게 사용해야 그 진정한 가치를 발휘할 수 있습니다. 오늘 배운 '렌더링을 포기하는 기술'을 통해 여러분의 React 애플리케이션이 더욱 빠르고 효율적으로 동작하기를 바랍니다.

다음 학습 방향으로는 React Profiler를 활용한 심층적인 성능 분석 방법과, Recoil, Zustand와 같은 최신 상태 관리 라이브러리들이 렌더링 최적화를 어떻게 지원하는지 탐구해보시길 추천합니다. 지속적인 학습과 실험을 통해 여러분의 프론트엔드 개발 역량을 한 단계 더 끌어올리세요!

---

## 참고 자료
- [ReactJS Hook Pattern ~Abandon Rendering~](https://dev.to/bishoy_semsem/reactjs-hook-pattern-abandon-rendering-477k) by Bishoy Semsem
- [This Week In React #266 : DoS, shadcn, Skills, Rspack | Expo 55 beta, Hermes, Expo Router | TC39, Rolldown, Yarn](https://dev.to/sebastienlorber/this-week-in-react-266-dos-shadcn-skills-rspack-expo-55-beta-hermes-expo-router-tc39-2h0p) by Sebastien Lorber
- [Introducing BoldKit: A Neubrutalism React Component Library](https://dev.to/aniruddha_agarwal_d3d6b0e/introducing-boldkit-a-neubrutalism-react-component-library-36g1) by Aniruddha Agarwal
- [Overcoming Geo-Restrictions in React: A Security Researcher’s Approach to Testing Blocked Features](https://dev.to/mohammad_waseem_c31f3a26f/overcoming-geo-restrictions-in-react-a-security-researchers-approach-to-testing-blocked-features-1i78) by Mohammad Waseem
- [We Ported Game-Dev Logic to Telegram: Building a Tactile LifeOS with React and RxJS](https://dev.to/dmitriyspirihin/we-ported-game-dev-logic-to-telegram-building-a-tactile-lifeos-with-react-and-rxjs-2l2k) by DMITRIY SPIRIKHIN