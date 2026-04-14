---
layout: post
title: "URL 바를 리액트 State 저장소로! `state-in-url`로 만드는 강력한 사용자 경험 🚀"
date: 2026-04-14T01:01:13.482Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거입니다! 2026년 4월 14일, 오늘도 흥미로운 프론트엔드 트렌드를 들고 여러분을 찾아왔습니다. 빠르게 변화하는 기술 세계에서 유저에게 더 나은 경험을 제공하고, 개발 효율을 높이는 방법은 항상 우리의 가장 큰 관심사죠.

오늘 살펴볼 주제는 바로 'URL'입니다. 웹 애플리케이션에서 URL은 단순히 페이지 경로를 나타내는 것을 넘어, 이제는 우리의 애플리케이션 '상태'를 담는 강력한 도구로 진화하고 있습니다. 특히 React 환경에서 `state-in-url` 라이브러리가 제시하는 아이디어는 사용자 경험과 개발 패턴에 새로운 지평을 열어줄 것으로 기대됩니다.

---



## 💡 소개: URL, 단순한 주소창을 넘어선 무한한 가능성

여러분은 웹 애플리케이션을 사용하면서 특정 필터나 검색 조건, 혹은 모달 창의 상태를 친구에게 공유하고 싶었던 경험이 있으신가요? 아니면 페이지를 새로고침했을 때, 이전의 복잡한 설정이 모두 초기화되어 당황했던 적은요? 기존의 React `useState` 훅은 컴포넌트 내부 상태를 관리하기엔 훌륭하지만, 이 상태를 URL과 동기화하여 공유 가능하고, 새로고침에도 유지되게 하는 것은 별도의 노력이 필요했습니다.

오늘 다룰 `state-in-url` 라이브러리는 이러한 고민을 해결해줍니다. URL의 쿼리 파라미터를 활용하여 React 컴포넌트의 실제 JavaScript 값을 저장하고 관리함으로써, 사용자 경험을 혁신적으로 개선하고 개발자에게는 더욱 직관적인 상태 관리 방식을 제공합니다. 이제 URL은 단순한 주소창이 아니라, 앱의 현재 상태를 완벽하게 반영하는 '살아있는' 인터페이스가 될 것입니다.

## 🚀 본문: `state-in-url`의 핵심 개념과 실전 활용

### 핵심 개념 설명: URL 쿼리 파라미터와 React State의 만남

`state-in-url`의 핵심은 애플리케이션의 특정 상태(예: 검색어, 필터 값, 페이지네이션 정보, 모달 열림 여부 등)를 URL의 쿼리 파라미터로 관리하는 것입니다. 이를 통해 다음과 같은 강력한 이점을 얻을 수 있습니다.

1.  **상태 공유 용이성:** 특정 상태를 가진 페이지 URL을 그대로 복사하여 공유하면, 다른 사용자도 동일한 상태의 페이지를 볼 수 있습니다.
2.  **새로고침 시 상태 유지:** 페이지를 새로고침하더라도 URL에 상태가 저장되어 있으므로, 애플리케이션이 이전 상태를 그대로 복원합니다.
3.  **브라우저 히스토리 연동:** 상태 변경이 URL 변경으로 이어지기 때문에, 브라우저의 뒤로 가기/앞으로 가기 버튼이 애플리케이션 상태 변경에도 자연스럽게 작동합니다.
4.  **SEO 친화적 (부분적으로):** 특정 상태에 따라 콘텐츠가 달라지는 페이지의 경우, URL에 상태 정보가 포함되어 있어 검색 엔진이 더 많은 정보를 색인화할 가능성이 생깁니다.

이 라이브러리는 숫자, 문자열, 불리언은 물론, 객체나 배열과 같은 복잡한 JavaScript 값까지도 URL 쿼리 파라미터로 직렬화(serialize)하고 역직렬화(deserialize)하는 기능을 제공합니다. 개발자는 마치 `useState`를 사용하는 것처럼 `useUrlState` 훅을 사용하여 URL 기반 상태를 쉽게 관리할 수 있습니다.

### 실제 코드 예시: Next.js와 TypeScript로 구현하는 URL 기반 필터링

우리는 Next.js 환경에서 상품 목록 필터링 기능을 `state-in-url`의 개념을 활용하여 구현해보겠습니다. 사용자가 검색어를 입력하거나 카테고리를 변경하면, 이 상태가 URL에 반영되고, 새로고침해도 유지되는 시나리오입니다.

먼저, `state-in-url` 라이브러리에서 영감을 받은 간단한 `useUrlState` 훅을 정의해봅시다. 실제 라이브러리는 더 많은 최적화와 기능을 제공하지만, 여기서는 핵심 원리를 보여줍니다.

```typescript
// hooks/useUrlState.ts
import { useState, useEffect, useCallback } from 'react';
import { useRouter } from 'next/router';

interface UseUrlStateOptions<T> {
  serializer?: (value: T) => string;
  deserializer?: (value: string) => T;
  debounceMs?: number; // 디바운싱 추가
}

/**
 * URL 쿼리 파라미터를 React State처럼 관리하는 훅
 * @param key URL 쿼리 파라미터의 키
 * @param defaultValue 초기 값
 * @param options 직렬화/역직렬화 함수, 디바운싱 설정
 */
function useUrlState<T extends Record<string, any>>(
  key: string,
  defaultValue: T,
  options?: UseUrlStateOptions<T>
): [T, (newValue: T | ((prev: T) => T)) => void] {
  const router = useRouter();
  const {
    serializer = JSON.stringify,
    deserializer = JSON.parse,
    debounceMs = 0,
  } = options || {};

  // URL에서 초기 상태를 파싱합니다.
  const getInitialState = useCallback((): T => {
    if (typeof window === 'undefined') return defaultValue; // SSR 방지
    try {
      const param = router.query[key];
      if (typeof param === 'string') {
        return deserializer(decodeURIComponent(param));
      }
    } catch (e) {
      console.error(`[useUrlState] Failed to deserialize URL param for key "${key}":`, e);
    }
    return defaultValue;
  }, [router.query, key, defaultValue, deserializer]);

  const [state, setState] = useState<T>(getInitialState);

  // URL 쿼리 파라미터 변경 감지 및 상태 동기화
  useEffect(() => {
    // router.isReady가 true가 될 때까지 기다립니다.
    if (!router.isReady) return;
    setState(getInitialState());
  }, [router.isReady, router.query, getInitialState]);

  // 상태 업데이트 및 URL 반영 (디바운싱 적용)
  const setUrlState = useCallback(
    (newValue: T | ((prev: T) => T)) => {
      setState((prev) => {
        const finalValue = typeof newValue === 'function' ? (newValue as (prev: T) => T)(prev) : newValue;

        const newQuery = {
          ...router.query,
          [key]: encodeURIComponent(serializer(finalValue)),
        };

        // 디바운싱 적용
        const timer = setTimeout(() => {
          router.push(
            {
              pathname: router.pathname,
              query: newQuery,
            },
            undefined,
            { shallow: true } // 페이지 리로드 없이 URL만 변경
          );
        }, debounceMs);

        return () => clearTimeout(timer); // 클린업 함수
      });
    },
    [router, key, serializer, debounceMs]
  );

  return [state, setUrlState];
}

export default useUrlState;
```

이제 이 훅을 사용하여 상품 필터 컴포넌트를 만들어보겠습니다.

```typescript
// components/ProductFilter.tsx
import React, { useEffect } from 'react';
import useUrlState from '../hooks/useUrlState'; // 위에서 정의한 훅 임포트

interface ProductFilterState {
  searchQuery: string;
  category: 'all' | 'electronics' | 'clothing' | 'books';
  minPrice: number;
}

const ProductFilter: React.FC = () => {
  const [filterState, setFilterState] = useUrlState<ProductFilterState>(
    'productFilters', // URL 쿼리 파라미터 키
    { searchQuery: '', category: 'all', minPrice: 0 }, // 초기값
    { debounceMs: 500 } // 검색어 입력 시 500ms 디바운싱
  );

  const handleSearchChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setFilterState((prev) => ({ ...prev, searchQuery: e.target.value }));
  };

  const handleCategoryChange = (e: React.ChangeEvent<HTMLSelectElement>) => {
    setFilterState((prev) => ({ ...prev, category: e.target.value }));
  };

  const handleMinPriceChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setFilterState((prev) => ({ ...prev, minPrice: parseInt(e.target.value) || 0 }));
  };

  // filterState가 변경될 때마다 상품 목록을 다시 불러오는 로직 (예시)
  useEffect(() => {
    console.log('--- 필터 상태 변경 감지 ---');
    console.log('현재 검색어:', filterState.searchQuery);
    console.log('현재 카테고리:', filterState.category);
    console.log('최소 가격:', filterState.minPrice);
    // 실제 API 호출 로직을 여기에 구현
    // fetchProducts(filterState);
  }, [filterState]);

  return (
    <div style={{ padding: '20px', border: '1px solid #ddd', borderRadius: '8px' }}>
      <h3>상품 필터</h3>
      <div style={{ marginBottom: '10px' }}>
        <label htmlFor="search">검색어:</label>
        <input
          id="search"
          type="text"
          value={filterState.searchQuery}
          onChange={handleSearchChange}
          placeholder="상품명 검색..."
          style={{ marginLeft: '10px', padding: '5px' }}
        />
      </div>
      <div style={{ marginBottom: '10px' }}>
        <label htmlFor="category">카테고리:</label>
        <select
          id="category"
          value={filterState.category}
          onChange={handleCategoryChange}
          style={{ marginLeft: '10px', padding: '5px' }}
        >
          <option value="all">모든 카테고리</option>
          <option value="electronics">가전</option>
          <option value="clothing">의류</option>
          <option value="books">도서</option>
        </select>
      </div>
      <div style={{ marginBottom: '10px' }}>
        <label htmlFor="minPrice">최소 가격:</label>
        <input
          id="minPrice"
          type="number"
          value={filterState.minPrice}
          onChange={handleMinPriceChange}
          placeholder="최소 가격"
          style={{ marginLeft: '10px', padding: '5px' }}
        />
      </div>
      <p style={{ marginTop: '15px', fontSize: '0.9em', color: '#555' }}>
        **URL 변경을 확인해보세요!** <br />
        예시 URL: `/products?productFilters=%7B%22searchQuery%22%3A%22react%22%2C%22category%22%3A%22books%22%2C%22minPrice%22%3A10000%7D`
      </p>
    </div>
  );
};

export default ProductFilter;
```

위 예시에서 `ProductFilter` 컴포넌트는 `useUrlState` 훅을 사용하여 `searchQuery`, `category`, `minPrice` 상태를 관리합니다. 이 상태들이 변경될 때마다 URL의 `productFilters` 쿼리 파라미터가 업데이트되고, 페이지를 새로고침하거나 URL을 공유해도 동일한 필터 상태가 유지되는 것을 확인할 수 있습니다. `shallow: true` 옵션 덕분에 페이지 전체가 리로드되지 않고 URL만 변경되어 부드러운 사용자 경험을 제공합니다.

### 실무 적용 팁: 현명하게 `state-in-url` 활용하기

`state-in-url`은 강력하지만, 모든 상황에 만능은 아닙니다. 현명하게 활용하기 위한 몇 가지 팁을 드립니다.

1.  **언제 사용하면 좋을까요?**
    *   **필터링, 검색, 페이지네이션:** 목록형 UI에서 가장 유용합니다. 사용자가 특정 조건으로 데이터를 조회한 후 그 상태를 유지하거나 공유해야 할 때.
    *   **탭, 모달, 아코디언 상태:** 특정 탭이 열려있거나 모달이 팝업된 상태를 URL에 반영하여 공유하거나 새로고침 시 유지할 수 있습니다.
    *   **테마, 언어 설정:** 전역적인 사용자 설정 중 URL에 반영되어야 할 때.
2.  **주의할 점:**
    *   **민감한 정보는 절대 금물:** 비밀번호, 토큰 등 민감한 정보는 URL에 노출되어서는 안 됩니다.
    *   **URL 길이 제한:** 대부분의 브라우저는 URL 길이에 제한이 있습니다 (보통 2000~4000자). 너무 많은 상태나 매우 큰 객체를 저장하면 문제가 될 수 있습니다.
    *   **잦은 업데이트:** 입력 필드처럼 실시간으로 자주 변경되는 상태를 URL에 직접 반영하면 URL 변경이 너무 빈번해질 수 있습니다. 위 예시처럼 `debounceMs` 옵션을 통해 디바운싱을 적용하는 것이 좋습니다.
    *   **복잡한 객체 직렬화:** 기본 `JSON.stringify`/`JSON.parse`로 처리하기 어려운 Map, Set, Date 객체 등은 `serializer`와 `deserializer` 옵션을 통해 커스텀 로직을 구현해야 합니다.
3.  **Next.js `shallow` 라우팅:** Next.js에서 `router.push` 시 `{ shallow: true }` 옵션을 사용하면 페이지 전체를 다시 불러오지 않고 URL 쿼리 파라미터만 변경하여 상태를 업데이트할 수 있습니다. 이는 성능과 사용자 경험 측면에서 매우 중요합니다.

## 맺음말: URL의 무한한 잠재력을 깨우다

오늘 우리는 URL 바를 단순한 주소창이 아닌, 애플리케이션의 '실시간 상태 저장소'로 활용하는 `state-in-url`의 개념과 실전 예시를 살펴보았습니다. 이 접근 방식은 사용자에게는 더욱 강력하고 직관적인 경험을, 개발자에게는 상태 관리의 새로운 패러다임을 제공합니다. 공유 가능한 URL, 새로고침에도 유지되는 상태, 그리고 브라우저 히스토리와의 자연스러운 연동은 현대 웹 애플리케이션이 추구해야 할 중요한 가치들입니다.

물론, 모든 상태를 URL에 저장할 필요는 없습니다. 컴포넌트 내부에서만 필요한 일시적인 상태는 `useState`를, 전역적으로 관리하되 URL과 연동될 필요가 없는 상태는 Context API나 Recoil, Zustand 같은 전역 상태 관리 라이브러리를 사용하는 것이 적절합니다. `state-in-url`은 이들 상태 관리 솔루션의 빈틈을 채워주는, 현명하게 선택될 때 큰 시너지를 낼 수 있는 도구입니다.

여러분도 여러분의 프로젝트에 `state-in-url` 아이디어를 적용하여 사용자 경험을 한 단계 업그레이드해보시는 건 어떨까요? 다음번에는 또 다른 흥미로운 프론트엔드 트렌드로 찾아오겠습니다!

---

## 참고 자료

-   [How I Built state-in-url: My Journey Turning the URL Bar into Real React State](https://dev.to/asmyshlyaev177/how-i-built-state-in-url-my-journey-turning-the-url-bar-into-real-react-state-a67) by Alex
-   [Deploying Full Stack Projects using Vercel and Render](https://dev.to/shiv11013/deploying-full-stack-projects-using-vercel-and-render-3c2j) by shiv-11013
-   [MUI v4/v5 Style Conflicts in single-spa — A Complete Debug Record](https://dev.to/czhoudev/mui-v4v5-style-conflicts-in-single-spa-a-complete-debug-record-438k) by Chloe Zhou
-   [How We Ran Two Portals on the Same Domain During a React Migration (Without Users Noticing)](https://dev.to/gustav_lotz_5628d28bc9eac/how-we-ran-two-portals-on-the-same-domain-during-a-react-migration-without-users-noticing-28g0) by gustav lotz
-   [How I Built KisanX — A Farmer-to-Consumer Marketplace with MERN Stack 🌾](https://dev.to/aniket_dev/how-i-built-kisanx-a-farmer-to-consumer-marketplace-with-mern-stack-45da) by Aniket Dubey