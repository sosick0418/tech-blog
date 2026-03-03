---
layout: post
title: "React 개발자라면 필독! 'useEffect' 늪에서 벗어나는 핵심 멘탈 모델"
date: 2026-03-03T01:00:44.347Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발의 최전선에서 늘 새로운 기술과 트렌드를 탐구하는 전문 블로거, 프론트엔드 마스터입니다!

2026년 3월 3일, 오늘도 프론트엔드 세계는 빠르게 진화하고 있네요. 다양한 흥미로운 소식들 중에서, 오늘은 React 개발자라면 한 번쯤은 머리를 싸매고 고민했을 바로 그 훅, `useEffect`에 대한 깊이 있는 통찰을 다뤄보고자 합니다.

---



`useEffect`는 React Hooks 중 가장 강력하면서도, 동시에 가장 오해받고 잘못 사용되기 쉬운 훅입니다. 잘못 사용하면 무한 렌더링, 메모리 누수, 예측 불가능한 버그의 주범이 되곤 하죠. 오늘 이 글에서는 `useEffect`를 '제대로' 이해하고 활용할 수 있는 핵심 멘탈 모델과 실용적인 팁을 공유하여, 여러분이 `useEffect`의 늪에서 벗어나 더욱 견고하고 효율적인 React 애플리케이션을 만들 수 있도록 돕겠습니다.

## 왜 `useEffect`는 늘 우리를 괴롭힐까요?

많은 개발자들이 `useEffect`를 "컴포넌트가 렌더링된 후에 어떤 작업을 수행할 때" 사용하는 훅으로만 생각합니다. 하지만 이러한 막연한 이해는 수많은 버그의 씨앗이 됩니다. `useEffect`의 본질을 파악하지 못하면, 우리는 불필요한 렌더링을 유발하거나, 의도치 않은 사이드 이펙트를 만들어내기 십상입니다. 이제 그 본질을 파헤쳐 봅시다.

### `useEffect`의 핵심 멘탈 모델: 외부 시스템과의 "동기화"

`useEffect`를 이해하는 가장 중요한 멘탈 모델은 바로 **"컴포넌트 외부의 시스템과 동기화"**하는 도구라는 것입니다. 이는 DOM 조작, 데이터 페칭, 구독 설정 및 해지, 외부 라이브러리 연동 등 React 컴포넌트 내부의 상태와 직접 관련 없는 '사이드 이펙트(side effect)'를 다룰 때 사용됩니다.

`useEffect`는 다음과 같은 세 가지 핵심 요소로 구성됩니다:

1.  **이펙트 함수 (Effect Function):** 실제로 수행할 사이드 이펙트 로직을 담습니다.
2.  **의존성 배열 (Dependency Array):** 이 배열 안의 값이 변경될 때만 이펙트 함수가 다시 실행됩니다. 빈 배열(`[]`)은 컴포넌트가 마운트될 때 한 번만 실행하고, 배열을 생략하면 매 렌더링마다 실행됩니다 (이는 거의 사용하지 않습니다).
3.  **클린업 함수 (Cleanup Function):** 이펙트가 다시 실행되거나 컴포넌트가 언마운트될 때 호출되어, 이펙트 함수가 설정한 외부 리소스나 구독 등을 정리합니다. 메모리 누수를 방지하는 핵심적인 역할이죠.

이펙트 함수는 컴포넌트의 렌더링 결과가 DOM에 반영된 *후*에 비동기적으로 실행됩니다. 즉, `useEffect`는 렌더링 자체에 직접적인 영향을 주지 않고, 렌더링이 완료된 후에 "외부 세계"와 상호작용하기 위한 다리 역할을 합니다.

## `useEffect` 올바르게 사용하기 (feat. 실제 코드 예시)

이제 실제 코드를 통해 `useEffect`의 올바른 사용법과 흔히 저지르는 실수를 살펴보겠습니다. 모든 예시는 TypeScript와 React를 기반으로 합니다.

### 🔴 잘못된 `useEffect` 사용 예시: 파생 상태 관리

많은 개발자들이 `useEffect`를 사용하여 이미 존재하는 상태로부터 파생되는 새로운 상태를 만들곤 합니다. 이는 불필요한 렌더링과 복잡성을 야기합니다.

```typescript
import React, { useState, useEffect } from 'react';

function BadExample() {
  const [count, setCount] = useState<number>(0);
  const [isEven, setIsEven] = useState<boolean>(false);

  // 🔴 잘못된 사용: isEven은 count에 의해 파생되는 상태입니다.
  // useEffect 없이 렌더링 시점에 계산하는 것이 훨씬 효율적입니다.
  useEffect(() => {
    setIsEven(count % 2 === 0);
  }, [count]); // count가 변경될 때마다 이펙트가 실행되어 isEven을 업데이트

  return (
    <div>
      <p>Count: {count}</p>
      <p>Is Even: {isEven ? 'Yes' : 'No'}</p>
      <button onClick={() => setCount(prev => prev + 1)}>Increment</button>
    </div>
  );
}
```

**✅ 개선 방안:** `isEven`은 `count` 값에 따라 즉시 계산될 수 있는 파생 상태이므로, `useEffect` 대신 렌더링 시점에 직접 계산하는 것이 올바릅니다.

```typescript
import React, { useState } from 'react';

function GoodExampleDerivedState() {
  const [count, setCount] = useState<number>(0);

  // ✅ 올바른 사용: 렌더링 시점에 파생 상태를 직접 계산
  const isEven: boolean = count % 2 === 0;

  return (
    <div>
      <p>Count: {count}</p>
      <p>Is Even: {isEven ? 'Yes' : 'No'}</p>
      <button onClick={() => setCount(prev => prev + 1)}>Increment</button>
    </div>
  );
}
```

### ✅ 올바른 `useEffect` 사용 예시 1: 데이터 페칭

외부 API로부터 데이터를 가져오는 것은 대표적인 사이드 이펙트입니다.

```typescript
import React, { useState, useEffect } from 'react';

interface Post {
  id: number;
  title: string;
  body: string;
}

function PostFetcher({ postId }: { postId: number }) {
  const [post, setPost] = useState<Post | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let isMounted = true; // 컴포넌트 마운트 상태 추적을 위한 플래그
    setLoading(true);
    setError(null);
    setPost(null); // 새로운 fetch 시작 시 기존 데이터 초기화

    const fetchPost = async () => {
      try {
        const response = await fetch(`https://jsonplaceholder.typicode.com/posts/${postId}`);
        if (!response.ok) {
          throw new Error(`Error: ${response.status}`);
        }
        const data: Post = await response.json();
        if (isMounted) { // 컴포넌트가 마운트된 상태에서만 상태 업데이트
          setPost(data);
        }
      } catch (err) {
        if (isMounted && err instanceof Error) {
          setError(err.message);
        }
      } finally {
        if (isMounted) {
          setLoading(false);
        }
      }
    };

    fetchPost();

    return () => {
      // 🧹 클린업: 컴포넌트 언마운트 또는 이펙트 재실행 시 플래그 변경
      // 비동기 작업이 완료되기 전에 컴포넌트가 언마운트되어 발생하는 오류 방지
      isMounted = false;
    };
  }, [postId]); // postId가 변경될 때마다 다시 fetch

  if (loading) return <p>Loading post...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!post) return <p>No post found.</p>;

  return (
    <div>
      <h3>{post.title}</h3>
      <p>{post.body}</p>
    </div>
  );
}
```

**핵심:** `postId`가 변경될 때마다 이펙트가 다시 실행되어 새로운 데이터를 가져옵니다. `isMounted` 플래그를 이용한 클린업은 비동기 작업 중 컴포넌트가 언마운트되는 경우 발생할 수 있는 메모리 누수 경고를 방지합니다.

### ✅ 올바른 `useEffect` 사용 예시 2: 이벤트 리스너 등록/해제

브라우저 이벤트 리스너를 등록하고 해제하는 것도 `useEffect`의 중요한 역할입니다.

```typescript
import React, { useState, useEffect } from 'react';

function WindowSizeLogger() {
  const [width, setWidth] = useState<number>(window.innerWidth);
  const [height, setHeight] = useState<number>(window.innerHeight);

  useEffect(() => {
    const handleResize = () => {
      setWidth(window.innerWidth);
      setHeight(window.innerHeight);
    };

    window.addEventListener('resize', handleResize);

    return () => {
      // 🧹 클린업: 컴포넌트 언마운트 시 이벤트 리스너 제거
      window.removeEventListener('resize', handleResize);
    };
  }, []); // 빈 배열: 마운트 시 한 번만 등록, 언마운트 시 해제

  return (
    <p>Window Size: {width}px x {height}px</p>
  );
}
```

**핵심:** 빈 의존성 배열(`[]`)을 사용하여 컴포넌트가 마운트될 때 한 번만 이벤트 리스너를 등록하고, 클린업 함수에서 언마운트 시 리스너를 제거하여 메모리 누수를 방지합니다.

## 실무 적용 팁: `useEffect`를 현명하게 다루는 법

1.  **`useEffect`는 최후의 수단:** 컴포넌트 내에서 상태나 props를 통해 해결할 수 있는 문제는 먼저 그 방법을 사용하세요. `useEffect`는 외부 시스템과 연동해야 할 때만 사용하세요.
2.  **의존성 배열을 정확히:** `eslint-plugin-react-hooks`의 `exhaustive-deps` 룰을 적극 활용하세요. 이 룰은 의존성 배열에 빠진 요소가 없는지 자동으로 알려주어 잠재적인 버그를 예방합니다.
3.  **클린업 함수는 필수:** 타이머, 구독, 이벤트 리스너 등 `useEffect` 내부에서 생성된 모든 외부 리소스는 클린업 함수를 통해 반드시 해제해야 합니다.
4.  **커스텀 훅으로 추상화:** 복잡한 `useEffect` 로직은 재사용 가능한 커스텀 훅으로 분리하여 코드를 깔끔하게 유지하고 로직을 추상화하세요. (예: `useFetch`, `useEventListener`)
5.  **Strict Mode 활용:** 개발 모드에서 React의 `StrictMode`를 사용하면 `useEffect`가 두 번 실행되어 클린업 로직의 잠재적 문제를 미리 발견하는 데 도움이 됩니다.

## 마무리하며: `useEffect`를 당신의 강력한 도구로!

`useEffect`는 React 컴포넌트가 외부 세계와 상호작용하는 데 필수적인 강력한 도구입니다. 하지만 그만큼 올바른 이해와 사용이 중요하죠. 오늘 배운 **"외부 시스템과의 동기화"**라는 멘탈 모델을 기억하고, 의존성 배열과 클린업 함수를 정확히 활용한다면 더 이상 `useEffect`가 여러분의 발목을 잡는 일이 없을 것입니다.

React의 생태계는 계속 발전하고 있으며, React Server Components와 같은 새로운 패러다임은 미래에 `useEffect`의 사용처를 더욱 줄여줄 수도 있습니다. 하지만 현재로서는 `useEffect`를 마스터하는 것이 견고한 프론트엔드 개발자가 되기 위한 핵심 역량임을 잊지 마세요!

이 글이 여러분의 `useEffect` 사용에 대한 이해를 한 단계 끌어올리는 데 도움이 되었기를 바랍니다. 다음에도 더 유익한 프론트엔드 기술 트렌드와 팁으로 찾아오겠습니다!

---

## 참고 자료

*   [Building a Real-time Multiplayer Game with React + Firebase - MMA XOX Case Study](https://dev.to/jebolwski/building-a-real-time-multiplayer-game-with-react-firebase-mma-xox-case-study-1ni7) by Mert Gökmen
*   [Stop Writing useEffect Wrong: The Mental Model That Changes Everything](https://dev.to/teguh_coding/stop-writing-useeffect-wrong-the-mental-model-that-changes-everything-4mdp) by Teguh Coding
*   [I Built a Multi-Provider AI Chat App with Django and React — Here's What Actually Happened](https://dev.to/arnabsahawrk/i-built-a-multi-provider-ai-chat-app-with-django-and-react-heres-what-actually-happened-49) by Arnab Saha
*   [What I Learned Building a Habit Tracker with React + AWS Amplify Gen 2](https://dev.to/bi_kaizen/what-i-learned-building-a-habit-tracker-with-react-aws-amplify-gen-2-5dge) by Dmytro
*   [Why React Developers Can Finally Delete Half Their Performance Code](https://dev.to/_itsglover/why-react-developers-can-finally-delete-half-their-performance-code-1oh9) by Glover