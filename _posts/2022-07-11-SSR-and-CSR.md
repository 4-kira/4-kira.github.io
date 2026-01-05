---
title: "오즈의 제작소: SSR과 CSR"
date: 2022-07-11 00:00:00 +0900
categories: [implementation, react]
tags: [react, SSR, CSR]
---

> [Archive] 이 글은 2022년에 다른 플랫폼에서 작성한 글을 이전하며 재구성한 것입니다.  
> 내용 중 일부는 현재 기준과 맞지 않을 수 있습니다.

## 렌더링(Rendering)

웹 프로그래밍에서의 렌더링이란 서버로부터 HTML파일을 받아 브라우저에 뿌려주는 일련의 과정을 뜻한다.

![velog](https://velog.velcdn.com/images/dltkdals224/post/e26aff80-9fde-47bd-ad52-009cd2255ec2/image.png)

렌더링 과정은 다음과 같다.

1) 브라우저(Loader)가 서버로부터 정보들을 불러온다.

2) 렌더링 엔진이 Parsing을 통해 HTML문서를 DOM트리로 만든다.

3) 외부 css파일과 함께 포함된 스타일 요소를 파싱해 CSSOM트리를 만든다.

> 스타일은 브라우저 기본 스타일(User Agent Stylesheet) 위에 사용자 정의 스타일이 적용되며,  
> 최종적으로는 CSS 우선순위 규칙(cascade)에 따라 스타일이 결정된다.)

4) 브라우저는 DOM트리와 CSSOM을 결합하여 렌더트리를 구축한다.

5) 뷰포트 내에서 노드들의 위치와 크기를 계산하는 과정이 진행된다.

> 해당 과정을 레이아웃 단계 혹은 리플로우 라고 부른다.  
> 
> 모든 변경이 Layout → Paint를 발생시키는 것은 아니다.  
> 변경 종류에 따라 (Layout + Paint / only Paint / only Composite)가 발생할 수 있다.

6) 렌더링 엔진이 페인트 이벤트를 발생시켜 렌더링 트리를 기반으로 화면을 그려낸다.

> 해당 과정을 페인팅 또는 래스터화 라고 부른다.

<br/>

### SSR(Server Side Rendering)

전통적인 웹 어플리케이션의 렌더링 방식.  
요청시마다 서버에서 처리한 후 새로고침으로 페이지에 대한 응답.

SSR을 채택해야 하는 상황은 다음과 같다.

- 네트워크가 느리거나 메인스크립트가 커서 로딩이 느릴 때  
(CSR은 한번에 모든 것을 불러오지만 SSR은 페이지마다 나눠불러오기 때문)
- SEO(serach engine optimization : 검색 엔진 최적화)가 필요할 때
- 최초 로딩이 빨라야하는 사이트를 개발 할 때
- 웹 사이트가 상호작용이 별로 없을 때
- 보안이 필요한 주요 비즈니스 로직을 숨겨야 할 때

#### MPA(Multiple Page Application)

MPA는 전통적인 웹 애플리케이션 구성 방식이다.  
MPA는 전통적인 웹 애플리케이션 구성 방식으로, 일반적으로 SSR 방식과 함께 사용되어 왔다.

MPA 애플리케이션의 가장 큰 단점은 프론트엔드와 백엔드가 강하게 결합되어 있다는 것인데,  
가령 사용자가 main이라는 페이지를 요청한다면 다음과 같은 과정을 거치기 때문이다.

1) 'main'에 대한 페이지 정보를 서버에 요청한다.

2) 'main'에 대한 요청을 받은 서버는 해당하는 UI 및 필요한 데이터를 HTML데이터로 파싱한다.

3) 브라우저는 전달받은 HTML 데이터를 지정된 인코딩으로 변환하여 사용자에게 보여준다.

브라우저는 해당 과정을 거치면서 페이지를 갱신하며, 사용자의 모든 이동에 대하여 페이지 새로고침이 발생하게된다.  
새로고침은 로딩 시간의 합을 증가시키며 이로인해 사용자 경험이 나빠질 수밖에 없다.  
일부 개선 기법을 통해 페이지 전체 새로고침을 줄일 수는 있었지만,
기본적으로 페이지 단위로 새로고침이 발생하는 구조 자체를 바꾸지는 못했다.

<br/>

### CSR(Client Side Rendering)

클라이언트단에서 JS를 통해 렌더링하는 방식.  
페이지에 담는 정보가 많아질수록 Javascript 번들이 커지게 되고, 이로 인해 초기 렌더링 속도가 느려질 수 있다.

CSR을 채택해야 하는 상황은 다음과 같다.

- 네트워크가 빠르거나, 메인 스크립트가 가벼울 때 (= 초기 JS 로딩 비용을 감당할 수 있는 환경)
- 사용자에게 보여줘야 하는 데이터의 양이 많을 때 (로딩 상태를 제어할 수 있는 장점)
- SEO가 필요없는 페이지를 구축할 때
- 웹 어플리케이션에 사용자와 상호작용할 것들이 많을 때 

> 로딩 중 상태를 명확히 보여주고, UI가 준비되기 전 사용자의 행동을 제어할 수 있다는 장점


#### SPA(Single Page Application)

MPA 애플리케이션의 단점을 개선해줄 수 있는 방법이다.  
SPA는 일반적으로 CSR 방식으로 구현된다.

SPA는 브라우저에 로드되고 난 뒤에 페이지 전체를 서버에 요청하는 것이 아닌,  
최초 한 번 기본 HTML과 Javascript 번들을 로드한 뒤 js가 실행되면서 페이지를 렌더링한다.  
렌더링 후 클라이언트 요청에 따라 페이지 특정 부분만 다시 렌더링하는 웹 어플리케이션 개념이다.

페이지의 화면 구성과 전환을 Javascript에서 처리하여, 동적으로 화면이 갱신되는 것처럼 보이게 하는 방식.

<br/>

### SSR과 CSR의 차이

- 초기 View 로딩 속도  

CSR은 최초 로딩 시 HTML과 Javascript 번들을 함께 불러오기 때문에 초기 로딩 속도가 SSR에 비해 느릴 수 있다.  
반면 최초 로딩 이후의 화면 전환이나 사용자 인터랙션은 빠른 편이다.

SSR은 요청마다 서버에서 HTML을 생성하기 때문에, 데이터가 많은 서비스의 경우 서버 부담이 커질 수 있다.

- SEO(검색 엔진 최적화) 문제  

CSR 방식에서는 Javascript 실행 전 HTML이 비어 있는 경우, 검색 엔진이 콘텐츠를 수집 못하는 문제가 발생할 수 있다.  
최근에는 검색엔진의 Javascript 해석 능력이 개선되었지만, 여전히 CSR 환경에서는 SEO 대응을 위한 추가적인 설정이 필요하다.

- 보안 문제  

SSR에서는 사용자 정보를 서버에서 세션 기반으로 관리할 수 있어 클라이언트에 민감한 정보가 노출될 가능성을 줄일 수 있다.  
CSR 환경에서는 인증 및 사용자 상태를 클라이언트에서 관리해야 하므로 보안 설계에 더욱 주의가 필요하다.

- 진화  

이러한 문제들을 보완하기 위해 프레임워크와 렌더링 방식도 함께 발전해왔다.  
React와 Vue 기반의 Next.js, Nuxt.js와 같은 프레임워크가 등장하며 SSR과 CSR을 함께 활용하는 방식이 보편화되고 있다.


<br/>

### SSR과 CSR의 상호 보완

이 두 가지 렌더링 방식의 단점을 상호보완하여 첫 번째 페이지 로딩에서는 서버 사이드 렌더링(SSR)을 사용하고,  
그 후에 모든 페이지 로드에는 클라이언트 사이드 렌더링(CSR)을 활용하는 방법을 많이 사용한다.

![velog](https://velog.velcdn.com/images/dltkdals224/post/e7ff3e34-48d9-4cb0-8da0-5d4fab4c8803/image.png)

일반적으로 SSR과 CSR을 상호보완하는 형식을 적용하기 위한 프로젝트 세팅에서 Next.js를 많이 사용한다.  
React만으로 SSR을 구현하는게 불가능한건 아니지만, 과정이 다소 복잡하다.

'오즈의 제작소' 에서도 Next.js를 사용한다.  
굿즈 상품을 판매하는 사이트이기 때문에 검색엔진 최적화(SEO)가 중요해 SSR의 사용이 필요하다.

CSR을 사용하는 구조에서도 검색엔진 최적화가 가능하긴 하다.  
react-helmet 패키지로 SEO에 필요한 메타태그들을 변경하고 react-snap 패키지를 통해 html파일을 분리하고 SEO친화적으로 바꿔낸다.

<br/>

## 오즈의 제작소에서 적용

오즈의 제작소에는 SSR 페이지와 CSR 페이지가 함께 존재했다.

특히 CSR로 동작하는 일부 페이지(ex. 꿀팁 페이지)는 기본 HTML만으로는 메타 정보가 부족해,  
공유 미리보기(Open Graph)나 검색엔진에서 원하는 정보가 잘 노출되지 않는 문제가 있었다.

이를 해결하기 위해 페이지별 SEO 데이터를 미리 정의해두고, Head/react-helmet을 통해 필요한 메타 태그와 JSON-LD를 주입하는 방식으로 구성했다.

```javascript
import React from 'react';
import { Helmet } from 'react-helmet';
import PropTypes from 'prop-types';

const JsonLdSEO = ({ index = 0 }) => {
  const jsonLd = [
    {
      '@context': 'https://schema.org',
      '@type': 'WebSite',
      name: '굿즈 제작 정보가 다! 오즈의 제작소',
      url: 'https://ozjejakso.com/aboutus',
      // ... (페이지 성격에 맞게 필요한 필드만 구성)
    },
    // ...
  ];

  return (
    <Helmet>
      <script type="application/ld+json">
        {JSON.stringify(jsonLd[index])}
      </script>
    </Helmet>
  );
};

JsonLdSEO.propTypes = {
  index: PropTypes.number,
};

export default JsonLdSEO;
```

```javascript
// ... (_app.js or page component 일부)

<Head>
  {/* 기본 메타 */}
  <title>{title}</title>
  <meta name="description" content={description} />

  {/* 공유 미리보기용(Open Graph / Twitter Card) */}
  <meta property="og:type" content="website" />
  <meta property="og:title" content={title} />
  <meta property="og:description" content={description} />
  <meta property="og:url" content={url} />
  <meta property="og:image" content={imageUrl} />

  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:title" content={title} />
  <meta name="twitter:description" content={description} />
  <meta name="twitter:image" content={imageUrl} />
</Head>
```

<br/>
