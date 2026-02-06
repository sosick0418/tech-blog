---
layout: post
title: "🚨 Tailwind + Shadcn/ui + Shadow DOM: 이 조합이 프론트엔드 개발자에게 '고통'인 이유"
date: 2026-02-06T01:00:38.048Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거입니다. 2026년 2월 6일, 오늘도 프론트엔드 세계는 흥미로운 이야기들로 가득하네요. 오늘은 최근 개발자들 사이에서 뜨거운 감자로 떠오른 한 가지 주제에 대해 깊이 파고들어 보겠습니다.

---



2026년 2월 6일, 프론트엔드 개발의 최전선에 계신 여러분, 안녕하세요! 오늘은 최근 ujja님의 "I Know This Will Upset Some Devs, but Tailwind + Shadcn/ui + Shadow DOM = Pain"이라는 글이 제 눈길을 사로잡았습니다. 이 글은 현대 프론트엔드 개발에서 각광받는 기술 스택인 Tailwind CSS, Shadcn/ui, 그리고 웹 컴포넌트의 핵심인 Shadow DOM이 함께 사용될 때 발생할 수 있는 예상치 못한 '고통'을 지적하고 있습니다.

특히 Sylwia Lask님의 "Is Learning CSS a Waste of Time in 2026?"이라는 질문과 맞물려, CSS의 근본적인 이해가 그 어느 때보다 중요해지는 시점이라는 것을 다시 한번 느끼게 되었습니다. 과연 이 세 가지 기술이 왜 충돌하며, 우리는 어떤 점을 주의해야 할까요? 함께 살펴보시죠!

## 이 조합이 왜 중요한가요?

최근 프론트엔드 개발은 생산성과 재사용성에 초점을 맞추고 있습니다. Tailwind CSS는 유틸리티 우선(utility-first) 방식으로 빠른 UI 개발을 돕고, Shadcn/ui는 이를 기반으로 고품질의 재사용 가능한 컴포넌트를 제공합니다. 여기에 웹 컴포넌트의 Shadow DOM은 컴포넌트의 완벽한 격리를 통해 재사용성과 유지보수성을 극대화하려는 시도입니다. 각각의 기술은 강력하지만, 이들이 만나면서 발생하는 시너지 효과보다는 상충되는 원리로 인해 예상치 못한 문제에 직면할 수 있습니다. 이는 개발 생산성 저하와 디버깅의 어려움으로 이어질 수 있어, 이 조합을 고려하는 모든 개발자가 반드시 인지해야 할 중요한 문제입니다.

## 핵심 개념 설명: 왜 '고통'이 발생하는가?

이 문제의 핵심은 각 기술의 근본적인 디자인 철학이 서로 다르다는 데 있습니다.

*   **Tailwind CSS:**
    Tailwind는 `bg-blue-500`, `text-white`, `p-4`와 같은 유틸리티 클래스를 HTML 요소에 직접 적용하여 스타일을 입히는 방식입니다. 이는 전역적으로 정의된 CSS 규칙에 의존하며, HTML 문서의 모든 요소에 걸쳐 일관된 스타일링 환경을 기대합니다.

*   **Shadcn/ui:**
    Shadcn/ui는 Tailwind CSS와 Radix UI를 기반으로 구축된 컴포넌트 라이브러리입니다. 개발자가 컴포넌트 코드를 직접 복사하여 프로젝트에 통합하고 커스터마이징할 수 있도록 설계되었습니다. 이 컴포넌트들 또한 Tailwind 클래스와 전역 CSS 변수(CSS Custom Properties)를 활용하여 스타일을 결정하고 테마를 적용합니다. 즉, Tailwind와 마찬가지로 전역적인 스타일링 컨텍스트를 가정합니다.

*   **Shadow DOM:**
    웹 컴포넌트의 핵심 기술 중 하나인 Shadow DOM은 DOM과 CSS를 주류 문서로부터 완벽하게 격리(encapsulation)시킵니다. `attachShadow({ mode: 'open' })`와 같이 생성된 Shadow Root 내부는 외부 CSS의 영향을 받지 않으며, 내부의 CSS 또한 외부로 유출되지 않습니다. 이는 컴포넌트의 스타일 충돌을 방지하고 재사용성을 높이는 강력한 메커니즘이지만, 동시에 외부 스타일링 도구와의 상호작용을 어렵게 만듭니다.

**문제의 시작: 격리와 전역 스타일링의 충돌**
바로 이 지점에서 '고통'이 시작됩니다. Shadow DOM의 강력한 격리 특성 때문에, Tailwind CSS의 유틸리티 클래스나 Shadcn/ui 컴포넌트들이 의존하는 전역적인 CSS 변수 또는 기본 스타일들이 Shadow DOM 내부로 전달되지 않습니다. Shadcn/ui 컴포넌트를 Shadow DOM 내에서 사용하려 하면, 의도한 스타일이 적용되지 않거나 깨지는 현상을 겪게 됩니다. Shadow DOM은 마치 외부 세계와 완벽히 단절된 섬과 같아서, 외부에서 아무리 멋진 옷(Tailwind 스타일)을 준비해 줘도 섬 안의 주민(Shadow DOM 내부 요소)은 이를 입을 수 없는 상황인 셈입니다.

## 실제 코드 예시: Shadow DOM의 격리 특성

다음은 Shadow DOM이 외부 Tailwind CSS 클래스를 어떻게 무시하는지 보여주는 간단한 React 컴포넌트 예시입니다. Next.js 환경에서 TypeScript와 React를 사용한다고 가정합니다.

```tsx
// src/components/ShadowButtonHost.tsx
import React, { useRef, useEffect } from 'react';

interface ShadowButtonHostProps {
  buttonText: string;
}

const ShadowButtonHost: React.FC<ShadowButtonHostProps> = ({ buttonText }) => {
  const shadowHostRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (shadowHostRef.current && !shadowHostRef.current.shadowRoot) {
      // Shadow DOM을 생성합니다. mode: 'open'으로 외부에서 접근 가능하게 합니다.
      const shadowRoot = shadowHostRef.current.attachShadow({ mode: 'open' });

      // Shadow DOM 내부에 스타일을 정의합니다.
      // 외부 Tailwind 클래스는 이곳으로 침투하지 못합니다.
      const style = document.createElement('style');
      style.textContent = `
        /* Shadow DOM 내부에서만 유효한 스타일 */
        button {
          background-color: #ef4444; /* Tailwind bg-red-500과 유사 */
          color: white;             /* Tailwind text-white */
          padding: 10px 20px;       /* Tailwind p-2 */
          border-radius: 6px;       /* Tailwind rounded-md */
          border: none;
          cursor: pointer;
          font-size: 16px;
          font-weight: 600;
          transition: background-color 0.2s ease-in-out;
        }
        button:hover {
          background-color: #dc2626; /* Tailwind bg-red-600과 유사 */
        }
      `;
      shadowRoot.appendChild(style);

      // Shadow DOM 내부에 버튼 요소를 생성합니다.
      const buttonElement = document.createElement('button');
      buttonElement.textContent = buttonText;
      // 여기에 'className="bg-green-500 text-black p-4"' 와 같은
      // 외부 Tailwind 클래스를 추가해도 Shadow DOM 내부에서는 무시됩니다!
      // Shadcn/ui 컴포넌트도 이와 유사한 이유로 제대로 렌더링되지 않습니다.
      shadowRoot.appendChild(buttonElement);
    }
  }, [buttonText]);

  return (
    <div ref={shadowHostRef} className="border border-dashed border-gray-400 p-4 rounded-md">
      <p className="mb-2 text-sm text-gray-600">
        여기는 Shadow DOM 외부 영역입니다. <span className="font-bold text-blue-600">Tailwind 스타일이 적용됩니다.</span>
      </p>
      {/* 이 div 태그에 ref가 연결되어 Shadow DOM의 호스트 역할을 합니다. */}
    </div>
  );
};

export default ShadowButtonHost;
```

```tsx
// app/page.tsx (Next.js 페이지)
import ShadowButtonHost from '@/components/ShadowButtonHost';
// import { Button } from '@/components/ui/button'; // Shadcn/ui 버튼

export default function HomePage() {
  return (
    <main className="flex min-h-screen flex-col items-center justify-center p-24 bg-gray-100">
      <h1 className="text-4xl font-extrabold mb-10 text-gray-800">
        Shadow DOM, Tailwind, Shadcn/ui 충돌 시연
      </h1>

      <div className="mb-8">
        <h2 className="text-2xl font-semibold mb-4 text-gray-700">
          1. Shadow DOM 내부 버튼 (외부 Tailwind 미적용)
        </h2>
        {/*
          아래 ShadowButtonHost 컴포넌트는 Shadow DOM을 생성합니다.
          이 컴포넌트에 'className="bg-blue-500 text-white p-8"' 와 같은
          Tailwind 클래스를 추가해도, Shadow DOM 내부의 버튼에는 적용되지 않습니다.
          버튼은 Shadow DOM 내부에 정의된 스타일을 따릅니다.
        */}
        <ShadowButtonHost buttonText="Shadow DOM 속 버튼" />
        <p className="mt-4 text-red-600 font-medium">
          **주의:** 위 버튼은 Shadow DOM 내부에 있어 외부 Tailwind 스타일링이 적용되지 않습니다.
          내부에서 직접 CSS를 정의해야 합니다.
        </p>
      </div>

      {/*
      <div className="mt-10">
        <h2 className="text-2xl font-semibold mb-4 text-gray-700">
          2. 일반 Shadcn/ui 버튼 (정상 작동)
        </h2>
        <Button className="bg-emerald-500 hover:bg-emerald-600 text-white font-bold py-2 px-4 rounded">
          일반 Shadcn/ui 버튼
        </Button>
        <p className="mt-4 text-green-700">
          이 버튼은 Shadow DOM 외부에 있어 Tailwind 및 Shadcn/ui 스타일이 정상적으로 적용됩니다.
        </p>
      </div>
      */}
      <p className="mt-10 text-gray-500 text-sm">
        (주석 처리된 Shadcn/ui 버튼 예시는 Shadow DOM과의 비교를 위해 추가되었습니다.)
      </p>
    </main>
  );
}
```

**코드 설명:**
위 `ShadowButtonHost` 컴포넌트는 `shadowHostRef`에 연결된 `div` 요소에 Shadow DOM을 생성합니다. `useEffect` 훅 내부에서 `<style>` 태그를 생성하여 Shadow DOM 내부에 직접 스타일을 주입하고, 그 아래에 `<button>` 요소를 추가합니다.

주목할 점은 `HomePage` 컴포넌트에서 `ShadowButtonHost`를 사용할 때, 부모 컴포넌트에서 `className="bg-blue-500 text-white"`와 같은 Tailwind 클래스를 적용하더라도, Shadow DOM 내부의 `<button>` 요소에는 해당 스타일이 적용되지 않는다는 것입니다. 버튼은 오직 Shadow DOM 내부에 정의된 `<style>` 태그의 CSS 규칙만을 따릅니다.

만약 Shadcn/ui 컴포넌트를 이 Shadow DOM 내부에 직접 렌더링하려 한다면, Shadcn/ui 컴포넌트가 의존하는 Tailwind 클래스나 전역 CSS 변수들이 Shadow DOM의 격리 때문에 전달되지 않아 스타일이 완전히 깨지거나 작동하지 않을 것입니다. 이를 해결하려면 Shadcn/ui 컴포넌트의 모든 스타일을 Shadow DOM 내부로 직접 가져오거나, 다른 복잡한 방법을 사용해야 합니다.

## 실무 적용 팁: '고통'을 줄이는 방법

그렇다면 이 '고통'스러운 조합을 피하거나, 어쩔 수 없이 사용해야 할 때 어떻게 대처해야 할까요?

1.  **조합을 신중하게 고려하세요:**
    새로운 프로젝트를 시작한다면, Tailwind + Shadcn/ui와 Shadow DOM을 함께 사용하는 것은 피하는 것이 좋습니다. 각 기술의 강점이 서로의 약점을 부각시킬 수 있습니다. 웹 컴포넌트의 격리 기능이 필수적이라면, Shadow DOM 내부의 컴포넌트 스타일링은 Tailwind 대신 CSS Modules, Styled Components, 또는 전통적인 CSS를 사용하는 것이 훨씬 효율적입니다.

2.  **Shadow DOM 내부 스타일링 전략:**
    만약 Shadow DOM 사용이 필수적이라면, Shadow DOM 내부에 직접 `<style>` 태그를 주입하거나, CSS Modules, 또는 Styled Components와 같은 CSS-in-JS 라이브러리를 사용하여 컴포넌트 스코프 스타일링을 적용해야 합니다. Shadcn/ui 컴포넌트를 사용해야 한다면, 해당 컴포넌트의 모든 Tailwind 관련 스타일을 추출하여 Shadow DOM 내부의 `<style>`로 옮기거나, CSS Custom Properties를 통해 제한적으로 테마를 전달하는 방법을 고려해야 합니다.

3.  **`::part` 셀렉터 활용 (가능하다면):**
    웹 컴포넌트가 `part` 속성을 노출하도록 설계되었다면, 외부에서 `::part()` 셀렉터를 통해 제한적으로 스타일을 커스터마이징할 수 있습니다. 예를 들어, Shadow DOM 내부의 `<button part="my-button">` 요소는 외부에서 `my-component::part(my-button) { background-color: blue; }`와 같이 스타일링할 수 있습니다. 하지만 이는 컴포넌트 설계 단계에서 고려되어야 하며, Shadcn/ui와 같은 기존 라이브러리가 이 기능을 제공하지 않을 수 있습니다.

4.  **CSS Custom Properties (변수) 활용:**
    Shadow DOM은 CSS Custom Properties를 상속받을 수 있습니다. 이를 통해 외부에서 정의한 테마 변수를 Shadow DOM 내부에서 활용하여 일관된 디자인을 유지할 수 있습니다. 예를 들어, `:root { --primary-color: #3b82f6; }`와 같이 정의하고 Shadow DOM 내부에서 `background-color: var(--primary-color);`를 사용할 수 있습니다. Shadcn/ui가 CSS 변수를 많이 사용하므로, 이 방법을 통해 일부 테마를 동기화할 수 있습니다.

5.  **근본적인 CSS 이해의 중요성:**
    결국, 이러한 복잡한 상황에 직면했을 때 가장 필요한 것은 CSS의 동작 원리, 특히 스코프, 상속, 명시도(specificity) 등에 대한 깊이 있는 이해입니다. 유틸리티 프레임워크가 개발 생산성을 높여주지만, 그 이면에 있는 CSS의 본질을 이해하지 못하면 예상치 못한 문제에 부딪혔을 때 해결하기 어렵습니다. Sylwia Lask님의 글처럼, 2026년에도 CSS 학습은 결코 낭비가 아닙니다.

## 마무리: 현명한 기술 스택 선택과 학습 방향

Tailwind, Shadcn/ui, 그리고 Shadow DOM은 각각 프론트엔드 개발에 강력한 도구이지만, 그들의 핵심 원리(전역적 유틸리티 스타일링 vs. 강력한 격리)가 충돌할 때 개발자에게 예상치 못한 난관을 안겨줄 수 있습니다. 기술 스택을 선택할 때는 각 기술의 장점뿐만 아니라, 서로 간의 호환성과 잠재적 충돌 지점을 면밀히 검토하는 것이 중요합니다.

특히 Shadow DOM과 같은 웹 컴포넌트 기술을 활용할 때는 스타일링 전략에 대한 깊은 고민이 필요하며, 이는 결국 CSS의 본질적인 이해로 귀결됩니다. 복잡한 프론트엔드 생태계 속에서 현명한 선택을 통해 더 효율적이고 견고한 애플리케이션을 구축하시길 바랍니다. 다음 포스팅에서는 Shadow DOM 내부에서 Shadcn/ui 컴포넌트를 사용하기 위한 좀 더 구체적인 워크어라운드에 대해 다뤄볼 수도 있겠네요!

---

## 참고 자료

*   [I Know This Will Upset Some Devs, but Tailwind + Shadcn/ui + Shadow DOM = Pain](https://dev.to/ujja/i-know-this-will-upset-some-devs-but-tailwind-shadcnui-shadow-dom-pain-44l7) by ujja
*   [I Built a Next.js SaaS Boilerplate So You Don't Have To - 3 Month Journey](https://dev.to/muhammadzubair/i-built-a-nextjs-saas-boilerplate-so-you-dont-have-to-3-month-journey-fk5) by Muhammad Zubair
*   [Mastering Data Sanitization in Legacy React Applications: A Lead QA Engineer's Approach](https://dev.to/bishoy_semsem/mastering-data-sanitization-in-legacy-react-applications-a-lead-qa-engineers-approach-5aep) by Bishoy Semsem
*   [Why I Didn’t Post for 3 Days — And What I Finally Learned 🔍](https://dev.to/usama_dev/why-i-didnt-post-for-3-days-and-what-i-finally-learned-4do1) by Usama
*   [Day 5 of #100DaysOfCode — Fetching Data in React (useEffect + fetch + axios)](https://dev.to/m_saad_ahmad/day-5-of-100daysofcode-fetching-data-in-react-useeffect-fetch-axios-46lk) by M Saad Ahmad