---
layout: post
title: "2026년, React 앱의 미래를 엿보다: InAppAI, 설명 대신 '실행'하는 AI 에이전트"
date: 2026-01-15T15:02:10.445Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거 Ishant입니다! (아, Ishant는 원문 저자고 저는 여러분의 친근한 블로거, '프론티'라고 불러주세요!)

오늘 날짜는 2026년 1월 15일입니다. 시간이 정말 빠르죠? 불과 몇 년 전만 해도 'AI'하면 챗봇이나 이미지 생성기를 떠올렸는데, 이제는 우리의 애플리케이션 속에서 직접 행동하는 AI 에이전트의 시대가 눈앞에 와 있습니다.

오늘도 어김없이 흥미로운 프론트엔드 트렌드들을 가져왔는데요, 그중에서도 특히 미래 지향적인 한 가지 주제에 주목하고 싶습니다. 바로 React 앱에 AI 에이전트를 통합하는 'InAppAI'에 대한 이야기입니다.

---



## 소개
2026년은 AI가 '무엇을 할지' 설명하는 단계를 넘어 '직접 실행하는' 시대로 접어들고 있습니다. 사용자들은 더 이상 복잡한 매뉴얼이나 튜토리얼을 원하지 않습니다. 대신, 앱 내에서 즉시 문제를 해결하고 작업을 자동화해주는 똑똑한 비서들을 기대하죠. 오늘 우리는 이러한 기대를 충족시킬 혁신적인 오픈소스 프로젝트, InAppAI(개념)에 대해 깊이 파헤쳐 보겠습니다.

## 본문

### 핵심 개념 설명: InAppAI와 능동형 AI 에이전트
InAppAI는 React 애플리케이션 내에서 직접 작업을 수행하는 오픈소스 AI 에이전트 프레임워크(개념)입니다. 기존의 챗봇이나 AI 어시스턴트가 사용자 질문에 답하거나 정보를 제공하는 데 그쳤다면, InAppAI 에이전트는 앱의 UI와 상호작용하고, API를 호출하며, 사용자 대신 특정 기능을 실행하는 것이 핵심입니다.

예를 들어, "최근 주문 목록을 보여줘"라고 말하면 단순히 목록을 나열하는 것을 넘어, '주문 목록 필터링' 기능을 직접 실행하여 결과를 보여주는 식이죠. 이는 대규모 언어 모델(LLM)의 추론 능력과 React 앱의 컴포넌트 기반 구조를 결합하여 가능해집니다. 에이전트는 사용자의 의도를 파악하고, 사전에 정의된 '도구(tools)'를 사용하여 앱의 기능들을 호출하며, 그 결과를 다시 사용자에게 전달하는 일련의 과정을 자율적으로 수행합니다.

이는 프론트엔드 개발의 패러다임을 바꿀 잠재력을 가지고 있습니다. 사용자들은 더 이상 복잡한 메뉴를 탐색하거나 여러 단계를 거칠 필요 없이, 자연어 명령 하나로 원하는 바를 이룰 수 있게 됩니다. 개발자 입장에서는 반복적인 UI 조작 로직을 AI 에이전트에게 위임하여 더 고차원적인 문제 해결에 집중할 수 있게 되죠.

### 실제 코드 예시: React/Next.js/TypeScript 활용
아직 `InAppAI`라는 이름의 표준화된 라이브러리가 널리 보급되지는 않았지만, 그 핵심 개념을 React/Next.js/TypeScript 환경에서 어떻게 구현할 수 있을지 가상의 시나리오를 통해 살펴보겠습니다. 여기서는 사용자가 자연어 명령으로 상품 목록을 필터링하는 AI 에이전트를 상상해봅니다.

**시나리오:** 사용자가 "50달러 미만의 빨간색 상품 보여줘"라고 입력하면, AI 에이전트가 이를 해석하여 상품 목록을 필터링하는 기능을 실행합니다.

```typescript
// src/components/ProductListAgent.tsx (React 컴포넌트)
import React, { useState, useCallback } from 'react';
// InAppAIAgentProvider, useAgent는 개념적인 라이브러리 인터페이스입니다.
// 실제 구현에서는 특정 LLM SDK (예: OpenAI JS SDK)와 커스텀 훅을 조합하여 구성할 수 있습니다.
import { InAppAIAgentProvider, useAgent } from 'inapp-ai'; 

interface Product {
  id: string;
  name: string;
  price: number;
  color: string;
}

const initialProducts: Product[] = [
  { id: '1', name: 'Red T-Shirt', price: 25, color: 'red' },
  { id: '2', name: 'Blue Jeans', price: 60, color: 'blue' },
  { id: '3', name: 'Green Cap', price: 15, color: 'green' },
  { id: '4', name: 'Red Shoes', price: 45, color: 'red' },
  { id: '5', name: 'Yellow Hat', price: 20, color: 'yellow' },
  { id: '6', name: 'Red Scarf', price: 30, color: 'red' },
];

const ProductListAgent: React.FC = () => {
  const [filteredProducts, setFilteredProducts] = useState<Product[]>(initialProducts);
  const [agentInput, setAgentInput] = useState<string>('');
  const [agentResponse, setAgentResponse] = useState<string>('');

  // AI 에이전트가 사용할 수 있는 '도구' 정의
  // 이 함수들은 에이전트가 LLM을 통해 호출할 수 있는 실제 앱 기능입니다.
  const tools = {
    filterProducts: useCallback(async (criteria: { price?: number; color?: string }) => {
      console.log('Agent is filtering products with criteria:', criteria);
      const result = initialProducts.filter(product => {
        const priceMatch = criteria.price ? product.price <= criteria.price : true;
        const colorMatch = criteria.color ? product.color.toLowerCase() === criteria.color.toLowerCase() : true;
        return priceMatch && colorMatch;
      });
      setFilteredProducts(result);
      return `조건에 맞는 상품 ${result.length}개가 필터링되었습니다.`;
    }, []), // 의존성 배열 비움 (initialProducts는 변경되지 않음)
    
    resetFilters: useCallback(async () => {
      setFilteredProducts(initialProducts);
      return '필터가 초기화되었습니다. 모든 상품을 표시합니다.';
    }, []),
  };

  // useAgent 훅을 통해 AI 에이전트와 상호작용
  const { sendToAgent, agentState } = useAgent({
    agentId: 'product-filter-agent', // 에이전트 식별자
    tools, // 정의한 도구들을 에이전트에 전달
    // 에이전트가 LLM과 통신하기 위한 백엔드 API 엔드포인트 (Next.js API Route)
    agentApiUrl: '/api/inapp-ai-agent', 
  });

  const handleSend = async () => {
    if (!agentInput.trim()) return;
    setAgentResponse('AI 에이전트가 요청을 처리 중입니다...');
    try {
      // 사용자의 메시지를 에이전트에 전달하고 응답을 받습니다.
      const response = await sendToAgent(agentInput);
      setAgentResponse(response || '에이전트로부터 특정 응답은 없지만, 작업이 수행되었을 수 있습니다.');
      setAgentInput(''); // 입력창 초기화
    } catch (error) {
      console.error('AI 에이전트 요청 실패:', error);
      setAgentResponse(`오류 발생: ${error instanceof Error ? error.message : String(error)}`);
    }
  };

  return (
    <div className="p-6 max-w-2xl mx-auto bg-white rounded-xl shadow-lg space-y-4">
      <h2 className="text-2xl font-bold text-gray-900">상품 관리 AI 에이전트</h2>
      <p className="text-gray-600">자연어 명령으로 상품 목록을 관리해보세요.</p>

      <div className="flex items-center space-x-2">
        <input
          type="text"
          className="flex-1 p-3 border border-gray-300 rounded-md focus:ring-blue-500 focus:border-blue-500"
          placeholder="예: 50달러 미만의 빨간색 상품 보여줘"
          value={agentInput}
          onChange={(e) => setAgentInput(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && handleSend()}
          disabled={agentState.isLoading}
        />
        <button
          className="px-5 py-3 bg-blue-600 text-white font-semibold rounded-md hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed"
          onClick={handleSend}
          disabled={agentState.isLoading}
        >
          {agentState.isLoading ? '처리 중...' : 'AI 에이전트에 요청'}
        </button>
      </div>

      {agentResponse && (
        <p className="text-sm text-gray-700 bg-gray-50 p-3 rounded-md border border-gray-200">
          <strong>AI 응답:</strong> {agentResponse}
        </p>
      )}

      <h3 className="text-xl font-semibold text-gray-800 mt-6">현재 상품 목록 ({filteredProducts.length}개)</h3>
      <ul className="list-disc pl-5 space-y-1 text-gray-700">
        {filteredProducts.length > 0 ? (
          filteredProducts.map(product => (
            <li key={product.id}>{product.name} (${product.price}, {product.color})</li>
          ))
        ) : (
          <li>조건에 맞는 상품이 없습니다.</li>
        )}
      </ul>
    </div>
  );
};

// Next.js 페이지 또는 App 컴포넌트에서 Provider로 감싸줍니다.
const App: React.FC = () => (
  <InAppAIAgentProvider> {/* 전역적으로 Agent Provider 설정 */}
    <ProductListAgent />
  </InAppAIAgentProvider>
);

export default App;

/*
// Next.js API Route (pages/api/inapp-ai-agent.ts) 예시
// 이 부분은 서버 사이드에서 InAppAI 에이전트의 LLM 호출을 처리합니다.
// 실제 구현에서는 OpenAI API 키 등을 안전하게 관리해야 합니다.

import type { NextApiRequest, NextApiResponse } from 'next';
import { InAppAIAgentServer } from 'inapp-ai/server'; // 서버 사이드 모듈 (개념적)
import OpenAI from 'openai'; // 실제 LLM SDK 예시

// 실제 구현에서는 환경 변수에서 API 키를 불러옵니다.
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!, 
});

// InAppAIAgentServer는 클라이언트에서 전달된 도구 정의와 사용자 메시지를 바탕으로
// LLM에 요청을 보내고, LLM이 어떤 도구를 호출해야 할지 결정하면,
// 해당 도구의 실행을 위한 정보를 클라이언트에 반환하는 역할을 합니다.
// (또는 직접 서버에서 도구를 실행할 수도 있습니다.)
const agentServer = {
  handleAgentRequest: async ({ agentId, userMessage, availableTools }: {
    agentId: string;
    userMessage: string;
    availableTools: Record<string, any>; // 클라이언트에서 정의된 도구 스키마
  }) => {
    // LLM에 도구 정보와 사용자 메시지를 함께 전달하여 추론을 요청
    const chatCompletion = await openai.chat.completions.create({
      model: 'gpt-4o', // 또는 다른 적절한 LLM
      messages: [
        { role: 'system', content: `You are an AI assistant for a product management application. You can use the following tools: ${JSON.stringify(availableTools)}` },
        { role: 'user', content: userMessage }
      ],
      tools: Object.keys(availableTools).map(name => ({
        type: 'function',
        function: {
          name,
          description: availableTools[name].description || `Call the ${name} function.`,
          parameters: availableTools[name].parameters, // 도구의 인자 스키마
        }
      })),
      tool_choice: 'auto', // LLM이 적절한 도구를 자동으로 선택하도록 함
    });

    const responseMessage = chatCompletion.choices[0].message;

    if (responseMessage.tool_calls && responseMessage.tool_calls.length > 0) {
      // LLM이 도구 호출을 요청했다면, 해당 정보를 클라이언트에 반환
      // 클라이언트에서는 이 정보를 바탕으로 실제 React 컴포넌트의 함수를 호출합니다.
      return {
        action: 'call_tool',
        toolCalls: responseMessage.tool_calls.map(tc => ({
          name: tc.function.name,
          arguments: JSON.parse(tc.function.arguments),
        }))
      };
    } else {
      // LLM이 텍스트 응답을 했다면, 그대로 반환
      return {
        action: 'reply',
        message: responseMessage.content,
      };
    }
  }
};


export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method === 'POST') {
    try {
      const { agentId, userMessage, availableTools } = req.body;
      // 서버 사이드에서 LLM과 통신하여 에이전트의 다음 행동을 결정
      const agentResponse = await agentServer.handleAgentRequest({
        agentId,
        userMessage,
        availableTools, 
      });
      res.status(200).json(agentResponse);
    } catch (error) {
      console.error('InAppAI API Error:', error);
      res.status(500).json({ error: 'Failed to process agent request' });
    }
  } else {
    res.setHeader('Allow', ['POST']);
    res.status(405).end(`Method ${req.method} Not Allowed`);
  }
}
*/
```
**코드 설명:**
1.  **`ProductListAgent.tsx`**: React 컴포넌트에서 `useState`로 상품 목록을 관리합니다.
2.  **`tools` 객체**: AI 에이전트가 호출할 수 있는 함수들을 정의합니다. 여기서는 `filterProducts`와 `resetFilters`가 있습니다. 이 함수들은 실제 React 컴포넌트의 상태를 변경하는 로직을 포함합니다.
3.  **`useAgent` 훅**: `inapp-ai` 라이브러리(가상)에서 제공하는 훅으로, `agentId`, `tools`, 그리고 `agentApiUrl`을 인자로 받습니다. `sendToAgent` 함수를 통해 사용자 메시지를 AI 에이전트에 전달하고, `agentState`로 에이전트의 상태(로딩 중 등)를 확인할 수 있습니다.
4.  **`agentApiUrl: '/api/inapp-ai-agent'`**: Next.js의 API Route를 통해 서버리스 함수로 LLM과의 통신을 처리합니다. 이는 클라이언트에서 직접 민감한 API 키를 노출하지 않고, LLM 호출 로직을 서버에서 안전하게 관리하기 위함입니다.
5.  **Next.js API Route (`pages/api/inapp-ai-agent.ts`)**: 이 부분은 클라이언트로부터 받은 사용자 메시지와 사용 가능한 도구 정보를 바탕으로 실제 LLM(예: OpenAI GPT-4o)에 요청을 보냅니다. LLM은 사용자의 의도를 파악하여 어떤 도구를 어떤 인자와 함께 호출해야 할지 결정하고, 그 정보를 다시 클라이언트에 반환합니다. 클라이언트는 이 정보를 받아 실제 `tools` 객체의 함수를 실행합니다.

이러한 구조를 통해 AI 에이전트는 사용자의 자연어 명령을 앱의 실제 기능 호출로 변환하여 실행할 수 있게 됩니다.

### 실무 적용 팁
InAppAI와 같은 능동형 AI 에이전트를 실제 프로젝트에 도입할 때 고려해야 할 몇 가지 팁입니다.

1.  **점진적 도입:** 모든 기능을 AI 에이전트로 대체하려 하지 말고, 사용자 피로도가 높은 특정 반복 작업을 자동화하는 것부터 시작하세요. (예: 복잡한 필터링, 데이터 입력 보조, 보고서 생성 등)
2.  **명확한 도구 정의:** 에이전트가 수행할 수 있는 '도구(tools)'를 명확하고 세분화하여 정의해야 합니다. 도구가 모호하면 에이전트의 오작동 가능성이 높아집니다. 각 도구의 목적, 입력, 출력을 상세히 문서화하는 것이 중요합니다.
3.  **사용자 경험 중심:** AI 에이전트의 작동 상태(생각 중, 실행 중, 완료)를 사용자에게 명확히 알려주고, 언제든지 사용자가 개입하거나 작업을 취소할 수 있는 UI를 제공해야 합니다. 투명성과 제어권은 사용자 신뢰 구축에 필수적입니다.
4.  **보안 및 데이터 프라이버시:** 에이전트가 민감한 데이터를 다루는 경우, 접근 권한 관리와 데이터 보안에 각별히 신경 써야 합니다. LLM 제공업체와의 데이터 처리 약관도 필수적으로 검토하고, 민감 정보는 LLM에 직접 노출되지 않도록 처리해야 합니다.
5.  **성능 및 비용 최적화:** LLM 호출은 네트워크 지연과 비용을 수반합니다. 불필요한 호출을 줄이고, 캐싱 전략을 고려하며, 비용 효율적인 모델을 선택하는 것이 중요합니다. 경우에 따라서는 소형 경량 모델(SLM)을 활용하는 것도 좋은 방법입니다.
6.  **에러 핸들링 및 폴백:** 에이전트가 사용자의 의도를 잘못 이해하거나, API 호출에 실패했을 때를 대비한 견고한 에러 핸들링과 폴백(fallback) 메커니즘을 구축해야 합니다. 사용자가 당황하지 않도록 친절한 안내와 대체 솔루션을 제공하는 것이 중요합니다.

## 마무리
InAppAI는 단순히 정보를 제공하는 것을 넘어, 사용자의 의도를 이해하고 실제 작업을 수행하는 '능동적인' 애플리케이션의 시대를 열고 있습니다. 2026년, 프론트엔드 개발자로서 우리는 이러한 AI 에이전트들을 우리의 React 앱에 어떻게 효과적으로 녹여낼지 고민해야 합니다.

지금 바로 여러분의 프로젝트에서 InAppAI와 같은 개념을 적용할 수 있는 지점을 찾아보고, 사용자 경험을 혁신할 새로운 가능성을 탐색해 보세요. 앞으로 AI 에이전트 기술은 더욱 발전하여, 우리의 앱을 훨씬 더 직관적이고 강력하게 만들 것입니다.

다음번에는 또 다른 흥미로운 프론트엔드 트렌드를 가지고 찾아오겠습니다!

## 참고 자료
- [🚇 I Accidentally Built a Full 3D Endless Runner Game in the Browser (It’s in KBs btw)](https://dev.to/swagking/i-accidentally-built-a-full-3d-endless-runner-game-in-the-browser-its-in-kbs-btw-fh) by Ishant Singh
- [My go-to patterns for full-stack/frontend projects](https://dev.to/snikidev/my-go-to-patterns-for-full-stackfrontend-projects-150a) by Niki
- [InAppAI: Open-Source AI Agents for React Apps](https://dev.to/chaoming/inappai-open-source-ai-agents-for-react-apps-1l86) by Chaoming Li
- [A Gradual Approach to React Folder Structure: From Package by Feature to Clean Architecture](https://dev.to/usapopopooon/a-gradual-approach-to-react-folder-structure-from-package-by-feature-to-clean-architecture-e44) by usapopopooon
- [Getting React on Rails to work with Turbo Streams](https://dev.to/bitcrowd/getting-react-on-rails-to-work-with-turbo-streams-2c46) by Max