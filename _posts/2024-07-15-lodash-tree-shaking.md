---
title: "lodash 라이브러리로 확인하는 tree-shaking"
date: 2024-07-15 00:00:00 +0900
categories: [implementation, react]
tags: [tree-shaking]
---

> [Archive] 이 글은 2024년에 다른 플랫폼에서 작성한 글을 이전하며 재구성한 것입니다.  
> 내용 중 일부는 현재 기준과 맞지 않을 수 있습니다.

## tree-shaking

JavaScript 맥락에서 죽은 코드 제거를 설명하기 위해 일반적으로 사용되는 용어.

import 및 export 문을 사용하여, JavaScript 파일에서 사용하기 위한 코드 모듈을 내보내고 가져오는 것을 감지한다.

최신 JavaScript 애플리케이션에서는 모듈 번들러(ex. webpack 또는 Rollup)를 사용하여  
여러 JavaScript 파일을 단일 파일로 번들링할 때 죽은 코드를 자동으로 제거한다.  
이는 프로덕션에 바로 사용할 수 있는 코드 생성에 중요한 작업으로, **최소한의 파일 크기로 코드를 최적화**하여 성능을 향상시킨다.

정적 분석과 코드 제거, 의존성 분석을 통해 이루어진다.  
정적 분석을 통해 사용되지 않는 코드를 식별하고, 의존성 분석을 통해 각 모듈의 필요성을 평가하여 최종 번들에서 불필요한 코드를 제거.

이어서 lodash 라이브러리를 기반으로 tree-shaking에 대해서 알아본다.  
사용된 라이브러리는 다음과 같다.

```shell
npm i lodash lodash-es
npm i -D @types/lodash rollup-plugin-visualizer
```

- rollup-plugin-visualizer: 번들 트리맵 + gzip/brotli 사이즈 확인
- babel-plugin-lodash: 빌드 단계에서 lodash import를 메서드 단위로 변환(자동화 실험)

<br/>

### 1) tree-shaking이 깨지기 쉬운 import 패턴

**큰 라이브러리에서 임의 유틸을 사용하는 패턴**은 번들러의 정적 분석이 막히는 순간, 번들에 불필요한 용량이 포함될 수 있다.  
lodash는 이 사례를 설명하기 좋은 대표 라이브러리이다.

```typescript
import { debounce, uniqBy, cloneDeep } from "lodash";

export function runLodashLab(input: Array<{ id: number; name: string }>) {
  // 실제 호출을 만들어 번들러가 제거하지 못하도록 고정
  const debounced = debounce(() => {
  }, 200);

  debounced();

  const copied = cloneDeep(input);
  return uniqBy(copied, "id");
}
```

![](/assets/img/posts/2024/lodash-1.png)

<br/>

### 2) (코드 레벨) path import로 엔트리 축소

가장 간단한 해결책은 번들러에게 트리 쉐이킹을 맡기기 전에, **작은 엔트리만 가져오도록 import를 바꾸는 방식**이다.

lodash에서 `{ debounce } from "lodash"` 같은 네임드 import는 편하지만, 번들러 입장에서는 분석이 불리할 수 있다.

```typescript
import debounce from "lodash/debounce";
import uniqBy from "lodash/uniqBy";
import cloneDeep from "lodash/cloneDeep";

export function runLodashLab(input: Array<{ id: number; name: string }>) {
  const debounced = debounce(() => {
  }, 200);

  debounced();

  const copied = cloneDeep(input);
  return uniqBy(copied, "id");
}
```

![](/assets/img/posts/2024/lodash-2.png)

하지만 이 접근은 장점이 명확한 대신, 코드베이스가 커질수록 수동 변경/관리 비용이 커진다는 단점이 있다.

<br/>

### 3) (빌드 레벨) babel-plugin-lodash로 자동화

**코드는 그대로 유지하면서도 번들 입력을 최적화하는 자동화**도 가능하다.

babel-plugin-lodash는 소스 코드의 lodash import를 빌드 단계에서 메서드 단위 import로 변환해준다.  
개발자는 여전히 `import { debounce } from "lodash"` 같은 직관적인 형태로 코딩하지만,  
빌드 결과물은 `import debounce from "lodash/debounce"`처럼 쪼개진 형태로 생성된다.

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import { visualizer } from "rollup-plugin-visualizer";

export default defineConfig({
  plugins: [
    react({
      babel: true  // babel 플러그인 활성화
        ? {
            plugins: ["lodash"],
          }
        : undefined,
    }),
    // ...
  ],
  build: {
    sourcemap: true,
  },
});
```

![](/assets/img/posts/2024/lodash-3.png)

트리 쉐이킹 자체는 번들러가 수행하지만,  
개발자는 번들러가 분석하기 쉬운 형태의 코드를 만들어주는 규칙을 파이프라인에 추가해 결과를 바꿀 수 있다.

<br/>

## 패키지 교체에 따른 효과 비교

### lodash-es는 왜 다른 결과를 보이는가

lodash-es는 ESM 형태로 배포되는 lodash 빌드로, 번들러가 트리 쉐이킹을 적용하기 쉬운 조건을 제공한다.  
이 접근은 개발자가 트리 쉐이킹을 수행한 것이 아닌, 트리 쉐이킹 친화적인 배포물을 선택한 구조.

ESM export/import로 구성되어 번들러가 모듈 그래프를 정적으로 분석하기 쉽다.  
사용한 함수만 남기고 나머지를 제거하는 최적화가 성립할 가능성이 커진다.

```typescript
import { debounce, uniqBy, cloneDeep } from "lodash-es";

export function runLodashLab(input: Array<{ id: number; name: string }>) {
  const debounced = debounce(() => {
  }, 200);

  debounced();

  const copied = cloneDeep(input);
  return uniqBy(copied, "id");
}
```

![](/assets/img/posts/2024/lodash-4.png)

기존에 lodash를 쓰고 있고 번들 이슈가 있다면, babel-plugin-lodash(자동 변환) 또는 path import로 먼저 수렴시킨다.  
lodash 루트 import가 섞이거나(팀 규모/리뷰 한계) 실수가 잦다면, lodash-es로 패키지 교체가 “안전장치”가 될 수 있다.

<br/>

## 결론

gzip 기준으로 봤을 때 최적 결과물과의 차이는 20~30KB 수준으로, 단일 페이지의 체감 성능에 큰 영향을 주기는 어렵다.

그럼에도 이 실험의 가치는 “얼마나 빨라졌는가”보다 번들 크기가 커질 수 있는 경로를 어떻게 통제할 수 있는가를 구체적인 사례로 확인했다는 점에 있다. 이러한 최적화는 하나만 놓고 보면 미미해 보이지만, 의존성이 늘어나고 코드베이스가 성장할수록 누적 효과를 통해 의미 있는 차이를 만들어낸다.

<br/>