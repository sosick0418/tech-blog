---
layout: post
title: "Next.js의 마법: ISR로 정적 페이지에 생명을 불어넣다!"
date: 2026-02-24T01:01:04.218Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 기술 블로거입니다!
오늘 날짜는 2026년 2월 24일이네요. 빠르게 변화하는 프론트엔드 기술 트렌드 속에서, 오늘도 여러분의 개발 여정에 도움이 될 만한 흥미로운 주제를 가져왔습니다.

---



## 소개
프론트엔드 개발의 세계는 속도와 사용자 경험의 끊임없는 추구입니다. 정적 사이트의 압도적인 성능과 서버 사이드 렌더링(SSR)의 동적 유연성 사이에서 항상 고민하셨나요? 2026년 2월, Next.js의 Incremental Static Regeneration (ISR)은 이 두 가지 장점을 모두 취할 수 있는 강력한 해법으로 자리매김했습니다. 이 패턴은 빌드 시점에 생성된 정적 페이지를 런타임에 주기적으로 업데이트하여, 마치 살아있는 페이지처럼 최신 데이터를 제공하면서도 정적 페이지의 장점을 그대로 유지하게 해줍니다.

## 본문

### 핵심 개념 설명: ISR, 정적 페이지의 진화

ISR은 Next.js 프레임워크의 핵심 기능 중 하나로, 빌드 후에도 정적 페이지를 업데이트할 수 있게 해주는 기술입니다. 기존의 정적 사이트 생성(SSG)은 빌드 시점에 모든 페이지를 미리 생성하므로 매우 빠르지만, 데이터가 변경되면 다시 빌드해야 하는 단점이 있었습니다. 반면, 서버 사이드 렌더링(SSR)은 항상 최신 데이터를 보여주지만, 요청마다 페이지를 생성해야 하므로 SSG보다 느릴 수 있습니다.

ISR은 이 두 가지 방식의 단점을 보완합니다. 페이지가 빌드된 후에도 `revalidate` 옵션을 통해 지정된 시간 간격으로 백그라운드에서 페이지를 재생성합니다. 사용자는 항상 이전에 생성된(캐시된) 페이지를 즉시 받게 되며, 재생성이 완료되면 다음 요청부터는 최신 버전의 페이지를 받게 됩니다. 이는 사용자에게 빠른 응답 속도를 제공하면서도 항상 신선한 콘텐츠를 유지할 수 있게 해줍니다.

결과적으로, ISR은 SEO에 유리하고, 사용자 경험을 향상시키며, 서버 부하를 줄이는 일석삼조의 효과를 얻을 수 있습니다. 마치 정적 페이지가 스스로를 업데이트하는 마법과도 같죠!

### 실제 코드 예시 (Next.js, TypeScript 활용)

ISR을 Next.js 애플리케이션에 적용하는 것은 매우 간단합니다. `getStaticProps` 함수에 `revalidate` 속성을 추가하기만 하면 됩니다. 아래는 상품 상세 페이지를 ISR로 구현하는 예시입니다.

```typescript
// pages/products/[id].tsx
import { GetStaticProps, GetStaticPaths, NextPage } from 'next';
import { useRouter } from 'next/router';

// 상품 데이터 타입을 정의합니다.
interface Product {
  id: string;
  name: string;
  price: number;
  updatedAt: string;
}

// 페이지 컴포넌트의 props 타입을 정의합니다.
interface ProductPageProps {
  product: Product;
}

// 상품 상세 페이지 컴포넌트
const ProductPage: NextPage<ProductPageProps> = ({ product }) => {
  const router = useRouter();

  // fallback: true 또는 'blocking'일 때,
  // 아직 생성되지 않은 페이지에 접근하면 로딩 상태를 보여줄 수 있습니다.
  if (router.isFallback) {
    return <div>로딩 중...</div>;
  }

  return (
    <div>
      <h1>{product.name}</h1>
      <p>상품 ID: {product.id}</p>
      <p>가격: {product.price.toLocaleString()}원</p>
      <p>최종 업데이트: {product.updatedAt}</p>
      <p style={{ fontSize: '0.8em', color: '#666' }}>
        (이 페이지는 {product.updatedAt}에 서버에서 생성되었으며, 10초마다 최신 데이터를 확인합니다.)
      </p>
    </div>
  );
};

// 동적 경로를 미리 정의합니다.
export const getStaticPaths: GetStaticPaths = async () => {
  // 실제 환경에서는 API 호출을 통해 모든 상품 ID를 가져옵니다.
  // 여기서는 예시로 2개의 상품 ID를 가정합니다.
  const products = [{ id: '1' }, { id: '2' }, { id: '3' }]; // 예시 상품 ID
  const paths = products.map((product) => ({
    params: { id: product.id },
  }));

  return {
    paths,
    fallback: 'blocking', // 중요: 'blocking' 또는 true를 사용해야 ISR이 작동합니다.
                          // 'blocking'은 페이지가 생성될 때까지 기다렸다가 보여줍니다.
                          // true는 로딩 상태를 보여준 후 페이지를 렌더링합니다.
  };
};

// 페이지의 데이터를 빌드 시점 또는 ISR 주기에 따라 가져옵니다.
export const getStaticProps: GetStaticProps<ProductPageProps> = async ({ params }) => {
  const id = params?.id as string;

  // 실제 환경에서는 `id`를 사용하여 API에서 상품 데이터를 가져옵니다.
  // 여기서는 임의의 데이터를 생성합니다.
  const productData: Product = {
    id,
    name: `프론트엔드 마스터 키트 ${id}`,
    price: Math.floor(Math.random() * 100000) + 29900, // 매번 다른 가격
    updatedAt: new Date().toLocaleString('ko-KR', { timeZone: 'Asia/Seoul' }),
  };

  // 만약 데이터가 없으면 404를 반환할 수 있습니다.
  if (!productData) {
    return {
      notFound: true,
    };
  }

  return {
    props: {
      product: productData,
    },
    revalidate: 10, // 10초마다 페이지 재생성을 시도합니다.
                    // 이 값을 통해 ISR이 동작합니다.
  };
};

export default ProductPage;
```

위 코드에서 가장 핵심적인 부분은 `getStaticProps` 함수 내의 `revalidate: 10` 입니다. 이는 Next.js에게 이 페이지를 최소 10초마다 한 번씩 백그라운드에서 다시 생성(revalidate)하도록 지시합니다. 사용자가 페이지에 접근하면, 10초가 지나지 않았다면 이전에 캐시된 페이지를 즉시 보여줍니다. 10초가 지났다면, 캐시된 페이지를 보여주는 동시에 백그라운드에서 최신 데이터를 가져와 페이지를 재생성합니다. 다음 사용자가 접근할 때는 새로 생성된 페이지를 보게 됩니다.

`getStaticPaths`의 `fallback: 'blocking'` 옵션도 중요합니다. 이는 `getStaticPaths`에 의해 미리 생성되지 않은 경로에 사용자가 접근했을 때, Next.js가 페이지를 생성할 때까지 기다렸다가 완전히 렌더링된 페이지를 보여주도록 합니다. `fallback: true`를 사용하면 로딩 상태를 먼저 보여준 후 페이지를 렌더링합니다.

### 실무 적용 팁

ISR을 효과적으로 사용하기 위한 몇 가지 팁을 공유합니다.

1.  **언제 ISR을 사용할까?**
    *   **블로그 게시물, 뉴스 기사, 제품 상세 페이지:** 데이터가 자주 업데이트되지만, 모든 요청마다 실시간으로 데이터를 가져올 필요는 없는 콘텐츠에 적합합니다. 예를 들어, 블로그 글의 조회수나 추천 상품 목록 등이 주기적으로 업데이트될 때 유용합니다.
    *   **정적 페이지의 장점을 유지하면서 데이터 신선도가 필요한 경우:** SEO 최적화가 중요하고, 빠른 초기 로딩 속도가 필수적인 페이지에 이상적입니다.

2.  **언제 ISR을 사용하지 않을까?**
    *   **사용자별로 완전히 다른 콘텐츠를 보여줘야 하는 페이지:** 개인화된 대시보드, 장바구니 페이지 등은 클라이언트 사이드 데이터 페칭이나 SSR이 더 적합합니다.
    *   **초 단위의 실시간 업데이트가 필요한 경우:** 주식 시세, 채팅 메시지, 게임 스코어보드 등은 웹소켓이나 클라이언트 사이드 렌더링(CSR)으로 데이터를 실시간으로 가져오는 것이 좋습니다.

3.  **`revalidate` 값 설정:**
    *   `revalidate` 값(초 단위)은 서비스의 데이터 업데이트 주기와 사용자 경험 사이의 균형을 고려하여 신중하게 설정해야 합니다. 너무 짧으면 불필요한 서버 부하가 발생할 수 있고, 너무 길면 데이터 신선도가 떨어질 수 있습니다.
    *   일반적으로 몇 분에서 몇 시간 단위로 설정하는 경우가 많습니다.

4.  **`fallback` 옵션 이해:**
    *   `fallback: false`: `getStaticPaths`에 정의되지 않은 경로는 404 페이지를 반환합니다. 모든 페이지를 미리 알고 있을 때 사용합니다.
    *   `fallback: true`: 빌드되지 않은 경로에 접근 시, `router.isFallback`이 `true`가 되며 로딩 상태를 보여준 후 백그라운드에서 페이지를 생성하고, 생성이 완료되면 해당 페이지를 보여줍니다. 이후 요청부터는 생성된 페이지를 제공합니다.
    *   `fallback: 'blocking'`: 빌드되지 않은 경로에 접근 시, 페이지가 완전히 생성될 때까지 기다린 후 렌더링된 페이지를 보여줍니다. 로딩 상태 없이 바로 콘텐츠를 보여줄 수 있어 SEO에 더 유리할 수 있습니다.

5.  **On-demand Revalidation 활용:**
    *   Next.js 12부터는 `revalidate` 주기와 상관없이 특정 이벤트(예: CMS에서 게시물 수정)가 발생했을 때 수동으로 페이지를 재생성할 수 있는 On-demand Revalidation 기능이 추가되었습니다. 이는 데이터 변경 시 즉시 최신 페이지를 반영해야 하는 경우에 매우 유용하며, `revalidate`와 함께 사용하면 더욱 강력한 제어력을 가질 수 있습니다.

## 마무리

Next.js의 ISR은 정적 페이지의 속도와 동적 페이지의 유연성을 결합하여 프론트엔드 개발에 새로운 지평을 열었습니다. 적절한 `revalidate` 주기 설정과 `fallback` 옵션의 이해는 ISR의 잠재력을 최대한 발휘하는 데 필수적입니다. 이제 여러분의 Next.js 애플리케이션에 ISR을 적용하여 사용자 경험과 성능이라는 두 마리 토끼를 모두 잡아보세요!

다음번에는 Next.js의 다른 렌더링 패턴(SSR, SSG, CSR)과 ISR의 심화된 활용법, 예를 들어 온디맨드 리밸리데이션(On-demand Revalidation)에 대해 더 깊이 탐구해보는 시간을 가져보겠습니다.

---

## 참고 자료
- [ReactJS(NextJs) Rendering Pattern　~Incremental Static Regeneration (ISR)~](https://dev.to/kkr0423/reactjsnextjs-rendering-pattern-incremental-static-regeneration-isr-261m) by Ogasawara Kakeru
- [How to Convert SVG to React Components: Complete Guide](https://dev.to/arenasbob2024cell/how-to-convert-svg-to-react-components-complete-guide-34jc) by arenasbob2024-cell
- [Stop Repeating React Setup: Introducing create-react-forge](https://dev.to/chiragmak10/stop-repeating-react-setup-introducing-create-react-forge-29hd) by Chirag
- [Patrones de diseño de componentes de React: una guía completa](https://dev.to/carlos_mruiz_e1fbb61ca7/patrones-de-diseno-de-componentes-de-react-una-guia-completa-4ndm) by Carlos m. Ruiz
- [React Router: Loaders, Actions & Form](https://dev.to/edriso/react-router-loaders-actions-form-2bbe) by Mohamed Idris