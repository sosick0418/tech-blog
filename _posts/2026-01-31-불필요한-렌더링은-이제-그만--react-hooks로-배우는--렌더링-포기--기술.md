---
layout: post
title: "불필요한 렌더링은 이제 그만! React Hooks로 배우는 '렌더링 포기' 기술"
date: 2026-01-31T01:01:03.152Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발의 최전선에서 여러분과 함께하는 전문 기술 블로거입니다!
오늘 날짜는 2026년 1월 31일, 프론트엔드 세계는 여전히 빠르게 진화하고 있습니다. 오늘은 특히 React 애플리케이션의 성능 최적화에 대한 깊이 있는 통찰을 제공할 주제를 가져왔습니다. 바로 "렌더링 포기" 전략입니다.

---



프론트엔드 애플리케이션의 성능은 사용자 경험에 직접적인 영향을 미칩니다. 특히 React와 같은 컴포넌트 기반 프레임워크에서는 불필요한 렌더링이 발생하여 애플리케이션 속도를 저하시키는 경우가 많죠. 오늘 우리는 React Hooks를 활용하여 이러한 불필요한 렌더링을 "포기"하고, 더욱 빠르고 효율적인 애플리케이션을 구축하는 비법을 파헤쳐 보겠습니다.

## 렌더링 최적화, 왜 중요할까요?

React는 상태(state)나 프롭스(props)가 변경되면 컴포넌트를 다시 렌더링하는 방식으로 UI를 업데이트합니다. 이는 선언적 UI의 장점이지만, 때로는 실제 DOM에 변경이 없거나 사용자에게 보이지 않는 부분까지 불필요하게 다시 계산하는 비효율을 낳기도 합니다. 이런 불필요한 렌더링은 CPU 자원을 소모하고, 배터리 수명을 단축시키며, 결국 사용자 인터페이스의 버벅거림(jank)으로 이어져 좋지 않은 사용자 경험을 제공하게 됩니다.

이러한 문제를 해결하기 위해 우리는 "렌더링 포기" 전략을 사용합니다. 여기서 "포기"란 말 그대로 렌더링을 아예 하지 않는다는 뜻이 아니라, *불필요한* 렌더링을 스마트하게 막아 React가 해야 할 작업을 줄이는 것을 의미합니다. React Hooks는 이 과정을 더욱 간결하고 강력하게 만들어 줍니다.

## 핵심 개념: React Hooks를 이용한 렌더링 제어

React에서 렌더링을 제어하는 핵심 도구는 `React.memo`, `useCallback`, `useMemo`, 그리고 `useRef`입니다. 이들을 조합하면 컴포넌트의 렌더링 조건을 섬세하게 조절할 수 있습니다.

1.  **`React.memo`:**
    함수형 컴포넌트의 프롭스가 변경되지 않았다면 리렌더링을 건너뛰도록 지시합니다. 클래스 컴포넌트의 `PureComponent`와 유사하지만, 함수형 컴포넌트에 특화되어 있습니다. 기본적으로 얕은 비교(shallow comparison)를 수행합니다.

2.  **`useCallback`:**
    컴포넌트가 리렌더링될 때마다 새로운 함수가 생성되는 것을 방지합니다. 특정 의존성 배열(dependencies array)의 값이 변경되지 않는 한, 이전에 생성된 함수를 재사용하게 하여 `React.memo`로 감싸진 자식 컴포넌트에 함수를 프롭스로 전달할 때 불필요한 리렌더링을 막는 데 매우 유용합니다.

3.  **`useMemo`:**
    `useCallback`이 함수를 메모이제이션(memoization)하는 반면, `useMemo`는 특정 값(객체, 배열 등)을 메모이제이션합니다. 의존성 배열의 값이 변경되지 않는 한, 이전에 계산된 값을 재사용하여 고비용 계산을 피하고 `React.memo`의 얕은 비교를 통과하게 합니다.

4.  **`useRef`:**
    렌더링에 영향을 주지 않으면서 변경 가능한(mutable) 값을 보존하고 싶을 때 사용합니다. DOM 요소에 직접 접근하거나, 렌더링 사이에 특정 값을 유지해야 하지만 그 값이 변경될 때 컴포넌트가 리렌더링될 필요가 없을 때 유용합니다.

## 실제 코드 예시: '렌더링 포기' 전략 적용하기

Next.js 환경에서 TypeScript를 사용하여 예시를 살펴보겠습니다.

### 문제 상황: 불필요한 자식 컴포넌트 렌더링

먼저, 부모 컴포넌트의 상태가 변경될 때마다 자식 컴포넌트가 불필요하게 리렌더링되는 상황을 보겠습니다.

```tsx
// components/ChildComponent.tsx
import React from 'react';

interface ChildProps {
  message: string;
  onButtonClick: () => void;
  data: { id: number; name: string };
}

export function ChildComponent({ message, onButtonClick, data }: ChildProps) {
  // 부모 컴포넌트가 렌더링될 때마다 이 로그가 출력됩니다.
  console.log('ChildComponent 렌더링');
  return (
    <div style={{ border: '1px solid gray', padding: '10px', margin: '10px' }}>
      <h3>자식 컴포넌트</h3>
      <p>메시지: {message}</p>
      <p>데이터 이름: {data.name}</p>
      <button onClick={onButtonClick}>자식 버튼 (부모 카운트 증가)</button>
    </div>
  );
}

// pages/index.tsx (혹은 ParentComponent.tsx)
import React, { useState } from 'react';
import { ChildComponent } from '../components/ChildComponent';

export default function HomePage() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  // 텍스트 입력 시 부모 컴포넌트와 자식 컴포넌트 모두 렌더링됩니다.
  console.log('ParentComponent 렌더링');

  const incrementCount = () => setCount(prev => prev + 1);
  const staticData = { id: 1, name: '정적 데이터' }; // 매 렌더링마다 새로운 객체 생성

  return (
    <div>
      <h1>부모 컴포넌트</h1>
      <p>카운트: {count}</p>
      <button onClick={incrementCount}>카운트 증가</button>
      <input
        type="text"
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="텍스트를 입력하세요"
        style={{ margin: '10px 0' }}
      />
      <ChildComponent
        message="저는 최적화되기 전의 자식입니다."
        onButtonClick={incrementCount}
        data={staticData}
      />
    </div>
  );
}
```
위 코드에서 `HomePage`의 `text` 상태가 변경될 때마다 `HomePage`와 `ChildComponent` 모두 "렌더링" 로그를 출력합니다. `ChildComponent`의 `message`, `onButtonClick`, `data` 프롭스는 `text` 상태와 무관함에도 불구하고 말이죠. 특히 `onButtonClick` 함수와 `staticData` 객체는 매 렌더링마다 새로운 참조값을 가지게 되어 `ChildComponent`가 프롭스가 변경되었다고 인식하게 됩니다.

### 해결책: `React.memo`, `useCallback`, `useMemo` 활용

이제 `React.memo`, `useCallback`, `useMemo`를 사용하여 자식 컴포넌트의 불필요한 렌더링을 막아보겠습니다.

```tsx
// components/MemoizedChildComponent.tsx
import React from 'react';

interface ChildProps {
  message: string;
  onButtonClick: () => void;
  data: { id: number; name: string };
}

// React.memo로 감싸서 프롭스 변경이 없을 경우 리렌더링을 방지합니다.
export const MemoizedChildComponent = React.memo(function ChildComponent({ message, onButtonClick, data }: ChildProps) {
  console.log('MemoizedChildComponent 렌더링'); // 이제 이 로그는 프롭스 변경 시에만 출력됩니다.
  return (
    <div style={{ border: '1px solid green', padding: '10px', margin: '10px' }}>
      <h3>최적화된 자식 컴포넌트</h3>
      <p>메시지: {message}</p>
      <p>데이터 이름: {data.name}</p>
      <button onClick={onButtonClick}>자식 버튼 (부모 카운트 증가)</button>
    </div>
  );
});

// pages/index.tsx (혹은 ParentComponent.tsx)
import React, { useState, useCallback, useMemo } from 'react';
import { MemoizedChildComponent } from '../components/MemoizedChildComponent'; // 변경된 컴포넌트 import

export default function HomePage() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  console.log('ParentComponent 렌더링');

  // useCallback으로 incrementCount 함수를 메모이제이션합니다.
  // 의존성 배열이 비어있으므로, 컴포넌트가 처음 마운트될 때 한 번만 생성됩니다.
  const incrementCount = useCallback(() => setCount(prev => prev + 1), []);

  // useMemo로 staticData 객체를 메모이제이션합니다.
  // 의존성 배열이 비어있으므로, 컴포넌트가 처음 마운트될 때 한 번만 생성됩니다.
  const staticData = useMemo(() => ({ id: 1, name: '정적 데이터' }), []);

  return (
    <div>
      <h1>부모 컴포넌트</h1>
      <p>카운트: {count}</p>
      <button onClick={incrementCount}>카운트 증가</button>
      <input
        type="text"
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="텍스트를 입력하세요"
        style={{ margin: '10px 0' }}
      />
      <MemoizedChildComponent
        message="저는 최적화된 자식입니다."
        onButtonClick={incrementCount} // 메모이제이션된 함수 전달
        data={staticData} // 메모이제이션된 객체 전달
      />
    </div>
  );
}
```

이제 `HomePage`에서 텍스트를 입력해도 `MemoizedChildComponent`는 "렌더링" 로그를 출력하지 않습니다. 이는 `React.memo`가 프롭스의 얕은 비교를 통해 `message`, `onButtonClick`, `data` 프롭스가 변경되지 않았음을 감지했기 때문입니다. `onButtonClick`과 `data`는 `useCallback`과 `useMemo` 덕분에 부모 컴포넌트가 리렌더링되어도 동일한 참조값을 유지합니다.

### `useRef` 활용 (렌더링과 무관한 값 저장)

`useRef`는 렌더링을 유발하지 않고 변경 가능한 값을 저장할 때 유용합니다. 예를 들어, 특정 DOM 요소에 접근하거나, 컴포넌트의 라이프사이클 동안 유지되어야 하는 타이머 ID 등을 저장할 때 사용합니다.

```tsx
import React, { useRef, useEffect } from 'react';

export default function TimerComponent() {
  const timerIdRef = useRef<number | null>(null); // 렌더링을 유발하지 않고 값을 유지

  useEffect(() => {
    // 컴포넌트 마운트 시 타이머 시작
    timerIdRef.current = window.setInterval(() => {
      console.log('타이머 작동 중...');
    }, 1000);

    // 컴포넌트 언마운트 시 타이머 정리
    return () => {
      if (timerIdRef.current) {
        window.clearInterval(timerIdRef.current);
      }
    };
  }, []); // 빈 의존성 배열로 한 번만 실행

  const stopTimer = () => {
    if (timerIdRef.current) {
      window.clearInterval(timerIdRef.current);
      console.log('타이머 중지됨.');
    }
  };

  return (
    <div>
      <h2>useRef를 이용한 타이머</h2>
      <p>콘솔을 확인하세요. 이 컴포넌트는 타이머 값 변경으로 리렌더링되지 않습니다.</p>
      <button onClick={stopTimer}>타이머 중지</button>
    </div>
  );
}
```
`timerIdRef.current`의 값은 변경되지만, 이 변경이 컴포넌트의 리렌더링을 유발하지 않습니다. 이는 상태(state)의 변경과는 다른 `useRef`의 중요한 특징입니다.

## 실무 적용 팁

1.  **무분별한 최적화는 독!:** 모든 컴포넌트에 `React.memo`를 적용하거나 모든 함수/객체를 `useCallback`/`useMemo`로 감싸는 것은 오히려 오버헤드를 발생시킬 수 있습니다. React Dev Tools의 프로파일러(Profiler)를 사용하여 렌더링 비용이 높은 컴포넌트를 식별하고, 그 부분에 집중하여 최적화하는 것이 현명합니다.
2.  **프롭스 드릴링 피하기:** 깊은 계층의 자식 컴포넌트에 프롭스를 전달해야 할 때는 Context API나 Recoil, Zustand 같은 전역 상태 관리 라이브러리를 고려해보세요. 이는 불필요한 중간 컴포넌트의 리렌더링을 줄이는 데 도움이 됩니다.
3.  **의존성 배열 관리:** `useCallback`과 `useMemo`의 의존성 배열(`[]`)은 매우 중요합니다. 배열 내의 값이 변경될 때만 메모이제이션된 값이 다시 계산되도록 주의 깊게 관리해야 합니다. 불필요하게 빈 배열을 사용하거나, 반대로 필요한 의존성을 누락하면 버그를 유발할 수 있습니다.
4.  **Custom Hooks 활용:** 반복적으로 사용되는 렌더링 최적화 로직이 있다면 Custom Hook으로 만들어 재사용성을 높일 수 있습니다. 예를 들어, 특정 값이 변경될 때만 실행되는 `useDebounce` 훅 같은 것을 만들 수 있습니다.

## 마무리하며

오늘 우리는 React Hooks를 활용한 "렌더링 포기" 전략, 즉 불필요한 리렌더링을 현명하게 제어하는 방법에 대해 깊이 있게 알아보았습니다. `React.memo`, `useCallback`, `useMemo`, `useRef`는 React 애플리케이션의 성능을 한 단계 끌어올릴 수 있는 강력한 도구들입니다.

하지만 기억하세요. 최적화는 항상 "필요할 때" 그리고 "측정 후에" 이루어져야 합니다. 과도한 최적화는 코드의 복잡성을 높이고 유지보수를 어렵게 만들 수 있습니다. React Dev Tools의 프로파일러를 적극 활용하여 애플리케이션의 병목 지점을 정확히 파악하고, 오늘 배운 기술들을 적재적소에 적용하여 사용자에게 최고의 경험을 제공하는 프론트엔드 개발자가 되시길 바랍니다!

다음 시간에는 React 애플리케이션의 성능 측정 도구에 대해 더 자세히 알아보는 시간을 가져보겠습니다.

---

## 참고 자료
- [ReactJS Hook Pattern ~Abandon Rendering~](https://dev.to/bishoy_semsem/reactjs-hook-pattern-abandon-rendering-477k) by Bishoy Semsem
- [This Week In React #266 : DoS, shadcn, Skills, Rspack | Expo 55 beta, Hermes, Expo Router | TC39, Rolldown, Yarn](https://dev.to/sebastienlorber/this-week-in-react-266-dos-shadcn-skills-rspack-expo-55-beta-hermes-expo-router-tc39-2h0p) by Sebastien Lorber
- [Introducing BoldKit: A Neubrutalism React Component Library](https://dev.to/aniruddha_agarwal_d3d6b0e/introducing-boldkit-a-neubrutalism-react-component-library-36g1) by Aniruddha Agarwal
- [Overcoming Geo-Restrictions in React: A Security Researcher’s Approach to Testing Blocked Features](https://dev.to/mohammad_waseem_c31f3a26f/overcoming-geo-restrictions-in-react-a-security-researchers-approach-to-testing-blocked-features-1i78) by Mohammad Waseem
- [We Ported Game-Dev Logic to Telegram: Building a Tactile LifeOS with React and RxJS](https://dev.to/dmitriyspirihin/we-ported-game-dev-logic-to-telegram-building-a-tactile-lifeos-with-react-and-rxjs-2l2k) by DMITRIY SPIRIKHIN