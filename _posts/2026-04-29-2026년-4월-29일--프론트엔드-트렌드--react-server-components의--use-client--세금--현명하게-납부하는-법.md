---
layout: post
title: "2026년 4월 29일, 프론트엔드 트렌드: React Server Components의 'use client' 세금, 현명하게 납부하는 법"
date: 2026-04-29T01:01:07.614Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발의 최전선에서 늘 새로운 지식을 탐구하는 여러분!
오늘은 2026년 4월 29일입니다. 빠르게 변화하는 프론트엔드 생태계 속에서 여전히 뜨거운 감자인 React Server Components(RSC)와 `use client` 지시문에 대해 이야기해보려 합니다.

---



## 💡 소개: 성능과 개발 경험 사이의 균형을 찾아서

Next.js 13부터 도입된 React Server Components(RSC)는 웹 애플리케이션의 초기 로딩 성능을 혁신적으로 개선하며 개발 패러다임을 바꾸고 있습니다. 하지만 이 강력한 도구에는 'use client'라는 특별한 지시문이 동반되며, 이를 어떻게 사용하느냐에 따라 성능 최적화라는 본래의 목표를 달성하지 못하고 오히려 불필요한 클라이언트 번들 크기 증가라는 '세금'을 납부하게 될 수도 있습니다. 오늘은 이 'use client'의 본질과, 이를 현명하게 활용하여 성능과 개발 경험 모두를 잡는 방법에 대해 깊이 있게 탐구해 보겠습니다.

## 🚀 본문: 'use client'의 두 얼굴과 실전 전략

### 핵심 개념 설명: RSC와 'use client'의 역할

React Server Components(RSC)는 기본적으로 서버에서 렌더링되고, 클라이언트 측 JavaScript 번들에 포함되지 않습니다. 이는 초기 페이지 로딩 속도 향상, 서버 자원 접근 용이성, 보안 강화 등 여러 이점을 제공합니다. 하지만 모든 UI가 서버에서만 렌더링될 수는 없습니다. 사용자 상호작용(클릭, 입력), 상태 관리(useState, useReducer), 브라우저 API 접근(localStorage, window 객체) 등 클라이언트 측에서만 가능한 기능들이 필요하죠.

이때 등장하는 것이 바로 `'use client'` 지시문입니다. 이 지시문은 파일의 최상단에 선언되어, 해당 파일과 그 의존성(import하는 모듈들)이 클라이언트 번들에 포함되어 클라이언트 측에서 렌더링되고 상호작용할 수 있도록 명시합니다. 즉, RSC가 '서버 우선'의 철학을 따른다면, `use client`는 '클라이언트 전환점'의 역할을 하는 셈입니다.

문제는 이 `'use client'` 지시문이 단순히 해당 컴포넌트 하나만을 클라이언트 컴포넌트로 만드는 것이 아니라는 점입니다. `'use client'`가 선언된 파일은 물론, 이 파일이 `import`하는 모든 모듈(심지어는 서버 컴포넌트로 설계될 수도 있었던 모듈까지)이 클라이언트 번들에 포함될 가능성이 커집니다. 이것이 바로 원문에서 언급된 "The 'use client' Tax"이자, "파일 단위 경계"의 문제입니다. 작은 상호작용 하나를 위해 거대한 컴포넌트 트리 전체가 클라이언트 번들로 끌려 들어갈 수 있는 위험이 있는 것이죠.

### 실제 코드 예시: RSC와 'use client'의 조화

Next.js 13+ App Router 환경에서 RSC와 `use client`를 어떻게 활용하는지 간단한 예시를 통해 살펴보겠습니다.

먼저, 기본적으로 서버 컴포넌트인 페이지와, 순수 서버 컴포넌트를 만들어봅니다.

```typescript jsx
// app/page.tsx (기본적으로 Server Component)
import ClientInteractiveComponent from '@/components/ClientInteractiveComponent';
import ServerGreeting from '@/components/ServerGreeting';

interface HomePageProps {
  // Next.js 13+에서는 페이지 컴포넌트도 props를 받을 수 있습니다.
  // 예를 들어, 검색 파라미터나 동적 라우팅 파라미터 등
  searchParams?: { [key: string]: string | string[] | undefined };
}

export default function HomePage({ searchParams }: HomePageProps) {
  // 서버에서만 접근 가능한 데이터베이스 쿼리 또는 API 호출
  const serverData = "서버에서 가져온 중요한 데이터입니다!";
  const initialCount = searchParams?.initialCount ? parseInt(searchParams.initialCount as string) : 0;

  return (
    <main className="p-8 bg-gray-50 min-h-screen">
      <h1 className="text-4xl font-extrabold text-blue-700 mb-6">
        환영합니다, 프론트엔드 개발자 여러분!
      </h1>
      <ServerGreeting message={serverData} />
      <ClientInteractiveComponent initialCount={initialCount} />
      <p className="mt-8 text-gray-600 text-sm">
        이 페이지는 대부분 서버에서 렌더링되어 초기 로딩이 빠릅니다.
      </p>
    </main>
  );
}
```

```typescript jsx
// components/ServerGreeting.tsx (순수 Server Component)
// 이 컴포넌트는 'use client' 지시문이 없으므로 기본적으로 서버 컴포넌트입니다.
// 클라이언트 번들에 포함되지 않습니다.
interface ServerGreetingProps {
  message: string;
}

export default function ServerGreeting({ message }: ServerGreetingProps) {
  return (
    <div className="bg-blue-100 p-4 rounded-lg shadow-md mb-6">
      <p className="text-blue-800 text-lg font-semibold">
        ✨ 서버에서 렌더링된 메시지: <span className="font-bold">{message}</span>
      </p>
      <p className="text-blue-600 text-sm mt-2">
        이 부분은 클라이언트 JS 없이도 완벽하게 동작합니다.
      </p>
    </div>
  );
}
```

이제 사용자 상호작용이 필요한 클라이언트 컴포넌트를 만들어보겠습니다.

```typescript jsx
// components/ClientInteractiveComponent.tsx
'use client'; // 이 파일은 클라이언트 컴포넌트임을 명시합니다.

import React, { useState, useEffect } from 'react';

interface ClientInteractiveComponentProps {
  initialCount: number;
}

export default function ClientInteractiveComponent({ initialCount }: ClientInteractiveComponentProps) {
  const [count, setCount] = useState(initialCount);

  useEffect(() => {
    // 클라이언트 측에서만 실행되는 로직 (예: localStorage 접근)
    const storedCount = localStorage.getItem('myCount');
    if (storedCount) {
      setCount(parseInt(storedCount));
    }
  }, []);

  const handleClick = () => {
    const newCount = count + 1;
    setCount(newCount);
    localStorage.setItem('myCount', newCount.toString()); // 클라이언트 측 저장
  };

  return (
    <div className="bg-green-100 p-6 rounded-lg shadow-md border border-green-300">
      <h2 className="text-2xl font-bold text-green-700 mb-4">
        클라이언트에서 상호작용하는 카운터
      </h2>
      <p className="text-green-800 text-xl mb-3">
        현재 카운트: <span className="font-extrabold">{count}</span>
      </p>
      <button
        onClick={handleClick}
        className="bg-green-600 hover:bg-green-700 text-white font-bold py-3 px-6 rounded-lg transition duration-300 ease-in-out transform hover:scale-105"
      >
        카운트 증가!
      </button>
      <p className="mt-4 text-sm text-gray-600">
        ⚠️ 이 컴포넌트는 'use client' 지시문 덕분에 클라이언트에서 상태를 관리하고 상호작용합니다.
        클라이언트 번들에 포함됩니다.
      </p>
    </div>
  );
}
```

이 예시에서 `HomePage`는 서버에서 렌더링되면서 `ServerGreeting`과 `ClientInteractiveComponent`를 자식으로 가집니다. `ServerGreeting`은 순수 서버 컴포넌트이므로 클라이언트 번들에 포함되지 않아 번들 크기 절감에 기여합니다. 반면, `ClientInteractiveComponent`는 `useState`와 `useEffect`를 사용하므로 `'use client'` 지시문이 필요하며, 이 컴포넌트는 클라이언트 번들에 포함되어 사용자 상호작용을 처리합니다.

여기서 핵심은 `HomePage`가 `ClientInteractiveComponent`를 `import`하고 렌더링하지만, `HomePage` 자체는 여전히 서버 컴포넌트라는 점입니다. 즉, 서버 컴포넌트가 클라이언트 컴포넌트를 자식으로 가질 수 있으며, 이 경우 클라이언트 컴포넌트만 클라이언트 번들로 보내집니다. 이것이 `'use client'` 세금을 최소화하는 가장 기본적인 방법입니다.

### 실무 적용 팁: 'use client' 세금을 최소화하는 전략

1.  **가장 작은 단위에 `'use client'` 적용:** 상호작용이 필요한 최소한의 컴포넌트에만 `'use client'`를 선언하세요. 불필요하게 큰 컴포넌트나 전체 페이지에 적용하는 것을 피해야 합니다.
    *   **예시:** 버튼 클릭 이벤트만 필요한 경우, 버튼 컴포넌트만 `use client`로 만들고, 버튼을 감싸는 부모 UI는 서버 컴포넌트로 유지합니다.

2.  **컴포지션(Composition) 활용:** 서버 컴포넌트가 클라이언트 컴포넌트를 렌더링할 때, 클라이언트 컴포넌트에 필요한 데이터나 비활성 UI 요소는 props로 전달하세요. 특히, 서버 컴포넌트를 클라이언트 컴포넌트의 `children`으로 전달하면, 해당 `children`은 여전히 서버에서 렌더링되어 클라이언트 번들에 포함되지 않습니다.

    ```typescript jsx
    // components/ClientWrapper.tsx
    'use client';
    import React from 'react';
    interface ClientWrapperProps {
      children: React.ReactNode; // 서버 컴포넌트가 될 수 있는 자식
      onClick: () => void;
    }
    export default function ClientWrapper({ children, onClick }: ClientWrapperProps) {
      return (
        <div onClick={onClick} className="border p-4 cursor-pointer">
          {children} {/* 이 부분은 서버에서 렌더링된 채로 클라이언트로 전달됩니다. */}
          <button>클라이언트 액션</button>
        </div>
      );
    }

    // app/page.tsx
    import ClientWrapper from '@/components/ClientWrapper';
    import ServerOnlyContent from '@/components/ServerOnlyContent'; // use client 없음

    export default function MyPage() {
      const handleClientClick = () => alert('클라이언트 클릭!');
      return (
        <ClientWrapper onClick={handleClientClick}>
          <ServerOnlyContent /> {/* ServerOnlyContent는 클라이언트 번들에 포함되지 않습니다! */}
        </ClientWrapper>
      );
    }
    ```

3.  **지연 로딩(Lazy Loading) 활용:** 당장 필요하지 않은 클라이언트 컴포넌트는 `React.lazy`와 `Suspense`를 사용하여 지연 로딩할 수 있습니다. 이는 초기 번들 크기를 더욱 줄이는 데 도움이 됩니다.

    ```typescript jsx
    // app/page.tsx
    import { Suspense, lazy } from 'react';
    import ServerContent from '@/components/ServerContent';

    const LazyClientComponent = lazy(() => import('@/components/LazyClientComponent'));

    export default function HomePage() {
      return (
        <div>
          <ServerContent />
          <Suspense fallback={<div>로딩 중...</div>}>
            <LazyClientComponent />
          </Suspense>
        </div>
      );
    }
    ```

4.  **번들 분석 도구 활용:** Next.js Bundle Analyzer와 같은 도구를 사용하여 어떤 컴포넌트와 모듈이 클라이언트 번들에 포함되는지 정기적으로 확인하고, 예상치 못한 번들 크기 증가가 발생하면 원인을 분석하세요.

5.  **서버 액션(Server Actions) 고려:** 일부 상호작용은 클라이언트 컴포넌트 없이도 서버 액션을 통해 처리할 수 있습니다. 예를 들어, 폼 제출과 같은 데이터 변경 작업은 서버 액션을 활용하여 클라이언트 JS를 최소화할 수 있습니다.

## 맺음말: 미래를 위한 현명한 선택

React Server Components와 `use client` 지시문은 웹 애플리케이션의 성능 최적화에 있어 강력한 도구임이 분명합니다. 하지만 'use client'가 가져오는 '세금'을 이해하고 현명하게 관리하는 것이 중요합니다. 무분별한 사용은 오히려 초기 성능 이점을 상쇄하고, 개발 복잡도를 높일 수 있습니다.

가장 작은 단위의 클라이언트 컴포넌트를 유지하고, 서버 컴포넌트와의 적절한 컴포지션을 통해 번들 크기를 최적화하는 것이 핵심입니다. 이러한 접근 방식은 사용자 경험을 향상시키고, 더 효율적인 프론트엔드 아키텍처를 구축하는 데 기여할 것입니다.

끊임없이 변화하는 프론트엔드 환경에서, 우리는 항상 새로운 기술의 장점과 단점을 깊이 이해하고, 우리의 프로젝트에 가장 적합한 방식을 찾아 적용해야 합니다. 'use client' 역시 그러한 맥락에서 지속적인 학습과 실험이 필요한 영역입니다.

---

## ## 참고 자료

- [Vue vs. React: Which JavaScript UI framework is best?](https://dev.to/hugodev/vue-vs-react-which-javascript-ui-framework-is-best-1coc) by Hugo Naili
- [The "use client" Tax](https://dev.to/lazarv/the-use-client-tax-1ed0) by Viktor Lázár
- [Slow Pages Cost Money. Here's How to Prove It.](https://dev.to/nosyos/slow-pages-cost-money-heres-how-to-prove-it-4fbm) by nosyos
- [React Inside Preact: Mounting React Components in a Form-JS Renderer](https://dev.to/samabaasi/react-inside-preact-mounting-react-components-in-a-form-js-renderer-2mjh) by Sam Abaasi
- [React Hooks Made Simple: Mastering useState (Without Losing Your Mind)](https://dev.to/kathirvel-s/react-hooks-made-simple-mastering-usestate-without-losing-your-mind-4m16) by Kathirvel S