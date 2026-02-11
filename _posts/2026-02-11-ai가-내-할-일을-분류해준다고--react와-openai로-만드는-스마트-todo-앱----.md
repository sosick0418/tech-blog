---
layout: post
title: "AI가 내 할 일을 분류해준다고? React와 OpenAI로 만드는 스마트 Todo 앱! 🚀"
date: 2026-02-11T01:01:08.969Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거입니다! 2026년 2월 11일, 오늘도 새로운 기술 트렌드를 함께 탐험해 볼 시간입니다. 프론트엔드 개발의 지평이 넓어지면서 AI 기술과의 융합은 이제 선택이 아닌 필수가 되어가고 있는데요. 오늘은 그중에서도 가장 실용적이고 흥미로운 주제를 다뤄보려 합니다.

---



## 소개

생산성 도구는 우리 일상과 개발 워크플로우에서 중요한 부분을 차지합니다. 특히 Todo 앱은 많은 분들이 사용하는 기본 중의 기본이죠. 그런데 만약 이 Todo 앱이 단순히 목록을 나열하는 것을 넘어, AI의 도움을 받아 자동으로 할 일을 분류하고 우선순위까지 제안해준다면 어떨까요? AI 기술이 프론트엔드 애플리케이션에 어떻게 통합되어 사용자 경험을 혁신할 수 있는지, 그 가능성을 탐구하는 것은 이제 프론트엔드 개발자에게 매우 중요한 역량이 되었습니다.

## 본문

### 핵심 개념 설명

오늘 우리가 만들 스마트 Todo 앱의 핵심은 **React**를 이용한 인터랙티브한 UI와 **OpenAI API**를 활용한 지능적인 할 일 분류 및 우선순위 제안 기능입니다.

1.  **OpenAI API**: OpenAI는 GPT-3.5 Turbo 또는 GPT-4와 같은 강력한 언어 모델을 API 형태로 제공합니다. 우리는 이 API에 사용자가 입력한 Todo 텍스트를 전송하여, 해당 텍스트의 내용을 분석하고 미리 정의된 카테고리(예: '업무', '개인', '쇼핑', '학습')와 우선순위('높음', '중간', '낮음')를 JSON 형태로 반환하도록 요청할 것입니다.
2.  **React**: React는 사용자 인터페이스를 구축하는 데 최적화된 라이브러리입니다. 사용자가 Todo를 입력하고, AI로부터 받은 분류 결과를 화면에 효율적으로 렌더링하며, 상태를 관리하는 데 사용됩니다.
3.  **Next.js API Route**: 클라이언트 측(브라우저)에서 직접 OpenAI API 키를 노출하는 것은 보안상 매우 위험합니다. Next.js의 API Route 기능을 활용하여 서버리스 함수 형태로 OpenAI API 호출을 처리하면, API 키를 안전하게 보호하면서 클라이언트 요청을 프록시할 수 있습니다.

이 세 가지 기술의 조합을 통해 사용자는 자연어 기반으로 Todo를 입력하기만 하면, AI가 자동으로 맥락을 이해하고 정리해주는 마법 같은 경험을 하게 될 것입니다.

### 실제 코드 예시 (React + Next.js + TypeScript)

간단한 React 기반의 Next.js 프로젝트에서 AI 기반 Todo 앱을 구현하는 핵심 코드들을 살펴보겠습니다.

먼저, OpenAI API 키를 안전하게 다루기 위해 `.env.local` 파일에 `OPENAI_API_KEY`를 설정해주세요.

```
// .env.local
OPENAI_API_KEY=sk-YOUR_OPENAI_API_KEY
```

**1. Next.js API Route (`pages/api/categorize-todo.ts`)**

클라이언트에서 전송된 Todo 텍스트를 받아 OpenAI API에 요청하고, 그 결과를 다시 클라이언트로 전달하는 서버리스 함수입니다.

```typescript
// pages/api/categorize-todo.ts
import type { NextApiRequest, NextApiResponse } from 'next';

// OpenAI API 응답의 타입을 정의합니다.
interface OpenAIResponse {
  choices: Array<{
    message: {
      content: string;
    };
  }>;
}

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  // POST 요청만 허용합니다.
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }

  const { todoText } = req.body;

  if (!todoText) {
    return res.status(400).json({ message: 'Todo text is required' });
  }

  try {
    const response = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`, // 환경 변수에서 API 키 로드
      },
      body: JSON.stringify({
        model: 'gpt-3.5-turbo', // 또는 더 강력한 gpt-4 모델 사용 가능
        messages: [
          {
            role: 'system',
            content: `당신은 할 일 항목을 분류하고 우선순위를 제안하는 유용한 비서입니다.
                      출력은 'category' (예: '업무', '개인', '쇼핑', '학습', '건강')와 'priority' (예: '높음', '중간', '낮음')를 포함하는 JSON 객체여야 합니다.
                      예시: {"category": "업무", "priority": "높음"}`
          },
          {
            role: 'user',
            content: `이 할 일을 분류하고 우선순위를 정해주세요: "${todoText}"`
          }
        ],
        temperature: 0.7, // 창의성 조절
        max_tokens: 60,   // 응답의 최대 길이
        response_format: { type: "json_object" }, // JSON 형식으로 응답 요청 (GPT-3.5 Turbo 이상)
      }),
    });

    if (!response.ok) {
      const errorData = await response.json();
      console.error('OpenAI API 오류:', errorData);
      return res.status(response.status).json({ message: '할 일 분류 실패', details: errorData });
    }

    const data: OpenAIResponse = await response.json();
    const content = data.choices[0]?.message?.content;

    if (!content) {
      return res.status(500).json({ message: 'OpenAI로부터 내용 없음' });
    }

    // OpenAI 응답이 JSON 문자열이므로 파싱합니다.
    let parsedContent;
    try {
      parsedContent = JSON.parse(content);
    } catch (parseError) {
      console.error('OpenAI 응답 JSON 파싱 실패:', content, parseError);
      return res.status(500).json({ message: 'OpenAI로부터 유효하지 않은 JSON 응답', raw: content });
    }

    res.status(200).json(parsedContent);

  } catch (error) {
    console.error('API 라우트 오류:', error);
    res.status(500).json({ message: '서버 내부 오류 발생' });
  }
}
```

**2. 메인 애플리케이션 컴포넌트 (`pages/index.tsx`)**

Todo 입력 폼과 Todo 목록을 관리하고, API Route를 호출하여 AI 분류 결과를 받아오는 로직을 포함합니다.

```typescript
// pages/index.tsx
import React, { useState } from 'react';
import { v4 as uuidv4 } from 'uuid'; // 'npm install uuid'로 설치
import TodoForm from '../components/TodoForm';
import TodoList from '../components/TodoList';

// Todo 항목의 타입을 정의합니다.
interface Todo {
  id: string;
  text: string;
  category?: string;
  priority?: '높음' | '중간' | '낮음';
  isCompleted: boolean;
}

// AI 분류 결과의 타입을 정의합니다.
interface CategorizationResult {
  category: string;
  priority: '높음' | '중간' | '낮음';
}

const Home: React.FC = () => {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // 새로운 Todo를 추가하고 AI 분류를 요청하는 함수
  const addTodo = async (text: string) => {
    setIsLoading(true);
    setError(null);
    try {
      const response = await fetch('/api/categorize-todo', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ todoText: text }),
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || '할 일 분류에 실패했습니다.');
      }

      const result: CategorizationResult = await response.json();

      setTodos((prevTodos) => [
        {
          id: uuidv4(), // 고유 ID 생성
          text,
          category: result.category,
          priority: result.priority,
          isCompleted: false,
        },
        ...prevTodos, // 새 할 일을 목록 상단에 추가
      ]);
    } catch (err: any) {
      console.error('Todo 추가 중 오류 발생:', err);
      setError(err.message || '할 일을 분류하는 데 실패했습니다. 일반 할 일로 추가합니다.');
      // AI 분류 실패 시에도 일단 할 일은 추가 (사용자 경험을 위해)
      setTodos((prevTodos) => [
        {
          id: uuidv4(),
          text,
          isCompleted: false,
        },
        ...prevTodos,
      ]);
    } finally {
      setIsLoading(false);
    }
  };

  // Todo 완료 상태를 토글하는 함수
  const toggleComplete = (id: string) => {
    setTodos((prevTodos) =>
      prevTodos.map((todo) =>
        todo.id === id ? { ...todo, isCompleted: !todo.isCompleted } : todo
      )
    );
  };

  return (
    <div className="container mx-auto p-4 max-w-2xl">
      <h1 className="text-4xl font-bold text-center mb-8 text-gray-800">
        AI 기반 스마트 Todo 앱 🚀
      </h1>
      <TodoForm onAddTodo={addTodo} isLoading={isLoading} />
      {error && (
        <p className="text-red-500 bg-red-100 p-3 rounded-md mb-4">{error}</p>
      )}
      <TodoList todos={todos} onToggleComplete={toggleComplete} />
    </div>
  );
};

export default Home;
```

**3. Todo 입력 폼 컴포넌트 (`components/TodoForm.tsx`)**

사용자 입력을 받아 `onAddTodo` 함수를 호출합니다.

```typescript
// components/TodoForm.tsx
import React, { useState } from 'react';

interface TodoFormProps {
  onAddTodo: (text: string) => void;
  isLoading: boolean; // 로딩 상태 표시
}

const TodoForm: React.FC<TodoFormProps> = ({ onAddTodo, isLoading }) => {
  const [todoText, setTodoText] = useState('');

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (todoText.trim()) {
      onAddTodo(todoText);
      setTodoText('');
    }
  };

  return (
    <form onSubmit={handleSubmit} className="flex gap-2 mb-4">
      <input
        type="text"
        value={todoText}
        onChange={(e) => setTodoText(e.target.value)}
        placeholder="새로운 할 일을 입력하세요 (예: 다음 주까지 보고서 제출)..."
        className="flex-grow p-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
        disabled={isLoading} // 로딩 중에는 입력 비활성화
      />
      <button
        type="submit"
        className="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed"
        disabled={isLoading}
      >
        {isLoading ? '분류 중...' : '추가'}
      </button>
    </form>
  );
};

export default TodoForm;
```

**4. Todo 목록 컴포넌트 (`components/TodoList.tsx`)**

Todo 항목들을 표시하고, AI가 분류한 카테고리 및 우선순위를 시각화합니다.

```typescript
// components/TodoList.tsx
import React from 'react';

interface Todo {
  id: string;
  text: string;
  category?: string;
  priority?: '높음' | '중간' | '낮음';
  isCompleted: boolean;
}

interface TodoListProps {
  todos: Todo[];
  onToggleComplete: (id: string) => void;
}

const TodoList: React.FC<TodoListProps> = ({ todos, onToggleComplete }) => {
  return (
    <ul className="space-y-2">
      {todos.length === 0 && <p className="text-center text-gray-500">아직 할 일이 없습니다. 새로운 할 일을 추가해보세요!</p>}
      {todos.map((todo) => (
        <li
          key={todo.id}
          className={`flex items-center justify-between p-3 border rounded-md shadow-sm ${
            todo.isCompleted ? 'bg-gray-100 line-through text-gray-500' : 'bg-white'
          }`}
        >
          <div className="flex items-center">
            <input
              type="checkbox"
              checked={todo.isCompleted}
              onChange={() => onToggleComplete(todo.id)}
              className="mr-3 h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded"
            />
            <div>
              <span className="block text-lg font-medium">{todo.text}</span>
              {(todo.category || todo.priority) && (
                <div className="text-sm text-gray-500 mt-0.5">
                  {/* 카테고리 태그 */}
                  {todo.category && <span className="mr-2 px-2 py-0.5 bg-blue-100 text-blue-800 rounded-full text-xs">{todo.category}</span>}
                  {/* 우선순위 태그 (색상으로 구분) */}
                  {todo.priority && (
                    <span
                      className={`px-2 py-0.5 rounded-full text-xs ${
                        todo.priority === '높음' ? 'bg-red-100 text-red-800' :
                        todo.priority === '중간' ? 'bg-yellow-100 text-yellow-800' :
                        'bg-green-100 text-green-800'
                      }`}
                    >
                      {todo.priority}
                    </span>
                  )}
                </div>
              )}
            </div>
          </div>
        </li>
      ))}
    </ul>
  );
};

export default TodoList;
```

(참고: 위 코드는 Tailwind CSS를 사용한 것으로 가정하며, `npm install -D tailwindcss postcss autoprefixer` 및 `tailwind.config.js` 설정이 필요합니다.)

### 실무 적용 팁

1.  **API 키 보안**: `process.env.OPENAI_API_KEY`와 같이 환경 변수를 사용하고, Next.js API Route처럼 서버리스 환경에서만 사용해야 합니다. 클라이언트 측 코드에 API 키를 직접 노출하는 것은 절대 금물입니다.
2.  **프롬프트 엔지니어링**: AI의 응답 품질은 프롬프트(지시문)에 크게 좌우됩니다. 위에 제시된 `system` 메시지처럼 명확하고 구체적인 지시와 원하는 출력 형식(JSON)을 명시하는 것이 중요합니다. 다양한 테스트를 통해 최적의 프롬프트를 찾아보세요.
3.  **에러 처리 및 로딩 상태**: API 호출은 네트워크 지연이나 실패 가능성이 있습니다. `isLoading` 상태를 통해 사용자에게 현재 작업이 진행 중임을 알리고, `error` 상태를 통해 문제가 발생했을 때 적절한 피드백을 제공하는 것이 중요합니다.
4.  **비용 관리**: OpenAI API는 사용량에 따라 비용이 발생합니다. `max_tokens`를 적절히 설정하여 불필요한 토큰 사용을 줄이고, 개발 단계에서는 비용 모니터링을 꾸준히 하는 것이 좋습니다.
5.  **사용자 경험 개선**: AI 분류 결과가 항상 완벽하지 않을 수 있습니다. 사용자가 직접 카테고리나 우선순위를 수정할 수 있는 기능을 추가하거나, AI의 분류에 대한 피드백을 받을 수 있는 메커니즘을 마련하여 모델을 개선하는 방안도 고려할 수 있습니다.

## 마무리

오늘 우리는 React와 OpenAI API를 활용하여 AI 기반 스마트 Todo 앱을 구축하는 방법을 알아보았습니다. 단순히 Todo 목록을 관리하는 것을 넘어, AI가 사용자의 의도를 파악하고 작업을 분류해주는 기능은 사용자 경험을 한 단계 끌어올릴 수 있는 강력한 잠재력을 보여줍니다.

이 예시는 AI를 프론트엔드 애플리케이션에 통합하는 첫걸음에 불과합니다. 여러분은 이 개념을 확장하여 다음과 같은 기능을 추가해 볼 수 있습니다:

*   **스트리밍 API**: OpenAI의 스트리밍 응답을 활용하여 Todo 분류 결과가 실시간으로 나타나게 구현해 보세요.
*   **추가 AI 기능**: Todo 요약, 관련 정보 검색, 마감일 예측 등 더 다양한 AI 기능을 통합해 보세요.
*   **사용자 맞춤 학습**: 사용자의 과거 분류 기록을 기반으로 AI 모델을 미세 조정하여 개인화된 분류 정확도를 높일 수 있습니다.
*   **UI/UX 개선**: 드래그 앤 드롭으로 우선순위 변경, 시각적인 대시보드 구현 등 사용자 친화적인 인터페이스를 개발해 보세요.

프론트엔드 개발의 미래는 AI와의 긴밀한 협력 속에 있습니다. 이 새로운 도구를 능숙하게 다루는 개발자가 되어 사용자에게 놀라운 경험을 선사하시길 바랍니다!

---

## 참고 자료

*   [Implemented a Feature where the Theme on my Portfolio changes based on the Holiday (Because it's fun) 💫](https://dev.to/francistrdev/implemented-a-feature-where-the-theme-on-my-portfolio-changes-based-on-the-holiday-because-its-31jo) by 👾 FrancisTRDev 👾
*   [I have got a lot of great feedback from the community. Although my portfolio is still in development, I would love to received some feedback on my Portfolio so far! Check out the post!](https://dev.to/francistrdev/i-have-got-a-lot-of-great-feedback-from-the-community-although-my-portfolio-is-still-in-2i7i) by 👾 FrancisTRDev 👾
*   [Forget Canvas Apps? Build Full-Code React + Azure OpenAI Apps on Power Platform](https://dev.to/ippu_ito_870812/forget-canvas-apps-build-full-code-react-azure-openai-apps-on-power-platform-kb) by Ippu Ito
*   [Build Your First AI-Powered Todo App: React + OpenAI Complete Tutorial](https://dev.to/paul_robertson_e844997d2b/build-your-first-ai-powered-todo-app-react-openai-complete-tutorial-3bam) by Paul Robertson
*   [I built a free, open-source App Store Screenshot Generator](https://dev.to/magicnull/i-built-a-free-open-source-app-store-screenshot-generator-39ko) by Magic.rb