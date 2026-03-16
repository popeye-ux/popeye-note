---
title: '前端效能優化：從 LCP 到 CLS 的完整指南'
description: '深入解析 Core Web Vitals 三大指標，並提供具體的優化策略與實作方法'
pubDate: '2024-02-20'
heroImage: '../../assets/blog-placeholder-3.jpg'
tags: ['效能', '前端', 'SEO']
---

網站效能直接影響使用者體驗和 SEO 排名。Google 的 Core Web Vitals 是目前最重要的效能指標，包含三個核心指標：LCP、INP 和 CLS。本文將深入解析這三個指標，並提供具體的優化方法。

## Core Web Vitals 三大指標

### 1. LCP（Largest Contentful Paint）

LCP 衡量頁面主要內容的載入速度，目標是在 **2.5 秒內**完成。

**常見的 LCP 元素：**
- 大型圖片（`<img>`、`<image>`）
- 背景圖片
- 大型文字區塊
- 影片縮圖

**優化策略：**

```html
<!-- 使用 fetchpriority 提高關鍵圖片的載入優先級 -->
<img src="hero.jpg" fetchpriority="high" alt="Hero Image" />

<!-- 預載入關鍵資源 -->
<link rel="preload" as="image" href="hero.jpg" />
```

### 2. INP（Interaction to Next Paint）

INP 於 2024 年 3 月取代 FID，衡量所有使用者互動的響應速度，目標是 **200ms 以內**。

**優化長任務（Long Tasks）：**

```javascript
// 將長任務分割為較小的片段
async function processLargeData(items) {
  const CHUNK_SIZE = 50;
  
  for (let i = 0; i < items.length; i += CHUNK_SIZE) {
    const chunk = items.slice(i, i + CHUNK_SIZE);
    processChunk(chunk);
    
    // 讓瀏覽器有機會處理其他任務
    await new Promise(resolve => setTimeout(resolve, 0));
  }
}
```

### 3. CLS（Cumulative Layout Shift）

CLS 衡量視覺穩定性，目標是分數低於 **0.1**。版面位移通常由以下原因造成：

- 圖片沒有設定尺寸
- 廣告動態插入
- 字型載入造成的 FOUT

**修復方法：**

```css
/* 永遠為圖片設定寬高 */
img {
  width: 100%;
  aspect-ratio: 16 / 9;
}

/* 使用 font-display: optional 避免字型替換 */
@font-face {
  font-family: 'MyFont';
  src: url('myfont.woff2') format('woff2');
  font-display: optional;
}
```

## 實用優化技巧

### 圖片優化

```html
<!-- 使用現代圖片格式 -->
<picture>
  <source srcset="image.avif" type="image/avif" />
  <source srcset="image.webp" type="image/webp" />
  <img src="image.jpg" alt="描述" width="800" height="600" loading="lazy" />
</picture>
```

### 程式碼分割

```javascript
// React 懶載入元件
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyComponent />
    </Suspense>
  );
}
```

### 快取策略

善用 HTTP 快取頭可以大幅減少重複請求：

```
# 靜態資源 - 長時間快取
Cache-Control: public, max-age=31536000, immutable

# HTML 頁面 - 短時間快取
Cache-Control: public, max-age=0, must-revalidate
```

## 量測工具

1. **Lighthouse** - Chrome DevTools 內建，適合開發時的本地測試
2. **PageSpeed Insights** - Google 提供，同時顯示實際使用者資料
3. **WebPageTest** - 詳細的效能分析，支援全球多個測試節點
4. **Chrome UX Report** - 真實使用者的效能資料

## 結語

效能優化是一個持續的過程，沒有一勞永逸的解法。建議養成定期量測的習慣，在每次迭代後確認效能沒有退化。

記住：**你無法改善你無法量測的事物**。先量測，再優化，最後驗證成果。
