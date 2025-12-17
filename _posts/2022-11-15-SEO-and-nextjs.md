---
title: "SEO를 위한 Next.js"
date: 2022-11-15 00:00:00 +0900
categories: [implementation, nextjs]
tags: [SEO, nextjs]
---

> [Archive] 이 글은 2022년에 다른 플랫폼에서 작성한 글을 이전한 것입니다.  
> 내용 중 일부는 현재 기준과 맞지 않을 수 있습니다.

## SEO

비즈니스 웹 사이트 소유주들은 검색 과정에서 자신의 사이트가 찾기 쉬운 위치에 놓이길 원한다.  
방문자 수는 이내 수익과 직결되는 사항이기 때문.

SEO(Search Engin Optimization)는 웹 사이트가 검색엔진 상위에 노출될 수 있도록 최적화하는 과정.

본 글은 프로젝트 과정에서 SEO와 관련한 작업을 거치며 읽은 문서들과 경험한 내용을 정리한 글.

SEO를 이루는 데에는 기획을 포함하는 전반적인 관리가 필요하며,
개발자 관점에서 SEO를 다루는 내용을 포함한다.

<br/>

### 1) Google SEO

![](/assets/img/posts/2022/after-quarter/google.png)

SEO 진행 과정에 있어 최우선으로 타겟이 되는 Google 검색엔진을 기준으로 한다.

2021년 기준, 대한민국의 경우는 네이버가 가장 높은 검색 점유율을 보유 중.  
구글 검색엔진에 대한 이해가 뒷받침된다면 기타 검색엔진에서도 유효한 효과를 기대할 수 있다.

검색엔진별로 SEO 점수를 측정하는 기준은 약간의 차이가 있다.  
Google을 포함한 대부분은 검색엔진은 다음의 단계를 차례대로 수행한다.

> 크롤링

검색엔진 로봇이 있는 웹에 대하여 데이터를 수집해가는 과정.

> 인덱싱

구글봇이 발견한 URL을 분석하고 가공하여 자사 DB에 저장.

> 랭킹

URL을 평가하여 사용자 검색어와 가장 관련성이 높은 페이지를 결정하고 SERP(Search Engin Result Page)로 구성.

다음은 SEO 점수 중요도의 배점과 관련한 표.  
키워드 관련성과 검색엔진 제어를 나누어 점수를 측정한다.

**Keyword Relevancy**

![](/assets/img/posts/2022/after-quarter/google-keyword-1.png)
![](/assets/img/posts/2022/after-quarter/google-keyword-2.png)

**Search Engine Control**

![](/assets/img/posts/2022/after-quarter/google-control.png)

<br/>

### 2) SEO 최적화 과정

#### - 페이지 속도 개선

배점표에 존재하지 않지만, Google이 SEO 점수 측정에 고려하는 주요 요소 중 하나는 페이지 렌더링 시간.
[구글 페이지 속도 테스트 도구](https://pagespeed.web.dev/?utm_source=psi&utm_medium=redirect)를 통해 모바일 환경과 데스크톱 환경을 나눠 속도를 수치로 확인할 수 있다.  

![](/assets/img/posts/2022/after-quarter/pagespeed.png)

제작 과정 중 다음의 사항들을 신경 쓰면 추후 수정할 부분이 적어질 것.

- 이미지의 크기를 적절히 조절하기
- script 무게 줄이기
- 데이터 캐싱 처리
- 리디렉션 최소화

#### - Semantic Markup기반 메타 태그 최적화

<div class="image-row">
<img src="/assets/img/posts/2022/after-quarter/meta-tag-1.png">
<img src="/assets/img/posts/2022/after-quarter/meta-tag-2.png">
</div>

```html
<header>, <footer>, <nav>
<main>, <article>, <section>, <aside> 
<ul>, <li>

<title>, <h1>, <h2>, <h3>, <h4>, <strong>, <a>
'alt' attribute사용 권장
'hidden' attribute사용 지양.
```

Google SEO 점수 배점표에 명시되어있지 않은 HTML Tag 들이 SEO 점수 측정에 영향을 미칠 수 있다.

```html
<meta>
```

페이지가 담고 있는 정보가 아닌 웹페이지 자체의 정보를 명시하기 위해 사용되는 HTML 태그.  
head 아래 배치되어 일반 사용자가 마주하는 웹페이지의 콘텐츠에는 아무 영향을 주지 않는다.

이중 OG(Open Graph)는 meta 태그의 한 종류로, URL로 공유되는 콘텐츠가 '보여지는 방식'을 결정.  
Facebook, KakaoTalk, Slack, Twitter 등의 SNS에서 공유될 때 동작한다.

![](/assets/img/posts/2022/after-quarter/open-graph.png)

OG 태그의 종류는 다양하지만, 다음의 태그들을 주로 사용한다.

```html
<meta property="og:title" content="땃쥐 1시간"/>
<meta property="og:description" content="땃쥐가 1시간동안 먹방을 합니다."/>
<meta property="og:site_name" content="Youtube"/>
<meta property="og:type" content="website"/>
<meta property="og:image" content="https://i.ytimg.com/vi/2IMabIUHnMQ/hqdefault.jpg?s…RhlIFAoQjAP&rs=AOn4CLDxmWtra4xrEZ-VNSu5CckM0tv9Mw"/>
<meta property="og:url" content="https://www.youtube.com/watch?v=2IMabIUHnMQ&t=65s"/>
```

#### - robot.txt

robot.txt는 웹 크롤러에게 교통 표지판 같은 역할을 수행한다.

크롤링 봇에게 웹 사이트 내 접근하지 말아야 할 부분을 알려 크롤링 트래픽을 차단해 성능 이슈의 발생을 방지하거나 페이지, 동영상, 기타 파일 등의 관계에 대한 정보를 제공하는 사이트맵 위치 같은 중요 정보를 포함한다.

[robot.txt 소개 및 가이드](https://developers.google.com/search/docs/crawling-indexing/robots/intro)

![](/assets/img/posts/2022/after-quarter/robot-txt.png)

robots.txt를 구성하는 요소는 크게 네 가지인데,  
"User-agent" 요소는 필수이며 나머지는 옵션이다.

- User-agent: robots.txt 에서 지정하는 크롤링 규칙이 적용되어야 할 크롤러를 지정한다.
- Allow: 크롤링을 허용할 경로를 지정한다.
- Disallow: 크롤링을 제한할 경로를 지정한다.
- Sitemap: 사이트맵이 위치한 경로의 전체 URL을 지정한다.  
(https:// 부터 /sitemap.xml 까지의 전체 절대경로)

다음은 Google 크롤링 봇에게는 nogooglebot 폴더 하단의 모든 파일에 대해 크롤링을 막고,  
그 외 모든 크롤링 봇에게는 모든 파일에 대해 크롤링을 허용하는 형식.

```html
User-agent: Googlebot
Disallow: /nogooglebot/

User-agent: *
Allow: /

Sitemap: http://www.example.com/sitemap.xml
```

#### - 페이지, 문서 제어 태그

> 캐노니컬 태그(Canonical tag)

URL 주소만 다른 동일한 페이지에서 동작하며, 대표가 되는 URL을 알려주는 역할을 하는 태그.

https://www.mysite.com/shop  
https://www.mysite.com/shop?ct=1

가령 위의 주소는 URL은 다르지만 모두 동일한 페이지로 연결될 수 있다.  
검색엔진은 해당 페이지들을 동일한 페이지의 중복으로 간주하여 크롤링 횟수가 적어지는 문제가 발생.

이러한 문제를 해결하기 위해 <head>태그 사이에 캐노니컬 태그를 삽입한다.

```html
<head>
   <link rel="canonical" href="https://www.mytest.com/detail">
</head>
```

> 캐노니컬(Canonical) 태그 적용 시 주의사항

- 완전 동일한 페이지인지 명확히 구분.
- 캐노니컬 태그를 절대경로로 적용.
- 캐노니컬 태그가 중복으로 사용되지 않도록 주의.

#### - 반응형을 통한 모바일 대응

데스크탑 페이지보다 모바일 페이지를 우선적으로 크롤링하여 조금 더 높은 점수를 부여한다.  
SEO를 위해서 반응형 웹 구현은 필수적.

[light house](https://search.google.com/test/mobile-friendly)를 통해 모바일 친화성을 테스트 할 수 있다.

<br/>

### 3) SEO 점수 확인

진단 사이트는 구글에서 따로 제공하지 않는다.  
대신 SEO 관련 마케팅 에이전시에서 만든 사이트들이 존재.

SEO Checker들은 개별 회사들만의 기준에 의해 만들어져, 같은 웹사이트를 넣어도 다른 점수가 나올 수 있다. 하지만 기준은 크게 다르지 않으므로 보기에 편리한 SEO Checker를 사용하면 된다.

하지만 개발환경이 백엔드와 함께 작업하지 않는 CRA환경이라면, SEO에 한계를 맞이할 수 있다.  
크롤러 봇이 읽은 HTML문서는 서버 사이드에서 사전에 구성되어야 하기 때문.

<br/>

## SEO를 위한 관점에서의 Next.js

Next.js는 React로 만드는 서버사이드 렌더링 프레임워크.

SEO와 관련한 부분에서의 이점외에도 Next.js는 파격적인 장점들을 갖고 있지만, 이 중 SEO와 관련있는 Dynamic Meta Tag와 SSR의 일부와 관련된 내용만을 작성한다.

<br/>

### 적용

Next.js가 서버 역할을 하기 때문에, 'next build'를 거쳐 웹에 Next 서버를 먼저 동작시키는 것이 선행으로 이루어져야 한다.

서버가 존재하지 않는다면 SSR 함수를 사용할 수 없게 되기 때문.

#### - Dynamic Meta Tag

Next.js에서는 _document.js 파일에서 meta 태그를 선언하는 방식으로 dynamic SEO를 구현한다.

_document.js파일은 next로 시작한 프로젝트 내에서 _app.js파일과 함께 각 페이지의 글로벌한 초기화에 관여한다.

해당 파일 내에 DefaultSEO를 기본으로 동작시키며, 그 외 dynamic SEO는 SSR을 통해 넘어온 props로 처리한다.


```jsx
// pages/_document.jsx
import Head from "next/head";

export default function MyDocument() {
  return (
    <>
      <Head>
        <title>이_땃쥐의 SEO</title>
        <meta name="description" content="SEO 설명" />
        <link rel="icon" href="/땃쥐.jpeg" />
      </Head>
      {/* ... */}
    </>
  );
}
```

```javascript
// constants/defaultSEO.js
export const DEFAULT_SEO = {
  title: "기본 제목",
  description: "기본 설명",
  openGraph: {
    type: "website",
    locale: "ko_KR",
    url: "https://example.com",
    images: [
      {
        url: "/default-og.jpeg",
        width: 800,
        height: 400,
        alt: "기본 OG 이미지",
      },
    ],
  },
};
```
<!-- {% raw %} -->
```jsx
// pages/detail/[id].jsx
import { NextSeo } from "next-seo";

const Detail = ({ data }) => {
  return (
    <>
      <NextSeo
        title={data.title}
        description={data.description}
        openGraph={{
          title: data.title,
          description: data.description,
          images: [{ url: data.image }],
        }}
      />
      <section>
        {/* ... */}
      </section>
    </>
  );
};

export default Detail;
```
<!-- {% endraw %} -->
```javascript
export async function getStaticPaths() {
  return {
    paths: [],
    fallback: "blocking",
  };
}

export async function getStaticProps({ params }) {
  const data = await fetch(`{API_URL}/${params.id}`).then((res) => res.json());

  return {
    props: { data },
  };
}
```

이와 같은 구조로 동적으로 meta태그 할당이 가능.  
사용된 next-seo는 next내에서 seo를 설정하는데 도움을 줄 수 있는 [라이브러리](https://www.npmjs.com/search?q=next-seo).

#### - Debugging

OG가 동작하는 SNS에서의 상호작용을 통해 SEO가 적용되었는지를 확인.

로컬에서 작업하는 내용에 대해 meta 태그가 적용될 수 있도록,  Ngrok같은 플랫폼을 통해 외부에서 접근할 수 있게 해줘야 한다.  
(Ngrok는 NAT와 방화벽 뒤의 로컬 서버를 임의의 안전한 터널을 통해 공개 인터넷에 노출될 수 있도록 해주는 플랫폼.)

로컬 클라이언트 서버 구동시키고, 터미널 하나 더 열어서 Ngrok 관련 작업을 시작.

`npm install -g ngrok`

`ngrok http 3000 (<- 본인의 포트)`  
`안되면 npx ngrok http 3000`

![](/assets/img/posts/2022/after-quarter/ngrok-1.png)
![](/assets/img/posts/2022/after-quarter/ngrok-2.png)

다음에서 마주하는 URL로 SNS에서 상호작용을 이루며 테스트.

![](/assets/img/posts/2022/after-quarter/ngrok-3.png)

페이스북과 카카오톡에서 한번 적용된 data의 meta tag는 캐싱 되어, 만료되기 전까지 값이 바뀌어도 이전의 값을 사용한다.

다음의 두 사이트에서 캐시를 초기화하면서 테스트를 진행.  
[Facebook cache 초기화](https://developers.facebook.com/tools/debug/),
[KakaoTalk cache 초기화](https://developers.kakao.com/tool/debugger/sharing)

<br/>
