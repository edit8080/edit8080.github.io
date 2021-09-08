---
title:  "Vanilla JavaScript로 간단한 SPA 구현하기"
excerpt: "JavaScript와 SPA"

categories:
 - JavaScript
tags:
 - [project, SPA]

toc: true
toc_sticky: true

date: 2021-09-06
last_modified_at: 2021-09-06

published: true
---

## 2021년 하반기 프로그래머스 프론트엔드 데브 매칭 참가 회고

지난 토요일에 2021년 하반기 프로그래머스 프론트엔드 데브 매칭 과제가 있었다.
좋은 경험이 될 것 같아 참가한 결과로는 문제를 해결할 수 없었지만 내가 몰랐던 새로운 과제를 마주하게 되어 오히려 새로운 자극이 되었던 것 같다.
(그렇게 크게 기대하지 않아서 실망감이 별로 없었던 것 같다. 😆)

과제의 핵심적인 구현사항은 역시 기존에 생각한 대로 외부 API 데이터를 호출하여 웹 페이지를 렌더링하는 전형적인 프론트엔드 과제였다.
이 부분은 예상하고 있었지만 Vanilla JS로 SPA를 구현해야하는 과정이 필요했기에 이 부분 때문에 문제를 해결하지 못했다.
다음에 비슷한 기회가 있을 때는 해결할 수 있도록 준비하기 위해 Vanilla JS로 SPA 구현하는 과정을 여러 자료를 참고하면서 구현해보았다.

## SPA 구현계획

SPA를 구현하기 위해서 단순한 페이지 렌더링이 아니라 실제 API에 맞게 동작하기 위한 이벤트 및 쿼리스트링에 초점을 두어 구성하기로 계획했다.
이를 단순화하고 서로 비교하기 위해 두 가지 기능을 구성하기로 했다.

1. 서로 다른 URL을 기준으로 페이지 구성하기
2. 동일한 URL이지만 쿼리 스트링이나 URL 파라미터를 통해 개별적으로 동작하는 페이지 구성하기

## Vanilla JS로 SPA 구현하기

SPA를 구현하는데 있어서 다음의 [블로그](https://kdydesign.github.io/2020/10/06/spa-route-tutorial/)를 참고하여 작성하였다.

나는 여기서 history를 활용하여 SPA를 구현하기로 결정하였다.
history를 활용하여 SPA를 구현하기 위해서 history 객체의 `pushState` 함수와 window의 `popstate` 이벤트를 주로 활용한다.
가장 먼저 `pushState` 함수는 브라우저의 세션 기록 스택에 상태를 추가하는 함수로 사용자의 방문 기록을 저장하고 지정한 URL 경로로 이동시켜주는 역할을 수행한다.

> pushState는 pushState(state, title, url) 구조로 이루어져있는데 state는 이전, 다음 페이지를 이동할 때 사용할 데이터를 저장할 때 활용되고, url은 페이지를 이동할 때 활용된다.
>
> 하지만 title은 아직 중점적으로 활용되지는 않는다고 한다. => [MDN 참고](https://developer.mozilla.org/ko/docs/Web/API/History/pushState)

다음으로 `popstate` 이벤트는 사용자의 세션 기록으로 인해 현재 활성화된 경로가 변경될 때 발생한다.
따라서, `pushState`로 저장한 방문 기록에 기반해 URL을 이동하게 되면 `popstate` 이벤트가 발생한다.
이를 통해, 이전 페이지나 다음 페이지로 페이지를 이동할 때 `popstate` 이벤트를 통해 SPA 페이지를 연속적으로 이동할 수 있다.

> 이전 방문 기록에 기반해 이동하기 때문에 직접적으로 URL을 입력하여 이동하는 경우 `popstate` 이벤트를 통해 페이지를 렌더링할 수 없다.

이후 핵심적인 router 로직을 구성하였다. index.html 구조와 각 페이지를 렌더링하는 HTML Element 태그는 간단하게 지정하였으므로 설명 없이 넘어가도록 한다. 궁금하다면 하단의 구현 결과의 최종 코드를 참고한다.

~~~javascript

/* router.js */
import { aboutEl, homeEl, productEl } from "./htmlEl.js";

const $app = document.querySelector("#app");
const $links = document.querySelectorAll("[route^='/']");

// 각 라우팅 페이지는 함수로 관리 -> 외부 API 데이터 처리 후 HTML Element를 구성하여 반환하는 형태로 확장
const routes = {
  "/": homeEl,
  "/about": aboutEl,
  "/product": productEl,
};

// 각 링크 요소에 대해 클릭 이벤트 추가 -> API 데이터로 렌더링한 요소의 클릭 이벤트 등으로 확장
$links.forEach(($link) => {
  $link.addEventListener("click", (e) => {
    const pathName = e.target.getAttribute("route"); // 지정한 경로 Fetch
    console.log(pathName);
    
    window.history.pushState({}, pathName, pathName); // 페이지 이동
    $app.innerHTML = routes[pathName](); // 이동한 페이지 렌더링
  });
});

// 이전 페이지, 다음 페이지로 이동할 때 렌더링
window.onpopstate = () => {
  $app.innerHTML = routes[window.location.pathname]();
};

// 초기 라우팅
$app.innerHTML = routes["/"]();

~~~

가장 먼저 각 페이지를 구성하는 HTML 요소들을 관리하였다. HTML 요소는 확장성을 고려해 함수로 관리하도록 설정하였다.
이후 각 페이지를 이동할 수 있는 버튼에 대해 이벤트를 추가해주었다. 지금 여기서는 클릭 이벤트를 사용하였고 `route` 속성값으로 저장한 URL 경로를 가져와 페이지를 이동하였다.
페이지를 이동하는데 위에서 설명한 `pushState` 함수를 활용하는 모습을 볼 수 있다.
마지막으로 이전 페이지와 다음 페이지로 이동해도 SPA를 렌더링할 수 있도록 하기 위해 `onpopstate` 이벤트를 통해 각 페이지를 렌더링할 수 있도록 구성하였다.

이렇게 해서 서로 다른 URL을 기준으로 페이지를 이동할 수 있는 방법은 구성하였다.
그렇다면 이제 2번 계획을 달성하기 위해 `/products/1` 이나 `/products?pId=1` 과 같은 URL에 대해 특정 제품에 대해 페이지를 렌더링하는 기능을 추가해야한다.
사실 이 부분은 router에서 수행하기보다 렌더링할 HTML 요소를 반환하는 함수를 구성하는 부분에서 담당한다.

우선 URL 경로에 매개변수나 쿼리 스트링을 추가해야하기 때문에 `pushState`를 통해 URL을 지정하는 부분의 pathName을 다음과 같이 살짝 변형시켜줬다.
나는 링크 버튼에 `data-id` 속성값으로 제품 ID를 부여하여 페이지를 이동할 때 활용할 수 있도록 하였다.

~~~javascript

// 매개변수(/:id 형태)
pathName + `${pathName === "/product" ? "/" + e.target.dataset.id : ""}`

// 쿼리스트링(?pid=1 형태)
pathName + `${pathName === "/product" ? "?pid=" + e.target.dataset.id : ""}`

~~~

이후 두 가지 경우에 대해 지정한 데이터를 사용할 수 있도록 파싱하는 과정이 필요한데, 매개변수를 통해 경로를 이동하는 경우 URL을 / 를 기준으로 파싱하여 사용하였다.

~~~javascript

// 매개변수 파싱
export function productEl() {
  const urlParse = window.location.pathname.split("/");
  const pid = urlParse[urlParse.length - 1];

  return `<h1>${pId}번째 제품입니다.</h1>`;
}

~~~

쿼리 스트링을 파싱할 때는 다음 [사이트](https://gent.tistory.com/62)의 정규표현식을 참고하였다.

~~~javascript

// 쿼리스트링 파싱
export function productEl() {
  let queryStr = {};

  // ?A=B 나 &A=B 기준으로 파싱
  window.location.search.replace(/[?&]+([^=&]+)=([^&]*)/gi, function (_, key, value) {
    queryStr[key] = value;
  });

  return `<h1>${queryStr.pid}번째 제품입니다.</h1>`;
}

~~~

마지막으로, 위에서 설명한 URL 파싱을 토대로 `onpopstate` 이벤트 함수에서도 유사한 방식으로 URL 파싱이 이루어지도록 변경하면 기본적인 SPA 구성이 완료된다.

## 구현

<iframe src="https://codesandbox.io/embed/amazing-fire-lvpfh?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="amazing-fire-lvpfh"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

> ‼ codesandbox 환경에서 onpopstate의 이전, 다음 페이지로 이동하는 기능이 정상적으로 동작하지 않는다. 로컬 환경에서는 정상적으로 동작하는 것을 확인하였다.

&lt;참고 자료&gt;

1. [Vanilla JS에서 SPA 라우팅 시스템 구현하기](https://kdydesign.github.io/2020/10/06/spa-route-tutorial/)
2. [MDN - History.pushState()](https://developer.mozilla.org/ko/docs/Web/API/History/pushState)
3. [MDN - popstate](https://developer.mozilla.org/ko/docs/Web/API/Window/popstate_event)
4. [URL(주소) Query String 쉽게 가져오기 (인자, 파라미터, param)](https://gent.tistory.com/62)
