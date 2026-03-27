---
title: '神機妙算-再談 computed'
description: '之前寫過一篇 computed 的筆記，總覺得只是建立一個心智表徵，知道 computed 怎麼使用，但實際的運作方式與限制卻是模模糊糊、不夠具體。因此，我重新研讀資料，以求對 `computed` 有更深入的理解。'
pubDate: '2024-10-24'
heroImage: '../../assets/blog-placeholder-1.jpg'
tags: ['JS','JavaScript', 'vue','vue3', 'computed','前端']
---

之前寫過一篇 `computed` 的筆記，總覺得只是建立一個心智表徵，知道 `computed` 怎麼使用，但實際的運作方式與限制卻是模模糊糊、不夠具體。因此，我重新研讀資料，以求對 `computed` 有更深入的理解。

## computed 是甚麼

`computed` 是 Vue 提供的計算方法，會觀察自己函式內響應式資料的變化，並返回一個值。就像一個臨時快照一般，當自己函式內的響應式資料發生變化，`computed` 就來拍一張快照，並把這張快照放在暫存裡面，所以 `computed` 是響應資料變化的結果，不能由結果回過頭去改變資料。

```vue
<template>
  <input v-model.number="firstNum" />
  <input v-model.number="secondNum" />
  {{ computdTotal }}
</template>

<script setup>
import { ref, computed } from "vue";
const firstNum = ref(1);
const secondNum = ref(2);

const computdTotal = computed(() => {  
  return firstNum.value + secondNum.value;
});
</script>
```

以上面的例子來說，就是 `firstNum` 或 `secondNum` 發生變化的時候，`computdTotal` 這個 `computed` 函式就會自動計算，並返回一個值。

## computed 的特色

- `computed` 會返回一個值（`RefImpl` 物件），並且在自己函式內的響應式資料發生變化時，重新計算之後再次傳值。
- `computed` 在元件的 `created` 生命週期就會建立並執行一次，之後則是觀察自己函式內的響應式資料有沒有變化，決定是否要重新計算。
- 自動追蹤所關注的響應式資料。
- 快取（cache）計算結果。
- 預設為 `getter` 唯讀特性，以符合單向資料流的設計。
- `computed` 只能在自己的函式中修改，沒有辦法用賦值的方式改變。以上例來說，沒辦法用 `computdTotal.value = 100` 來重新賦值，主控台會報錯。

## computed 的 getter 與 setter 與單向資料流

前面有提到 `computed` 是觀察自己函式內的響應式資料是否改變，才決定要不要重新計算，因此它是被動地讀取資料產生運算結果，不能反過來去覆寫本來觀察的響應式資料。所以 `computed` 預設只有 `getter` 屬性，只能取值，是唯讀的。Vue 官網有特別警告：

> 计算属性的 getter 应只做计算而没有任何其他的副作用，这一点非常重要，请务必牢记。举例来说，不要改变其他状态、在 getter 中做异步请求或者更改 DOM！一个计算属性的声明中描述的是如何根据其他值派生一个值。因此 getter 的职责应该仅为计算和返回该值。

以上例而言，讓我們 `console.log(computdTotal)` 這個 `computed`，可以看到它是一個 `ComputedRefImpl` 物件，並且有提供 `setter` 方法。

所以如果要寫回響應式資料，可以使用物件的 `setter` 方法，以下為 Vue 官網的範例：

```vue
<template>
  <input v-model="firstName">
  <input v-model="lastName">
  <input v-model="fullName">
</template>

<script setup>
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

const fullName = computed({
  // getter
  get() {
    console.log("2.getter");
    return firstName.value + ' ' + lastName.value
  },
  // setter
  set(newValue) {
    console.log("1.setter", fullName.value);
    // Note: we are using destructuring assignment syntax here.
    [firstName.value, lastName.value] = newValue.split(' ')
  }
})
</script>
```

以上例畫面結果而言，會錯以為是 `computed` 的結果 `fullName` 反過頭去改變了 `firstName.value` 與 `lastName.value`。

但是由下圖可以觀察到當我們改變 `fullName` 輸入框的值（`John Doe` → `John Doe6`）的時候，`console` 會先出現 `setter`，此時 `fullName` 還沒被改變，然後再出現 `getter`，所以可以推測這個過程是：

1. 修改 `fullName` 輸入框的資料之後，並不是透過 `setter` 去改變 `fullName`，而是透過 `setter` 帶入 `newValue`，改變了 `firstName.value` 與 `lastName.value`。
2. 接著 `computed` 觀察到是 `lastName.value` 這 1 個 `getter` 所關注的響應資料發生變化，才觸發了 `getter`，傳出了新的值，改變了 `fullName.value`。

所以即使是加上了 `setter`，`computed` 依然是透過所關注的響應式資料的改變來傳值，而非直接透過 `setter` 修改，符合 Vue 單向資料流的設計。

## 由快取（cache）來談 computed 與 method 的差別

就像開頭講的，`computed` 會在關注的響應式資料發生變化時，對於計算結果拍一張臨時快照，並把快照放在快取中。如果響應式資料沒有改變，那快照就不會重拍。

而且當響應式資料發生改變時，不管畫面上使用幾次，`computed` 只會執行一次，其他次都是使用放在快取中的快照。但是 `methods` 則是畫面上呼叫幾次就會執行幾次。

```vue
<template>
  <input v-model.number="firstNum" />
  <input v-model.number="secondNum" />
  <div>computed ： {{ computedTotal }}</div>
  <div>computed ： {{ computedTotal }}</div>
  <div>computed ： {{ computedTotal }}</div>
  <div>method : {{ methodCount() }}</div>
  <div>method : {{ methodCount() }}</div>
  <div>method : {{ methodCount() }}</div>
</template>

<script setup>
import { ref, computed } from "vue";
const firstNum = ref(1);
const secondNum = ref(2);

const computedTotal = computed(() => {
  console.log("computed");
  return firstNum.value + secondNum.value;
});

const methodCount = function () {
  console.log("method");
  return firstNum.value + secondNum.value;
};
</script>
```

`image.png`

畫面上 `computedTotal` 與 `methodCount()` 都呼叫了 3 次，然而 `computedTotal` 只執行了 1 次，`methodCount` 卻執行了 3 次。

在使用大量計算的情境中，所關注響應資料沒有更動的前提下，`computed` 不會重新計算求值，減少性能上的消耗。反之，若是有需要隨時更新、不希望暫存的情境，則可以使用 `method`。

## computed 的限制

- `computed` 不能用來處理非同步事件，也就是不能拿來打 API。
- `computed` 無法帶入參數。
- `computed` 傳出的值不能直接修改，應該修改其所關注的響應式資料，從而觸發 `computed` 的 `getter` 方法來改變 `computed` 的結果。
- 只有響應式資料（`ref` / `reactive`）才會被 `computed` 當作追蹤數據源。

## 結語

在 Vue.js 中，`computed` 屬性是強大且實用的工具，能有效提升應用的性能和程式碼可讀性。通過快取（cache）複雜的計算結果，我們可以避免不必要的重新運算，讓應用在處理大量資料時依然保持高效流暢。在實際開發中，適當地運用 `computed` 不僅能提高開發效率，還能使程式碼更易於維護與理解。希望這篇文章能幫助你更好地掌握 `computed` 的使用技巧，並運用於未來的專案中！

## 參考資料

- <https://ithelp.ithome.com.tw/articles/10274622>
- <https://ithelp.ithome.com.tw/articles/10275281>
- <https://n4563810.medium.com/筆記-vue-js-methods-v-s-computed-3459e5184d19>
- <https://medium.com/@yhosutun2491/vue-前端新手深入系列-vue-computed-計算屬性-4a20540be80e>


