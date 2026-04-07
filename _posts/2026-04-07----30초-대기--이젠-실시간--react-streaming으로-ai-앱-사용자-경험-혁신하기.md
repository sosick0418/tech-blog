---
layout: post
title: "⚡️ 30초 대기? 이젠 실시간! React Streaming으로 AI 앱 사용자 경험 혁신하기"
date: 2026-04-07T01:01:05.785Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거 개발하는 코알라입니다!
오늘 날짜는 2026년 4월 7일이네요. 벌써 4월이라니, 시간이 정말 빠르죠? 오늘도 빠르게 변화하는 프론트엔드 기술 세계에서 주목할 만한 소식을 들고 왔습니다.

---



최근 AI 기술이 프론트엔드 개발에 깊숙이 파고들면서, AI 모델의 응답을 기다리는 동안 발생하는 사용자 경험 저하 문제가 큰 숙제로 떠올랐습니다. "로딩 중..."이라는 메시지를 30초씩 보고 있는 사용자는 인내심의 한계를 느끼기 마련이죠. 하지만 이제 React Streaming과 Server Components를 활용하면 이런 문제를 해결하고, AI 기반 애플리케이션의 사용자 경험을 혁신할 수 있습니다.

오늘은 React Streaming을 통해 AI 발표 자료 생성기가 어떻게 30초의 대기 시간을 실시간에 가까운 경험으로 탈바꿈시켰는지 살펴보겠습니다. 이 기술은 단순히 로딩 시간을 줄이는 것을 넘어, 사용자가 콘텐츠가 점진적으로 생성되는 과정을 지켜보며 마치 마법을 경험하는 듯한 느낌을 선사합니다.

## 💡 핵심 개념: React Server Components와 Streaming UI

기존의 웹 애플리케이션은 대부분 클라이언트에서 모든 렌더링을 처리했습니다. 하지만 AI 모델처럼 서버에서 많은 연산이 필요한 경우, 클라이언트는 서버의 응답을 모두 기다려야 했죠. 여기서 React Server Components (RSC)와 Streaming UI 패턴이 빛을 발합니다.

*   **React Server Components (RSC)**: 서버에서 렌더링되어 클라이언트로 전송되는 컴포넌트입니다. 데이터 페칭과 같은 무거운 작업을 서버에서 처리하므로, 클라이언트 번들 크기를 줄이고 초기 로딩 속도를 향상시킵니다. 중요한 점은, RSC는 클라이언트 컴포넌트와 달리 번들로 다운로드되지 않으며, 서버에서만 실행됩니다.
*   **Streaming UI**: RSC와 함께 사용될 때 강력한 시너지를 냅니다. 서버에서 컴포넌트가 준비되는 대로 클라이언트로 스트리밍하여 전송하는 방식입니다. 이는 전체 페이지가 로드될 때까지 기다리지 않고, 사용자가 UI의 일부를 즉시 볼 수 있게 하여 '느리지만 진행 중'이라는 긍정적인 인상을 줍니다. React의 `Suspense` 컴포넌트와 결합하여 특정 컴포넌트가 로딩되는 동안 대체 UI(스켈레톤 등)를 보여줄 수 있습니다.

이러한 접근 방식은 특히 AI 응답처럼 시간이 걸리는 작업에서 사용자에게 즉각적인 피드백을 제공하며, 실제 로딩 시간 단축 효과뿐만 아니라 사용자가 느끼는 '체감 속도'를 비약적으로 향상시킵니다.

## 🧑‍💻 실시간 AI 발표 자료 생성기 구현 예시 (Next.js, TypeScript)

Next.js App Router 환경에서 AI 발표 자료 생성기가 어떻게 React Streaming을 활용하는지 간단한 예시 코드를 통해 살펴보겠습니다. 목표는 AI가 슬라이드를 생성하는 동안, 각 슬라이드가 준비되는 대로 화면에 나타나도록 하는 것입니다.

### 1. 메인 페이지 (Server Component)

`app/page.tsx` 파일은 서버 컴포넌트로, 전체 레이아웃을 구성하고 `Suspense`를 사용하여 AI 생성 컴포넌트의 로딩 상태를 관리합니다.

```tsx
// app/page.tsx
import { Suspense } from 'react';
import PresentationGenerator from './presentation-generator'; // Client Component
import { PresentationSkeleton } from './presentation-skeleton'; // Server Component (Loading Fallback)

export default function HomePage() {
  return (
    <div className="container mx-auto p-4">
      <h1 className="text-3xl font-bold mb-6">AI 기반 실시간 발표 자료 생성기</h1>
      <p className="mb-4 text-gray-600">주제를 입력하고 AI가 실시간으로 발표 자료를 만드는 마법을 경험하세요!</p>
      <Suspense fallback={<PresentationSkeleton />}>
        {/* AI 생성 로직은 Client Component 내부에서 API 호출을 통해 처리됩니다. */}
        <PresentationGenerator />
      </Suspense>
    </div>
  );
}
```

### 2. 발표 자료 생성기 (Client Component)

`app/presentation-generator.tsx`는 사용자 입력을 받고, AI API를 호출하여 스트리밍된 슬라이드를 받아 화면에 표시하는 클라이언트 컴포넌트입니다. `use client` 지시자를 통해 클라이언트 컴포넌트임을 명시합니다.

```tsx
// app/presentation-generator.tsx
'use client';

import { useState, useRef, FormEvent } from 'react';
import { motion } from 'framer-motion'; // 부드러운 애니메이션 효과를 위해 사용

interface Slide {
  id: string;
  title: string;
  content: string;
}

export default function PresentationGenerator() {
  const [prompt, setPrompt] = useState('');
  const [slides, setSlides] = useState<Slide[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const slideIdRef = useRef(0); // 고유한 슬라이드 ID 생성을 위한 ref

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    if (!prompt.trim()) return;

    setSlides([]); // 새로운 생성 시작 시 기존 슬라이드 초기화
    setIsLoading(true);

    try {
      // Next.js API Route를 통해 AI 스트리밍 응답을 받습니다.
      const response = await fetch('/api/generate-slides', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ prompt }),
      });

      if (!response.body) {
        throw new Error("API 응답 본문을 읽을 수 없습니다.");
      }

      const reader = response.body.getReader();
      const decoder = new TextDecoder();
      let buffer = '';

      while (true) {
        const { value, done } = await reader.read();
        if (done) break;

        buffer += decoder.decode(value, { stream: true });
        // 각 슬라이드가 JSON 형태로 스트리밍된다고 가정하고 파싱합니다.
        // 실제 구현에서는 더 견고한 스트리밍 파싱 로직이 필요합니다.
        const parts = buffer.split('\n\n'); // 예시: 각 슬라이드가 두 개의 개행 문자로 구분
        buffer = parts.pop() || ''; // 마지막 불완전한 부분은 버퍼에 남김

        parts.forEach(part => {
          if (part.trim()) {
            try {
              const newSlide = JSON.parse(part) as Omit<Slide, 'id'>;
              setSlides(prevSlides => [
                ...prevSlides,
                { id: `slide-${slideIdRef.current++}`, ...newSlide }
              ]);
            } catch (error) {
              console.error('슬라이드 JSON 파싱 오류:', error, part);
            }
          }
        });
      }
    } catch (error) {
      console.error('발표 자료 생성 중 오류 발생:', error);
      alert('발표 자료 생성 중 오류가 발생했습니다. 다시 시도해 주세요.');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div>
      <form onSubmit={handleSubmit} className="flex gap-2 mb-8">
        <input
          type="text"
          value={prompt}
          onChange={(e) => setPrompt(e.target.value)}
          placeholder="발표 주제를 입력하세요 (예: React Server Components의 미래)"
          className="flex-grow p-2 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
          disabled={isLoading}
        />
        <button
          type="submit"
          className="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700 disabled:opacity-50 transition-colors"
          disabled={isLoading}
        >
          {isLoading ? '생성 중...' : '발표 자료 생성'}
        </button>
      </form>

      {isLoading && slides.length === 0 && (
        <div className="text-center text-gray-500 mt-8">발표 자료를 생성 중입니다. 잠시만 기다려주세요...</div>
      )}

      <div className="grid gap-6">
        {slides.map((slide, index) => (
          <motion.div
            key={slide.id}
            initial={{ opacity: 0, y: 20 }} // 초기 상태
            animate={{ opacity: 1, y: 0 }}   // 애니메이션 종료 상태
            transition={{ delay: index * 0.1, duration: 0.3 }} // 각 슬라이드별 지연 및 지속 시간
            className="p-6 border border-gray-200 rounded-lg shadow-sm bg-white hover:shadow-md transition-shadow"
          >
            <h2 className="text-2xl font-semibold mb-3 text-blue-800">{slide.title}</h2>
            <p className="text-gray-700 leading-relaxed">{slide.content}</p>
          </motion.div>
        ))}
      </div>

      {!isLoading && slides.length === 0 && (
        <div className="text-center text-gray-400 mt-10 p-4 border border-dashed rounded-lg">
          아직 생성된 발표 자료가 없습니다. 위에 주제를 입력하고 AI의 마법을 경험해보세요!
        </div>
      )}
    </div>
  );
}
```

### 3. 로딩 스켈레톤 (Server Component)

`Suspense`의 `fallback`으로 사용될 스켈레톤 UI입니다. 로딩 중에도 사용자에게 UI의 구조를 미리 보여주어 더 나은 경험을 제공합니다.

```tsx
// app/presentation-skeleton.tsx
export function PresentationSkeleton() {
  return (
    <div className="animate-pulse grid gap-6">
      <div className="h-10 bg-gray-200 rounded w-3/4 mb-4"></div>
      <div className="p-6 border border-gray-100 rounded-lg shadow-sm bg-gray-50">
        <div className="h-6 bg-gray-200 rounded w-1/2 mb-3"></div>
        <div className="h-4 bg-gray-200 rounded mb-2"></div>
        <div className="h-4 bg-gray-200 rounded w-5/6"></div>
      </div>
      <div className="p-6 border border-gray-100 rounded-lg shadow-sm bg-gray-50">
        <div className="h-6 bg-gray-200 rounded w-2/3 mb-3"></div>
        <div className="h-4 bg-gray-200 rounded mb-2"></div>
        <div className="h-4 bg-gray-200 rounded w-4/5"></div>
      </div>
      <div className="p-6 border border-gray-100 rounded-lg shadow-sm bg-gray-50">
        <div className="h-6 bg-gray-200 rounded w-1/3 mb-3"></div>
        <div className="h-4 bg-gray-200 rounded mb-2"></div>
        <div className="h-4 bg-gray-200 rounded w-3/4"></div>
      </div>
    </div>
  );
}
```

### 4. AI 스트리밍 API (Next.js API Route)

`app/api/generate-slides/route.ts` 파일은 AI 모델의 응답을 시뮬레이션하고, 각 슬라이드를 스트리밍 방식으로 클라이언트에 전송합니다.

```typescript
// app/api/generate-slides/route.ts
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
  const { prompt } = await request.json();

  if (!prompt) {
    return NextResponse.json({ error: 'Prompt is required' }, { status: 400 });
  }

  // AI 응답 스트리밍을 시뮬레이션합니다.
  const encoder = new TextEncoder();
  const readableStream = new ReadableStream({
    async start(controller) {
      const slidesData = [
        { title: `"${prompt}" 주제 소개`, content: `이 발표에서는 ${prompt}의 중요성과 핵심 개념을 다룹니다.` },
        { title: `핵심 개념 1: React Server Components`, content: `RSC가 무엇이며, 서버에서 렌더링되어 클라이언트 번들을 줄이는 방법을 설명합니다.` },
        { title: `핵심 개념 2: Streaming UI와 Suspense`, content: `Streaming UI 패턴을 통해 사용자 경험을 어떻게 개선하는지, Suspense와 함께 사용하는 방법을 알아봅니다.` },
        { title: `실제 적용 사례: AI 발표 자료 생성기`, content: `AI 발표 자료 생성기에서 RSC와 Streaming이 어떻게 실시간 경험을 제공하는지 구체적인 예시를 들어 설명합니다.` },
        { title: `결론 및 향후 전망`, content: `오늘 다룬 내용을 요약하고, React의 미래와 AI 앱 개발에 미칠 영향에 대해 이야기합니다.` },
      ];

      for (const slide of slidesData) {
        const jsonString = JSON.stringify(slide) + '\n\n'; // 각 슬라이드를 JSON 문자열로 만들고 구분자 추가
        controller.enqueue(encoder.encode(jsonString));
        await new Promise(resolve => setTimeout(resolve, Math.random() * 1500 + 500)); // 네트워크 지연 시뮬레이션
      }
      controller.close();
    },
  });

  return new NextResponse(readableStream, {
    headers: {
      'Content-Type': 'text/plain; charset=utf-8', // 스트리밍 JSON 청크를 위해 text/plain 사용
      'Cache-Control': 'no-cache', // 캐시 방지
      'Connection': 'keep-alive', // 연결 유지
    },
  });
}
```

이 예시를 통해 사용자가 주제를 입력하면, AI가 각 슬라이드를 하나씩 생성하여 스트리밍 방식으로 클라이언트에 전송하고, 클라이언트는 이를 즉시 렌더링하는 과정을 볼 수 있습니다. `framer-motion`을 사용하면 새로 추가되는 슬라이드에 부드러운 애니메이션 효과를 주어 사용자 경험을 더욱 풍부하게 만들 수 있습니다.

## 🚀 실무 적용 팁

React Streaming과 Server Components는 강력하지만, 올바르게 사용해야 그 효과를 극대화할 수 있습니다.

1.  **컴포넌트 분리 전략**:
    *   **Server Components**: 데이터 페칭, 민감한 정보 처리, 큰 라이브러리 로드, 정적 콘텐츠 렌더링에 적합합니다. 클라이언트 번들에 포함되지 않아 초기 로딩 성능에 유리합니다.
    *   **Client Components**: 사용자 상호작용 (클릭, 입력), 이벤트 처리, 브라우저 API 접근, 클라이언트 전용 라이브러리(ex. `framer-motion`, 상태 관리 라이브러리) 사용에 적합합니다. `use client` 지시자를 반드시 사용해야 합니다.
    *   두 컴포넌트 타입 간의 명확한 경계를 설정하는 것이 중요합니다.

2.  **Suspense 활용 극대화**:
    *   `Suspense`는 UI의 특정 부분이 준비될 때까지 기다리는 동안 대체 콘텐츠(fallback)를 보여줍니다. 여러 `Suspense` 바운더리를 사용하여 복잡한 UI의 각 부분을 독립적으로 로딩하고 사용자에게 더 세분화된 피드백을 제공할 수 있습니다.
    *   데이터 페칭, 코드 스플리팅 등 비동기 작업에 폭넓게 적용 가능합니다.

3.  **에러 바운더리 (`Error Boundary`)**:
    *   스트리밍 과정이나 데이터 처리 중 예상치 못한 오류가 발생할 수 있습니다. `Error Boundary`를 사용하여 특정 컴포넌트 트리에서 발생하는 오류를 격리하고, 사용자에게 친화적인 오류 메시지를 표시하여 애플리케이션 전체가 망가지는 것을 방지해야 합니다.

4.  **스트리밍 데이터 형식 설계**:
    *   API에서 스트리밍되는 데이터의 형식을 신중하게 설계해야 합니다. 위 예시처럼 JSON 청크를 `\n\n`과 같은 구분자로 보내는 방식도 있지만, Server-Sent Events (SSE)나 GraphQL Subscriptions 같은 표준화된 스트리밍 프로토콜을 고려할 수도 있습니다. 클라이언트에서 이 데이터를 견고하게 파싱하는 로직이 필수적입니다.

5.  **캐싱 전략 이해**:
    *   Next.js App Router는 강력한 캐싱 메커니즘을 제공합니다. RSC의 데이터 페칭은 기본적으로 캐싱될 수 있으며, `revalidate` 옵션 등을 통해 캐싱 동작을 제어할 수 있습니다. 스트리밍과 캐싱을 함께 이해하고 활용하면 성능을 더욱 최적화할 수 있습니다.

## 맺음말

AI 기술의 발전은 프론트엔드 개발자들에게 새로운 도전과 기회를 동시에 제공하고 있습니다. React Server Components와 Streaming UI 패턴은 이러한 도전 과제, 특히 AI 모델의 느린 응답 시간으로 인한 사용자 경험 저하 문제를 해결하는 강력한 도구입니다. 30초의 기다림을 실시간에 가까운 경험으로 바꾸는 것은 단순히 기술적 개선을 넘어, 사용자에게 깊은 인상을 남기는 마법과도 같습니다.

오늘 보여드린 AI 발표 자료 생성기 예시처럼, 이 기술들은 AI 챗봇, 실시간 데이터 대시보드, 복잡한 데이터 시각화 등 다양한 분야에서 혁신적인 사용자 경험을 만들어낼 잠재력을 가지고 있습니다. Next.js App Router와 함께 React Streaming을 깊이 탐구하여, 여러분의 애플리케이션을 한 단계 더 발전시켜 보세요!

---

## 참고 자료

-   [Building Compliance-First Property Management Software](https://dev.to/jonomor_ecosystem/building-compliance-first-property-management-software-3eon) by Jonomor
-   [Why Web Apps Should Never Lose Your Session Again](https://dev.to/rohith_kn/why-web-apps-should-never-lose-your-session-again-4pn8) by Rohith
-   [I Built an AI Presentation Generator Using React Streaming — Here's What Actually Works](https://dev.to/jiade/i-built-an-ai-presentation-generator-using-react-streaming-heres-what-actually-works-1ha4) by 吴迦
-   [De 3 segundos a 300ms: cómo optimicé el performance de una app Next.js en producción](https://dev.to/juantorchia/de-3-segundos-a-300ms-como-optimice-el-performance-de-una-app-nextjs-en-produccion-3ceh) by Juan Torchia
-   [Next.js App Router: la guía que me hubiera gustado tener cuando migré de Pages Router](https://dev.to/juantorchia/nextjs-app-router-la-guía-que-me-hubiera-gustado-tener-cuando-migre-de-pages-router-5cef) by Juan Torchia