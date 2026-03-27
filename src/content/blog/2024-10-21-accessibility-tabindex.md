---
title: '友善的鍵盤–tabindex'
description: '在 WCAG 2.0 及 2.1 中，提及「頁面上的所有功能都應該可透過鍵盤操作。」主要是用 `Tab` 鍵來上下選取例如 `input`、`button` 及超連結等元素，並且在鍵盤聚焦時，要提供永久可視且具明顯的焦點標示（AA）。'
pubDate: '2024-10-21'
heroImage: '../../assets/blog-placeholder-1.jpg'
tags: ['JS','JavaScript', 'vue','vue3', 'computed','前端']
---

## 使用 `Tab` 鍵的操作方法如下

- `Tab`：正向選取對焦。
- `Shift + Tab`：反向選取對焦。
- 使用方向鍵可以在 `select` 元素中切換選項。

首先要知道的是：原生的 `<a>` 連結、`<button>` 按鈕及 `<input>` 輸入，預設就是可以依據 HTML DOM 的結構順序由上而下、由左而右，使用鍵盤 `Tab` 鍵來依序聚焦。

至於其他元素則需要使用 `tabindex` 這個標籤來設置，就讓我來整理一下 `tabindex` 如何使用。

## `tabindex` 是什麼

`tabindex` 屬性用來指定元素是否可以使用鍵盤取得聚焦（focus）的狀態，同時指定使用者按 `Tab` 鍵的時候，DOM 元素聚焦的順序。

## 使用方法

```html
<input id="my-input" tabindex="n" />

<div class="my-class" tabindex="n">myDiv</div>
```

`tabindex="n"` 的 `n` 可以是 `-1`、`0` 或大於 `0` 的整數。

- 當 `tabindex="-1"` 的時候，不能使用鍵盤 `Tab` 鍵來聚焦，但是可以用 JavaScript 的 `focus()` 方法來聚焦。
- 當 `tabindex="0"` 的時候，可以使用 `Tab` 鍵依照 DOM 元素的排列順序取得焦點（focus）。
- 當 `tabindex="1"` 或大於 `1` 的時候，可以依照數字順序使用 `Tab` 鍵聚焦，但是並不建議使用這種方式。應該讓能獲得鍵盤焦點的原生元素，自然地依照 DOM 的排列順序排列。螢幕閱讀器會依據 DOM 結構生成內部的資料結構，其結構會與 DOM 中元素的排列順序相同，並按照這個順序進行導航。為了確保鍵盤的 `Tab` 鍵導航與螢幕閱讀器導航一致，應避免使用 `tabindex` 值大於 `0` 的設定，而是將它們按操作順序適當地排列在 DOM 中。

## `tabindex="-1"`

<iframe height="300" style="width: 100%;" scrolling="no" title="tabindex-順序" src="https://codepen.io/popeye_ux/embed/GRVvxNv?default-tab=" frameborder="no" loading="lazy" allowtransparency="true">
  See the Pen <a href="https://codepen.io/popeye_ux/pen/GRVvxNv">
  tabindex-順序</a> by POPEYE (<a href="https://codepen.io/popeye_ux">@popeye_ux</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

## `tabindex="0"`

<iframe height="300" style="width: 100%;" scrolling="no" title="tabindex-順序" src="https://codepen.io/popeye_ux/embed/GRVvxNv?default-tab=" frameborder="no" loading="lazy" allowtransparency="true">
  See the Pen <a href="https://codepen.io/popeye_ux/pen/GRVvxNv">
  tabindex-順序</a> by POPEYE (<a href="https://codepen.io/popeye_ux">@popeye_ux</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

## `tabindex > 1`

<iframe height="300" style="width: 100%;" scrolling="no" title="tabindex-順序" src="https://codepen.io/popeye_ux/embed/GRVvxNv?default-tab=" frameborder="no" loading="lazy" allowtransparency="true">
  See the Pen <a href="https://codepen.io/popeye_ux/pen/GRVvxNv">
  tabindex-順序</a> by POPEYE (<a href="https://codepen.io/popeye_ux">@popeye_ux</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

原生 `input` 依照 DOM 元素順序聚焦，如果中間插入一個 `tabindex="0"` 的 `div`，也會照規矩依照順序聚焦。

## `overflow` 為 `auto` 或 `scroll` 的情形

還有一種情形為 `div` 元素的 CSS 設定 `overflow` 為 `auto` 或 `scroll`，這種狀況會產生卷軸。如果無法取得焦點，則無法使用鍵盤的空白鍵來滾動卷軸，這時可以在 `div` 元素上設定 `tabindex="0"`，使其依據 DOM 元素順序取得焦點，取得焦點後就可以使用空白鍵操作卷軸。

## 結論

最後，UI 視覺上的順序應該與語法（`tabindex`）的設定順序一致，並且謹記不要在非互動元素上加上 `tabindex="0"` 或 `tabindex="1"`，以避免螢幕閱讀器無法區分互動元素及非互動元素，而造成使用者的困擾。

## 參考資料

- https://medium.com/a11yvillage/巧用-tabindex-讓你的-ui-更鍵盤友善-ad3d087e26fe
- https://bettertop.wordpress.com/2019/05/11/tabindex0與-1．鍵盤與網頁元素的無障礙距離/


