---
layout: post
title: "React Router v6 -> v7 마이그레이션, 3초 만에 끝내는 마법? AST Codemod와 ast-grep의 힘!"
date: 2026-05-02T01:00:33.227Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거 개발자 K입니다. 2026년 5월 2일, 오늘도 빠르게 변화하는 프론트엔드 세계의 흥미로운 소식을 들고 찾아왔습니다.

---



## 대규모 프로젝트 마이그레이션, 아직도 수동으로 하시나요?

React 생태계는 항상 발전하며 새로운 버전과 기능이 끊임없이 출시됩니다. 최근 React Router v7이 발표되면서, 더 간결해진 API와 개선된 성능으로 많은 개발자의 기대를 모으고 있습니다. 하지만 대규모 프로덕션 애플리케이션에서 새로운 버전으로 마이그레이션하는 작업은 결코 만만치 않습니다. 수백, 수천 개의 파일에 걸쳐 변경 사항을 수동으로 적용하는 것은 엄청난 시간 소모와 함께 휴먼 에러의 위험을 높이죠.

오늘 우리는 이 지루하고 고통스러운 과정을 획기적으로 단축시켜 줄 마법 같은 기술, 바로 **AST 기반 Codemod**와 Rust로 개발된 강력한 도구 **`ast-grep`**에 대해 깊이 파고들어 볼 예정입니다. 이 기술을 활용하면 며칠이 걸릴 작업을 단 몇 초 만에, 그것도 "Zero-False-Positive"로 안전하게 완료할 수 있습니다.

## 코드의 구조를 이해하고 변환하는 힘: AST Codemod와 ast-grep

### 핵심 개념 설명: AST와 Codemod, 그리고 ast-grep

우리가 작성하는 코드는 컴퓨터가 이해할 수 있는 형태로 변환되기 위해 여러 단계를 거칩니다. 이 과정에서 중요한 역할을 하는 것이 바로 **AST(Abstract Syntax Tree, 추상 구문 트리)**입니다. AST는 소스 코드를 프로그램의 문법적 구조에 따라 트리 형태로 표현한 데이터 구조입니다. 컴파일러, 린터, 코드 포매터(Prettier 등) 등 대부분의 코드 분석 및 변환 도구는 이 AST를 기반으로 작동합니다.

**Codemod**는 이러한 AST를 조작하여 코드 변환을 자동화하는 도구를 총칭합니다. 단순한 텍스트 찾기-바꾸기가 아닌, 코드의 의미론적 구조를 이해하고 특정 패턴을 찾아 변환하기 때문에 훨씬 더 정교하고 안전한 리팩토링이 가능합니다. `jscodeshift`와 같은 도구가 대표적인 Codemod 프레임워크입니다.

그리고 오늘 주목할 도구는 바로 **`ast-grep`**입니다. Rust로 개발된 `ast-grep`는 YAML 기반의 간결한 규칙 정의를 통해 AST 패턴 매칭 및 변환을 수행하는 CLI 도구입니다. `jscodeshift`보다 러닝 커브가 낮고, 뛰어난 성능을 자랑하며, Rust의 안정성과 신뢰성을 바탕으로 대규모 코드베이스에 안전하게 적용할 수 있습니다. `ast-grep`는 코드의 특정 패턴을 찾고, 그 패턴에 맞는 코드를 새로운 형태로 변환하는 강력한 기능을 제공하여 복잡한 마이그레이션 작업을 자동화하는 데 최적화되어 있습니다.

### 실제 코드 예시: NavLink의 `activeClassName` 마이그레이션

React Router v6에서 v7로 마이그레이션할 때, `NavLink` 컴포넌트의 `activeClassName` prop이 `className` prop으로 통합되면서 `isActive` 상태를 활용하는 방식으로 변경되었습니다. 이처럼 사소하지만 수많은 파일에 걸쳐 수정해야 하는 변경 사항은 Codemod의 힘을 빌리기에 아주 좋은 예시입니다.

**기존 React Router v6 코드 (TypeScript/JSX)**:

```tsx
// src/components/MyNav.tsx
import { NavLink } from 'react-router-dom';

function MyNav() {
  return (
    <nav>
      <NavLink to="/home" activeClassName="active-link">
        홈
      </NavLink>
      <NavLink to="/about" activeClassName="active-link">
        소개
      </NavLink>
      <NavLink to="/contact" activeClassName="active-link">
        연락처
      </NavLink>
    </nav>
  );
}

export default MyNav;
```

**React Router v7으로 변환될 코드**:

```tsx
// src/components/MyNav.tsx
import { NavLink } from 'react-router-dom';

function MyNav() {
  return (
    <nav>
      <NavLink
        to="/home"
        className={({ isActive }) => (isActive ? 'active-link' : '')}
      >
        홈
      </NavLink>
      <NavLink
        to="/about"
        className={({ isActive }) => (isActive ? 'active-link' : '')}
      >
        소개
      </NavLink>
      <NavLink
        to="/contact"
        className={({ isActive }) => (isActive ? 'active-link' : '')}
      >
        연락처
      </NavLink>
    </nav>
  );
}

export default MyNav;
```

이러한 변경을 `ast-grep`로 자동화하는 YAML 규칙은 다음과 같이 작성할 수 있습니다.

**`ast-grep` 규칙 (rule.yaml)**:

```yaml
# rules/navlink-activeclassname.yaml
id: navlink-activeclassname-to-classname
message: NavLink의 activeClassName을 className prop으로 마이그레이션합니다.
severity: info
language: tsx # TypeScript JSX 파일에 적용
rule:
  pattern: |
    <NavLink $$$PROPS activeClassName="$ACTIVE_CLASS" $$$REST_PROPS>
  fix: |
    <NavLink
      $$$PROPS
      className={({ isActive }) => (isActive ? '$ACTIVE_CLASS' : '')}
      $$$REST_PROPS
    >
```

**규칙 설명**:
*   `id`: 규칙의 고유 식별자입니다.
*   `message`: 이 규칙이 수행하는 작업에 대한 설명입니다.
*   `severity`: 규칙의 심각도를 나타냅니다.
*   `language`: 이 규칙이 적용될 파일의 언어를 지정합니다 (여기서는 `tsx`).
*   `rule.pattern`: 찾을 코드 패턴을 정의합니다.
    *   `<NavLink ...>`: `NavLink` 컴포넌트를 찾습니다.
    *   `$$$PROPS`, `$$$REST_PROPS`: `ast-grep`의 메타변수로, 임의의 JSX prop들을 나타냅니다. `activeClassName` 앞뒤의 다른 prop들을 모두 캡처합니다.
    *   `activeClassName="$ACTIVE_CLASS"`: `activeClassName` prop과 그 값을 `$ACTIVE_CLASS`라는 메타변수로 캡처합니다.
*   `fix`: 패턴이 발견되었을 때 적용할 변환 규칙을 정의합니다.
    *   캡처된 `$$$PROPS`와 `$$$REST_PROPS`를 그대로 유지합니다.
    *   `activeClassName` 대신 `className` prop을 추가하고, `$ACTIVE_CLASS`로 캡처된 값을 `({ isActive }) => (isActive ? '$ACTIVE_CLASS' : '')` 형식으로 사용하도록 변환합니다.

이 `rule.yaml` 파일을 프로젝트 루트에 저장하고 다음 명령어를 실행하면, `src` 디렉토리 내의 모든 `tsx` 파일에 해당 규칙이 적용되어 자동으로 코드가 변환됩니다.

```bash
# ast-grep 설치 (Rust가 설치되어 있어야 함)
# cargo install ast-grep

# 현재 디렉토리에서 규칙을 실행하고 변경사항 미리보기
sg scan -r rules/navlink-activeclassname.yaml --fix

# 실제 파일에 변경사항 적용
sg scan -r rules/navlink-activeclassname.yaml --fix --force
```

이처럼 `ast-grep`는 복잡한 코드 변환도 간결한 YAML 규칙으로 정의하여 빠르게 적용할 수 있게 해줍니다.

### 실무 적용 팁

1.  **점진적 적용 및 테스트 우선**: Codemod를 적용하기 전에는 반드시 해당 변경 사항에 대한 테스트(유닛 테스트, 통합 테스트)를 충분히 확보해야 합니다. `ast-grep`는 "zero false positives"를 지향하지만, 복잡한 프로젝트에서는 예상치 못한 부작용이 발생할 수 있습니다. 작은 단위부터 적용하고 철저히 테스트하세요.
2.  **버전 관리 시스템 활용**: Git과 같은 버전 관리 시스템을 적극 활용하여 Codemod 적용 전후의 변경 사항을 추적하고, 문제가 발생했을 경우 언제든 쉽게 롤백할 수 있도록 준비해야 합니다.
3.  **커스터마이징 능력**: `ast-grep`는 매우 유연하여 프로젝트의 특정 요구 사항에 맞는 커스텀 규칙을 쉽게 작성할 수 있습니다. 공식 문서를 참고하여 AST 패턴 매칭 및 변환에 익숙해지는 것이 중요합니다.
4.  **커뮤니티와 자료 활용**: 이미 공개된 `ast-grep` 규칙이나 다른 Codemod 프로젝트의 사례들을 참고하면 효율적인 규칙 작성에 도움이 됩니다. 또한, 원문 블로그와 같이 실제 마이그레이션 경험을 공유하는 글들을 통해 실질적인 팁을 얻을 수 있습니다.

## 마무리하며: 미래의 리팩토링, AST Codemod와 함께

React Router v6에서 v7로의 마이그레이션 예시를 통해 AST 기반 Codemod와 `ast-grep`의 강력함을 경험했습니다. 이 기술은 단순히 React Router 마이그레이션에만 국한되지 않습니다. 프레임워크 버전 업그레이드, 레거시 코드 리팩토링, 코드 컨벤션 강제 등 대규모 코드베이스를 다루는 모든 상황에서 개발자의 생산성을 극대화하고 휴먼 에러를 최소화하는 핵심 도구가 될 것입니다.

더 이상 반복적이고 지루한 수동 작업에 시간을 낭비하지 마세요. `ast-grep`와 같은 Codemod 도구를 활용하여 코드를 더 스마트하고 효율적으로 관리하는 방법을 익히는 것은 미래의 프론트엔드 개발자에게 필수적인 역량이 될 것입니다. 지금 바로 `ast-grep` 공식 문서를 탐색하고, 여러분의 프로젝트에 적용할 만한 Codemod 규칙을 직접 작성해보면서 이 강력한 도구의 매력에 빠져보시길 추천합니다!

---

## 참고 자료

*   [I Built a Zero-False-Positive Codemod That Migrates React Router v6 v7 in 3 Seconds](https://dev.to/ankit_raj_16a4c518f4c1689/automating-react-router-v6-to-v7-migration-with-ast-codemods-2888) by Ankit raj
*   [I Built a Full-Stack Invoice App from Scratch. Here's the Complete Breakdown](https://dev.to/carter254g/i-built-a-full-stack-invoice-app-from-scratch-heres-the-complete-breakdown-2mmb) by Carter
*   [React Native Performance Optimization After Migrating to the New Architecture](https://dev.to/jacobnoah9876/react-native-performance-optimization-after-migrating-to-the-new-architecture-l38) by Jacob Noah
*   [React Rn-Rendering Wrokflow](https://dev.to/ankitsinghmyself/react-rn-rendering-wrokflow-1e58) by Ankit Singh
*   [React Performance for Senior Developers (Practical, No Cargo Culting)](https://dev.to/grantwatsondev/react-performance-for-senior-developers-practical-no-cargo-culting-1l02) by Grant Watson