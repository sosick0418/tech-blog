---
layout: post
title: "🚀 프론트엔드 성능의 비밀 병기: Next.js ISR로 정적 페이지를 동적으로!"
date: 2026-02-23T08:03:10.822Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거, **프론트엔드 마스터K**입니다! 🚀

2026년 2월 23일, 오늘날의 프론트엔드 기술은 그야말로 눈부신 발전을 거듭하고 있습니다. 사용자 경험과 성능 최적화는 이제 선택이 아닌 필수가 되었죠. 오늘은 이 두 마리 토끼를 잡을 수 있는 Next.js의 강력한 기능, **ISR (Incremental Static Regeneration)**에 대해 깊이 파헤쳐 보겠습니다.

---



## 💡 왜 ISR이 중요할까요? 정적 페이지의 한계를 넘어서

현대 웹 애플리케이션에서 사용자 경험과 검색 엔진 최적화(SEO)는 무엇보다 중요합니다. 이를 위해 정적 페이지 생성(SSG)은 페이지 로딩 속도를 극대화하고 서버 부하를 줄이는 훌륭한 방법으로 각광받아 왔습니다. 하지만 SSG는 치명적인 단점이 있었죠. 바로 **데이터가 업데이트될 때마다 전체 웹사이트를 재빌드하고 재배포해야 한다는 점**입니다. 이는 콘텐츠가 자주 바뀌는 블로그, 뉴스 사이트, 이커머스 등에서 큰 비효율을 초래합니다.

여기서 Next.js의 **ISR(Incremental Static Regeneration)**이 등장합니다. ISR은 SSG의 놀라운 성능 이점을 유지하면서도, 페이지를 주기적으로 또는 필요에 따라 "증분적으로" 업데이트할 수 있게 하여 정적 페이지의 한계를 극복합니다. 사용자는 항상 최신 데이터를 보면서도, SSG가 제공하는 빠른 로딩 속도를 경험할 수 있게 되는 것이죠.

## 🛠️ ISR, 핵심 개념 파헤치기

ISR은 Next.js 프레임워크가 제공하는 기능으로, 빌드 시점에 페이지를 HTML 파일로 미리 생성(SSG)한 후, 설정된 시간(revalidate)이 지나면 백그라운드에서 해당 페이지를 다시 생성하여 최신 데이터로 업데이트하는 방식입니다.

핵심은 다음과 같습니다:

1.  **초기 요청**: 사용자가 페이지에 처음 접근하면, 빌드 시점에 생성된 정적 페이지를 즉시 제공합니다. 매우 빠르게 로딩되죠.
2.  **백그라운드 재생성**: 페이지가 제공된 후, `revalidate` 옵션에 설정된 시간이 경과하고 다음 요청이 들어오면, Next.js는 백그라운드에서 해당 페이지를 다시 빌드합니다.
3.  **Stale-while-revalidate**: 이 과정에서 사용자는 여전히 이전에 캐시된(Stale) 페이지를 보게 됩니다. 백그라운드 재생성이 완료되면, 다음 요청부터는 새로 생성된(Fresh) 페이지를 제공합니다. 마치 HTTP 캐시 헤더의 `stale-while-validate`와 유사한 개념입니다.

이러한 방식으로 ISR은 초기 로딩 속도를 보장하면서도, 데이터 업데이트에 유연하게 대응할 수 있게 합니다.

### 🧑‍💻 실제 코드 예시 (Next.js & TypeScript)

간단한 블로그 포스트 페이지를 ISR로 구현하는 예시를 살펴보겠습니다. `getStaticProps` 함수 내에서 `revalidate` 옵션을 설정하는 것이 핵심입니다.

```typescript
// pages/posts/[id].tsx
import { GetStaticProps, GetStaticPaths, NextPage } from 'next';

interface Post {
  id: string;
  title: string;
  content: string;
  updatedAt: string;
}

interface PostPageProps {
  post: Post;
}

const PostPage: NextPage<PostPageProps> = ({ post }) => {
  if (!post) {
    return <div>로딩 중이거나 게시물을 찾을 수 없습니다...</div>;
  }

  return (
    <div style={{ padding: '20px', maxWidth: '800px', margin: '0 auto' }}>
      <h1>{post.title}</h1>
      <p>최종 업데이트: {new Date(post.updatedAt).toLocaleDateString()}</p>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </div>
  );
};

export const getStaticPaths: GetStaticPaths = async () => {
  // 실제 API 호출을 통해 모든 게시물 ID를 가져와야 합니다.
  // 여기서는 예시를 위해 하드코딩합니다.
  const posts = [
    { id: '1', title: '첫 번째 글', content: '...', updatedAt: new Date().toISOString() },
    { id: '2', title: '두 번째 글', content: '...', updatedAt: new Date().toISOString() },
  ];
  const paths = posts.map(post => ({
    params: { id: post.id },
  }));

  return {
    paths,
    fallback: 'blocking', // 'blocking' 또는 true/false 설정 가능
    // 'blocking': 새 경로 요청 시 SSR처럼 동작하여 페이지를 생성 후 제공
    // true: 로딩 상태를 보여주고 클라이언트 측에서 데이터 페칭 시도
    // false: 정의되지 않은 경로는 404 페이지
  };
};

export const getStaticProps: GetStaticProps<PostPageProps, { id: string }> = async ({ params }) => {
  const postId = params?.id;

  // 실제 API 호출을 통해 게시물 데이터를 가져옵니다.
  // 예시를 위해 가상의 데이터를 사용합니다.
  const response = await fetch(`https://api.example.com/posts/${postId}`);
  const post: Post = await response.json();

  if (!post) {
    return {
      notFound: true, // 게시물이 없으면 404 페이지 반환
      revalidate: 60, // 60초 후 재검증 시도
    };
  }

  return {
    props: {
      post,
    },
    revalidate: 60, // 이 페이지는 60초마다 백그라운드에서 재생성됩니다.
    // 즉, 60초가 지나고 새로운 요청이 들어오면 기존 캐시를 보여주고,
    // 백그라운드에서 데이터를 다시 가져와 페이지를 업데이트합니다.
  };
};

export default PostPage;
```

위 코드에서 주목할 부분은 `getStaticProps` 내의 `revalidate: 60` 입니다. 이는 해당 페이지가 배포된 후 60초마다 백그라운드에서 데이터를 다시 가져와 페이지를 재생성하도록 지시합니다. 사용자는 최대 60초 전의 데이터를 볼 수 있지만, 그 이후의 요청부터는 새로 업데이트된 페이지를 보게 됩니다.

`getStaticPaths`는 동적 라우트(`[id].tsx`)에서 어떤 경로들을 미리 생성할지 Next.js에게 알려주는 함수입니다. `fallback: 'blocking'`은 미리 생성되지 않은 경로에 대한 요청이 들어오면, 서버에서 해당 페이지를 즉시 생성하여 응답하고, 이후 캐시하여 ISR 주기에 포함시키겠다는 의미입니다.

### 💡 실무 적용 팁

1.  **`revalidate` 값 설정 전략**:
    *   **데이터 변경 빈도 고려**: 뉴스 기사처럼 실시간에 가까운 업데이트가 필요한 경우 `revalidate` 값을 짧게 (예: 10초~60초) 설정할 수 있습니다. 반면, 거의 변경되지 않는 회사 소개 페이지 같은 경우엔 길게 (예: 1시간~24시간) 설정하여 서버 부하를 줄일 수 있습니다.
    *   **너무 짧은 값의 주의점**: `revalidate` 값이 너무 짧으면 사실상 SSR과 유사하게 매 요청마다 서버에서 페이지를 생성하는 부담이 생길 수 있습니다. 적절한 균형을 찾는 것이 중요합니다.
    *   **사용자 경험 vs. 서버 리소스**: 사용자가 최신 데이터를 보게 하는 것과 서버 리소스 사용량 사이의 균형점을 찾아야 합니다.

2.  **On-Demand Revalidation (Next.js 12.2+)**:
    *   ISR의 `revalidate` 옵션은 시간 기반으로 자동 업데이트되지만, 데이터베이스나 CMS에서 특정 콘텐츠가 변경되었을 때 즉시 페이지를 업데이트하고 싶을 때가 있습니다. Next.js 12.2부터는 `revalidate` API를 통해 특정 경로에 대한 재생성을 수동으로 트리거할 수 있습니다.
    *   예를 들어, CMS에서 게시물 수정 후 웹훅을 통해 Next.js 서버의 `revalidate` 엔드포인트를 호출하여 해당 게시물 페이지를 즉시 업데이트할 수 있습니다. 이는 ISR의 유연성을 극대화하는 강력한 기능입니다.

3.  **에러 핸들링 및 폴백 페이지**:
    *   `getStaticProps` 내부에서 데이터 페칭에 실패할 경우, `notFound: true`를 반환하여 404 페이지를 보여주거나, 폴백 데이터를 제공하는 로직을 구현할 수 있습니다.
    *   `fallback: 'blocking'`을 사용하는 경우, 새로운 페이지가 생성되는 동안 사용자는 빈 화면을 보지 않고 이전에 캐시된 페이지 또는 로딩 상태를 자연스럽게 경험할 수 있도록 UI를 고려해야 합니다.

4.  **CDN 활용**:
    *   ISR로 생성된 정적 파일들은 CDN(Content Delivery Network)에 캐시될 수 있으므로, 전 세계 사용자에게 더욱 빠르게 콘텐츠를 전달할 수 있습니다. Next.js를 Vercel에 배포하는 경우, Vercel의 엣지 네트워크가 자동으로 이를 처리해줍니다.

## 🌟 마무리하며: 성능과 유연성을 동시에 잡는 ISR

Next.js의 ISR은 정적 웹사이트의 장점인 뛰어난 성능과 동적 웹사이트의 장점인 최신 데이터 제공 능력을 효과적으로 결합한 매우 강력한 기능입니다. 초기 로딩 속도, SEO, 그리고 콘텐츠 관리의 유연성까지 한 번에 잡을 수 있죠.

오늘 살펴본 ISR 개념과 코드 예시, 그리고 실무 적용 팁들을 바탕으로 여러분의 Next.js 프로젝트에 ISR을 적극적으로 도입해보시길 바랍니다. 웹 애플리케이션의 성능과 사용자 경험이 한 단계 더 도약할 수 있을 거예요! 다음 시간에는 ISR과 함께 활용하면 좋은 On-Demand Revalidation에 대해 더 자세히 알아보는 시간을 가져보겠습니다.

---

## ## 참고 자료

*   [ReactJS(NextJs) Rendering Pattern　~Incremental Static Regeneration (ISR)~](https://dev.to/kkr0423/reactjsnextjs-rendering-pattern-incremental-static-regeneration-isr-261m) by Ogasawara Kakeru
*   [How to Convert SVG to React Components: Complete Guide](https://dev.to/arenasbob2024cell/how-to-convert-svg-to-react-components-complete-guide-34jc) by arenasbob2024-cell
*   [Stop Repeating React Setup: Introducing create-react-forge](https://dev.to/chiragmak10/stop-repeating-react-setup-introducing-create-react-forge-29hd) by Chirag
*   [Patrones de diseño de componentes de React: una guía completa](https://dev.to/carlos_mruiz_e1fbb61ca7/patrones-de-diseno-de-componentes-de-react-una-guia-completa-4ndm) by Carlos m. Ruiz
*   [React Router: Loaders, Actions & Form](https://dev.to/edriso/react-router-loaders-actions-form-2bbe) by Mohamed Idris