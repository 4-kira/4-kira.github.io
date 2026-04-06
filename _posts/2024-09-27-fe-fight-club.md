---
title: "프론트엔드 파이트 클럽"
date: 2024-09-27 00:00:00 +0900
categories: [essay, event]
tags: [css architecture, package manager]
---

> [Archive] 이 글은 2024년에 다른 플랫폼에서 작성한 글을 이전한 것입니다.  
> 내용 중 일부는 현재 기준과 맞지 않을 수 있습니다.

## 프론트엔드 다이빙: 프론트엔드 파이트 클럽

오프라인 커뮤니티 '프론트엔드 다이빙 클럽'에서 개최한 스핀오프 모임.  
기존 프론트엔드 파이트 클럽(이하 프파클)의 발표 + 토론 형태가 아닌, 패널토론 + 채팅참관 형태로 진행.

3가지 주제에 대해 TOSS의 개발자들이 패널 토론을 진행한다.

<br/>

### Round 1. 동료 개발자 핵심 역량

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fc4zGhk%2FbtsJO1aD3uP%2FAAAAAAAAAAAAAAAAAAAAALrOPP4Yz11ADsyPVo4MNGGFHbc7EQmpl6OgLnnUReJD%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3DzHPky3aC7jqNcheRFyt4W7ARWOw%253D)

홍팀 주장
- 일이라고 하는 것은 "문제를 해결하는 것".
- 커뮤니케이션 능력도 당연히 중요하지만, 문제 해결 능력에 비교할 수는 없다.
- 커뮤니케이션 능력은 문제 해결 능력을 키우는 것보다는 훨씬 쉽다.

청팀 주장
- 문제는 어떠한 방식으로든 결국 해결된다.
- 커뮤니케이션 자체도 일이고, 같이 문제를 정의하고 내가 문제 해결사가 되면 된다.

#### 생각 정리

문제 해결역량이 커뮤니케이션 능력보다 중요하다고 생각했다.  
회사는 "일을 하는 곳"이고 일은 "문제를 해결하는 것"이기 때문에, 역량의 중요도가 우선되어야 한다.

개인을 상대하는 개인으로써는 후자가 선호될 여지가 있지만,  
공동체의 더 나은 미래를 생각한다면 답은 정해져있다고 생각했다.  
그래서 '대결이 성사될 수 있는 내용인가?' 생각했는데, 채팅이나 같이 모임을 참관했던 한O님은 생각이 달랐다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FdCwM3R%2FbtsJPglaDSN%2FAAAAAAAAAAAAAAAAAAAAADIzDyDJGxH2AAbifwlVkwhBHgQzkfSktaVDhZN2987o%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3DzTOyu4aNUIu7sr7J6e3h1F1groE%253D)

AI의 발전으로 불가능한 문제해결이 거의 존재하지 않게 된 시점에서는  
커뮤니케이션 역량이 지금보다도 훨씬 더 중요해질 수도 있겠다는 생각이 들었다.

커뮤니케이션 역량 강화를 위해선 어떤 노력들이 필요한 것일까를 고민하다가  
이런 블로그 글도 문서화의 한 종류로써 커뮤니케이션 역량 강화의 기초가 될수도 있겠구나 생각했다.

<br/>

### Round 2. 스타일링 방식

<br/>
![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FZtpVO%2FbtsJQnJ0bgK%2FAAAAAAAAAAAAAAAAAAAAAK7CrtmuhwypUUDTaJr89Byn78nyWWH0A7H5Au1hEUvX%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3DydLcd1%252F3%252FRNMKDZxXe%252FZaHKDwPw%253D)

홍팀 주장
- CSS-in-JS는 생태계에서 환영받지 못하는 이유들이 존재한다.

> 1) CSS-in-JS는 실제로 런타임에서 성능 문제를 일으킬 수 있다.  
> ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fdzw8WW%2FbtsJQDlxBda%2FAAAAAAAAAAAAAAAAAAAAAOJO0b864-yzALsKc06m2NctacaQg8RD8nPf3RFN6Wmh%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3DurxYwZ5xgCVgYZicpKmzIres6lE%253D)
> Chakra UI의 경우 @emotion/styled 의존성을 통해 런타임 CSS-in-JS를 사용중인데,  
> 개발팀은 이러한 CSS-in-JS의 성능 문제를 인지하고 있음을 공식적으로 언급했다.

> 2) useInsertionEffect  
> React는 CSS-in-JS의 런타임 주입 방식을 사용해야 하는 경우 useInsertionEffect를 통한 최적화를 권장한다.  
> 하지만, 해당 방식이 최적의 해결책은 아니라고 설명한다.

> 3) Next.js에서의 CSS-in-JS와 관련한 일부 제약 사항  
> Next.js 13 이상에서는 React 18의 새로운 streaming rendering 방식을 사용하는데,
> 이 방식에서는 \<head>가 \<body>보다 먼저 렌더링.
> 하지만 CSS-in-JS는 일반적으로 <body> 렌더링 후에 스타일을 생성하기 때문에 문제가 발생할 수 있다.

- emotion docs의 확인을 통한 실질적 성능 문제 확인.

> emotion 라이브러리를 임의의 동적 스타일을 업데이트 하는 데 걸리는 시간을 측정.  
> ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fb5cKu5%2FbtsJPMQ8i6h%2FAAAAAAAAAAAAAAAAAAAAAK2h90dZR-BzeGo2eq514ki_9NOn6Bt33f6H3fVNU1Br%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3DpDEcaFNAwWM2JzNHiZvUmoVUTuc%253D)
> 결과로 pure process가 emotion의 사용보다 최대 6배 가량 빠르다는 것을 확인할 수 있다.

청팀 주장
- tailwind CSS는 추가 라이브러리 사용으로 런타임 오버헤드가 발생할 수 있다.

> ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2Fch3VCn%2FbtsJQBH5IhS%2FAAAAAAAAAAAAAAAAAAAAAOAymk26WveMmEE29OVBTEQa3Zr6baWcp3W6oD291m1f%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3Dp%252BHziOmsjhsOaVWvDlEiuvLyao0%253D)
> Tailwind CSS는 "zero runtime"을 표방하지만 실제로는 추가 라이브러리 사용으로 런타임 오버헤드가 발생할 수 있으며, 클래스 이름 결합이나 동적 스타일링을 위해 추가 라이브러리가 필요할 수 있다.
> 
> CSS-in-JS는 런타임에 스타일을 생성하지만, 최근 성능 개선으로 부담이 줄어들고 있으며,  
> JavaScript의 모든 기능을 활용할 수 있어 유연한 스타일링이 가능하다.

- CSS-in-JS가 좋은 모습을 보여주는 런타임에서의 역할이 있다.

> CSS-in-JS가 훌륭한 모습을 보여주는 런타임에서의 역할이 있다. (= 성능 이슈는 불필요한 걱정이다.)
> ![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2FDURuQ%2FbtsJQHH2HRi%2FAAAAAAAAAAAAAAAAAAAAADzsnEx7axWOnoGXHs7Ko1NA6wkGSU_KBfg3pjSIxnzn%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1769871599%26allow_ip%3D%26allow_referer%3D%26signature%3DVkueRLXIjzQU6b4%252FkN4qmTWVTzw%253D)
> 런타임에 classname이 추가되어야 할 대, emotion은 css를 잘못 사용한 것보다 훨씬 빠르다.  
> 각 서비스에서 프로파일러를 돌려보면서 emotion이 더 적절할 수 있는지 파악하며 개발하는게 중요하다.

#### 생각 정리

CSS와 CSS-in-JS 선택은 상황에 따라 달라질 수 있다.  
CSS-in-JS가 런타임 성능 이슈를 가질 수 있지만, 동적 스타일링에서는 장점을 보인다.  

프로파일러를 통한 성능 측정은 매우 중요한 접근 방식.

각 프로젝트의 특성과 요구사항을 고려하며, 개발 편의성과 성능 사이의 균형을 찾는 것이 핵심.

#### 궁금한 점

- yarn과 yarn berry의 차이
- 프로파일러를 통한 실질적인 성능 측정 코드

<br/>

### Round 3. 패키지 매니저

홍팀 주장
- yarn은 npm을 대체하기 위해 새로운 방식의 접근을 선택했지만, 제약 사항이 다소 존재한다.

> 2022년 Vercel이 만든 'Turbopack'이라는 번들러가 있는데,  
> maintainer가 yarn berry에서 지원하게 수정할 생각이 없다고 선언.
> 
> 만약 yarn 매니저 기반으로 개발을 진행하다가 이런 상황을 마주하면 어떻게 해결할 것인가?

- yarn PnP의 말도안되는 Zip 파일 형식

> yarn PnP는 각 패키지를 개별 Zip 파일로 저장하며 무결성을 보장하고 수정 여부를 쉽게 알 수 있게 한다.  
> Zip 파일은 압축된 형태이기 때문에 내용을 직접 보거나 수정하기 어렵다.
> 
> pnpm은 content-addressable 저장소를 사용하여 패키지를 관리한다.  
> 이 방식은 파일 시스템에서 직접 패키지 내용을 볼 수 있게 하여 굉장히 직관적으로 바라볼 수 있다.

청팀 주장
- pnpm은 결국 npm이다. (internet explorer를 업그레이드 해도 결국 explorer일 뿐)
- yarn은 패키지 매니저를 근본있게 수정하며 잘 설계했다고 생각한다.

> 전통적 의미에서의 근본이 아닌건 맞지만, 근본이 잘못되었다면 파괴적인 행동을 할 필요도 있다.  
> 잘못된 것들이 있다면 이를 바로잡아야 하는 것도 중요한 요소.

- 커뮤니티에서 자주 사용되는 라이브러리의 경우 일반적으로 yarn을 통한 매니징이 가능하다.

> toss는 yarn korea user group을 형성하고자 노력하고 있다.

#### 생각 정리

npm의 문제점들을 고려할 때, yarn이 조금 더 나은 선택이지 않을까 생각했다.  
npm은 표준으로 사용되어 왔지만, 성능, 일관성, 보안 면에서 개선이 필요한 부분이 존재.

"근본이 잘못되었다면 파괴적인 행동을 할 필요도 있다"는 말이 특히 인상 깊었다.  
yarn이 npm의 문제점을 해결하기 위해 변화를 시도한 것과 같은 접근이 개발 생태계를 발전시킨다고 생각한다.

#### 궁금한 점

- npm과 pnpm의 차이
- yarn PnP의 Zip 파일 형식이 갖는 이점들
- npm이 근본적으로 패키지 매니징 차원에서 실질적으로 얼마나 문제를 일으키는지

<br/>