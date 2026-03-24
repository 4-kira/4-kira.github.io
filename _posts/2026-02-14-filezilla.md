---
title: "cafe24 호스팅과 FileZilla를 사용한 정적 파일 서빙"
date: 2026-02-14 00:00:00 +0900
categories: [implementation, tools]
tags: [hosting, cafe24, filezilla]
---

FTP로 파일을 업로드 하는 과정에서 마주한 에러들과 정적 파일 서빙을 이해하게 된 과정을 정리.

<br/>

## FTP 배포 기본 흐름

카페24 웹호스팅에 파일을 올리는 과정은 다음과 같다.

1. 카페24 관리자 페이지(my.cafe24.com)에서 FTP 접속 정보 확인
2. FileZilla 설치 후 접속 (호스트 / 아이디 / 비밀번호 / 포트 21)
3. `/www/` 폴더가 웹 루트 — 여기에 파일 업로드
4. 업로드 즉시 반영 (서버 재시작 불필요)

<br/>

## 빌드 파일이 반영되지 않을 때

### - 브라우저 캐시

가장 흔한 원인이다. `Ctrl + Shift + R`로 강력 새로고침하거나, 개발자도구 Network 탭에서 "Disable cache"를 체크한 후 새로고침한다.

### - 해시 파일명 문제

React는 빌드할 때마다 JS/CSS 파일명에 랜덤 해시값이 붙는다.

```
이전 빌드: main.abc123.js
새 빌드:   main.zzz999.js  ← 파일명 자체가 바뀜
```

`index.html`만 덮어쓰면 안 된다. `static/` 폴더 전체를 교체해야 한다.

올바른 배포 순서는 다음과 같다.

1. FileZilla에서 `/www/static/` 폴더 **통째로 삭제**
2. 새 빌드의 `static/` 폴더 **새로 업로드**
3. `index.html` **덮어쓰기**
4. `Ctrl + Shift + R` 새로고침

### - MIME 타입 에러

```
Failed to load module script: Expected a JavaScript module script
but the server responded with a MIME type of "text/plain".
```

카페24 Apache 서버가 `.js` 파일을 JavaScript가 아닌 텍스트 파일로 응답하기 때문이다.  
브라우저는 MIME 타입이 틀리면 스크립트 실행을 거부한다.

`/www/` 루트에 `.htaccess` 파일을 만들어 업로드한다.

```apache
AddType application/javascript .js
AddType text/css .css
AddType application/json .json
```

FileZilla에서 숨김 파일이 보이지 않으면 상단 메뉴 → **서버** → **숨김 파일 강제 표시**를 선택한다.

### dist 폴더째로 올리는 실수

`/www/` 안에 `dist/` 폴더 자체가 들어가면 안 된다. `dist/` **안의 내용물**을 `/www/`에 바로 올려야 한다.

```
올바른 구조:
/www/
  ├── index.html
  ├── static/
  └── asset-manifest.json

잘못된 구조:
/www/
  └── dist/         ← dist 폴더째로 올린 경우
        ├── index.html
        └── static/
```

<br/>

## 정적 파일 서빙의 동작 원리

FTP 업로드만으로 배포가 가능한 이유는 **정적 파일 서빙** 방식 때문이다.

```
로컬에서 npm run build
    ↓
FileZilla로 /www/에 파일 업로드 (서버 디스크에 파일 저장)
    ↓
유저가 브라우저에서 URL 접속
    ↓
카페24 Apache 서버가 /www/index.html 읽어서 응답
    ↓
브라우저가 index.html에 적힌 JS/CSS 파일 추가 요청
    ↓
화면 렌더링
```

핵심은 서버가 **파일을 그대로 전달만 할 뿐**, 코드를 실행하거나 컴파일하지 않는다는 점이다.  
그래서 서버 재시작 없이 파일 교체만으로 즉시 반영된다.

<br/>

## 카페24 호스팅의 가능 / 불가능

| 가능                         | 불가능                         |
| ---------------------------- | ------------------------------ |
| React / Vue SPA 배포         | Node.js (Express 등) 서버 실행 |
| 외부 API 호출 (fetch, axios) | Python / Django / FastAPI      |
| PHP 파일 실행                | 커스텀 포트 사용               |
| 이미지, CSS, JS 서빙         | DB 직접 운영                   |

백엔드가 필요한 경우엔 FE는 카페24, 백엔드는 Railway / Render / AWS 등 별도 서버로 분리하는 구조가 일반적이다.

<br/>

## 카페24 vs Netlify / Vercel

순수 정적 파일 서빙 용도만으로는 Netlify나 Vercel이 훨씬 편하다.

|                  | 카페24                     | Netlify / Vercel          |
| ---------------- | -------------------------- | ------------------------- |
| 배포 방법        | FTP로 직접 업로드          | git push → 자동 빌드+배포 |
| 빌드             | 로컬에서 직접 빌드 후 올림 | 서버에서 자동으로 빌드    |
| 가격             | 유료 (월 정액)             | 무료 플랜 있음            |
| HTTPS            | 별도 SSL 신청 필요         | 자동 발급                 |
| CDN              | 기본 제공 약함             | 글로벌 CDN 기본 내장      |
| PHP 지원         | 됨                         | 안 됨                     |
| 국내 도메인 연동 | 편함                       | 번거로움                  |

### - 카페24가 유리한 경우

- `.co.kr` 도메인을 카페24에서 구매하여 한 곳에서 관리하고 싶을 때
- PHP로 간단한 서버 로직(메일 발송, 폼 처리)이 필요할 때
- 고객사 납품용으로 국내 고객센터가 필요할 때

순수 React 앱 배포가 목적이라면 Netlify나 Vercel이 낫다.  
카페24의 본래 강점은 **PHP + MySQL 조합의 소규모 웹사이트**와 **국내 쇼핑몰 연동**이다.
