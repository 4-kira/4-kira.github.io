---
title: "Cloudflare 서비스 중단 사태"
date: 2025-11-20 00:00:00 +0900
categories: [knowledge, CS]
tags: [cloudflare]
---

## 개요

[2025년 11월 18일 Cloudflare 서비스 중단에 대한 공식 블로그 글](https://blog.cloudflare.com/ko-kr/18-november-2025-outage/)  
한국 시간(KST) 기준  
약 2025년 11월 18일 20시 17분 부터 2025년 11월 18일 23시 42분 까지  
Cloudflare 서비스가 중단되는 사태가 발생.

![Cloudflare Issue](/assets/img/posts/2025/2025-11/cloudflare-issue.png){: width="40%"}
[Cloudflare Status](https://www.cloudflarestatus.com/incidents/8gmgl950y3h7)에서 Cloudflare 상태에 대한 정보를 확인 가능.

해당 시간의 Reddit에서 참고된 바, 일본 오사카 도쿄 영국도 다운.

Cloudflare는 Anycast로 전 세계에 동일한 IP 프리픽스를 광고한다.  

```
        (Cloudflare Tokyo PoP)
              ↑
BGP Advertise |   "104.16.0.0/12은 여기 있다"
              |
      ┌──────────────────────┐
      │ ISP / 글로벌 라우터들    │
      └──────────────────────┘
              |
              |   "104.16.0.0/12 경로 배웠음"
BGP Advertise |
              ↓
        (Cloudflare LA PoP)
```

즉 장애가 특정 지역에서 시작해도, 같은 경로를 공유하는 글로벌 트래픽 일부가 동시 영향권에 들어갈 수 있다.

<br/>

## Cloudflare의 주요 기능

### Anycast 리버스 프록시(https 프런트 도어)

많은 서비스가 다음의 구조를 따른다.

`user` - `cloudflare edge(=proxy server)` - `origin server`

대기 시간과 대역폭 사용을 줄이기 위해 데이터 소스와 가장 가까운 곳에서 컴퓨팅을 하는 엣지 컴퓨팅 기반으로 웹 트래픽에 대한 리버스 프록시 역할.

네트워크 엣지는 해당 장치 또는 해당 장치를 포함하는 로컬 네트워크가 인터넷과 통신하는 곳을 뜻하며,  
엣지가 하는 핵심적인 역할은 다음과 같다.

#### - TLS 종료

사용자가 https://service.com에 접속.  
하지만 실제로는 Cloudflare 엣지에 TLS 세션을 맺고, 엣지가 별도의 연결로 origin에 연결하여 데이터를 전달.

origin 서버는 TLS, 인증서 갱신, Cipher 설정 등을 Cloudflare에 위임하여 운영.

#### - L7 보안 계층 (WAF, Bot Management, Rate limiting)

공격 트래픽 / 크롤러 / 스팸 봇을 엣지에서 걸러낸 후, 깨끗한 트래픽만 origin으로 전송.

LLM API 같은 곳은 해당 레이어가 없으면 DDoS, 크롤링, abuse로 터지기 때문에 사실상 필수적 방패로써 동작.

#### - 캐싱 + 성능 최적화

정적 파일(JS, CSS, 이미지, 폰트)과 일부 동적 응답까지 캐싱.

<br/>

### 권한 DNS 기반의 보호

언론에서 Cloudflare는 전 세계 웹사이트의 약 20% 트래픽을 처리하는 인프라로 설명된다.

해당 20% 중 상당수가 DNS + 프록시를 동시에 Cloudflare에 얹어둔 상태라는 뜻.

#### - Authoritative DNS

도메인이 어디 IP/호스트로 가야 하는지를 Cloudflare가 관리하는 경우가 많다.

Cloudflare DNS가 장애 나거나 오작동하면,  
도메인이 아예 resolve 안 되거나 프록시 우회 경로가 사라진다.

#### - DNS + 보안

많은 서비스가 DNS 레코드 - Cloudflare 프록시(CNAME/Proxy) 구조로 붙어 있다.  
DNS 레이어와 프록시 레이어가 같이 먹통이 되면, 우회할 수 있는 경로가 없음.

<br/>

## Cloudflare를 사용하는 주요 서비스

### 대화형 AI / LLM 서비스

> ex) ChatGPT, Claude 등

prompt injection, 대량 계정 생성/스크래핑, API abuse를 엣지에서 막으며  
다양한 대륙에서 레이턴시를 감소시키기 위해 cloudflare를 사용한다.

그래서 이와 같은 구조를 가지게 되는데,  

`사용자` → `Cloudflare (TLS, 인증, 보안, 캐싱)` → `API Gateway(K8s/Envoy 등)` → `모델 서버`

이 상태에서 Cloudflare 장애가 발생하면, 모델 서버는 멀쩡한데도 접속이 500/502로 터진다.

<br/>

### 소셜 커뮤니케이셜 플랫폼

> X(트위터), Discord 등

스크롤을 통해 타임라인을 로드할 때마다 JS 번들, 이미지, 썸네일이 전송되는데,  
그걸 전 세계 유저에게 가까운 엣지에서 제공.

또한 거대한 트래픽을 자체 데이터센터가 다 받기엔 부담이기 때문에 Cloudflare가 앞단에서 흡수한다.

그래서 Cloudflare가 미끄러지면 타임라인과 미디어에 대한 로딩 실패가 발생한다.

<br/>

### 실시간 게임

> League of Legends 등

클라이언트 패치, 스킨 리소스, 동영상, 웹 클라이언트 파일을 CDN으로 배포하며,  
런처가 띄우는 웹 뷰, 뉴스 페이지, 상점 페이지 등도 Cloudflare 엣지를 지나간다.

로그인/매칭/계정 관리 등에 쓰이는 웹/REST API 앞단에서 WAF 및 DDoS 방어를 위해서도 Cloudflare를 사용한다.

실시간 게임 서버 UDP 트래픽 자체를 전부 Cloudflare로 통과시키지 않는 경우도 있지만,  
로그인·매칭·컨트롤 플레인 + 런처/패치가 Cloudflare에 엮여 있기 때문에 에러가 발생한다.

![Cloudflare Issue](/assets/img/posts/2025/2025-11/cloudflare-game-issue.png){: width="80%"}

<br/>

## 장애 원인과 기술적 분석

Cloudflare로 향하는 모든 요청은 명확하게 정의된 경로를 거친다.  
요청은 HTTP 및 TLS 계층에서 종료되고, 핵심 프록시 시스템(Fontline(=FL))으로 전달.  
**핵심 프록시를 통과하는 동안 네트워크에서 제공되는 다양한 보안 및 성능 관련 제품을 실행**한다.  
WAF 규칙 및 DDoS 방어 적용, 그리고 트래픽을 Developer Platform과 R2로 라우팅하는 작업이 포함된다.

**이 FL 모듈 중 하나인 Bot Management가 서비스 중단의 원인**.
Bot Management는 네트워크를 통과하는 모든 요청을 대상으로 봇 점수를 생성하고 어떤 봇이 사이트에 액세스하도록 허용할지 여부를 제어한다.  
이 모델의 입력으로 사용하는 '피처 구성 파일'은 몇 분마다 갱신되어 네트워크 전체에 배포된다.  
(새로운 유형의 봇 공격에 대응할 수 있도록 해주기 때문에 악성 행위자가 빠르게 전략을 바꾸는 상황에 대처)

그러나 **ClickHouse 쿼리 동작의 변경**으로 인해 이 **파일이 중복된 “피처” 행을 많이 포함**하게 되었고,  
봇 모듈에서 오류가 발생하며 핵심 프록시 시스템에서 HTTP 5xx 오류 코드가 반환되게 된 것.

ClickHouse 소프트웨어의 동작과 관련되어 더 자세히 이해하기 위해서는 [2025년 11월 18일 Cloudflare 서비스 중단에 대한 공식 블로그 글](https://blog.cloudflare.com/ko-kr/18-november-2025-outage/)을 참고한다.

<br/>

## 시사점

Cloudflare는 단순 CDN 업체가 아니라, 인터넷의 L7 프록시 + DNS + 보안을 통합한 거대한 인프라 레이어.  
11월 18일 사고는 “단일 벤더가 인터넷의 상당 부분을 프록시/보안 계층으로 장악했을 때 생기는 시스템 리스크”를 보여준 사례.

![Cloudflare Stock](/assets/img/posts/2025/2025-11/cloudflare-stock.png){: width="100%"}

당일 cloudflare의 주식 가격.  
개장 후 1시간 내에 10% 하락.

<br/>
