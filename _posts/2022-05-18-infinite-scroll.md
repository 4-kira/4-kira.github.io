---
title: "오즈의 제작소: 무한 스크롤"
date: 2022-05-18 00:00:00 +0900
categories: [implementation, react]
tags: [react, infinite-scroll, SWR, intersection-observer]
---

> [Archive] 이 글은 2022년에 다른 플랫폼에서 작성한 글을 이전하며 재구성한 것입니다.  
> 내용 중 일부는 현재 기준과 맞지 않을 수 있습니다.

## 무한 스크롤

### IntersectionObserver 채택 이유

IntersectionObserver를 간편하게 사용할 수 있는 React Hook인 react-intersection-observer로 무한 스크롤을 구현.  
IntersectionObserver를 사용하는 방법이 일반적으로 Scroll event감지를 사용하는 방법보다 유리하다.

![velog](https://velog.velcdn.com/images/dltkdals224/post/ba44f314-b9e2-4f4b-a751-4ff1cc703c51/image.png)

Scroll event는 매우 자주 발생하고, 그 때마다 핸들러가 실행되기 때문에  
debounce, throttle(보통 throttle 또는 rAF 기반 제어)을 통해 호출 수를 제어하는 방법이 필수로 요구된다.

또한 Scroll event에서는 현재 창의 높이값을 조사하기 위해 offsetTop을 사용하는데,  
이는 강제 동기 레이아웃 계산(reflow / layout thrash)을 유발할 수 있다.  
레이아웃 계산 요청이 반복되면 reflow 비용이 누적되어 jank가 생길 수 있다.

IntersectionObserver를 사용하는 방법은 해당 문제들에 대한 고민을 덜어낼 수 있다.

<br/>

### IntersectionObserver 사용

[Intersectioin Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)에는 다음과 같이 정의되어 있다.

IntersectionObserver는 target element와 상위 element들의 viewport(현재 화면에 보이는 직사각형의 영역)가 교차되는 부분을 비동기적으로 관찰하는 API이다.

IntersectionObserver는 다음 4가지 상황에서 보통 사용한다.

- 페이지 스크롤로 이미지에 대한 lazy-loading을 구현할 때
- Infinite scrolling을 통해 스크롤로 새로운 컨텐츠를 불러올 때
- 광고의 수익을 계산하기 위해 광고의 가시성을 참고할 때
- 사용자가 결과를 볼 것인지에 따라 애니메이션 동작 여부를 결정할 때

<br/>

## 코드

```javascript
import { useEffect, useState } from 'react';
import { useInView } from 'react-intersection-observer';

// ... (HoneytipCard, PostListWrapper, InfiniteScrollCheck, useGetHoneytipList 등)

const MainMid = ({ step }) => {
  // ... (queryStep, sort, size, setCurrentFilter 등)

  /** viewport 진입 감지 */
  const [ref, inView] = useInView();

  /** Pagination + 누적 리스트 */
  const [page, setPage] = useState(1);
  const [honeytipList, setHoneytipList] = useState([]);

  /** SWR 기반 패칭 (page 변경 시 자동 재호출) */
  const { honeytipList: honeytipListFromServer } = useGetHoneytipList({
    step: queryStep ?? '',
    page,
    sort,
    size,
  });

  /** 필터(step/sort) 변경 시: 페이지/리스트 초기화 **/
  useEffect(() => {
    setHoneytipList([]);
    setPage(1);

    if (sort === 'id,desc') {
      setCurrentFilter('최신 순');
    }
  }, [step, sort]);


  /** "데이터 반영 + 다음 페이지 트리거"를 하나의 흐름으로 관리 **/
  useEffect(() => {
    if (honeytipListFromServer) {
      const next = honeytipListFromServer.content ?? [];

      setHoneytipList((prev) =>
        page === 1 ? next : [...prev, ...next]
      );
    }

    if (inView) {
      setPage((prev) => prev + 1);
    }
  }, [inView, page, honeytipListFromServer]);

  return (
    <>
      {/* ... */}

      <PostLis>
        {honeytipList.map((post) => (
          <HoneytipCard key={post.id} {...post} />
        ))}

        <InfiniteScrollCheck ref={ref} />
      </PostLis
      tWrapper>

      {/* ... */}
    </>
  );
};
```

<br/>

## 문제와 해결

### 스크롤 위치가 초기화되는 문제

초기 구현에서는 별도의 누적 배열 없이, useSWR로 받아온 데이터를 그대로 렌더링에 사용하였다.

이 방식에서는 sentinel(ref)이 viewport에 진입할 때마다 page 값이 변경되고,  
이에 따라 useSWR의 key가 바뀌면서 데이터가 다시 조회된다.

이때 useSWR은 새로운 key에 대해 기존 데이터를 이어 붙이는 것이 아니라, 첫 번째 인덱스부터 데이터를 다시 구성한다.  
그 결과, 렌더링 단계에서 리스트가 통째로 교체되며 스크롤이 상단으로 이동했다가 다시 제자리로 돌아오는 부자연스러운 깜박임이 발생하였다.

이를 해결하기 위해, 서버에서 내려받은 데이터를 그대로 사용하지 않고 렌더링 전용으로 사용할 honeytipList 배열을 별도로 두어 기존 데이터 뒤에 새로운 콘텐츠를 추가하는 방식으로 구현하였다.

<br/>

### step / sort 변경에서 상태 초기화 문제

해당 페이지에서는 카드(step) 변경과 콘텐츠 정렬(sort) 기능이 함께 존재하였다.

문제는 정렬이 적용된 상태에서 step이 변경될 경우,  
변경된 step에 대해 동일한 정렬 조건으로 다시 데이터를 요청하기 위해 추가적인 분기 로직이 필요해진다는 점이었다.

로직이 복잡해지는 것을 피하기 위해 step이 변경되는 경우에는 항상 page와 누적 리스트를 초기화하고,
정렬 기준 또한 기본값으로 되돌리는 방식으로 처리하였다.

```javascript
useEffect(() => {
  setHoneytipList([]);
  setPage(1);

  if (sort === 'id,desc') {
    setCurrentFilter('최신 순');
  }
}, [step, sort]);
```

이 effect에서 step과 sort 변경에 따른 모든 초기화 작업을 한 번에 처리하도록 구성하였다.

<br/>