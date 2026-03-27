---
title: 'VUE 表格元件進階-分頁'
description: '上一篇《VUE 表格元件進階－排序》有提到把表格做成公用的子元件，子元件本身有排序的功能，透過父元件傳入的資料設定表格的欄位，表格的內容也透過父元件傳入子元件進行排列。'
pubDate: '2025-03-28'
heroImage: '../../assets/blog-placeholder-1.jpg'
tags: ['JS', 'vue', 'table','pagination','前端']
---

上一篇《VUE 表格元件進階－排序》有提到把表格做成公用的子元件，子元件本身有排序的功能，透過父元件傳入的資料設定表格的欄位，表格的內容也透過父元件傳入子元件進行排列。也就是說：

- **父元件**負責資料
- **表格子元件**負責排列與排序功能

這一篇則加入**分頁功能**，把分頁元件當成表格元件的子元件，讓整體表格功能更完整。

---

## 元件職責說明

### 父元件
負責取得資料，並將資料傳給表格子元件。

### 表格元件
- 新增 `currentPage` 的 `ref` 變數，用來儲存使用者點擊的頁次，初始值為 `1`
- 新增 `perPage` 的 `ref` 變數，用來儲存每頁顯示的資料筆數
- 把資料總筆數傳遞給分頁元件
- 透過 `props` 把 `currentPage` 與 `perPage` 傳給分頁元件 (`pagination`)
- 當使用者點擊分頁元件上的頁次時，分頁元件會把 `currentPage` 透過 `emit` 傳回表格元件
- 表格元件再透過 `computed` 觀察 `currentPage` 的變化，計算目前分頁要呈現的資料並渲染到 DOM 上

### 分頁元件
- 取得表格元件傳入的 `currentPage`、`perPage` 與總筆數
- 使用者點擊頁次後，透過 `emit` 回傳目前點擊的頁次給表格元件
- 被點擊的頁碼應呈現 `active` 狀態
- 提供上一頁與下一頁功能

---

## Step 1：表格元件新增 `currentPage`、`perPage` 的 `ref` 變數

```js
// DataTable 元件
const perPage = ref(10);
const currentPage = ref(1);
```

---

## Step 2：表格元件傳遞 props 給分頁元件

表格元件把 `currentPage`、`perPage` 與資料總筆數傳遞給分頁元件，其中 `currentPage` 可以使用 `v-model` 雙向綁定。

```vue
<pagination
  :perPage="perPage"
  :totalRows="tempData.length"
  v-model="currentPage"
></pagination>
```

---

## Step 3：分頁元件接收表格元件傳入的 props

分頁元件透過 `props` 取得表格元件傳入的 `totalRows`、`perPage` 與 `currentPage`。  
因為 `currentPage` 是透過 `v-model` 傳入，所以分頁元件需用 `modelValue` 承接，並設定：

```js
defineEmits(['update:modelValue'])
```

當使用者點擊頁次時，再將值傳回表格元件。

```js
// Props
const props = defineProps({
  align: {
    type: String,
    default: 'left',
  },
  totalRows: {
    type: Number,
    default: 0,
  },
  perPage: {
    type: Number,
    default: 10,
  },
  modelValue: {
    type: Number,
    default: 1,
  },
});

// Emits
const emit = defineEmits(['update:modelValue']);

// Reactive state
const currentPage = ref(props.modelValue);
```

---

## Step 3-1：分頁元件的 DOM 佈局前，先建立輔助函式

以下是幾個確保數值為整數的 helper function，避免分頁計算出錯。

```js
// Constants
const DEFAULT_PER_PAGE = 10;
const DEFAULT_TOTAL_ROWS = 0;

// Helper functions
// 若 toInteger(value) 轉換後為 0、NaN、null、undefined，
// 則回傳 DEFAULT_PER_PAGE（預設每頁顯示數量）。
// 如果轉換後有數值，則使用該數值。
const sanitizePerPage = (value) =>
  // 確保 perPage 至少是 1，避免無效的 0 或負數
  Math.max(toInteger(value) || DEFAULT_PER_PAGE, 1);

// 若轉換後的數值為 0、NaN、null、undefined，
// 則使用 DEFAULT_TOTAL_ROWS（預設總筆數）。
// 若有正常數值，則使用該數值。
const sanitizeTotalRows = (value) =>
  // 確保 totalRows 至少是 0，因為總筆數不能是負數
  Math.max(toInteger(value) || DEFAULT_TOTAL_ROWS, 0);

const toInteger = (value, defaultValue = NaN) => {
  // 轉換為整數，忽略小數和非數字部分
  // 強制使用十進位，避免舊瀏覽器誤判 0 或 0x 為其他進位
  // 轉換失敗時回傳 NaN
  const integer = parseInt(value, 10);
  return isNaN(integer) ? defaultValue : integer;
};
```

---

## Step 3-2：利用 `computed` 算出總頁數

```js
// Computed properties 計算總頁數
const numberOfPages = computed(() => {
  const result = Math.ceil(
    sanitizeTotalRows(props.totalRows) / sanitizePerPage(props.perPage)
  );
  console.log(result);
  return result < 1 ? 1 : result;
});
```

---

## Step 3-3：在 DOM 上渲染分頁數與綁定事件

```vue
<nav aria-label="Page navigation example">
  <ul class="pagination pagination-sm" :class="justifyContent">
    <!-- 往前的箭號，如果 currentPage 為 1，則呈現 disabled 樣式 -->
    <li class="page-item" :class="currentPage === 1 ? 'disabled' : ''">
      <a class="page-link" @click="Previous" aria-label="Previous">
        <span aria-hidden="true">‹</span>
      </a>
    </li>

    <!-- 用 v-for 渲染出頁次 -->
    <li
      class="page-item"
      :class="currentPage === item ? 'active' : ''"
      v-for="item in numberOfPages"
      :key="item"
    >
      <!-- 如果頁次等於 1、最後頁、頁次小於等於 5，
           或目前頁次與 item 的差距小於 2，則顯示數字 -->
      <a
        v-if="
          Math.abs(currentPage - item) < 2 ||
          item === 1 ||
          item === numberOfPages ||
          numberOfPages <= 5
        "
        class="page-link"
        @click="onClick(item)"
      >
        {{ item }}
      </a>

      <a v-else-if="Math.abs(currentPage - item) === 2" class="page-link">
        ...
      </a>
    </li>

    <!-- 往後的箭號，如果 currentPage 為總頁數，則呈現 disabled 樣式 -->
    <li
      class="page-item"
      :class="currentPage === numberOfPages ? 'disabled' : ''"
    >
      <a class="page-link" @click="Next" aria-label="Next">
        <span aria-hidden="true">›</span>
      </a>
    </li>
  </ul>
</nav>
```

---

## Step 4：分頁元件點擊事件處理

這裡透過：

```js
emit('update:modelValue', currentPage.value)
```

把雙向綁定的 `currentPage` 傳回表格元件。

```js
// Methods

// 往前一頁
const Previous = () => {
  if (currentPage.value !== 1) {
    currentPage.value -= 1;
    emit('update:modelValue', currentPage.value);
  }
};

// 往後一頁
const Next = () => {
  if (currentPage.value !== numberOfPages.value) {
    currentPage.value += 1;
    emit('update:modelValue', currentPage.value);
  }
};

// 點擊分頁
const onClick = (pageNumber) => {
  console.log(pageNumber);
  if (pageNumber === currentPage.value) return;
  currentPage.value = pageNumber;
  emit('update:modelValue', currentPage.value);
};
```

---

## Step 5：表格元件觀察 `currentPage` 的改變，渲染當前分頁資料

```js
// 計算當前頁面的資料
const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * perPage.value;
  return tempData.value.slice(start, start + perPage.value);
});
```

---

## 補充：Vue 3.4 之後可以改用 `defineModel()`

在 Vue 3.4 之後，元件使用 `v-model` 時，可以不用再寫：

```js
const emit = defineEmits(['update:modelValue']);
```

而是直接使用 `defineModel()`，讓寫法更直覺：

```js
const currentPage = defineModel();
```

這樣在點擊事件中，也可以把：

```js
emit('update:modelValue', currentPage.value);
```

拿掉。

### 改寫後的事件處理

```js
// Methods

// 往前一頁
const Previous = () => {
  if (currentPage.value !== 1) {
    currentPage.value -= 1;
  }
};

// 往後一頁
const Next = () => {
  if (currentPage.value !== numberOfPages.value) {
    currentPage.value += 1;
  }
};

// 點擊分頁
const onClick = (pageNumber) => {
  console.log(pageNumber);
  if (pageNumber === currentPage.value) return;
  currentPage.value = pageNumber;
};
```

---

## 重點整理

- 父元件負責取得資料並傳遞給表格元件
- 表格元件負責資料顯示、排序與分頁資料切片
- 分頁元件負責頁碼顯示、上一頁 / 下一頁與頁碼切換
- 使用 `v-model` 可以簡化 `currentPage` 在元件間的雙向綁定
- Vue 3.4 之後可使用 `defineModel()` 讓寫法更簡潔

---

## 範例檔案
https://stackblitz.com/edit/vitejs-vite-yegibrrm?file=src%2Fcomponents%2Fpagination.vue
原文最後提到「範例檔案」，但未附上實際檔案內容，因此此整理僅涵蓋文章文字內容。
