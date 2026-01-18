---
layout: post
title: "🚨 Math.random()은 이제 그만! React와 Web Crypto API로 만드는 '진짜' 공정한 추첨 시스템"
date: 2026-01-18T09:30:23.436Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발의 최전선에 계신 여러분! 2026년 1월 18일, 오늘도 흥미로운 프론트엔드 기술 트렌드를 가지고 찾아온 프론트엔드 전문 기술 블로거입니다.

오늘은 우리에게 너무나 익숙하지만 그 위험성을 간과하기 쉬운 한 가지 주제, 바로 '난수 생성'에 대해 이야기해보려 합니다. 특히 추첨이나 보안이 중요한 서비스에서 `Math.random()` 사용이 왜 위험하며, 그 대안으로 `Web Crypto API`를 어떻게 활용할 수 있는지 React와 TypeScript를 통해 자세히 알아보겠습니다.

---



우리는 웹 애플리케이션을 개발하면서 무작위성이 필요한 순간을 자주 마주합니다. 사용자에게 무작위 추천을 제공하거나, 게임에서 주사위를 굴리거나, 혹은 중요한 경품 추첨을 진행할 때 말이죠. 이때 많은 개발자들이 손쉽게 `Math.random()`을 사용하곤 합니다.

하지만 과연 `Math.random()`으로 생성된 숫자가 '진정으로' 무작위적일까요? 특히 사용자들의 신뢰와 직결되는 추첨 시스템이나 보안이 중요한 기능에서는 `Math.random()`이 치명적인 약점이 될 수 있습니다. 오늘은 이 문제점을 파악하고, 더욱 안전하고 공정한 무작위성을 확보하는 방법을 React와 Web Crypto API를 통해 살펴보겠습니다.

## `Math.random()`의 불편한 진실과 `Web Crypto API`의 등장

### `Math.random()`: 편리하지만 예측 가능한 유사 난수

`Math.random()`은 웹 개발에서 가장 흔하게 사용되는 난수 생성 함수입니다. 0(포함)과 1(제외) 사이의 부동 소수점 유사 난수를 반환하죠. 여기서 핵심은 '유사 난수(Pseudo-Random Number)'라는 점입니다. 이는 특정 알고리즘에 의해 생성되며, 초기 시드(seed) 값만 알면 다음 숫자를 예측할 수 있다는 의미입니다.

브라우저 환경에서 `Math.random()`은 시스템 시간이나 다른 환경 변수들을 시드로 사용하기 때문에, 이론적으로는 예측이 어렵다고 생각할 수 있습니다. 하지만 숙련된 공격자나 특정 상황에서는 패턴을 파악하여 난수를 예측할 가능성이 존재합니다. 이는 특히 온라인 카지노, 경품 추첨, 보안 토큰 생성 등 '공정성'과 '보안'이 핵심인 서비스에서는 절대 용납될 수 없는 문제입니다.

### `Web Crypto API`: 암호학적으로 안전한 난수의 표준

이러러한 `Math.random()`의 한계를 극복하기 위해 등장한 것이 바로 `Web Crypto API`입니다. `Web Crypto API`는 암호학적으로 안전한 난수 생성기(CSPRNG: Cryptographically Secure Pseudo-Random Number Generator)를 제공합니다. 이 API는 운영체제 수준에서 제공하는 고품질의 엔트로피(무작위성) 소스를 활용하여, 예측 불가능하고 편향되지 않은 진정한 의미의 무작위 값을 생성합니다.

특히 `window.crypto.getRandomValues()` 메서드는 지정된 TypedArray를 암호학적으로 안전한 무작위 값으로 채워줍니다. 이는 보안에 민감한 모든 무작위성 요구 사항에 대한 표준적인 해결책이라고 할 수 있습니다.

## React와 TypeScript로 구현하는 공정한 추첨 시스템

이제 `Web Crypto API`를 활용하여 React 컴포넌트에서 공정한 추첨 번호를 생성하는 방법을 살펴보겠습니다. 우리의 목표는 1부터 100 사이의 숫자를 암호학적으로 안전하게 생성하는 것입니다.

```typescript
import React, { useState, useCallback } from 'react';

// Math.random()을 사용한 예시 (경고용)
const generatePseudoRandomNumber = (max: number): number => {
  console.warn("경고: Math.random()은 보안이 중요한 추첨에는 부적합합니다.");
  return Math.floor(Math.random() * max) + 1;
};

const FairRaffleGenerator: React.FC = () => {
  const [randomNumber, setRandomNumber] = useState<number | null>(null);
  const min = 1;
  const max = 100; // 1부터 100 사이의 난수 생성

  /**
   * Web Crypto API를 사용하여 암호학적으로 안전한 난수를 생성합니다.
   * @returns {number | null} 생성된 난수 또는 오류 발생 시 null
   */
  const generateSecureRandomNumber = useCallback((): number | null => {
    // 1. Web Crypto API 지원 여부 확인
    if (typeof window.crypto === 'undefined' || typeof window.crypto.getRandomValues === 'undefined') {
      console.error("오류: Web Crypto API를 사용할 수 없습니다. 브라우저를 업데이트해주세요.");
      alert("Web Crypto API를 지원하지 않는 브라우저입니다. 최신 브라우저를 사용해주세요.");
      return null;
    }

    // 2. Uint32Array 생성: 32비트 부호 없는 정수 1개를 담을 배열
    const array = new Uint32Array(1);

    // 3. getRandomValues로 배열을 암호학적으로 안전한 난수로 채움
    window.crypto.getRandomValues(array);

    // 4. 생성된 난수를 원하는 범위로 변환
    // array[0]은 0부터 2^32 - 1 (약 42억) 사이의 값을 가집니다.
    // 이 값을 0부터 1 사이의 부동 소수점으로 정규화합니다.
    const randomValue = array[0] / (0xFFFFFFFF + 1); // 0xFFFFFFFF는 2^32 - 1

    // 5. 정규화된 값을 [min, max] 범위의 정수로 변환
    const range = max - min + 1;
    const result = Math.floor(randomValue * range) + min;

    setRandomNumber(result);
    return result;
  }, [min, max]);

  return (
    <div style={{ padding: '20px', border: '1px solid #ddd', borderRadius: '8px', maxWidth: '400px', margin: '20px auto' }}>
      <h3>✨ 공정한 추첨 번호 생성기 (Web Crypto API 활용)</h3>
      <p style={{ fontSize: '1.2em', fontWeight: 'bold' }}>
        현재 추첨 번호: {randomNumber !== null ? <span style={{ color: '#007bff' }}>{randomNumber}</span> : '생성 전'}
      </p>
      <button
        onClick={generateSecureRandomNumber}
        style={{
          padding: '10px 20px',
          fontSize: '1em',
          backgroundColor: '#28a745',
          color: 'white',
          border: 'none',
          borderRadius: '5px',
          cursor: 'pointer'
        }}
      >
        공정한 번호 생성!
      </button>
      <p style={{ fontSize: '0.85em', color: '#666', marginTop: '15px' }}>
        * 이 번호는 <code style={{ backgroundColor: '#e9ecef', padding: '2px 4px', borderRadius: '3px' }}>Web Crypto API</code>를 사용하여 암호학적으로 안전하게 생성됩니다.
      </p>

      {/* Math.random() 예시 버튼 (비교용) */}
      <hr style={{ margin: '20px 0' }} />
      <h4 style={{ color: '#dc3545' }}>⚠️ Math.random()으로 생성 (비권장)</h4>
      <button
        onClick={() => setRandomNumber(generatePseudoRandomNumber(max))}
        style={{
          padding: '8px 15px',
          fontSize: '0.9em',
          backgroundColor: '#ffc107',
          color: '#343a40',
          border: 'none',
          borderRadius: '5px',
          cursor: 'pointer'
        }}
      >
        일반 난수 생성 (비권장)
      </button>
    </div>
  );
};

export default FairRaffleGenerator;
```

**코드 설명:**

1.  **`Uint32Array(1)` 생성**: `getRandomValues`는 TypedArray를 인자로 받습니다. 우리는 32비트 부호 없는 정수 1개를 담을 배열을 생성합니다.
2.  **`window.crypto.getRandomValues(array)` 호출**: 이 메서드가 `array`를 암호학적으로 안전한 난수로 채웁니다.
3.  **범위 변환 로직**: `getRandomValues`는 0부터 2^32-1 (0xFFFFFFFF) 사이의 값을 반환합니다. 이를 우리가 원하는 `min`에서 `max` 범위의 숫자로 변환하기 위해, 먼저 0부터 1 사이의 부동 소수점으로 정규화한 다음, `Math.floor(randomValue * range) + min` 공식을 사용하여 원하는 범위의 정수를 얻습니다.
4.  **`useCallback` 사용**: `generateSecureRandomNumber` 함수는 컴포넌트 리렌더링 시에도 동일한 함수 인스턴스를 유지하도록 `useCallback`으로 감싸 효율성을 높였습니다.

## 실무 적용 팁

*   **보안 민감 영역에 적극 활용**: 비밀번호 초기화 토큰, 세션 ID, 일회용 코드(OTP) 생성 등 보안에 민감한 모든 영역에서 `Web Crypto API`를 우선적으로 고려해야 합니다. 단순한 UI 애니메이션이나 개발 테스트 용도가 아니라면 `Math.random()`은 지양하는 것이 좋습니다.
*   **폴백(Fallback) 처리**: 모든 브라우저가 `Web Crypto API`를 지원하는 것은 아닙니다(물론 2026년인 지금은 대부분 지원하지만, 구형 브라우저나 특정 환경을 고려해야 할 수 있습니다). `typeof window.crypto !== 'undefined' && typeof window.crypto.getRandomValues !== 'undefined'`와 같이 지원 여부를 확인하고, 미지원 시에는 사용자에게 경고 메시지를 표시하거나 기능 사용을 제한하는 등의 폴백 처리를 구현해야 합니다.
*   **서버 사이드 난수**: 만약 서버 사이드(Node.js 등)에서 난수 생성이 필요하다면, Node.js의 내장 `crypto` 모듈을 사용하는 것이 적절합니다. `Web Crypto API`는 클라이언트 사이드(브라우저) 환경에 특화되어 있습니다.
*   **범위 변환의 정확성**: `getRandomValues`는 바이트 배열을 반환하므로, 이를 특정 숫자 범위로 정확하게 매핑하는 로직이 중요합니다. 예시 코드에서처럼 `(0xFFFFFFFF + 1)`로 나누어 0~1 사이의 부동 소수점을 얻는 방식이 일반적입니다.

## 마무리하며: 신뢰할 수 있는 서비스의 시작

오늘 우리는 `Math.random()`의 숨겨진 위험성을 깨닫고, `Web Crypto API`를 통해 암호학적으로 안전한 난수를 생성하는 방법을 React 컴포넌트와 함께 알아보았습니다. `Math.random()`은 간편하지만 예측 가능성 때문에 중요한 서비스에서는 사용을 지양해야 합니다. 대신 `Web Crypto API`의 `getRandomValues()` 메서드를 사용하여 사용자에게 진정으로 공정하고 신뢰할 수 있는 경험을 제공할 수 있습니다.

프론트엔드 개발자로서 우리는 단순히 멋진 UI를 만드는 것을 넘어, 서비스의 근간이 되는 '신뢰성'과 '보안'을 확보하는 데에도 깊이 관여해야 합니다. `Web Crypto API`는 이러한 목표를 달성하기 위한 강력한 도구 중 하나입니다. 다음번에는 `Web Crypto API`의 다른 기능들, 예를 들어 암호화나 해싱 기능에 대해서도 다뤄보면 좋을 것 같네요!

여러분의 서비스가 더욱 안전하고 신뢰받는 서비스로 거듭나길 바랍니다. 다음 포스팅에서 또 만나요!

---

## 참고 자료

*   [Adding Multi-Lingual Support in React](https://dev.to/shubham_gupta_6f6c50fefd4/adding-multi-lingual-support-in-react-i90) by shubham gupta
*   [How I Built a Free Derivative Calculator with Next.js 15 & React 19](https://dev.to/geek_6872d0d95c515aa70202/how-i-built-a-free-derivative-calculator-with-nextjs-15-react-19-3i5o) by GEEK
*   [Composition vs Inheritance: Why React Chose the "Has-A" Over "Is-A" Relationship](https://dev.to/tishonator/composition-vs-inheritance-why-react-chose-the-has-a-over-is-a-relationship-5108) by Tihomir Ivanov
*   [From React Rookie to Pro: Mastering the Modern Ecosystem and Landing Your Dream Gig! (React Day 10)](https://dev.to/vasughanta09/from-react-rookie-to-pro-mastering-the-modern-ecosystem-and-landing-your-dream-gig-react-day-10-51n4) by Vasu Ghanta
*   [Stop Using `Math.random()` for Raffles: Building a Truly Fair Wheel with React & Web Crypto API](https://dev.to/gaven_yang_438e60f53e6ff2/stop-using-mathrandom-for-raffles-building-a-truly-fair-wheel-with-react-web-crypto-api-2ahh) by gaven yang