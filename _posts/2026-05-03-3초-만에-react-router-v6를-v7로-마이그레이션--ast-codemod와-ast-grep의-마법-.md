---
layout: post
title: "3초 만에 React Router v6를 v7로 마이그레이션? AST Codemod와 ast-grep의 마법!"
date: 2026-05-03T01:00:33.709Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 기술 블로거입니다! 2026년 5월 3일, 오늘도 뜨거운 프론트엔드 세계의 최신 소식을 들고 찾아왔습니다. 빠르게 변화하는 기술 스택 속에서 개발 생산성을 극대화하는 방법은 언제나 우리의 주요 관심사죠.

오늘은 그중에서도 개발자의 고통을 덜어주고, 생산성을 획기적으로 끌어올릴 수 있는 마법 같은 기술, **AST 기반 Codemod**에 대한 이야기를 나누고자 합니다. 특히 최근 React Router v6에서 v7로의 마이그레이션을 단 3초 만에, 그것도 `ast-grep`이라는 Rust 기반 도구를 활용해 **오류 없이(Zero-False-Positive)** 성공시킨 사례는 정말 인상 깊었습니다. 수 시간, 아니 어쩌면 며칠이 걸릴 수 있는 수작업을 자동화하는 이 기술, 함께 깊이 파헤쳐 볼까요?

---



## 소개: 반복적인 마이그레이션, 이제 그만!

프론트엔드 개발자라면 누구나 한 번쯤 라이브러리 버전 업그레이드나 API 변경으로 인한 대규모 코드 수정 작업에 시달려 본 경험이 있을 겁니다. 수백, 수천 개의 파일을 일일이 열어보며 변경 사항을 적용하는 것은 시간 소모적일 뿐만 아니라, 작은 실수 하나가 치명적인 버그로 이어질 수 있는 고된 작업입니다. 이러한 반복적이고 지루한 작업은 개발자의 소중한 시간을 낭비하고, 개발 피로도를 높이는 주범이죠.

하지만 만약 이러한 작업을 단 몇 초 만에, 그것도 사람이 하는 것보다 훨씬 정확하게 자동화할 수 있다면 어떨까요? 바로 **AST(Abstract Syntax Tree) 기반 Codemod**가 그 해답을 제시합니다. 특히 최근 `ast-grep`이라는 강력한 도구를 활용해 React Router v6에서 v7으로의 마이그레이션을 '제로 거짓 양성'으로 성공시킨 사례는 이러한 자동화의 잠재력을 여실히 보여줍니다. 오늘은 이 마법 같은 기술의 핵심 원리와 실제 적용 방법을 함께 탐구해보고자 합니다.

## 본문: 코드를 코드로 다루는 기술, AST Codemod

### 핵심 개념 설명: Codemod와 AST란 무엇인가?

Codemod는 **코드베이스를 자동으로 변환하는 도구 또는 프로세스**를 의미합니다. 단순히 텍스트를 찾아 바꾸는 것(정규표현식)을 넘어, 코드의 구조와 의미를 이해하고 변환하는 것이 핵심이죠. 그리고 Codemod가 코드의 구조와 의미를 이해하게 해주는 것이 바로 **AST(Abstract Syntax Tree, 추상 구문 트리)**입니다.

AST는 말 그대로 **소스 코드를 추상화된 나무(트리) 구조로 표현한 것**입니다. 우리가 작성하는 코드는 컴퓨터가 이해하기 전에 파싱(Parsing) 과정을 거치는데, 이때 코드의 각 요소(변수, 함수, 연산자, 리터럴 등)들이 노드(Node)가 되고, 이 노드들이 계층적인 관계를 맺어 트리를 형성합니다. 마치 문법적인 구조를 시각적으로 표현한 지도와 같다고 할 수 있습니다.

**AST 기반 Codemod의 작동 방식:**

1.  **파싱(Parsing):** 원본 소스 코드를 AST로 변환합니다.
2.  **변환(Transformation):** AST를 순회하며(traverse) 특정 패턴을 찾아내고, 정의된 규칙에 따라 AST 노드를 수정합니다. 예를 들어, 특정 함수 호출 방식을 다른 방식으로 변경하거나, 변수 이름을 일괄적으로 바꿀 수 있습니다.
3.  **역변환(Pretty-Printing/Code Generation):** 수정된 AST를 다시 사람이 읽을 수 있는 소스 코드로 변환합니다.

정규표현식은 텍스트 패턴 매칭에만 능한 반면, AST는 코드의 문법적, 구조적 의미를 정확히 파악하기 때문에 **훨씬 더 정교하고 안전한 코드 변환**이 가능합니다. 예를 들어, `const foo = bar;`라는 코드에서 `bar`라는 변수 이름만 바꾸고 싶을 때, 정규표현식은 주석이나 문자열 리터럴 안의 `bar`까지 잘못 바꿀 위험이 있지만, AST는 '변수 선언문의 우변에 있는 식별자 `bar`'를 정확히 찾아낼 수 있습니다.

### `ast-grep`: Rust 기반의 강력한 AST 변환 도구

기존에는 JavaScript 생태계에서 `jscodeshift`와 같은 도구가 주로 사용되었지만, 최근에는 Rust 기반의 `ast-grep`이 주목받고 있습니다. `ast-grep`은 다음과 같은 특징으로 강력한 Codemod 도구로 자리매김하고 있습니다:

*   **성능:** Rust로 작성되어 매우 빠르고 효율적입니다. 대규모 코드베이스에서도 신속하게 작업을 처리할 수 있습니다.
*   **간편한 패턴 매칭:** YAML 기반의 패턴 정의를 통해 복잡한 AST 구조를 명확하고 간결하게 표현할 수 있습니다. 이는 개발자가 Codemod 규칙을 쉽게 작성하고 이해하는 데 큰 도움이 됩니다.
*   **정확성:** AST 기반이므로 코드의 의미를 정확히 파악하여 오탐(false positive) 없이 원하는 코드만 변환할 수 있습니다.

`ast-grep`은 특정 코드 패턴을 검색하고, 이를 다른 패턴으로 교체하는 방식으로 동작합니다. 마치 `grep`이 텍스트를 검색하듯이, `ast-grep`은 AST 패턴을 검색하는 것이죠.

### 실제 코드 예시: React Router v6에서 v7 마이그레이션 (TypeScript/React 활용)

React Router v7에서는 `useNavigate` 훅의 일부 동작 방식이 변경되었습니다. 특히, 숫자 인자를 받아 상대 경로로 이동하던 기능이 제거되었습니다. 예를 들어 `navigate(-1)`처럼 이전 페이지로 돌아가는 기능은 더 이상 지원되지 않고, `history.back()`이나 `navigate('..', { relative: 'path' })`와 같은 방식으로 변경해야 합니다.

이러한 변경 사항을 수동으로 적용한다면, 프로젝트 내의 모든 `navigate(-1)` 호출을 찾아 수정해야 할 것입니다. `ast-grep`을 사용하면 이 과정을 자동화할 수 있습니다.

**문제 상황 (React Router v6):**

```typescript
// src/components/MyComponent.tsx
import React from 'react';
import { useNavigate } from 'react-router-dom';

function MyComponent() {
  const navigate = useNavigate();

  const handleGoBack = () => {
    navigate(-1); // v7에서는 더 이상 지원되지 않습니다.
  };

  const handleGoForward = () => {
    navigate(1); // 이 또한 변경되어야 합니다.
  };

  return (
    <div>
      <button onClick={handleGoBack}>이전 페이지로</button>
      <button onClick={handleGoForward}>다음 페이지로</button>
    </div>
  );
}

export default MyComponent;
```

**목표 (React Router v7):**

`navigate(-1)`은 `history.back()`으로, `navigate(1)`은 `navigate('..', { relative: 'path' })` 또는 더 정교한 방법으로 변경되어야 합니다. 여기서는 `navigate(-1)`을 `history.back()`으로 변경하는 Codemod를 예시로 들어보겠습니다.

```typescript
// src/components/MyComponent.tsx (Codemod 적용 후)
import React from 'react';
import { useNavigate } from 'react-router-dom';

function MyComponent() {
  const navigate = useNavigate();

  const handleGoBack = () => {
    history.back(); // Codemod로 변경됨
  };

  const handleGoForward = () => {
    // navigate(1)은 상황에 따라 더 복잡한 로직이 필요할 수 있습니다.
    // 여기서는 간단히 주석 처리하거나, 특정 경로로 대체하는 방식으로 가정합니다.
    // navigate('/some-path');
  };

  return (
    <div>
      <button onClick={handleGoBack}>이전 페이지로</button>
      <button onClick={handleGoForward}>다음 페이지로</button>
    </div>
  );
}

export default MyComponent;
```

**`ast-grep` 규칙 작성 (YAML):**

`ast-grep`은 YAML 파일로 규칙을 정의합니다. `navigate(-1)`을 `history.back()`으로 변경하는 간단한 규칙은 다음과 같이 작성할 수 있습니다.

```yaml
# rules/react-router-v7-migrate.yml
id: react-router-navigate-back-migration
message: "React Router v7: Change navigate(-1) to history.back()"
language: typescript # TypeScript 파일에 적용
rule:
  # 패턴 매칭: $NAVIGATE라는 변수가 -1을 인자로 받는 함수 호출을 찾습니다.
  # 여기서 $NAVIGATE는 useNavigate 훅으로 생성된 함수여야 합니다.
  # 실제 ast-grep에서는 스코프 분석을 통해 더 정확하게 특정할 수 있지만,
  # 예시를 위해 간단한 패턴을 사용합니다.
  pattern: $NAVIGATE(-1)
  # constraints를 사용하여 $NAVIGATE가 특정 타입이나 이름 패턴을 가지도록 제한할 수 있습니다.
  # 예를 들어, $NAVIGATE가 useNavigate 훅에서 온 것인지 확인하는 로직 추가 가능
fix: history.back()
```

**`ast-grep` 실행:**

이 규칙 파일을 `rules/react-router-v7-migrate.yml`로 저장하고, 프로젝트 루트에서 다음 명령어를 실행하면 됩니다.

```bash
# 먼저 ast-grep을 설치합니다.
# cargo install ast-grep # Rust가 설치되어 있다면
# npm install -g @ast-grep/cli # Node.js 환경에서
# ag -h # 설치 확인

# 규칙을 실행하고 변경 사항을 미리 봅니다. (dry run)
ag -r rules/react-router-v7-migrate.yml --dry-run src/

# 실제로 변경 사항을 적용합니다.
ag -r rules/react-router-v7-migrate.yml src/
```

이처럼 단 몇 줄의 YAML 설정과 명령어 실행으로 수많은 파일을 일괄적으로, 그리고 정확하게 수정할 수 있습니다. React Router v7 마이그레이션의 경우, `useRoutes`와 `createBrowserRouter`로의 변경 등 더 복잡한 패턴들도 `ast-grep`의 강력한 패턴 매칭 기능을 활용하여 자동화할 수 있습니다.

### 실무 적용 팁

1.  **점진적 적용:** Codemod를 전체 코드베이스에 한 번에 적용하기보다는, 작은 단위(예: 특정 디렉토리, 특정 컴포넌트)부터 시작하여 예상대로 동작하는지 확인하는 것이 좋습니다.
2.  **버전 관리:** Codemod를 실행하기 전에는 반드시 현재 코드베이스를 커밋하여 안전한 롤백 지점을 확보하세요. 만약 Codemod가 잘못 적용되더라도 쉽게 되돌릴 수 있습니다.
3.  **철저한 검토:** '제로 거짓 양성'을 목표로 하지만, 항상 최종 결과물은 사람이 검토하는 것이 중요합니다. 특히 복잡한 로직이 얽힌 부분에서는 수동 검증이 필수적입니다.
4.  **커스텀 Codemod:** 라이브러리 마이그레이션뿐만 아니라, 팀 내의 코딩 컨벤션을 강제하거나, 사내에서 자주 사용되는 패턴을 리팩토링하는 등 프로젝트 특화된 Codemod를 직접 만들어 활용할 수 있습니다.
5.  **CI/CD 통합:** 반복적인 Codemod 작업이 있다면, CI/CD 파이프라인에 통합하여 자동화된 코드 검사 및 수정 프로세스를 구축하는 것을 고려해볼 수 있습니다.
6.  **학습과 이해:** Codemod를 효과적으로 사용하려면 AST의 기본 개념과 사용하는 도구(`ast-grep` 등)의 패턴 문법을 이해하는 것이 중요합니다. 처음에는 어렵게 느껴질 수 있지만, 한 번 익혀두면 개발 생산성에 엄청난 이점을 가져다줍니다.

## 마무리: 자동화된 미래, 개발자의 손안에

오늘 우리는 React Router v6에서 v7로의 마이그레이션 사례를 통해 AST 기반 Codemod와 `ast-grep`의 강력한 힘을 엿보았습니다. 수 시간이 걸릴 수 있는 지루하고 오류 투성이인 수작업을 단 3초 만에, 그것도 완벽하게 처리하는 이 기술은 프론트엔드 개발의 미래를 더욱 효율적이고 즐겁게 만들 잠재력을 가지고 있습니다.

더 이상 반복적인 코드 수정에 에너지를 낭비하지 마세요. 코드를 코드로 다루는 기술, AST와 Codemod를 익혀 여러분의 개발 생산성을 한 단계 끌어올리세요. `ast-grep`과 같은 도구를 직접 사용해보고, 간단한 규칙부터 만들어 적용해보는 것을 추천합니다. 이 기술은 라이브러리 마이그레이션을 넘어, 코드 품질 향상, 리팩토링, 코드 컨벤션 강제 등 다양한 영역에서 여러분의 든든한 조력자가 될 것입니다.

## 참고 자료

- [I Built a Zero-False-Positive Codemod That Migrates React Router v6 v7 in 3 Seconds](https://dev.to/ankit_raj_16a4c518f4c1689/automating-react-router-v6-to-v7-migration-with-ast-codemods-2888) by Ankit raj
- [I Built a Full-Stack Invoice App from Scratch. Here's the Complete Breakdown](https://dev.to/carter254g/i-built-a-full-stack-invoice-app-from-scratch-heres-the-complete-breakdown-2mmb) by Carter
- [React Native Performance Optimization After Migrating to the New Architecture](https://dev.to/jacobnoah9876/react-native-performance-optimization-after-migrating-to-the-new-architecture-l38) by Jacob Noah
- [React Rn-Rendering Wrokflow](https://dev.to/ankitsinghmyself/react-rn-rendering-wrokflow-1e58) by Ankit Singh
- [React Performance for Senior Developers (Practical, No Cargo Culting)](https://dev.to/grantwatsondev/react-performance-for-senior-developers-practical-no-cargo-culting-1l02) by Grant Watson