---
title: 'React Hooks 完整指南：從入門到進階'
description: '深入了解 React Hooks 的原理與最佳實踐，包含自訂 Hook 的設計模式'
pubDate: '2024-01-25'
heroImage: '../../assets/blog-placeholder-5.jpg'
tags: ['React', '前端', 'Hooks']
---

React Hooks 自 React 16.8 引入以來，徹底改變了我們撰寫 React 元件的方式。本文將從基礎到進階，帶你全面了解 Hooks 的使用。

## 基礎 Hooks

### useState

最基本的狀態管理 Hook：

```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>目前數字：{count}</p>
      <button onClick={() => setCount(prev => prev + 1)}>+1</button>
      <button onClick={() => setCount(prev => prev - 1)}>-1</button>
    </div>
  );
}
```

**重要：** 當新狀態依賴舊狀態時，使用函式形式的更新器（`prev => prev + 1`），避免 stale closure 問題。

### useEffect

處理副作用（Side Effects）：

```tsx
import { useEffect, useState } from 'react';

function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  
  useEffect(() => {
    let cancelled = false;
    
    async function fetchUser() {
      const data = await api.getUser(userId);
      if (!cancelled) {
        setUser(data);
      }
    }
    
    fetchUser();
    
    // 清理函式：避免競態條件（Race Condition）
    return () => { cancelled = true; };
  }, [userId]); // 依賴陣列
  
  return <div>{user?.name ?? '載入中...'}</div>;
}
```

### useReducer

複雜狀態邏輯的最佳選擇：

```tsx
type Action = 
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset' };

function reducer(state: number, action: Action): number {
  switch (action.type) {
    case 'increment': return state + 1;
    case 'decrement': return state - 1;
    case 'reset': return 0;
    default: return state;
  }
}

function Counter() {
  const [count, dispatch] = useReducer(reducer, 0);
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>重置</button>
    </div>
  );
}
```

## 效能優化 Hooks

### useMemo

快取計算結果：

```tsx
import { useMemo } from 'react';

function ProductList({ products, filterText }: Props) {
  // 只有當 products 或 filterText 改變時才重新計算
  const filteredProducts = useMemo(
    () => products.filter(p => p.name.includes(filterText)),
    [products, filterText]
  );
  
  return <ul>{filteredProducts.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

### useCallback

快取函式參考：

```tsx
import { useCallback } from 'react';

function Parent() {
  const [count, setCount] = useState(0);
  
  // 如果不使用 useCallback，每次渲染都會產生新的函式
  const handleClick = useCallback(() => {
    console.log('clicked!');
  }, []); // 空依賴陣列 = 函式永遠不變
  
  return <Child onClick={handleClick} />;
}
```

## 自訂 Hook

自訂 Hook 是抽取元件邏輯的最佳方式：

### useLocalStorage

```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue] as const;
}

// 使用方式
function App() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  return <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
    切換主題
  </button>;
}
```

### useFetch

```tsx
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const controller = new AbortController();
    
    async function fetchData() {
      try {
        setLoading(true);
        const res = await fetch(url, { signal: controller.signal });
        if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
        const json = await res.json();
        setData(json);
      } catch (err) {
        if (err instanceof Error && err.name !== 'AbortError') {
          setError(err);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchData();
    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}
```

## Hook 的規則

記住 Hook 的兩個基本規則：

1. **只在最頂層呼叫 Hook** - 不要在條件判斷、迴圈或巢狀函式中呼叫 Hook
2. **只在 React 函式中呼叫 Hook** - 只在 React 函式元件或自訂 Hook 中呼叫

## 結語

Hooks 是 React 中非常強大的功能，善用它們可以讓你的元件程式碼更簡潔、更易測試。最重要的是理解每個 Hook 的設計意圖，才能在正確的場景使用它們。

如果你有任何問題或有趣的自訂 Hook 想分享，歡迎留言討論！
