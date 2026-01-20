---
layout: post
title: "`useEffectEvent`의 등장: React 19에서 Side Effect를 깔끔하게 다루는 법"
date: 2026-01-20T01:53:50.948Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거 개발자 K입니다. 2026년 1월 20일, 오늘도 여러분께 프론트엔드 기술의 흥미로운 소식을 전해드리려 합니다.

React 개발자라면 `useEffect` 훅이 얼마나 강력하면서도 때로는 다루기 까다로운지 잘 아실 겁니다. `deps` 배열 관리의 어려움, 불필요한 `effect` 재실행, 그리고 오래된 클로저(stale closure) 문제까지. 이러한 고충을 해결하기 위해 React 19에서 새로운 훅인 `useEffectEvent`가 등장했습니다. 오늘은 이 `useEffectEvent` 훅이 무엇이며, 어떻게 우리의 `useEffect` 라이프를 더 깔끔하고 효율적으로 만들어줄지 깊이 파헤쳐 보겠습니다.

---



## 1. 소개: `useEffect`의 고통, `useEffectEvent`가 끝낸다?

React 개발에서 `useEffect` 훅은 컴포넌트의 생명주기 동안 발생하는 다양한 사이드 이펙트(데이터 fetching, 구독 설정, DOM 직접 조작 등)를 관리하는 핵심 도구입니다. 하지만 `useEffect`의 `deps` 배열에 함수를 포함해야 할 때마다, 해당 함수가 `props`나 `state`에 의존하여 자주 재생성되면 `effect`가 불필요하게 재실행되는 문제가 발생하곤 했습니다. 이는 성능 저하뿐만 아니라 예측하기 어려운 버그의 원인이 되기도 했죠.

이러한 `useEffect`의 고질적인 문제를 해결하고자, React 19에서는 `useEffectEvent`라는 새로운 훅을 도입했습니다. `useEffectEvent`는 `effect` 내부에서 최신 `props`와 `state`에 안정적으로 접근할 수 있는 "이벤트 핸들러"를 생성하여, `effect`의 `deps` 배열을 더 간결하고 의미 있게 유지할 수 있도록 돕습니다. 이제 더 이상 불필요한 `effect` 재실행에 대한 걱정 없이, 오직 `effect`의 핵심 로직에만 집중할 수 있게 된 것입니다.

## 2. 본문: `useEffectEvent`의 핵심과 실전 활용

### 핵심 개념 설명: `useEffect`의 한계와 `useEffectEvent`의 등장

기존 `useEffect`의 가장 큰 문제 중 하나는 `deps` 배열에 함수를 포함해야 할 때 발생합니다. 예를 들어, `effect` 안에서 부모 컴포넌트로부터 받은 콜백 함수(`onSomething`)를 호출해야 한다고 가정해 봅시다. 만약 `onSomething` 함수가 `useCallback`으로 메모이제이션되지 않았다면, 부모 컴포넌트가 리렌더링될 때마다 `onSomething`은 새로운 참조를 가지게 되고, 이는 `effect`의 재실행을 유발합니다. `onSomething`이 단순히 로깅이나 분석 이벤트를 전송하는 함수라면, `effect`가 재실행될 필요가 없는데도 말이죠.

이 문제를 해결하기 위해 `onSomething`을 `deps`에서 제거하면, `effect` 내부의 `onSomething`은 `effect`가 처음 실행될 때의 오래된 `onSomething`을 참조하는 "오래된 클로저(stale closure)" 문제가 발생합니다. 최신 `state`나 `props`에 접근하지 못하는 것이죠. `useCallback`을 사용하여 `onSomething`을 안정화할 수도 있지만, 이 역시 `useCallback`의 `deps` 관리가 필요하며, `effect` 내부에서 일어나는 모든 이벤트성 로직에 `useCallback`을 적용하는 것은 번거로울 수 있습니다.

여기서 `useEffectEvent`가 빛을 발합니다. `useEffectEvent`는 `effect`의 `deps` 배열에 포함되어도 안정적인(stable) 참조를 가지면서, 호출될 때는 항상 최신 `props`와 `state`에 접근할 수 있는 특별한 함수를 생성합니다. 마치 `effect` 내부에 선언된 이벤트 핸들러처럼 작동하며, `effect`의 핵심 로직(구독, 타이머 설정 등)과 이벤트성 로직(데이터 전송, 로깅 등)을 명확하게 분리할 수 있게 해줍니다.

**`useEffectEvent`의 주요 특징:**

*   `effect` 내부에서만 호출 가능합니다.
*   `effect`의 `deps` 배열에 포함되어도 안정적인 참조를 가집니다. (즉, `effect`가 이 함수 때문에 재실행되지 않습니다.)
*   호출될 때는 항상 최신 `props`와 `state`에 접근합니다.
*   `useCallback`과 달리, 함수 자체를 메모이제이션하는 것이 아니라 `effect` 내 이벤트 로직을 분리하는 데 초점을 맞춥니다.

### 실제 코드 예시 (React/TypeScript 활용)

`ChatRoom` 컴포넌트를 예로 들어, `useEffectEvent`가 어떻게 `effect`를 개선하는지 살펴보겠습니다. 이 컴포넌트는 특정 채팅방에 연결하고, 메시지를 보내는 로직을 가지고 있습니다. `onNewMessage` 콜백을 통해 부모 컴포넌트에 새로운 메시지를 알립니다.

```tsx
// React 19 환경을 가정합니다.
import React, { useState, useEffect, useCallback } from 'react';
// React 19에서는 'react'에서 useEffectEvent를 직접 import할 수 있습니다.
// 현재는 TypeScript 정의를 위해 가상으로 선언합니다.
declare function useEffectEvent<Args extends any[], R>(callback: (...args: Args) => R): (...args: Args) => R;

// =========================================================================
// Case 1: useEffectEvent를 사용하지 않았을 때의 문제점 (가정)
// =========================================================================
// function ChatRoomProblematic({
//   roomId,
//   onNewMessage,
// }: {
//   roomId: string;
//   onNewMessage: (message: string) => void;
// }) {
//   const [messageCount, setMessageCount] = useState(0);

//   useEffect(() => {
//     console.log(`[Problematic] Connecting to chat room: ${roomId}`);
//     const connection = {
//       connect: () => console.log(`[Problematic] Connected to ${roomId}`),
//       disconnect: () => console.log(`[Problematic] Disconnected from ${roomId}`),
//       sendMessage: (msg: string) => {
//         console.log(`[Problematic] Sending message "${msg}" to ${roomId}`);
//         // onNewMessage가 deps에 없으면 stale closure, 있으면 불필요한 effect 재실행
//         onNewMessage(`[${roomId}] ${msg}`);
//         setMessageCount(prev => prev + 1);
//       },
//     };
//     connection.connect();

//     // 1초 후 메시지 전송 시뮬레이션
//     const timer = setTimeout(() => {
//       connection.sendMessage(`Hello from ${roomId} (count: ${messageCount})`);
//     }, 1000);

//     return () => {
//       clearTimeout(timer);
//       connection.disconnect();
//     };
//   }, [roomId, onNewMessage, messageCount]); // onNewMessage, messageCount 때문에 effect가 자주 재실행될 수 있음

//   return (
//     <div>
//       <h3>[Problematic] Chat Room: {roomId}</h3>
//       <p>Messages sent: {messageCount}</p>
//     </div>
//   );
// }

// =========================================================================
// Case 2: useEffectEvent를 사용하여 개선된 코드
// =========================================================================
function ChatRoomImproved({
  roomId,
  onNewMessage,
}: {
  roomId: string;
  onNewMessage: (message: string) => void;
}) {
  const [messageCount, setMessageCount] = useState(0);

  // useEffectEvent를 사용하여 onNewMessage 로직을 안정적인 이벤트로 래핑합니다.
  // 이 함수는 deps에 포함되지 않아도 항상 최신 onNewMessage와 messageCount에 접근합니다.
  const handleNewMessageEvent = useEffectEvent((msg: string) => {
    onNewMessage(msg); // 항상 최신 onNewMessage 참조
    setMessageCount((prev) => prev + 1); // 항상 최신 messageCount 참조
  });

  useEffect(() => {
    console.log(`[Improved] Connecting to chat room: ${roomId}`);
    const connection = {
      connect: () => console.log(`[Improved] Connected to ${roomId}`),
      disconnect: () => console.log(`[Improved] Disconnected from ${roomId}`),
      sendMessage: (msg: string) => {
        console.log(`[Improved] Sending message "${msg}" to ${roomId}`);
        // 이제 안정적인 handleNewMessageEvent를 호출합니다.
        handleNewMessageEvent(`[${roomId}] ${msg}`);
      },
    };
    connection.connect();

    // 1초 후 메시지 전송 시뮬레이션
    const timer = setTimeout(() => {
      // handleNewMessageEvent 내부에서 최신 messageCount를 참조하므로,
      // 여기서는 직접 전달할 필요가 없습니다.
      connection.sendMessage(`Hello from ${roomId}`);
    }, 1000);

    return () => {
      clearTimeout(timer);
      connection.disconnect();
    };
  }, [roomId, handleNewMessageEvent]); // handleNewMessageEvent는 안정적 참조이므로 effect는 roomId 변경 시에만 재실행됩니다.

  return (
    <div>
      <h3>[Improved] Chat Room: {roomId}</h3>
      <p>Messages sent: {messageCount}</p>
    </div>
  );
}

// App 컴포넌트 (사용 예시)
function App() {
  const [currentRoom, setCurrentRoom] = useState('general');

  // onNewMessage 콜백은 안정적이어도 되고, 불안정해도 useEffectEvent 덕분에 문제가 없습니다.
  // 여기서는 useCallback을 사용하여 안정적으로 만들었지만, 필수적이지 않습니다.
  const handleMessage = useCallback((msg: string) => {
    console.log('App received message:', msg);
  }, []);

  return (
    <div style={{ padding: '20px', fontFamily: 'sans-serif' }}>
      <h1>`useEffectEvent` 데모</h1>
      <div style={{ marginBottom: '20px' }}>
        <button onClick={() => setCurrentRoom('tech')} style={{ marginRight: '10px', padding: '8px 15px' }}>
          Join Tech Room
        </button>
        <button onClick={() => setCurrentRoom('random')} style={{ padding: '8px 15px' }}>
          Join Random Room
        </button>
      </div>
      <ChatRoomImproved roomId={currentRoom} onNewMessage={handleMessage} />
      <p style={{ marginTop: '20px', fontSize: '0.9em', color: '#666' }}>
        `roomId`를 변경하여 `effect`가 재실행되는 것을, 메시지 전송 로직은 안정적으로 동작하는 것을 확인해보세요.
      </p>
    </div>
  );
}

export default App;
```

위 `ChatRoomImproved` 예시에서 `handleNewMessageEvent`는 `useEffectEvent`로 래핑되어 있습니다. 이 덕분에 `useEffect`의 `deps` 배열에 `handleNewMessageEvent`를 포함하더라도, 이 함수는 안정적인 참조를 유지하므로 `roomId`가 변경될 때만 `effect`가 재실행됩니다. `handleNewMessageEvent` 내부에서는 `onNewMessage`와 `messageCount`의 최신 값을 항상 참조할 수 있어, 오래된 클로저 문제도 발생하지 않습니다.

### 실무 적용 팁

1.  **언제 `useEffectEvent`를 사용해야 하는가?**
    *   `useEffect` 내부에서 최신 `props`나 `state`에 접근해야 하지만, 해당 `props`/`state`의 변경이 `effect`의 재실행을 유발해서는 안 될 때.
    *   특히, 로깅, 분석 이벤트 전송, 외부 API 호출 시 콜백 함수 등 `effect`의 "이벤트성" 로직을 분리하는 데 매우 유용합니다.
    *   `effect`의 `deps` 배열이 너무 길어지거나, 불필요한 재실행으로 인해 성능 문제가 의심될 때 고려해볼 수 있습니다.
2.  **`effect`의 `deps` 배열 간결화:** `useEffectEvent`를 사용하면 `effect`의 `deps` 배열에서 `props`나 `state`에 의존하는 콜백 함수들을 제거할 수 있어, `deps` 배열을 더 간결하고 의미 있게 유지할 수 있습니다. 이는 코드 가독성 향상에도 기여합니다.
3.  **`useEffectEvent`는 `effect` 외부에서 호출할 수 없습니다.** `useEffectEvent`로 생성된 함수는 오직 `useEffect`의 콜백 함수 내부에서만 호출되어야 합니다. 이는 이 훅의 설계 의도와 관련이 있습니다.
4.  **`useCallback`과의 차이:** `useCallback`은 특정 함수 자체를 메모이제이션하여 참조 안정성을 보장하지만, `useEffectEvent`는 `effect` 내부에서 호출될 이벤트성 로직이 최신 값에 접근하면서도 `effect`의 재실행을 유발하지 않도록 돕는 데 특화되어 있습니다. 두 훅은 서로 다른 목적을 가지고 있으며, 상황에 맞게 적절히 활용해야 합니다.

## 3. 마무리: 더 예측 가능한 React Side Effect 관리

`useEffectEvent`는 React 19에서 `useEffect`의 복잡성을 줄이고, 코드의 예측 가능성을 높이는 데 기여할 중요한 도구입니다. 이 훅을 통해 우리는 `effect`의 핵심 로직과 이벤트성 로직을 명확하게 분리하고, 불필요한 `effect` 재실행과 오래된 클로저 문제로부터 자유로워질 수 있습니다.

React 팀은 `useEffectEvent`와 같은 새로운 훅을 통해 개발자들이 더욱 견고하고 효율적인 애플리케이션을 구축할 수 있도록 지속적으로 노력하고 있습니다. React 19의 도입은 이러한 노력의 결실이며, 프론트엔드 개발자들에게 더욱 강력하고 유연한 도구를 제공할 것입니다.

앞으로는 `useEffectEvent`를 실제 프로젝트에 적용해보고, React 19에서 새롭게 추가되거나 개선된 다른 기능들(예: `use` 훅, 서버 컴포넌트의 발전 등)도 함께 탐색하며 React 생태계의 변화에 발맞춰 나가는 것이 중요할 것입니다.

---

## 참고 자료

*   [Stop Configuring, Start Shipping: My Ultimate FastAPI + React SaaS Stack](https://dev.to/mehdidch/stop-configuring-start-shipping-my-ultimate-fastapi-react-saas-stack-2ad5) by Mehdi Chafai
*   [Adding Multi-Lingual Support in React](https://dev.to/shubham_gupta_6f6c50fefd4/adding-multi-lingual-support-in-react-i90) by shubham gupta
*   [Building in Public: CV Analyzer - Act 2 · Scene 1 - Laying Down the App’s Spine (Main Navigation)](https://dev.to/voke_vawkei/building-in-public-cv-analyzer-act-2-scene-1-laying-down-the-apps-spine-main-navigation-2i89) by Voke
*   [ReactJS Hook Pattern ~useEffectEvent Pattern~](https://dev.to/kkr0423/reactjs-hook-pattern-useeffectevent-pattern-29d) by Ogasawara Kakeru
*   [Stop Configuring Telegram Bots via Commands: Building a Visual Moderator with Mini Apps](https://dev.to/shuvalov/stop-configuring-telegram-bots-via-commands-building-a-visual-moderator-with-mini-apps-3ac7) by DMITRY