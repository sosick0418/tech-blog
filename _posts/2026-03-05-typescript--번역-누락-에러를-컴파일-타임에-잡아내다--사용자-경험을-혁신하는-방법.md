---
layout: post
title: "TypeScript, 번역 누락 에러를 컴파일 타임에 잡아내다: 사용자 경험을 혁신하는 방법"
date: 2026-03-05T01:06:48.872Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 기술 블로거입니다!

오늘 날짜는 2026년 3월 5일입니다. 빠르게 변화하는 프론트엔드 생태계 속에서 오늘은 특히 개발 생산성과 사용자 경험을 동시에 혁신할 수 있는 흥미로운 주제를 다뤄보려 합니다.

---



## 소개

글로벌 서비스를 개발할 때 다국어(i18n) 지원은 이제 선택이 아닌 필수가 되었습니다. 하지만 번역 키가 누락되어 애플리케이션에 엉뚱한 문자열이 보이거나, 심지어 빈 화면이 뜨는 경험, 다들 있으실 겁니다. 이러한 번역 누락 문제는 대부분 런타임에 발견되어 사용자에게 직접적인 불편을 초래하고, 개발팀은 뒤늦게 부랴부랴 수정 패치를 배포해야 하는 상황에 놓이곤 합니다.

오늘은 이처럼 치명적인 런타임 에러를 사전에 방지하고, 개발 단계에서부터 번역의 일관성과 정확성을 보장하는 혁신적인 방법을 소개합니다. 바로 TypeScript의 강력한 타입 시스템을 활용하여 번역 누락 에러를 *컴파일 타임*에 잡아내는 것이죠. 사용자 경험은 물론 개발 생산성까지 한 단계 끌어올릴 수 있는 이 기술, 함께 알아볼까요?

## 본문

### 핵심 개념 설명: 런타임 오류에서 컴파일 타임 안정성으로

대부분의 React i18n 라이브러리는 번역 키를 찾지 못할 경우 기본 문자열을 반환하거나 오류를 발생시킵니다. 문제는 이러한 오류가 사용자가 해당 화면에 접근했을 때, 즉 애플리케이션이 이미 배포된 후에야 발견된다는 점입니다. 이는 사용자 경험을 저해하고, 서비스의 신뢰도를 떨어뜨리는 주요 원인이 됩니다.

이 문제의 핵심은 "번역 키의 유효성을 런타임에 검사한다"는 점에 있습니다. 우리가 원하는 것은 코드를 작성하는 순간, 즉 컴파일 타임에 번역 키가 유효한지 여부를 확인하는 것입니다. TypeScript는 바로 이 지점에서 빛을 발합니다.

핵심 아이디어는 간단합니다. 모든 번역 파일(예: `ko.json`, `en.json`)에 정의된 키들을 TypeScript의 타입으로 정의하고, 번역 함수(`t` 함수)가 이 타입에 정의된 키만을 인자로 받도록 강제하는 것입니다. 이렇게 하면 존재하지 않는 번역 키를 사용하려 할 때 TypeScript 컴파일러가 즉시 오류를 발생시켜 개발자가 문제를 미리 인지하고 수정할 수 있게 됩니다.

### 실제 코드 예시 (React/Next.js/TypeScript 활용)

먼저, 번역 파일이 있다고 가정해봅시다.

```json
// src/locales/ko.json
{
  "common.greeting": "안녕하세요!",
  "common.welcomeMessage": "환영합니다, {{name}}님!",
  "error.networkFailed": "네트워크 연결에 실패했습니다."
}

// src/locales/en.json
{
  "common.greeting": "Hello!",
  "common.welcomeMessage": "Welcome, {{name}}!",
  "error.networkFailed": "Network connection failed."
}
```

이제 이 번역 파일의 키들을 TypeScript 타입으로 정의합니다. 가장 간단한 방법은 `typeof` 연산자를 활용하여 JSON 파일의 타입을 추론하는 것입니다.

```typescript
// src/i18n/types.ts
// 이 파일은 스크립트를 통해 자동 생성하는 것이 일반적입니다. (아래 팁 참조)
// 여기서는 설명을 위해 수동으로 작성했다고 가정합니다.

import ko from '../locales/ko.json'; // 기본 언어 파일을 기준으로 타입을 생성합니다.

// ko.json의 모든 최상위 키를 포함하는 타입을 생성합니다.
// 중첩된 객체의 키를 다루려면 재귀적인 타입 정의가 필요합니다.
// 여기서는 간단화를 위해 최상위 키만 다루는 예시를 보여줍니다.
type NestedKeys<T> = T extends object
  ? {
      [K in keyof T]: K extends string
        ? T[K] extends object
          ? `${K}.${NestedKeys<T[K]>}`
          : K
        : never;
    }[keyof T]
  : '';

export type TranslationKeys = NestedKeys<typeof ko>;

// 예시: TranslationKeys는 'common.greeting' | 'common.welcomeMessage' | 'error.networkFailed' 와 같은 형태가 됩니다.
```

다음으로, 이 `TranslationKeys` 타입을 활용하는 `useTranslation` 훅을 구현합니다.

```typescript
// src/i18n/useTranslation.ts
import { useState, useEffect } from 'react';
import { TranslationKeys } from './types';
import ko from '../locales/ko.json';
import en from '../locales/en.json';

// 실제 앱에서는 사용자 설정에 따라 동적으로 로드됩니다.
const locales = {
  ko: ko,
  en: en,
};

type Params = Record<string, string | number>;

export function useTranslation() {
  const [currentLocale, setCurrentLocale] = useState<'ko' | 'en'>('ko'); // 예시: 현재 언어 상태 관리
  const [translations, setTranslations] = useState<typeof ko>(locales[currentLocale]);

  useEffect(() => {
    setTranslations(locales[currentLocale]);
  }, [currentLocale]);

  const t = (key: TranslationKeys, params?: Params): string => {
    // 중첩된 키를 처리하는 간단한 로직 (예: 'common.greeting' -> translations.common.greeting)
    const keys = key.split('.');
    let value: any = translations;
    for (const k of keys) {
      if (value && typeof value === 'object' && k in value) {
        value = value[k];
      } else {
        // 컴파일 타임에 걸러지겠지만, 런타임 안전성을 위한 폴백
        console.warn(`[i18n] Missing translation for key: ${key}`);
        return `MISSING_KEY: ${key}`;
      }
    }

    let translatedString: string = String(value);

    // 변수 삽입 (예: {{name}})
    if (params) {
      for (const paramKey in params) {
        translatedString = translatedString.replace(
          new RegExp(`{{${paramKey}}}`, 'g'),
          String(params[paramKey])
        );
      }
    }

    return translatedString;
  };

  const changeLanguage = (lang: 'ko' | 'en') => {
    setCurrentLocale(lang);
  };

  return { t, changeLanguage, currentLocale };
}
```

이제 React 컴포넌트에서 이 훅을 사용해봅시다.

```tsx
// src/components/MyComponent.tsx
import React from 'react';
import { useTranslation } from '../i18n/useTranslation';

function MyComponent() {
  const { t, changeLanguage } = useTranslation();

  return (
    <div>
      <h1>{t('common.greeting')}</h1>
      <p>{t('common.welcomeMessage', { name: '프론트엔드 개발자' })}</p>
      <button onClick={() => changeLanguage('en')}>Change to English</button>
      <button onClick={() => changeLanguage('ko')}>Change to Korean</button>

      {/* 🚨 이곳에서 TypeScript 오류가 발생합니다! */}
      {/* 'common.nonExistentKey'는 TranslationKeys 타입에 존재하지 않으므로 컴파일 오류가 납니다. */}
      {/* <p>{t('common.nonExistentKey')}</p> */}

      {/* 'error.networkFailed'는 정상적으로 작동합니다. */}
      <p>{t('error.networkFailed')}</p>
    </div>
  );
}

export default MyComponent;
```

위 코드에서 `t('common.nonExistentKey')`와 같이 존재하지 않는 키를 사용하려 하면, TypeScript 컴파일러가 즉시 오류를 보고하여 배포 전에 문제를 해결할 수 있게 됩니다. 이는 개발자가 코드를 작성하는 시점에 번역 파일과의 불일치를 파악할 수 있게 해주는 강력한 기능입니다.

### 실무 적용 팁

1.  **자동화된 타입 생성 스크립트:** `src/i18n/types.ts` 파일을 수동으로 관리하는 것은 번거롭고 오류 발생 가능성이 높습니다. 대신, `json-to-ts`와 같은 라이브러리를 사용하거나 간단한 Node.js 스크립트를 작성하여 모든 `.json` 번역 파일을 스캔하고 `TranslationKeys` 타입을 자동으로 생성하도록 만드세요. 이 스크립트를 `package.json`의 `prebuild` 또는 `predev` 스크립트에 포함시키면 좋습니다.

    ```json
    // package.json 예시
    {
      "scripts": {
        "generate:i18n-types": "node scripts/generate-i18n-types.js",
        "dev": "npm run generate:i18n-types && next dev",
        "build": "npm run generate:i18n-types && next build"
      }
    }
    ```
    (실제 `generate-i18n-types.js` 스크립트 구현은 파일 시스템 접근 및 타입 문자열 생성 로직이 필요합니다.)

2.  **CI/CD 파이프라인 통합:** 자동화된 타입 생성 스크립트를 CI/CD 파이프라인에 포함시켜, 코드가 빌드될 때마다 번역 키의 유효성을 검증하도록 합니다. 이렇게 하면 개발자가 로컬에서 놓친 오류도 배포 전에 잡아낼 수 있습니다.

3.  **동적 키 처리:** 때로는 번역 키가 API 응답에서 오거나 런타임에 동적으로 결정될 수 있습니다. 이런 경우에는 `t(dynamicKey as TranslationKeys)`와 같이 타입 단언(type assertion)을 사용해야 할 수도 있습니다. 하지만 이는 타입 안전성을 포기하는 것이므로, 가능한 한 동적 키 사용을 최소화하고, 만약 사용해야 한다면 해당 키들이 번역 파일에 실제로 존재하는지 런타임 검사를 추가하는 것이 좋습니다.

4.  **변수 삽입(Interpolation)의 타입 안전성:** `{{name}}`과 같은 변수 삽입 시, `params` 객체의 키들이 실제로 번역 문자열에 필요한 변수와 일치하는지까지 검사할 수 있다면 더 좋습니다. 이는 좀 더 복잡한 타입 추론과 리터럴 타입 활용이 필요하며, 특정 i18n 라이브러리(예: `react-i18next`의 `Trans` 컴포넌트)가 제공하는 기능을 활용하는 것이 효율적일 수 있습니다.

5.  **모노레포 환경:** 여러 서비스가 하나의 번역 파일을 공유하는 모노레포 환경에서는, 번역 관련 타입 정의를 공유 패키지로 분리하여 모든 프로젝트에서 일관된 타입 안전성을 유지할 수 있습니다.

## 마무리

TypeScript를 활용하여 번역 누락 에러를 컴파일 타임에 잡아내는 접근 방식은 단순한 코드 개선을 넘어, 애플리케이션의 안정성과 사용자 경험을 획기적으로 향상시킵니다. 개발자는 더 이상 런타임에 발생할지 모르는 번역 오류에 대한 걱정 없이 코드를 작성할 수 있게 되고, 사용자는 완벽하고 일관된 다국어 환경을 경험할 수 있습니다.

이러한 접근 방식은 i18n 라이브러리에만 국한되지 않고, 애플리케이션 전반에서 사용되는 상수 문자열, 라우트 경로, API 엔드포인트 등 모든 종류의 문자열 관리에 적용되어 개발 생산성과 코드 품질을 높일 수 있습니다. 다음 단계로는 더 정교한 타입 추론, 린팅 규칙 통합, 그리고 번역 관리 시스템(TMS)과의 연동을 통해 이 시스템을 더욱 강력하게 만드는 방법을 탐구해볼 수 있을 것입니다.

여러분의 프론트엔드 개발 여정에 이 아이디어가 큰 도움이 되기를 바랍니다!

---

## 참고 자료

-   [React: Singletons aren't as evil as you think](https://dev.to/link2twenty/react-singletons-arent-as-evil-as-you-think-44m8) by Andrew Bone
-   [How I Made Missing Translations a Compile-Time TypeScript Error](https://dev.to/akocan98/how-i-made-missing-translations-a-compile-time-typescript-error-5bjc) by Amer Kočan
-   [Why AI Crawlers Can't See Your React App](https://dev.to/jsvisible/why-ai-crawlers-cant-see-your-react-app-137n) by JSVisible
-   [I replaced console.log with a real-time error dashboard that runs on localhost](https://dev.to/meghshyams/i-replaced-consolelog-with-a-real-time-error-dashboard-that-runs-on-localhost-342a) by Meghshyam Sonar
-   [I Built a macOS Menu Bar App to Track My Freelance Time — Here's How](https://dev.to/splyy/i-built-a-macos-menu-bar-app-to-track-my-freelance-time-heres-how-4bil) by MZP