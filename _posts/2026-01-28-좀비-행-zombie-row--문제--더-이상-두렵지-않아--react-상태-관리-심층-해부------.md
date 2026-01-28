---
layout: post
title: "좀비 행(Zombie Row) 문제, 더 이상 두렵지 않아! React 상태 관리 심층 해부 🧟‍♂️"
date: 2026-01-28T03:48:13.369Z
categories: [frontend, tech]
tags: [react, javascript, nextjs, typescript, tailwindcss]
---

안녕하세요, 프론트엔드 개발 전문 블로거 개발자 K입니다!
2026년 1월 28일, 오늘도 뜨거운 프론트엔드 트렌드를 들고 여러분을 찾아왔습니다.

---



React 개발자라면 한 번쯤 겪어봤을 '좀비 행(Zombie Row)' 문제, 즉 데이터를 삭제했는데도 UI에 잔재가 남아 사용자 경험을 해치거나 디버깅을 어렵게 하는 현상 말이죠. 오늘은 이 골치 아픈 문제를 React의 강력한 상태 관리 기법을 활용하여 깔끔하게 해결하는 방법을 심층적으로 알아보겠습니다.

## '좀비 행' 문제, 왜 생길까요?

'좀비 행' 문제는 사용자가 특정 데이터를 삭제했지만, UI 상에서는 해당 데이터가 여전히 보이거나 예상치 못한 방식으로 동작하는 현상을 일컫습니다. 이는 주로 비동기 데이터 처리와 UI 업데이트 간의 불일치에서 발생합니다.

**주요 원인은 다음과 같습니다:**

1.  **서버 응답 전 클라이언트 상태 미리 업데이트:** 사용자에게 빠른 피드백을 주기 위해 클라이언트 상태를 먼저 변경했지만, 서버에서 오류가 발생했을 때 롤백 로직이 없는 경우.
2.  **데이터 삭제 후 UI 컴포넌트 언마운트 또는 상태 동기화 누락:** 서버에서 데이터가 성공적으로 삭제되었더라도, 클라이언트 UI가 이를 반영하지 못하고 이전 상태를 계속 렌더링하는 경우.
3.  **데이터 캐싱 문제:** 클라이언트 측에서 데이터를 캐싱하고 있는데, 서버 데이터 변경 후 캐시가 제대로 무효화되지 않아 오래된 데이터를 계속 보여주는 경우.

이러한 문제들은 사용자에게 혼란을 주고, 애플리케이션의 신뢰도를 떨어뜨릴 수 있습니다.

## 좀비 행 문제를 해결하는 전략

좀비 행 문제를 해결하기 위한 가장 효과적인 전략은 **Optimistic UI Update(낙관적 업데이트)**와 **견고한 에러 핸들링 및 롤백 로직**을 결합하는 것입니다.

*   **Optimistic UI Update**: 사용자가 어떤 액션을 취했을 때, 서버의 응답을 기다리지 않고 즉시 UI를 업데이트하여 사용자에게 즉각적인 피드백을 제공하는 기법입니다. 이는 사용자 경험을 크게 향상시키지만, 서버 요청 실패 시 UI를 이전 상태로 되돌리는 '롤백' 로직이 필수적입니다.
*   **Server-Side Revalidation / Query Invalidation**: 서버 데이터 변경 후 클라이언트 캐시를 무효화하여 최신 데이터를 다시 불러오도록 강제하는 방법입니다. React Query나 SWR 같은 라이브러리에서 강력하게 지원하는 기능입니다.
*   **Proper State Management**: `useState`, `useReducer`, 또는 전역 상태 관리 라이브러리(Redux, Zustand 등)를 사용하여 애플리케이션의 상태를 일관되고 예측 가능하게 관리하는 것이 중요합니다.

## 실제 코드 예시: Optimistic UI를 이용한 아이템 삭제

이제 React(TypeScript) 환경에서 간단한 아이템 리스트를 만들고, Optimistic UI 업데이트를 적용하여 좀비 행 문제를 해결하는 코드를 살펴보겠습니다.

```typescript
// components/ItemList.tsx
import React, { useState, useEffect } from 'react';

// 아이템 데이터 타입을 정의합니다.
interface Item {
  id: string;
  name: string;
}

const ItemList: React.FC = () => {
  const [items, setItems] = useState<Item[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  // 컴포넌트 마운트 시 아이템 목록을 불러옵니다.
  useEffect(() => {
    fetchItems();
  }, []);

  // API에서 아이템 목록을 비동기로 가져오는 함수
  const fetchItems = async () => {
    setLoading(true);
    setError(null);
    try {
      // 실제 API 엔드포인트에 맞게 수정해주세요.
      const response = await fetch('/api/items'); 
      if (!response.ok) {
        throw new Error('Failed to fetch items');
      }
      const data: Item[] = await response.json();
      setItems(data);
    } catch (err) {
      setError((err as Error).message);
    } finally {
      setLoading(false);
    }
  };

  // 아이템 삭제 핸들러 (Optimistic UI 적용)
  const handleDeleteItem = async (idToDelete: string) => {
    // 1. Optimistic UI Update: 서버 응답 전에 UI에서 즉시 제거하여 사용자에게 빠른 피드백을 제공합니다.
    const previousItems = items; // 실패 시 롤백을 위해 이전 상태를 저장합니다.
    setItems(prevItems => prevItems.filter(item => item.id !== idToDelete));

    try {
      // 실제 API 엔드포인트에 맞게 수정해주세요.
      const response = await fetch(`/api/items/${idToDelete}`, {
        method: 'DELETE',
        headers: {
          'Content-Type': 'application/json',
          // 필요한 경우 인증 토큰 등을 추가합니다.
        },
      });

      if (!response.ok) {
        // 서버에서 에러 응답을 보낸 경우
        const errorData = await response.json();
        throw new Error(errorData.message || 'Failed to delete item');
      }
      // 성공 시, UI는 이미 업데이트되었으므로 추가 작업이 필요 없습니다.
      console.log(`Item ${idToDelete} successfully deleted.`);

    } catch (err) {
      // 2. Rollback: API 호출 실패 시 이전 상태로 되돌립니다.
      setError((err as Error).message);
      setItems(previousItems); // 이전 상태로 롤백합니다.
      console.error(`Failed to delete item ${idToDelete}:`, err);
      alert(`아이템 삭제 실패: ${(err as Error).message}`); // 사용자에게 알림
    }
  };

  if (loading) return <p>아이템 목록을 불러오는 중입니다...</p>;
  if (error) return <p style={{ color: 'red' }}>에러 발생: {error}</p>;
  if (items.length === 0) return <p>표시할 아이템이 없습니다.</p>;

  return (
    <div>
      <h1>나의 아이템 리스트</h1>
      <ul>
        {items.map(item => (
          <li key={item.id} style={{ display: 'flex', alignItems: 'center', marginBottom: '8px' }}>
            <span>{item.name}</span>
            <button 
              onClick={() => handleDeleteItem(item.id)} 
              style={{ marginLeft: '15px', padding: '5px 10px', cursor: 'pointer' }}
            >
              삭제
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default ItemList;
```

위 코드에서는 `handleDeleteItem` 함수 내에서 Optimistic UI 업데이트 로직이 핵심입니다.

1.  `previousItems`에 현재 `items` 상태를 저장하여 롤백에 대비합니다.
2.  `setItems`를 호출하여 즉시 해당 아이템을 UI에서 제거합니다.
3.  `fetch` API를 통해 서버에 삭제 요청을 보냅니다.
4.  요청이 성공하면 별다른 조치 없이 종료됩니다.
5.  요청이 실패하면 `catch` 블록에서 `setItems(previousItems)`를 호출하여 UI를 이전 상태로 되돌립니다. 이 과정에서 사용자에게 에러 메시지를 보여주는 것도 중요합니다.

## 실무 적용 팁

*   **React Query / SWR 활용**: 실제 프로덕션 환경에서는 `useState`만으로 모든 데이터 페칭 및 캐싱 로직을 관리하기 어렵습니다. React Query나 SWR 같은 데이터 페칭 라이브러리를 사용하면 Optimistic UI 업데이트, 재검증, 캐싱, 에러 핸들링을 훨씬 쉽게 관리할 수 있습니다. 특히 `useMutation` 훅의 `onMutate`, `onError`, `onSettled` 콜백을 활용하면 Optimistic UI 로직을 매우 깔끔하게 구현할 수 있습니다.
*   **불변성(Immutability) 유지**: React 상태를 업데이트할 때는 항상 불변성을 유지해야 합니다. 즉, 상태를 직접 변경하지 않고, 항상 새로운 배열이나 객체를 생성하여 업데이트해야 합니다. `filter`, `map`, `spread operator` 등을 적극적으로 활용하세요.
*   **견고한 에러 핸들링 및 롤백**: 네트워크 문제, 서버 오류 등 다양한 실패 상황에 대비하여 사용자에게 적절한 피드백(토스트 메시지, 경고창 등)을 제공하고, UI를 이전 상태로 롤백하는 로직을 견고하게 설계해야 합니다.
*   **로딩 상태 관리**: 삭제 요청 중임을 사용자에게 시각적으로 알리는 로딩 인디케이터를 추가하면 좋습니다. 예를 들어, 삭제 버튼을 비활성화하거나 스피너를 보여주어 중복 요청을 방지하고 사용자 경험을 향상시킬 수 있습니다.

## 마무리하며

좀비 행 문제는 React 애플리케이션에서 흔히 발생하는 문제이지만, Optimistic UI 업데이트와 견고한 상태 관리 전략을 통해 충분히 극복할 수 있습니다. 오늘 다룬 `useState` 기반의 접근 방식은 문제 해결의 기본 원리를 이해하는 데 도움이 되며, 실제 프로젝트에서는 React Query와 같은 전문 라이브러리를 활용하면 더욱 효율적으로 문제를 해결할 수 있습니다.

다음번에는 React Query를 활용하여 좀비 행 문제를 어떻게 더욱 우아하게 해결할 수 있는지에 대해 더 깊이 다뤄보겠습니다. 그때까지 여러분의 React 애플리케이션에 좀비가 나타나지 않기를 바랍니다! 🧟‍♀️🚫

---

## 참고 자료

*   [The Personal Branding Playbook Developers Don't Want to Admit They Need 😎](https://dev.to/thebitforge/the-personal-branding-playbook-developers-dont-want-to-admit-they-need-1p72) by TheBitForge
*   [I Built a 'Fourth Clover' for Writers: A Minimalist Next.js Blogging Platform 🍀](https://dev.to/aryan-dani/i-built-a-fourth-clover-for-writers-a-minimalist-nextjs-blogging-platform-oja) by Aryan Dani
*   [Is That Mole Dangerous? Build a Real-Time Skin Lesion Classifier with WebGPU and EfficientNetV2 🚀](https://dev.to/wellallytech/is-that-mole-dangerous-build-a-real-time-skin-lesion-classifier-with-webgpu-and-efficientnetv2-4f9i) by wellallyTech
*   [Solving the Zombie Row Problem: A Deep Dive into React State Management](https://dev.to/icraftcode/solving-the-zombie-row-problem-a-deep-dive-into-react-state-management-1618) by ICraftCode
*   [Open Source - Seeking Expert Feedback on React Component Library Updates](https://dev.to/madhavilosettyintel/open-source-seeking-expert-feedback-on-react-component-library-updates-m43) by Madhavi Losetty