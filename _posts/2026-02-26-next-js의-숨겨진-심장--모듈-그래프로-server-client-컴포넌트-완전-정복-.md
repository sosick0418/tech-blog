---
layout: post
title: "Next.js의 숨겨진 심장: 모듈 그래프로 Server/Client 컴포넌트 완전 정복!"
date: 2026-02-26T01:00:34.923Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발의 최전선에서 늘 새로운 기술을 탐구하는 블로거, **프론트엔드 마스터**입니다! 어느덧 2026년 2월 26일이네요. 오늘도 여러분의 개발 여정에 도움이 될 만한 흥미로운 주제를 들고 왔습니다.

---



최근 Next.js 개발의 핵심은 단연 Server Components와 Client Components입니다. 하지만 이 둘이 어떻게 상호작용하며 애플리케이션의 성능과 구조를 결정하는지 명확히 이해하기란 쉽지 않죠. 오늘은 Next.js의 '모듈 그래프'라는 숨겨진 아키텍처를 통해 이 복잡한 퍼즐을 풀어보려 합니다. 이 개념을 제대로 이해한다면, 여러분의 Next.js 애플리케이션은 더욱 견고하고 효율적으로 거듭날 것입니다.

## 모듈 그래프, Next.js 아키텍처의 청사진

### 핵심 개념 설명
모듈 그래프(Module Graph)는 우리 코드의 모든 파일과 그 파일들이 서로 어떻게 의존하는지를 시각화한 지도와 같습니다. 어떤 파일이 다른 파일을 import 하는 순간, 이 파일들은 그래프의 노드가 되고 그 관계는 엣지(edge)가 되죠. Next.js는 이 모듈 그래프를 기반으로 애플리케이션을 빌드하고 최적화합니다.

특히 Next.js 13+ 버전의 App Router에서는 Server Components와 Client Components의 구분이 매우 중요하며, 이 둘의 상호작용은 바로 이 모듈 그래프에 의해 결정됩니다. Next.js 빌드 시스템은 모듈 그래프를 분석하여 각 컴포넌트가 서버에서 실행될지, 클라이언트에서 실행될지를 판단하고, 이에 따라 적절한 번들(bundle)을 생성합니다.

*   **Server Components (RSC):** 기본적으로 모든 컴포넌트는 Server Component로 간주됩니다. 서버에서 렌더링되고, 클라이언트로 HTML만 전송됩니다. 초기 로딩 속도와 SEO에 유리하며, 클라이언트 번들에 JavaScript를 포함하지 않아 번들 사이즈를 줄입니다.
*   **Client Components:** `use client` 지시자를 파일 상단에 명시하면 해당 파일과 그 파일이 import 하는 모든 모듈은 Client Component로 처리됩니다. 클라이언트에서 렌더링되며, 인터랙티브한 UI와 상태 관리에 적합합니다.

가장 중요한 점은 Next.js가 모듈 그래프를 따라가며 `use client` 지시자를 발견하는 순간, 그 지점부터 아래로 이어지는 모든 의존성을 클라이언트 번들로 포함시킨다는 것입니다. 즉, `use client`는 서버와 클라이언트 코드의 **경계선(boundary)** 역할을 합니다. 서버 컴포넌트는 클라이언트 컴포넌트를 자식(children)이나 prop으로 포함할 수 있지만, 클라이언트 컴포넌트가 직접 서버 컴포넌트를 import 할 수는 없습니다.

### 실제 코드 예시 (React/Next.js/TypeScript 활용)
간단한 Next.js 프로젝트 구조를 통해 모듈 그래프와 Server/Client 컴포넌트의 관계를 살펴보겠습니다.

```tsx
// app/page.tsx (기본적으로 Server Component)
import ClientInteractiveSection from '../components/ClientInteractiveSection';
import ServerStaticData from '../components/ServerStaticData';
import type { Post } from '@/types/Post'; // TypeScript 타입 임포트

// 가상의 데이터 fetch 함수 (서버에서 실행)
async function getPosts(): Promise<Post[]> {
  // 실제 API 호출 로직 (예: fetch('https://api.example.com/posts'))
  return new Promise(resolve =>
    setTimeout(() =>
      resolve([
        { id: 1, title: 'Server Component의 힘' },
        { id: 2, title: 'Client Component와의 조화' },
      ]),
    1000)
  );
}

export default async function HomePage() {
  const posts = await getPosts(); // 서버에서 데이터 패칭

  return (
    <div className="container">
      <h1>Next.js 모듈 그래프 이해하기</h1>
      <ServerStaticData posts={posts} />
      <ClientInteractiveSection initialCount={0} />
    </div>
  );
}
```

```tsx
// components/ServerStaticData.tsx (Server Component)
interface Post {
  id: number;
  title: string;
}

interface ServerStaticDataProps {
  posts: Post[];
}

export default function ServerStaticData({ posts }: ServerStaticDataProps) {
  return (
    <section>
      <h2>서버에서 가져온 정적 데이터</h2>
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </section>
  );
}
```

```tsx
// components/ClientInteractiveSection.tsx (Client Component)
'use client'; // 이 파일과 관련된 모든 의존성은 클라이언트 번들로 포함됩니다.

import { useState } from 'react';

interface ClientInteractiveSectionProps {
  initialCount: number;
}

export default function ClientInteractiveSection({ initialCount }: ClientInteractiveSectionProps) {
  const [count, setCount] = useState(initialCount);

  return (
    <section>
      <h2>클라이언트에서 인터랙티브 섹션</h2>
      <p>현재 카운트: {count}</p>
      <button onClick={() => setCount(prev => prev + 1)}>카운트 증가</button>
      <button onClick={() => setCount(prev => prev - 1)}>카운트 감소</button>
    </section>
  );
}
```

```tsx
// types/Post.ts (타입 정의 파일)
export interface Post {
  id: number;
  title: string;
}
```

위 예시에서 `app/page.tsx`와 `components/ServerStaticData.tsx`는 Server Component입니다. 이들은 서버에서 렌더링되어 HTML로만 클라이언트에 전달되므로, 이들의 JavaScript 코드는 클라이언트 번들에 포함되지 않습니다. 반면, `components/ClientInteractiveSection.tsx`는 `use client` 지시자 덕분에 Client Component가 됩니다. 이 파일과 `useState` 같은 React 훅을 포함하는 React 라이브러리의 일부는 클라이언트 번들에 포함되어 사용자의 브라우저에서 실행됩니다.

`app/page.tsx`가 `ClientInteractiveSection`을 import 하는 순간, Next.js의 모듈 그래프는 이 경계를 인식하고 `ClientInteractiveSection`과 그 의존성(예: React의 `useState`)을 클라이언트 번들에 포함시킵니다.

### 실무 적용 팁
1.  **`use client`의 전략적 배치:** `use client`는 필요한 최소한의 범위에만 적용하세요. 예를 들어, 거대한 컴포넌트 전체를 클라이언트로 만드는 대신, 인터랙션이 필요한 특정 부분만 작은 Client Component로 분리하여 import 하는 것이 좋습니다. 이는 클라이언트 번들 크기를 최소화하는 데 핵심입니다.
2.  **서버 컴포넌트의 활용 극대화:** 데이터 패칭, 민감한 정보 처리, SEO에 중요한 정적 콘텐츠 렌더링 등은 Server Component에서 처리하여 초기 로딩 성능을 최적화하세요.
3.  **번들 사이즈 분석:** Next.js 빌드 후 제공되는 번들 분석 도구(예: `npx @next/bundle-analyzer`)를 활용하여 어떤 모듈이 클라이언트 번들에 포함되었는지 확인하고, 불필요한 코드가 유입되지 않도록 모듈 그래프를 역추적하며 최적화하세요. 특히, Server Component에서만 필요한 라이브러리가 실수로 Client Component에 import 되어 클라이언트 번들에 포함되지 않도록 주의해야 합니다.
4.  **클라이언트 컴포넌트의 자식으로 서버 컴포넌트 전달:** 클라이언트 컴포넌트 내에서 서버 컴포넌트의 렌더링 결과를 사용하고 싶다면, 서버 컴포넌트를 직접 import 하는 대신 `children` prop으로 전달하는 방식을 활용하세요. 이는 클라이언트 컴포넌트가 서버 컴포넌트를 재렌더링할 필요 없이, 서버에서 이미 렌더링된 HTML을 받아 사용하도록 합니다.

```tsx
// components/ClientWrapper.tsx (Client Component)
'use client';

export default function ClientWrapper({ children }: { children: React.ReactNode }) {
  // 클라이언트 측 인터랙션 로직
  return (
    <div className="client-wrapper">
      {children} {/* 서버에서 렌더링된 내용을 받아서 표시 */}
      <button>클라이언트 액션</button>
    </div>
  );
}

// app/page.tsx (Server Component)
import ClientWrapper from '../components/ClientWrapper';
import ServerContent from '../components/ServerContent'; // Server Component

export default function HomePage() {
  return (
    <ClientWrapper>
      <ServerContent /> {/* 서버 컴포넌트가 ClientWrapper의 자식으로 전달 */}
    </ClientWrapper>
  );
}
```

## 마무리하며
Next.js의 모듈 그래프는 Server Components와 Client Components의 작동 방식을 이해하는 데 필수적인 개념입니다. 이 숨겨진 아키텍처를 파악함으로써 우리는 더 효율적인 번들링, 더 빠른 초기 로딩, 그리고 더 나은 사용자 경험을 제공하는 Next.js 애플리케이션을 구축할 수 있습니다.

오늘 다룬 내용을 바탕으로 여러분의 Next.js 프로젝트에서 모듈 그래프를 의식적으로 활용해보세요. 다음 단계로는 Next.js의 스트리밍(Streaming)과 서버 액션(Server Actions)이 모듈 그래프와 어떻게 연관되는지 깊이 파고들어보는 것을 추천합니다. Happy Coding!

## 참고 자료
- [Cursor-Reactive Gradients: Making CSS Respond to Mouse Position](https://dev.to/sammiihk/cursor-reactive-gradients-making-css-respond-to-mouse-position-5ga3) by Sammii
- [Offline-first React without the boilerplate — how I built connectivity-js](https://dev.to/connectivity-js/offline-first-react-without-the-boilerplate-how-i-built-connectivity-js-4lb7) by connectivity-js
- [I Fixed Stale Data Across Every Screen in React Native with 3 React Query Patterns](https://dev.to/likhit/i-fixed-stale-data-across-every-screen-in-react-native-with-3-react-query-patterns-5fd5) by Likhit Kumar V P
- [How We Engineered a 73% Cache Hit Rate Travel App with Vite + React](https://dev.to/nishant_paudel_b5720c48ad/how-we-engineered-a-73-cache-hit-rate-travel-app-with-vite-react-32dg) by Nishant Paudel
- [Next.js Module Graphs: The Hidden Architecture Behind Server and Client Components](https://dev.to/zwelhtetyan/nextjs-module-graphs-the-hidden-architecture-behind-server-and-client-components-4h) by Zwel👻