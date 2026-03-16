---
title: 'TypeScript 實用技巧：讓你的程式碼更安全'
description: '分享 10 個在實際專案中非常有用的 TypeScript 技巧，幫你寫出更健壯的程式碼'
pubDate: '2024-03-05'
heroImage: '../../assets/blog-placeholder-2.jpg'
tags: ['TypeScript', '程式設計', '進階']
---

TypeScript 是目前最受歡迎的程式語言之一，它為 JavaScript 加入了靜態型別系統，幫助我們在開發階段就發現潛在的錯誤。今天分享幾個我在實際專案中非常常用的 TypeScript 技巧。

## 1. 使用 `satisfies` 運算子

TypeScript 4.9 引入了 `satisfies` 運算子，它讓你能同時確保型別正確，又保留更精確的推斷型別：

```typescript
type Color = 'red' | 'green' | 'blue';
type Palette = Record<string, Color | [number, number, number]>;

const palette = {
  red: [255, 0, 0],
  green: '#00ff00',
  blue: [0, 0, 255],
} satisfies Palette;

// palette.red 的型別是 [number, number, number]，而不是 Color | [number, number, number]
palette.red.at(0); // ✅ 可以正常調用陣列方法
```

## 2. 善用泛型約束

泛型約束能讓你的函式更靈活，同時保持型別安全：

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: '小明', age: 25, email: 'ming@example.com' };
const name = getProperty(user, 'name');  // 型別為 string
const age = getProperty(user, 'age');    // 型別為 number
// getProperty(user, 'phone'); // ❌ 編譯錯誤！
```

## 3. 使用 Template Literal Types

Template Literal Types 讓你能夠建立基於字串的複雜型別：

```typescript
type EventName = 'click' | 'focus' | 'blur';
type EventHandler = `on${Capitalize<EventName>}`;
// 型別為 "onClick" | "onFocus" | "onBlur"

type ApiEndpoint = 'users' | 'posts' | 'comments';
type GetEndpoint = `/api/${ApiEndpoint}`;
// 型別為 "/api/users" | "/api/posts" | "/api/comments"
```

## 4. 使用 Discriminated Unions

當你有多種不同狀態時，Discriminated Unions 是最佳選擇：

```typescript
type LoadingState = { status: 'loading' };
type SuccessState = { status: 'success'; data: User[] };
type ErrorState = { status: 'error'; error: string };

type FetchState = LoadingState | SuccessState | ErrorState;

function renderState(state: FetchState) {
  switch (state.status) {
    case 'loading':
      return '載入中...';
    case 'success':
      return `共 ${state.data.length} 筆資料`;  // TypeScript 知道這裡有 data
    case 'error':
      return `錯誤：${state.error}`;              // TypeScript 知道這裡有 error
  }
}
```

## 5. 使用 `infer` 關鍵字

`infer` 讓你能夠在條件型別中推斷型別：

```typescript
type UnpackPromise<T> = T extends Promise<infer U> ? U : T;

type Result = UnpackPromise<Promise<string>>;  // string
type Same = UnpackPromise<number>;             // number

// 取得函式的回傳型別
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
```

## 6. Readonly 與 DeepReadonly

保護你的資料不被意外修改：

```typescript
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

const config: DeepReadonly<{
  api: { baseUrl: string; timeout: number };
}> = {
  api: { baseUrl: 'https://api.example.com', timeout: 5000 },
};

// config.api.baseUrl = 'other'; // ❌ 無法修改
```

## 7. 使用型別守衛（Type Guards）

型別守衛讓你能夠在執行時期安全地縮小型別範圍：

```typescript
function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'name' in obj &&
    'email' in obj
  );
}

function processData(data: unknown) {
  if (isUser(data)) {
    console.log(data.name);  // TypeScript 知道 data 是 User
  }
}
```

## 結語

TypeScript 提供了非常豐富的型別系統，善用這些功能可以讓你的程式碼更安全、更易維護。最重要的是，這些技巧都能在開發階段就幫你發現潛在的錯誤，省去大量的除錯時間。

希望這些技巧對你有所幫助！如果有任何問題，歡迎在下方留言討論。
