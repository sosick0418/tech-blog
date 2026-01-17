---
layout: post
title: "리액트 핵심: 상속 대신 '합성(Composition)'을 선택한 이유"
date: 2026-01-17T12:17:25.875Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 기술 블로거입니다!
2026년 1월 17일, 오늘도 흥미로운 프론트엔드 트렌드를 함께 살펴보겠습니다.

---



객체지향 프로그래밍(OOP)의 '상속'에 익숙한 개발자라면 리액트가 왜 '합성'을 강조하는지 궁금할 수 있습니다. 리액트는 'Has-A' 관계를 통해 더 유연하고 재사용성 높은 컴포넌트를 구축합니다. 이 원칙을 이해하는 것은 현대 리액트 개발의 필수입니다.

### 상속 vs. 합성: 리액트의 선택

전통적인 **상속(Inheritance)**은 'Is-A' 관계(예: `Dog extends Animal`)로 부모의 기능을 물려받습니다. 이는 코드 중복을 줄이는 데 효과적이지만, 단일 상속 제약, 부모-자식 간의 높은 결합도, 예상치 못한 부작용 등의 한계가 있습니다.

반면, 리액트의 **합성(Composition)**은 'Has-A' 관계에 기반합니다. 이는 컴포넌트가 다른 컴포넌트를 *포함(has)*하여 새로운 기능을 만드는 방식입니다. 예를 들어, `Card` 컴포넌트가 `Header`, `Body`, `Footer` 컴포넌트들을 *가지고* 있는 형태입니다. 리액트는 합성을 통해 다음과 같은 이점을 얻습니다.

*   **유연성 및 재사용성:** 작은 컴포넌트들을 조합하여 다양한 기능을 만들 수 있습니다.
*   **낮은 결합도:** 컴포넌트들이 독립적으로 작동하며, 변경의 영향이 적습니다.
*   **예측 가능한 데이터 흐름:** 단방향 데이터 흐름과 잘 어울려 이해하기 쉽습니다.
*   **테스트 용이성:** 각 컴포넌트가 독립적이므로 단위 테스트가 쉽습니다.

### 코드 예시: 합성(Composition)으로 유연한 UI 구축 (React/TypeScript)

리액트에서 합성은 주로 `children` prop을 통한 콘텐츠 주입과 컴포넌트를 prop으로 전달하는 방식으로 구현됩니다. 다음은 아이콘을 포함하는 `Button` 컴포넌트 예시입니다.

먼저, 간단한 `Icon` 컴포넌트와 `Button` 컴포넌트를 정의합니다. `Button`은 `children`으로 텍스트를 받고, `icon` prop으로 아이콘 컴포넌트를 받을 수 있도록 합니다.

```tsx
// components/Icon.tsx
import React from 'react';

interface IconProps {
  name: string;
}

export const Icon: React.FC<IconProps> = ({ name }) => {
  // 실제로는 SVG나 폰트 아이콘 라이브러리를 사용합니다.
  return <span style={{ marginRight: '8px' }}>{name}</span>;
};
```

```tsx
// components/Button.tsx
import React from 'react';

interface ButtonProps {
  children: React.ReactNode;
  icon?: React.ReactNode; // 아이콘 컴포넌트를 prop으로 받음
  onClick: () => void;
}

export const Button: React.FC<ButtonProps> = ({ children, icon, onClick }) => {
  return (
    <button
      onClick={onClick}
      style={{ padding: '10px 15px', backgroundColor: '#007bff', color: 'white', border: 'none', borderRadius: '5px', display: 'flex', alignItems: 'center', cursor: 'pointer' }}
    >
      {icon && <span style={{ marginRight: '8px' }}>{icon}</span>}
      {children}
    </button>
  );
};
```

이제 Next.js 페이지에서 이 컴포넌트들을 합성하여 사용해봅시다.

```tsx
// app/page.tsx (Next.js App Router)
import { Button } from '../components/Button';
import { Icon } from '../components/Icon';

export default function HomePage() {
  const handleClick = () => alert('버튼이 클릭되었습니다!');

  return (
    <div style={{ padding: '20px', display: 'flex', gap: '15px' }}>
      <Button onClick={handleClick}>기본 버튼</Button>
      <Button onClick={handleClick} icon={<Icon name="🚀" />}>로켓 발사</Button>
      <Button onClick={handleClick} icon={<Icon name="✅" />}>작업 완료</Button>
    </div>
  );
}
```
위 예시에서 `Button` 컴포넌트는 `children`으로 텍스트를, `icon` prop으로는 `Icon` 컴포넌트 또는 다른 형태의 React 노드를 받습니다. 이처럼 작은 단위의 컴포넌트들을 조합하여 새로운 기능을 가진 컴포넌트를 만드는 것이 리액트 합성의 핵심입니다.

### 실무 적용 팁

1.  **책임 분리 원칙(SRP) 준수:** 각 컴포넌트가 하나의 명확한 책임만 갖도록 설계하여 재사용성을 높입니다.
2.  **`children` Prop 적극 활용:** 컴포넌트의 내부 콘텐츠를 유연하게 주입해야 할 때는 `children` prop을 사용하세요.
3.  **컴포넌트를 Prop으로 전달:** 특정 UI 영역을 완전히 교체해야 할 경우, 컴포넌트 자체를 prop으로 전달하는 방식을 고려하세요.
4.  **커스텀 훅(Custom Hooks)으로 로직 재사용:** UI 재사용은 합성으로, 상태 관리나 비즈니스 로직의 재사용은 커스텀 훅으로 분리하여 코드를 깔끔하게 유지합니다.

### 마무리: 합성은 리액트의 철학이자 강력한 무기

오늘 우리는 리액트가 왜 상속 대신 합성을 선택했는지, 그리고 이 철학이 어떻게 더 유연하고 견고하며 유지보수하기 쉬운 프론트엔드 애플리케이션을 만들 수 있게 하는지 살펴보았습니다. 'Has-A' 관계를 기반으로 컴포넌트를 설계하고 `children` prop, 컴포넌트 prop 전달, 커스텀 훅 등을 적절히 활용하는 것은 리액트 개발자로서 갖춰야 할 핵심 역량입니다.

합성 패턴에 익숙해질수록 여러분의 리액트 코드는 더욱 깔끔하고 확장 가능해질 것입니다. 더 나아가 React Context API, Render Props, HOC(High-Order Components)와 같은 고급 합성 패턴에 대해서도 학습해보시면 좋습니다. 이 모든 것은 결국 리액트가 지향하는 선언적이고 컴포넌트 기반의 개발 패러다임을 더욱 깊이 이해하는 데 도움이 될 것입니다.

---

## 참고 자료
- [Code-ing in a Tree: Adding Auto-Save to freeCodeCamp](https://dev.to/sagi0312/code-ing-in-a-tree-adding-auto-save-to-freecodecamp-137b) by Anju Karanji
- [Building a City Boy Meme Generator with Next.js and Fabric.js (No Watermarks!)](https://dev.to/_75729aa6b13e505e279f7/building-a-city-boy-meme-generator-with-nextjs-and-fabricjs-no-watermarks-3i85) by 姚路行
- [MdBin Levels Up: From Custom Markdown Pipeline to Streamdown](https://dev.to/sivarampg/mdbin-levels-up-from-custom-markdown-pipeline-to-streamdown-55g2) by Sivaram
- [Composition vs Inheritance: Why React Chose the "Has-A" Over "Is-A" Relationship](https://dev.to/tishonator/composition-vs-inheritance-why-react-chose-the-has-a-over-is-a-relationship-5108) by Tihomir Ivanov
- [async/await is NOT just syntax sugar for Promises](https://dev.to/bishoy_semsem/asyncawait-is-not-just-syntax-sugar-for-promises-312e) by Bishoy Semsem