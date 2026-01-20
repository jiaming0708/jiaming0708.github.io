---
title: 前端元件設計思維探討
date: 2026-01-20 13:32:20
updated: 2026-01-20 13:32:20
categories:
- Frontend
tags:
- Frontend
- Thinking
thumbnail: cover.jpg
---

一位同事發的 PR 是要調整 Ranking 的 render 邏輯，看完以後覺得可以紀錄一下雙方的思維差異。

<!-- more -->

## 原始版本

``` typescript
// useStartsRating.ts
const MAX_STAR_SLOTS = 5;

const calcStartNumber = (score: number) => {
  const normalizedScore = Number.isFinite(score) ? score : 0;
  return Math.max(0, Math.ceil(normalizedScore / 10));
};

/** 計算全星數量 */
export const getFullStarCount = (score: number): number => {
  const starCount = calcStartNumber(score);
  return Math.min(MAX_STAR_SLOTS, Math.floor(starCount / 2));
};

/** 是否顯示半星 */
export const showHalfStar = (score: number): boolean => {
  const starCount = calcStartNumber(score);
  return starCount % 2 === 1 && getFullStarCount(score) < MAX_STAR_SLOTS;
};

/** 空星（用灰色滿星表示）數量：滿分五顆星，可包含半星 */
export const getEmptyStarCount = (score: number): number => {
  const full = getFullStarCount(score);
  const half = showHalfStar(score) ? 1 : 0;
  return Math.max(0, MAX_STAR_SLOTS - full - half);
};
```

``` vue
<i-codicon-star-full
  v-for="num in getFullStarCount(i.score)"
  :key="`full-${num}`"
  class="text-lg text-c-orange-500"
/>
<i-codicon-star-half v-if="showHalfStar(i.score)" class="text-lg text-c-orange-500" />
<i-codicon-star-full
  v-for="num in getEmptyStarCount(i.score)"
  :key="`empty-${num}`"
  class="text-lg text-c-dark-400"
/>
```

> i-codicon-star-full 這幾個是 icon 元件

## 第二版

當我看到的時候，留了一句話給同事

> 要顯示幾顆星的計算應該是一樣，剩下的應該是看圖片要怎麼換，那這樣分空、半、滿 不會太重複嗎?

然後同事就說，他去思考一下，過一陣子就說調整一版了

``` typescript
const MAX_STAR_SLOTS = 5;
interface StarRating {
  full: number;
  half: boolean;
  empty: number;
}
export type StarIconType = 'full' | 'half' | 'empty';
/** 計算星星評分顯示（滿分五顆星，可包含半星） */
export const getStarRating = (score: number): StarRating => {
  const normalizedScore = Number.isFinite(score) ? score : 0;
  const starCount = Math.max(0, Math.ceil(normalizedScore / 10));
  const full = Math.min(MAX_STAR_SLOTS, Math.floor(starCount / 2));
  const half = starCount % 2 === 1 && full < MAX_STAR_SLOTS;
  const empty = Math.max(0, MAX_STAR_SLOTS - full - (half ? 1 : 0));
  return { full, half, empty };
};
/**
 * 產生要呈現的星星 icon 序列（先滿星、再半星、最後空星），方便在 template 用單一 v-for 顯示。
 */
export const getStarIcons = (score: number): StarIconType[] => {
  const { full, half, empty } = getStarRating(score);
  return [
    ...Array.from({ length: full }, () => 'full' as const),
    ...(half ? (['half'] as const) : []),
    ...Array.from({ length: empty }, () => 'empty' as const),
  ];
};
```

``` vue
<template v-for="(type, idx) in getStarIcons(i.score)" :key="`${type}-${idx}`">
  <i-codicon-star-half v-if="type === 'half'" class="text-lg text-c-orange-500" />
  <i-codicon-star-full
    v-else
    class="text-lg"
    :class="type === 'full' ? 'text-c-orange-500' : 'text-c-dark-400'"
  />
</template>
```

這邊先停下來討論一下這兩個版本的差異:
- 迴圈
  - 第一版是每個 icon 根據需要的數量去跑迴圈，寫了三個 for
  - 第二版是移到上層，只有一個 for 來跑完全部
- 計算每顆星的呈現
  - 第一版是每個類型分開計算
  - 第二版是合在一起計算

## 我的版本

我的設計概念是這樣，會有兩個元件
- 一個元件是 render 單顆星，由外面決定呈現的樣式 empty/half/full
- 另一個元件是 n 顆星，由外面決定 n (可以預設5)，然後給傳 score 當參數進去
  迴圈 i = 1...n
  如果 i < score 則 full
  如果 i > score 但不是整數則 half
  其餘 empty

完整的程式如下

### StarIcon.vue
``` vue
<script setup lang="ts">
interface Props {
  type: 'full' | 'half' | 'empty';
}

defineProps<Props>();
</script>

<template>
  <i-codicon-star-half v-if="type === 'half'" class="text-lg text-c-orange-500" />
  <i-codicon-star-full
    v-else
    class="text-lg"
    :class="type === 'full' ? 'text-c-orange-500' : 'text-c-dark-400'"
  />
</template>

```

### StarRating.vue

``` vue
<script setup lang="ts">
import StarIcon from './StarIcon.vue';

interface Props {
  score: number;
  maxStars?: number;
}

const props = withDefaults(defineProps<Props>(), {
  maxStars: 5,
});

type StarType = 'full' | 'half' | 'empty';

/** 根據當前位置和分數計算星星類型 */
const getStarType = (index: number): StarType => {
  // 將 score (0-100) 轉換為星星數 (0-maxStars)
  // score 100 = maxStars 顆星
  const starScore = (props.score / 100) * props.maxStars;

  if (index <= starScore) {
    return 'full';
  }

  // 檢查是否需要半星：當 index-1 < starScore < index
  if (index - 1 < starScore && starScore < index) {
    return 'half';
  }

  return 'empty';
};
</script>

<template>
  <StarIcon v-for="i in maxStars" :key="i" :type="getStarType(i)" />
</template>
```

### 思維差異

依照同事的思維，是將 `計算` 和 `render` 分開來，而我的思維則是這兩個可以放在一起，除非今天計算的部分有需要抽出去，不然這應該屬於元件的行為
另外一個思維，是我將元件的職責分開來，第一個元件只負責 render 星星，第二個元件只負責計算要怎麼呈現

這邊不確定是不是框架所延伸的思維差異，我之前是寫 angular 轉 react，在 react 的世界中就很多會這樣拆元件，在 angular 就比較不會這樣拆
因為我沒有真正的 vue 實戰，也無法確認這件事情

### AI 優化版本

請 AI 分析原始版本和我的版本差異，有提到計算是當下呼叫方法，當量大的時候會容易有效能的問題，因此請 AI 幫忙改一個版本

> Copilot 搭配 Claude Sonnet 4.5

``` vue
<script setup lang="ts">
import { computed } from 'vue';
import StarIcon from './StarIcon.vue';

interface Props {
  score: number;
  maxStars?: number;
}

const props = withDefaults(defineProps<Props>(), {
  maxStars: 5,
});

type StarType = 'full' | 'half' | 'empty';

/** 計算所有星星的類型（只計算一次，結果會被快取） */
const starTypes = computed((): StarType[] => {
  // 將 score (0-100) 轉換為星星數 (0-maxStars)
  const starScore = (props.score / 100) * props.maxStars;

  return Array.from({ length: props.maxStars }, (_, index) => {
    const position = index + 1; // 星星位置從 1 開始

    if (position <= starScore) {
      return 'full';
    }

    // 檢查是否需要半星：當 position-1 < starScore < position
    if (position - 1 < starScore && starScore < position) {
      return 'half';
    }

    return 'empty';
  });
});
</script>

<template>
  <StarIcon v-for="(type, index) in starTypes" :key="index" :type="type" />
</template>
```

這是 AI 說的原理，可以在量大的時候發揮更好的效能

```
// 當 score 或 maxStars 改變時：
props.score = 85  →  starTypes 重新計算  →  ['full', 'full', 'full', 'full', 'half']
                 ↓
                快取這個結果
                 ↓
// 後續渲染直接使用快取結果，不再重新計算
render 1 → 使用快取 ⚡
render 2 → 使用快取 ⚡
render 3 → 使用快取 ⚡
```
