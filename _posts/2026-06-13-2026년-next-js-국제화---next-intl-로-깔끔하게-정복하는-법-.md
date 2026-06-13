---
layout: post
title: "2026년 Next.js 국제화, `next-intl`로 깔끔하게 정복하는 법!"
date: 2026-06-13T01:00:35.675Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거입니다! 2026년 6월 13일, 오늘도 빠르게 변화하는 프론트엔드 세계의 흥미로운 소식들을 가지고 왔습니다.

---



## 소개
글로벌 서비스가 보편화되면서 국제화(i18n)는 더 이상 선택이 아닌 필수가 되었습니다. 하지만 막상 프로젝트에 적용하려 하면 복잡한 로케일 관리, 메시지 번들링, 그리고 프레임워크와의 통합 문제로 개발자들의 발목을 잡곤 합니다. 특히 Next.js의 앱 라우터(App Router) 시대에 접어들면서, 서버 컴포넌트와 클라이언트 컴포넌트 간의 경계를 넘나드는 국제화 구현은 더욱 까다로워졌죠.

오늘 소개해드릴 `next-intl` 라이브러리는 이러한 Next.js 환경에서의 국제화 작업을 혁신적으로 간소화해줍니다. 2026년 현재, `next-intl`은 Next.js 앱 라우터와 완벽하게 통합되어, 개발자가 언어 장벽 없이 사용자에게 최적의 경험을 제공할 수 있도록 돕는 강력한 도구로 자리매김했습니다.

## 본문

### 핵심 개념 설명
`next-intl`은 Next.js의 앱 라우터 철학에 맞춰 설계된 국제화 라이브러리입니다. 주요 특징은 다음과 같습니다:

1.  **앱 라우터 완벽 지원:** `app` 디렉토리 구조와 서버 컴포넌트, 클라이언트 컴포넌트 모두에서 자연스럽게 작동합니다.
2.  **타입스크립트 기반:** 메시지 파일의 키에 대한 타입 추론을 제공하여 런타임 오류를 줄이고 개발 생산성을 높입니다.
3.  **성능 최적화:** 서버 컴포넌트에서 필요한 메시지만 로딩하고, 클라이언트 컴포넌트에서는 번들 크기를 최소화하여 뛰어난 성능을 보장합니다.
4.  **간결한 API:** 직관적인 API를 통해 복잡한 국제화 로직을 쉽게 구현할 수 있습니다.

`next-intl`은 URL을 통해 로케일을 관리하는 방식을 권장합니다 (예: `/en/dashboard`, `/ko/dashboard`). 이를 통해 SEO 친화적인 구조를 가지며, 서버 컴포넌트에서 요청된 로케일에 맞는 메시지를 효율적으로 로드할 수 있습니다.

### 실제 코드 예시 (React/Next.js/TypeScript 활용)

`next-intl`을 Next.js 프로젝트에 적용하는 기본적인 단계를 살펴보겠습니다.

#### 1. 설치 및 초기 설정
먼저 `next-intl`과 관련 패키지를 설치합니다:
```bash
npm install next-intl
# 또는 yarn add next-intl
```

그 다음, 프로젝트 루트에 `i18n.ts` 파일을 생성하여 국제화 설정을 정의합니다.

```typescript
// i18n.ts
import { getRequestConfig } from 'next-intl/server';
import { notFound } from 'next/navigation';

// 지원하는 로케일 목록
const locales = ['en', 'ko'];

export default getRequestConfig(async ({ locale }) => {
  // 로케일이 지원 목록에 없으면 404 페이지를 렌더링
  if (!locales.includes(locale as any)) notFound();

  // 해당 로케일의 메시지 파일을 동적으로 로드
  // `messages` 폴더에 `en.json`, `ko.json` 파일이 있다고 가정
  return {
    messages: (await import(`./messages/${locale}.json`)).default
  };
});
```

#### 2. 미들웨어 설정
Next.js 미들웨어를 사용하여 요청 URL에서 로케일을 감지하고 라우팅을 처리합니다.

```typescript
// middleware.ts
import createMiddleware from 'next-intl/middleware';

export default createMiddleware({
  // 지원하는 로케일 목록
  locales: ['en', 'ko'],

  // 기본 로케일
  defaultLocale: 'en',

  // 로케일 접두사를 항상 사용하도록 설정 (예: /en, /ko)
  localePrefix: 'always'
});

export const config = {
  // 미들웨어가 적용될 경로를 지정
  // '/'를 포함하여 모든 경로에 적용하고, API 라우트, 정적 파일 등은 제외
  matcher: ['/', '/(ko|en)/:path*', '/((?!_next|_vercel|.*\\..*).*)']
};
```

#### 3. 레이아웃에 `NextIntlClientProvider` 적용
`app/[locale]/layout.tsx` 파일에서 `NextIntlClientProvider`를 사용하여 클라이언트 컴포넌트에서 국제화 기능을 사용할 수 있도록 설정합니다.

```typescript
// app/[locale]/layout.tsx
import { NextIntlClientProvider } from 'next-intl';
import { getMessages } from 'next-intl/server';
import './globals.css'; // 전역 CSS 임포트

export default async function LocaleLayout({
  children,
  params: { locale }
}: {
  children: React.ReactNode;
  params: { locale: string };
}) {
  // 서버 컴포넌트에서 모든 메시지를 로드
  const messages = await getMessages();

  return (
    <html lang={locale}>
      <body>
        <NextIntlClientProvider messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  );
}
```

#### 4. 메시지 파일 생성
`messages` 폴더 안에 각 로케일별 JSON 파일을 생성합니다.

```json
// messages/en.json
{
  "home": {
    "title": "Welcome to Next.js i18n!",
    "description": "This is an example using next-intl.",
    "greeting": "Hello, {name}!"
  },
  "localeSwitcher": {
    "label": "Change language:",
    "english": "English",
    "korean": "Korean"
  }
}
```

```json
// messages/ko.json
{
  "home": {
    "title": "Next.js 국제화에 오신 것을 환영합니다!",
    "description": "next-intl을 사용한 예시입니다.",
    "greeting": "안녕하세요, {name}님!"
  },
  "localeSwitcher": {
    "label": "언어 변경:",
    "english": "영어",
    "korean": "한국어"
  }
}
```

#### 5. 서버 컴포넌트에서 사용
`getTranslator` 또는 `getMessages`를 사용하여 서버 컴포넌트에서 메시지를 가져옵니다.

```typescript
// app/[locale]/page.tsx (Server Component)
import { getTranslator } from 'next-intl/server';
import { LocaleSwitcher } from '@/components/LocaleSwitcher'; // 클라이언트 컴포넌트 임포트

export default async function HomePage({ params: { locale } }: { params: { locale: string } }) {
  // 'home' 네임스페이스의 번역 함수를 가져옵니다.
  // getTranslator는 필요한 메시지만 로드하여 번들 크기를 최적화합니다.
  const t = await getTranslator(locale, 'home');

  return (
    <main className="flex min-h-screen flex-col items-center justify-center p-24">
      <h1 className="text-4xl font-bold mb-4">{t('title')}</h1>
      <p className="text-lg mb-8">{t('description')}</p>
      <p className="text-xl mb-8">{t('greeting', { name: '개발자' })}</p>
      <LocaleSwitcher />
    </main>
  );
}
```

#### 6. 클라이언트 컴포넌트에서 사용
`useTranslations` 훅을 사용하여 클라이언트 컴포넌트에서 메시지를 가져옵니다.

```typescript
// components/LocaleSwitcher.tsx (Client Component)
'use client';

import { useTranslations } from 'next-intl';
import { usePathname, useRouter } from 'next/navigation';
import { ChangeEvent } from 'react';

export function LocaleSwitcher() {
  const t = useTranslations('localeSwitcher'); // 'localeSwitcher' 네임스페이스의 번역 함수를 가져옵니다.
  const router = useRouter();
  const pathname = usePathname();

  const handleLocaleChange = (e: ChangeEvent<HTMLSelectElement>) => {
    const newLocale = e.target.value;
    // 현재 경로를 기반으로 로케일만 변경하여 이동합니다.
    // pathname은 '/ko/dashboard'와 같은 형태이므로, 첫 3글자(/ko)를 교체합니다.
    const newPath = `/${newLocale}${pathname.substring(3)}`;
    router.push(newPath);
  };

  // 현재 로케일을 알아내기 위해 pathname에서 추출합니다.
  const currentLocale = pathname.substring(1, 3);

  return (
    <div className="mt-8">
      <label htmlFor="locale-select" className="mr-2 text-gray-700">{t('label')}</label>
      <select
        id="locale-select"
        onChange={handleLocaleChange}
        defaultValue={currentLocale}
        className="border border-gray-300 rounded-md p-2"
      >
        <option value="en">{t('english')}</option>
        <option value="ko">{t('korean')}</option>
      </select>
    </div>
  );
}
```

### 실무 적용 팁

1.  **메시지 파일 네임스페이스 활용:**
    프로젝트 규모가 커지면 모든 메시지를 하나의 JSON 파일에 담는 것은 비효율적입니다. `home`, `auth`, `products`, `common` 등 기능별로 네임스페이스를 분리하여 메시지 파일을 관리하세요. `getTranslator(locale, 'home')`처럼 특정 네임스페이스만 로드하여 성능을 최적화할 수 있습니다.

2.  **폴백 언어 설정:**
    `i18n.ts` 또는 `NextIntlClientProvider`에서 `defaultLocale`을 설정하여 특정 언어의 메시지가 누락되었을 때 기본 언어로 폴백되도록 할 수 있습니다. 이는 사용자 경험에 매우 중요합니다.

3.  **변수 및 복수형 처리:**
    `next-intl`은 메시지 내 변수 삽입 (`{name}`)과 복수형 처리 (`{count, plural, one {item} other {items}}`)를 지원합니다. 이를 통해 동적인 문구를 유연하게 처리할 수 있습니다.
    ```json
    // messages/en.json
    {
      "cart": {
        "itemCount": "{count, plural, one {# item} other {# items}} in your cart."
      }
    }
    ```
    ```typescript
    // 사용 예시
    t('cart.itemCount', { count: 1 }); // "1 item in your cart."
    t('cart.itemCount', { count: 5 }); // "5 items in your cart."
    ```

4.  **성능 최적화:**
    *   **서버 컴포넌트 우선:** 가능한 한 많은 텍스트를 서버 컴포넌트에서 번역하여 클라이언트 측 번들 크기를 줄이세요.
    *   **동적 임포트:** `i18n.ts`에서 메시지 파일을 동적으로 임포트하여 필요한 로케일의 파일만 로드하도록 합니다.
    *   **캐싱:** Next.js의 내장 캐싱 메커니즘과 `next-intl`의 서버 측 메시지 로딩은 기본적으로 효율적으로 작동합니다.

5.  **테스팅 전략:**
    국제화된 애플리케이션은 각 로케일별로 UI가 올바르게 렌더링되는지 확인하는 E2E 테스트가 필수적입니다. Cypress나 Playwright 같은 도구를 사용하여 URL을 변경하며 언어별 화면을 검증하는 시나리오를 추가하세요.

## 마무리

2026년, Next.js와 함께 글로벌 서비스를 구축하는 프론트엔드 개발자에게 `next-intl`은 더 이상 선택이 아닌 필수 도구입니다. 복잡하게 느껴졌던 국제화 작업을 간결하고 효율적으로 만들어주며, Next.js 앱 라우터의 강력한 기능들을 최대한 활용할 수 있도록 돕습니다. 타입스크립트 지원으로 개발 안정성까지 확보하니, 이보다 더 좋은 조합은 찾기 어려울 것입니다.

오늘 소개해드린 내용을 바탕으로 여러분의 Next.js 프로젝트에 `next-intl`을 적용해보세요. 처음에는 설정이 다소 복잡하게 느껴질 수 있지만, 한번 구축하고 나면 깔끔한 코드와 뛰어난 유지보수성에 만족하실 겁니다. 다음 학습 방향으로는 `next-intl`의 공식 문서를 통해 고급 서식 지정, 리치 텍스트 번역, 그리고 커스텀 로더 구현 등을 살펴보시길 권장합니다. 글로벌 사용자에게 최고의 경험을 선사하는 여러분의 서비스를 기대합니다!

## 참고 자료
- [The Best JavaScript Slider Library? Introducing Pagiflow: A Zero-Dependency, High-Performance Carousel](https://dev.to/pagiflow/the-best-javascript-slider-library-introducing-pagiflow-a-zero-dependency-high-performance-12ch) by Pagiflow
- [Localizing a Shopify Customer Account Extension: 42 Languages, One Rails Server, and a Cache That Never Hit](https://dev.to/hanswys/localizing-a-shopify-customer-account-extension-42-languages-one-rails-server-and-a-cache-that-oh6) by hans
- [Why modal.open() should return Promise<TResult>, not Promise<any>](https://dev.to/alexey79/why-modalopen-should-return-promise-not-promise-12g) by Oleksii Kyrychenko
- [next-intl: The Complete Next.js i18n Guide (2026)](https://dev.to/stacknotice/next-intl-the-complete-nextjs-i18n-guide-2026-139o) by Carlos Oliva Pascual
- [Vue to React Migration in Action: Real CRM Admin Case](https://dev.to/smirk9581/vue-to-react-migration-in-action-real-crm-admin-case-5257) by Ryan John