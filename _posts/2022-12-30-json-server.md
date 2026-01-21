---
title: "json-server 라이브러리 사용"
date: 2022-12-30 00:00:00 +0900
categories: [implementation, javascript]
tags: [json-server]
---

> [Archive] 이 글은 2022년에 다른 플랫폼에서 작성한 글을 이전한 것입니다.  
> 내용 중 일부는 현재 기준과 맞지 않을 수 있습니다.

## json-server 란?

json-server는 짧은 시간 내에 REST API를 구축할 수 있도록 해주는 JavaScript 라이브러리.  
단순 목데이터보다 유려한 사용성을 가지며, 이후 이루어지는 서버와의 API 통신 연결 시간을 단축시켜줄 수 있다.

따라서 프로젝트 프로토타입의 제작이나, 소규모 프로젝트 제작 과정에서 유용하게 사용된다.

<br/>

## json-server 사용

### 1) 설치 및 시작하기

`npm install -g json-server`  
`yarn -g add json-server`  
진행하는 프로젝트 최상단에 db.json이라는 파일을 생성한다.  
모든 data는 db.json 내에서만 관리.

`json-server --watch db.json --port 4000`  
그리고 터미널을 하나 추가로 열어 리액트 서버를 가동해 주면, json-server를 사용 가능하다.

기능들을 자세히 살펴보기 전, DB에 임의의 data를 넣도록 한다.  
(해당 과정은 Postman을 통해 진행)

우선 db.json 내 객체에 data를 담을 table을 만들어줘야 한다.  
"products"를 key값으로 하는 배열을 생성.
```json
{
  "products": []
}
```
이제 "products"라는 URI를 통해 실제 백엔드 서버와 통신하듯 동작할 수 있는 상태가 된다.

![](/assets/img/posts/2022/after-quarter/json-server-1.png)
![](/assets/img/posts/2022/after-quarter/json-server-2.png)

POST요청에 대해 data가 정상적으로 저장됨을 확인할 수 있다.  
(레코드를 고유하게 식별해 데이터의 무결성을 유지해주는 id는 실제 server의 DB와 마찬가지로 중복될 수 없음)

<br/>

### 2) 제공 기능
json-server는 단순한 data의 삽입과 호출 외에도 수정, 삭제와 더불어 필터링, 페이지네이션, 정렬 등의 기능을 제공한다.
 
#### - URI routing

복수형(Plural)과 단수형(Singular)을 분리해 놓았지만, 모두 복수형으로 사용해도 무방하다.  
따라서, 기본적으로 제공하는 CRUD별 라우팅 형태는 다음과 같다.

```
GET    /products
GET    /products/1
POST   /products
PUT    /products/1
PATCH  /products/1
DELETE /products/1
```

기본적으로 URI 구분자에 사용되는 값은 id.  
하지만 routes.json 파일을 만들어 임의의 custom routing 경로를 생성할 수도 있다.

<br/>

#### - filter

GET 요청으로 불러오는 data에 대해 필터링을 적용할 수 있다.

- . 연산자

. 연산자를 통해 deep properties에 대해 접근할 수 있다.

```
GET /products?id=1&id=2
// id가 1인 data와 id가 2인 data

GET /products?seller.name=ddatjui
// 2단 depth 탐색
```
&와 |연산자 또한 사용할 수 있다.

- _start와 _end

값의 시작과 끝을 정해 제한할 수 있다. (id를 통한 제한이 아님)

```
GET /products?_start=20&_end=30
// 20번째부터 30번째까지(id와 무관)의 데이터
```

- _limit

데이터의 개수를 통해 필터링.

```
GET /products/_limit=10
// 불러올 data를 10개로 제한하여 호출
```

- _gte, _lte, _ne, _like

데이터의 최댓값, 최솟값, 불일치, 포함을 여부로 필터링.

_gte : 크거나 같다.  
_lte : 작거나 같다.

_ne : 일치하지 않는다.   
_like : 문자를 포함한다.

```
GET /products?id_gte=10&id_lte=20
// 10이상 20이하의 모든 id를 갖는 data

GET /products?id_ne=1
// id가 1이 아닌 모든 data

GET /products?title_like=server
// id값에 1이 포함되어있는 모든 data
```

- q (full-text search)

데이터 객체 key에 상관없이 full-text search.

```
GET /products?q=https
// https라는 text에 대해 full-text searching 시행
```

- _embed, _expand

순서대로 자식과 부모의 자원을 탐색하여 필터링.

<br/>

#### - pagination

_page와 _limit 인자를 통해 페이지네이션을 구현.  
_page로 몇 번째 data 묶음인지를, _limit을 통해 한 페이지를 구성하는 data의 수를 제한한다.  
( _limit의 default value는 10. )

```
GET /products?_page=7
// 10개 단위로 묶은 data의 7번째를 보여줌
```

<br/>
 
#### - sort

_sort와 _order 인자를 통해 정렬을 구현한다.  
_sort로 정렬을 적용할 key를, _order을 통해 정렬할 방식을 정한다.  
(_order의 기본 정렬 방식은 오름차순(ascending))

Multiple fields 대한 정렬도 제공.

```
GET /products?_sort=id&_order=desc
// id에 대하여 내림차순으로 정렬.

GET /products?_sort=id,price&_order=desc,asc
// user에 대하여 내림차순으로, price에 대하여 오름차순으로 정렬.
// 정렬 우선 적용은 앞의 key(여기서는 user)를 대상으로.
```

<br/>
 
#### - delete

전체 삭제를 해야 하는 경우, DELETE /products 따위의 방식을 사용할 수 없다.  
고차함수 등을 통해 모든 데이터의 개수만큼 API 호출이 필요하다.

```typescript
const useDeleteAllCartItem = (datas) => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: () => {
      datas.map((data) => deleteCartItemApi(data.id));
      // datas의 수 만큼 delete API를 호출하는 방법밖에 없더라...
    },
    onSuccess: () => {
      queryClient.refetchQueries(["cartData"]);
    },
  });
};

const deleteAllCartData = useDeleteAllCartItem(reversedCartData);

const confirmDeleteAllCartData = async () => {
  const confirmFlag = confirm("모든 제품을 장바구니에서 제거하시겠습니까?");
  if (confirmFlag) {
    deleteAllCartData.mutate();
  }
};
```

(상위 코드는) React-Query를 사용하는 과정 내에서 작성되었다.

보다 더 자세하고 정확한 기능들은 [깃허브](https://github.com/typicode/json-server를 직접 참고.