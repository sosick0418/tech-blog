---
layout: post
title: "Next.js의 숨겨진 보물, Rewrites를 완벽 해부하기"
date: 2026-02-27T01:00:47.909Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거입니다. 2026년 2월 27일, Next.js를 단순히 기본 라우팅이나 서버 사이드 렌더링(SSR) 정도로만 활용하고 계신가요? 그렇다면 애플리케이션의 유연성을 극대화할 수 있는 강력한 기능, `Rewrites`를 놓치고 있을 수 있습니다. `Rewrites`는 URL 구조를 유연하게 관리하고, 외부 서비스와 매끄럽게 연동하며, 심지어 SEO 최적화에도 간접적으로 기여할 수 있는 Next.js의 숨겨진 보물 같은 기능입니다. 오늘은 `Rewrites`가 무엇이며, 어떻게 우리의 Next.js 프로젝트를 한 단계 더 발전시킬 수 있는지 깊이 파헤쳐 보겠습니다.

---



## Rewrites란 무엇인가?

Next.js의 `Rewrites`는 들어오는 요청의 URL을 **서버 내부적으로** 변경하여 다른 경로로 라우팅하는 기능입니다. 여기서 중요한 점은 이 모든 과정이 서버에서 이루어지기 때문에, 클라이언트(브라우저)는 URL이 변경되었다는 사실을 인지하지 못하며, 주소창의 URL은 원래 요청했던 경로를 그대로 유지합니다.

이는 `Redirects`와 명확히 구분됩니다.
*   **Redirects**: 클라이언트에 HTTP 3xx 상태 코드(예: 301 Moved Permanently)를 반환하여, 브라우저가 새로운 URL로 다시 요청하도록 지시합니다. 이 경우 브라우저의 주소창 URL이 변경됩니다. 주로 URL이 영구적으로 변경되었거나 다른 페이지로 이동해야 할 때 사용됩니다.
*   **Rewrites**: 서버 내부적으로 요청 경로를 다시 작성합니다. 클라이언트에는 200 OK 상태 코드를 반환하며, 브라우저 주소창 URL은 변경되지 않습니다. 주로 내부 API 프록싱, URL 구조 숨기기, A/B 테스트 등에 활용됩니다.

간단히 말해, `Redirects`는 "이리 가세요!"라고 클라이언트에게 알려주는 것이고, `Rewrites`는 "제가 알아서 처리해서 보여드릴게요!"라고 서버가 내부적으로 처리하는 것입니다.

`Rewrites`는 주로 다음과 같은 상황에서 빛을 발합니다:
1.  **외부 API 프록시**: 백엔드 API 엔드포인트를 직접 노출하지 않고 Next.js 서버를 통해 프록시하여 보안을 강화하고 CORS 문제를 회피합니다.
2.  **URL 구조 유연성 확보**: 사용자에게는 짧고 의미 있는 URL을 제공하면서, 실제 내부 경로는 다른 구조를 사용할 수 있습니다.
3.  **A/B 테스트**: 특정 사용자 그룹에게는 URL 변경 없이 특정 페이지의 다른 버전을 보여줄 수 있습니다.

## 실제 코드 예시 (Next.js & TypeScript)

`Rewrites`는 `next.config.js` 파일의 `rewrites` 함수에서 설정합니다. 이 함수는 `source`와 `destination` 속성을 가진 객체 배열을 반환합니다. `async` 함수로 정의하여 비동기 로직을 포함할 수도 있습니다.

```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  async rewrites() {
    return [
      // 1. 사용자 친화적인 URL 제공 예시: `/old-page`로 접근하면 `/new-content` 컴포넌트 렌더링
      {
        source: '/old-page',
        destination: '/new-content',
      },
      // 2. 동적 경로(Dynamic Routes) Rewrite 예시: `/blog/post-title` -> `/articles/post-title`
      {
        source: '/blog/:slug',
        destination: '/articles/:slug', // 실제 컴포넌트 경로 (pages/articles/[slug].tsx)
      },
      // 3. 외부 API 프록시 예시: 클라이언트의 `/api/users` 요청을 `https://api.external.com/v1/users`로 전달
      {
        source: '/api/users/:path*', // `:path*`는 모든 하위 경로를 포함
        destination: `https://api.external.com/v1/users/:path*`,
      },
      // 4. 환경별 조건부 Rewrite 예시: 개발 환경에서만 Mock API 사용
      {
        source: '/api/data',
        destination: process.env.NODE_ENV === 'development'
          ? 'http://localhost:3000/api/mock-data' // 개발 시 Next.js 앱 내부의 Mock API 사용
          : 'https://api.production.com/real-data', // 프로덕션 시 실제 외부 API 사용
      },
    ];
  },
};

module.exports = nextConfig;
```
위 예시에서 `source`는 사용자가 브라우저에서 요청하는 경로 패턴이고, `destination`은 Next.js 서버가 실제 콘텐츠를 가져올 내부 또는 외부 경로입니다. 특히 외부 API를 프록시하는 경우, 클라이언트 요청은 `'/api/users/...'`로 보이지만, 서버는 내부적으로 `https://api.external.com/v1/users/...`로 요청을 처리하여 CORS 문제를 해결하고 백엔드 엔드포인트를 숨길 수 있어 매우 효과적입니다.

## 실무 적용 팁

`Rewrites`는 강력하지만, 올바르게 사용하지 않으면 오히려 복잡성을 증가시키거나 예상치 못한 문제를 일으킬 수 있습니다. 몇 가지 실무 팁을 공유합니다.

*   **보안에 유의하세요**: 외부 API를 프록시할 때, 민감한 API 키나 인증 정보를 클라이언트에 노출하지 않도록 주의해야 합니다. `next.config.js` 내부에서 `process.env`와 같은 서버 환경 변수를 활용하여 안전하게 처리해야 합니다.
*   **디버깅**: `Rewrites` 규칙이 예상대로 작동하지 않을 경우, 브라우저의 개발자 도구 네트워크 탭을 확인하거나, Next.js 서버 로그를 자세히 살펴보세요. `next.config.js` 내부에 `console.log`를 활용하여 매칭되는 `source`와 `destination`을 추적하는 것도 좋은 방법입니다.
*   **SEO 고려**: `Rewrites`는 클라이언트에게 URL 변경을 알리지 않으므로, 브라우저 주소창의 URL은 변경되지 않습니다. 이는 SEO 관점에서 일관된 URL을 유지하는 데 도움이 될 수 있습니다. 그러나 만약 특정 콘텐츠의 "공식" URL을 변경해야 한다면, `Rewrites`보다는 `Redirects`를 사용하는 것이 더 적절합니다. 일관된 URL 전략을 유지하는 것이 중요합니다.

## 마무리하며

Next.js의 `Rewrites` 기능은 단순한 라우팅을 넘어, 애플리케이션의 유연성과 확장성을 극대화하는 강력한 도구입니다. 복잡한 URL 구조를 단순화하고, 외부 서비스와의 통합을 매끄럽게 하며, 환경별 설정을 유연하게 관리하는 데 필수적인 역할을 합니다.

오늘 다룬 내용을 통해 여러분의 Next.js 프로젝트가 한 단계 더 발전할 수 있기를 바랍니다. `Rewrites` 외에도 `Redirects`, `Headers` 등 `next.config.js`에서 제공하는 다양한 라우팅 관련 기능을 함께 학습하여 Next.js의 잠재력을 최대한 활용해 보시길 권장합니다.

---

## 참고 자료
- [Understanding Next.js Rewrites](https://dev.to/cole_ruche/understanding-next-js-rewrites-234j) by Emeruche Ikenna
- [amazing Offline-first opensource for web apps
https://github.com/minseong0324/connectivity-js](https://dev.to/_e35bd2c8a72ed0504f893/amazing-offline-first-opensource-for-web-appshttpsgithubcomminseong0324connectivity-js-1d14) by _e35bd2c8a72ed0504f893
- [How to Use React Query with React Router Loaders (Pre-fetch & Cache Data)](https://dev.to/edriso/how-to-use-react-query-with-react-router-loaders-pre-fetch-cache-data-kag) by Mohamed Idris
- [Creé un plugin que simplifica la integración de Google Maps en Vite y React](https://dev.to/franqsanz/cree-un-plugin-que-simplifica-la-integracion-de-google-maps-en-vite-y-react-11k9) by Franco Andrés
- [The Smallest Schema Library on the Market (and the Form Hook Built on Top of It)](https://dev.to/sakobume/the-smallest-schema-library-on-the-market-and-the-form-hook-built-on-top-of-it-2j71) by Sarkis M