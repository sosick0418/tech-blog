---
layout: post
title: "정적 페이지의 한계를 넘어서: Next.js ISR로 성능과 신선함을 동시에 잡는 법!"
date: 2026-02-25T01:00:42.974Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거 개발자 K입니다. 2026년 2월 25일, 오늘도 빠르게 변화하는 프론트엔드 기술의 최전선에서 흥미로운 주제를 가져왔습니다.

---



웹 애플리케이션 개발에서 사용자 경험의 핵심은 바로 '속도'와 '최신성'입니다. 정적 페이지는 뛰어난 성능과 SEO(검색 엔진 최적화) 이점을 제공하지만, 데이터 업데이트에 취약하다는 단점이 있었죠. 반면 서버 사이드 렌더링(SSR)은 항상 최신 데이터를 보여주지만, 첫 로딩 속도에서 불리할 수 있습니다.

이 두 가지 장점을 모두 취할 수 있는 혁신적인 렌더링 패턴이 바로 Next.js의 Incremental Static Regeneration, 줄여서 ISR입니다. 오늘은 ISR이 무엇인지, 어떻게 활용하는지 깊이 있게 다루며, 여러분의 Next.js 프로젝트에 어떻게 적용할 수 있을지 알아보겠습니다.

---

### ISR, 정적 페이지의 진화를 이끌다

ISR은 Next.js 프레임워크가 제공하는 강력한 기능으로, 빌드 시점에 생성된 정적 페이지를 배포 후에도 주기적으로 업데이트할 수 있게 해줍니다. 즉, 한 번 만들어진 정적 페이지가 영원히 '정적'으로 머무는 것이 아니라, 설정된 주기에 따라 최신 데이터를 반영하며 '재생성'되는 방식입니다.

기존 SSG(Static Site Generation)는 빌드 시점에 모든 페이지를 미리 생성하여 CDN에 배포하기 때문에 매우 빠른 로딩 속도와 뛰어난 SEO를 자랑합니다. 하지만 데이터가 변경되어도 다음 빌드 전까지는 변경 사항이 반영되지 않는 한계가 있습니다. 반대로 SSR(Server-Side Rendering)은 요청이 올 때마다 서버에서 페이지를 렌더링하므로 항상 최신 데이터를 보여주지만, 매 요청마다 서버 부하가 발생하고 첫 로딩 시간이 길어질 수 있습니다.

ISR은 이 두 가지 방식의 장점을 결합합니다. 처음 사용자에게는 빌드된 정적 페이지를 제공하여 빠른 로딩 경험을 선사하고, 이후 설정된 `revalidate` 시간(초 단위)이 지나면 백그라운드에서 페이지를 다시 생성하여 업데이트된 내용을 다음 사용자에게 제공합니다. 이는 'stale-while-revalidate' 전략과 유사하게 작동하며, 사용자 경험을 저해하지 않으면서 데이터의 신선도를 유지합니다.

---

### ISR 코드 예시: Next.js에서 구현하기

ISR은 `getStaticProps` 함수 내에서 `revalidate` 속성을 설정함으로써 간단하게 구현할 수 있습니다. 다음은 특정 상품 상세 페이지를 ISR로 구현하는 예시입니다.

```typescript
import { GetStaticPaths, GetStaticProps } from 'next';
import { useRouter } from 'next/router';
import React from 'react';

// 상품 데이터 타입을 정의합니다.
interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
  lastUpdatedAt: string; // 데이터 업데이트 시간을 추적하기 위한 필드
}

// 페이지 컴포넌트의 props 타입을 정의합니다.
interface ProductPageProps {
  product: Product;
}

// 상품 상세 페이지 컴포넌트
const ProductDetailPage: React.FC<ProductPageProps> = ({ product }) => {
  const router = useRouter();

  // fallback: true일 때, 페이지가 아직 생성되지 않았다면 로딩 UI를 보여줍니다.
  if (router.isFallback) {
    return <div>상품 정보를 로딩 중입니다...</div>;
  }

  return (
    <div>
      <h1>{product.name}</h1>
      <p>가격: {product.price.toLocaleString()}원</p>
      <p>{product.description}</p>
      <small>최종 업데이트: {new Date(product.lastUpdatedAt).toLocaleString()}</small>
      <p>
        <a href="/">홈으로 돌아가기</a>
      </p>
    </div>
  );
};

// Next.js가 미리 생성할 경로(페이지)를 정의합니다.
export const getStaticPaths: GetStaticPaths = async () => {
  // 실제 API 호출을 통해 미리 생성할 상품 ID 목록을 가져옵니다.
  // 여기서는 예시로 일부 ID만 하드코딩합니다.
  const products = [{ id: '101' }, { id: '102' }, { id: '103' }]; 

  const paths = products.map((product) => ({
    params: { id: product.id },
  }));

  return {
    paths,
    // fallback: true: 빌드 시점에 생성되지 않은 경로에 대한 요청이 오면,
    // Next.js는 해당 페이지를 백그라운드에서 생성하고 캐싱합니다.
    // 사용자는 이 과정 동안 로딩 상태를 보게 됩니다.
    fallback: true, 
    
    // fallback: 'blocking': 빌드 시점에 생성되지 않은 경로에 대한 요청이 오면,
    // Next.js는 페이지가 완전히 생성될 때까지 기다렸다가 페이지를 제공합니다.
    // 사용자는 로딩 상태 없이 바로 완성된 페이지를 보게 됩니다.
    // fallback: 'blocking', 

    // fallback: false: getStaticPaths에 명시되지 않은 경로는 404 페이지를 반환합니다.
  };
};

// 각 페이지에 필요한 데이터를 가져옵니다.
export const getStaticProps: GetStaticProps<ProductPageProps> = async ({ params }) => {
  const productId = params?.id as string;

  // 실제 API 호출을 통해 상품 데이터를 가져옵니다.
  // fetch 대신 axios 등 다른 라이브러리를 사용할 수도 있습니다.
  const res = await fetch(`https://api.example.com/products/${productId}`);
  const product: Product = await res.json();

  // 데이터가 없는 경우 404 페이지를 반환합니다.
  if (!product || Object.keys(product).length === 0) {
    return {
      notFound: true,
    };
  }

  return {
    props: {
      product: {
        ...product,
        lastUpdatedAt: new Date().toISOString(), // 예시를 위해 업데이트 시간 기록
      },
    },
    // ISR의 핵심! 60초마다 페이지를 재생성(revalidate)합니다.
    // 60초가 지나고 첫 요청이 오면, 기존 캐시된 페이지를 보여주고
    // 백그라운드에서 새로운 데이터를 가져와 페이지를 다시 빌드합니다.
    // 다음 요청부터는 업데이트된 페이지를 제공합니다.
    revalidate: 60, 
  };
};

export default ProductDetailPage;
```

위 코드에서 `getStaticProps` 내의 `revalidate: 60` 부분이 핵심입니다. 이는 Next.js에게 이 페이지를 60초마다 백그라운드에서 다시 생성(revalidate)하도록 지시합니다. 사용자가 페이지에 접근하면, 60초가 지나지 않았다면 캐시된 정적 페이지를 즉시 보여줍니다. 60초가 지난 후 첫 요청이 들어오면, 여전히 캐시된 페이지를 보여주지만, 동시에 백그라운드에서 새로운 데이터를 가져와 페이지를 다시 빌드하고 다음 요청부터는 업데이트된 페이지를 제공합니다.

또한 `getStaticPaths`의 `fallback: true` 설정은 빌드 시점에 미리 생성되지 않은 경로에 대한 요청이 들어왔을 때, 해당 페이지를 즉시 생성하여 보여준 후 캐싱하도록 합니다. 이는 대규모 데이터셋을 가진 페이지에서 모든 경로를 미리 빌드할 필요 없이 ISR의 이점을 누릴 수 있게 해줍니다.

---

### ISR 실무 적용 팁: 현명하게 활용하기

ISR은 매우 강력하지만, 몇 가지 고려사항을 통해 더욱 효율적으로 활용할 수 있습니다.

**1. 적절한 `revalidate` 주기 설정:**
데이터의 변경 빈도와 신선도 요구사항에 따라 `revalidate` 값을 신중하게 결정해야 합니다. 블로그 게시물처럼 업데이트가 잦지 않은 콘텐츠는 긴 주기를, 가격 변동이 잦은 상품 페이지는 짧은 주기를 설정하는 것이 좋습니다. 너무 짧은 주기는 불필요한 서버 부하를 유발할 수 있습니다.

**2. `fallback` 옵션 이해:**
`fallback: true`는 사용자에게 로딩 상태를 보여준 후 페이지를 렌더링합니다. `fallback: 'blocking'`은 페이지가 생성될 때까지 기다렸다가 완전히 렌더링된 페이지를 보여줍니다. 어떤 사용자 경험을 제공할지에 따라 선택하세요. `fallback: false`는 `getStaticPaths`에 명시되지 않은 경로는 404를 반환합니다.

**3. 수동 재검증 (On-demand Revalidation):**
특정 이벤트(예: 관리자 페이지에서 상품 정보 수정) 발생 시 즉시 페이지를 업데이트해야 하는 경우가 있습니다. Next.js 12부터는 `revalidate` API를 통해 특정 경로를 수동으로 재검증할 수 있습니다. 이는 Next.js 서버리스 함수나 API 라우트에서 호출하여 캐시를 즉시 무효화하고 페이지를 재생성하는 데 사용됩니다.

```typescript
// pages/api/revalidate.ts (혹은 app/api/revalidate/route.ts)
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  // 보안을 위해 비밀 토큰을 확인합니다.
  if (req.query.secret !== process.env.MY_SECRET_TOKEN) {
    return res.status(401).json({ message: 'Invalid token' });
  }

  try {
    // 쿼리 파라미터로 받은 경로를 재검증합니다.
    const path = req.query.path as string;
    await res.revalidate(path);
    return res.json({ revalidated: true, path });
  } catch (err) {
    // 오류 발생 시
    console.error(err);
    return res.status(500).send('Error revalidating');
  }
}
```
이처럼 특정 API 엔드포인트를 만들어 외부 시스템에서 호출함으로써 필요한 시점에만 페이지를 업데이트할 수 있습니다. 예를 들어, CMS에서 글을 발행하거나 수정할 때 이 API를 호출하여 해당 블로그 게시물 페이지를 즉시 업데이트할 수 있습니다.

**4. 에러 핸들링:**
`getStaticProps` 내에서 데이터를 가져오는 과정에서 에러가 발생했을 때, 기존 캐시된 페이지를 유지하고 에러를 로깅하는 등의 전략을 고려해야 합니다. 데이터 페칭 로직에 견고한 `try-catch` 블록을 포함하여 빌드 실패나 사용자에게 빈 페이지가 노출되는 것을 방지하는 것이 중요합니다.

---

### 마무리하며: ISR, 미래의 웹을 위한 필수 도구

오늘 우리는 Next.js의 Incremental Static Regeneration(ISR)에 대해 자세히 알아보았습니다. ISR은 정적 페이지의 놀라운 성능과 SEO 이점을 유지하면서도, 데이터의 신선도를 실시간에 가깝게 유지할 수 있게 해주는 혁신적인 렌더링 전략입니다.

복잡한 캐싱 전략 없이도 빠르고 항상 최신 상태를 유지하는 웹사이트를 구축하고자 한다면, ISR은 Next.js 개발자에게 필수적인 도구입니다. 블로그, 뉴스 사이트, 전자상거래 등 데이터 업데이트가 필요한 다양한 유형의 웹 애플리케이션에서 ISR의 강력함을 경험해보세요.

다음번에는 ISR을 더욱 효율적으로 관리하는 고급 전략이나, Next.js 13+ App Router에서의 데이터 페칭 전략 등 더 깊이 있는 주제로 찾아뵙겠습니다. 계속해서 프론트엔드 기술 트렌드에 주목해주세요!

---

## 참고 자료

- [ReactJS(NextJs) Rendering Pattern　~Incremental Static Regeneration (ISR)~](https://dev.to/kkr0423/reactjsnextjs-rendering-pattern-incremental-static-regeneration-isr-261m) by Ogasawara Kakeru
- [How to Convert SVG to React Components: Complete Guide](https://dev.to/arenasbob2024cell/how-to-convert-svg-to-react-components-complete-guide-34jc) by arenasbob2024-cell
- [Stop Repeating React Setup: Introducing create-react-forge](https://dev.to/chiragmak10/stop-repeating-react-setup-introducing-create-react-forge-29hd) by Chirag
- [Patrones de diseño de componentes de React: una guía completa](https://dev.to/carlos_mruiz_e1fbb61ca7/patrones-de-diseno-de-componentes-de-react-una-guia-completa-4ndm) by Carlos m. Ruiz
- [React Router: Loaders, Actions & Form](https://dev.to/edriso/react-router-loaders-actions-form-2bbe) by Mohamed Idris