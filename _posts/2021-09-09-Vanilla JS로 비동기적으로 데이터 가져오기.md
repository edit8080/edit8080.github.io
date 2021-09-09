---
title:  "Vanilla JS로 비동기 데이터를 Fetch해 페이지 렌더링하기"
excerpt: "Promise와 async/await로 외부 API 데이터 fetch하기"

categories:
 - JavaScript
tags:
 - [fetch, Promise, async/await]

toc: true
toc_sticky: true

date: 2021-09-09
last_modified_at: 2021-09-09

published: true
---

## 1. 비동기적으로 데이터 가져오기

흔히 프론트엔드 과제 테스트를 진행하면 외부 API 데이터를 Fetch하여 사용하는 로직이 많이 필요하다.
기존에 MDN 홈페이지를 보면서 `fetch()` 함수를 통해 데이터를 사용했지만 잘 정리되지 않아서 시간을 많이 잡아먹었던 경험이 있었다.
이번에 봤던 프론트엔드 데브매칭때도 비슷한 문제가 나왔지만 역시 `Promise`로 수행할지 `async/await`를 사용하여 수행할지 확실히 결정하지 않아
갈팡질팡하게 되어 시간을 많이 잡아먹었다.
따라서 이번 기회에 `Promise`와 `async/await`를 사용해서 데이터를 Fetch하고 두 방식간에 구현 로직의 차이가 있는지 확인하고자 했다.

## 2. 구현 계획

데이터를 단순히 Fetch하기만 하는 것은 재미가 없으니 이전 시간에 구현했던 [&lt;Vanilla JS로 간단한 SPA만들기&gt;](https://edit8080.github.io/javascript/Vanilla-JavaScript%EB%A1%9C-SPA-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0/) 프로젝트를 활용하기로 결정하였다.
따라서 이번 시간에 진행할 프로젝트 계획은 다음과 같다.

1. Promise를 활용하여 데이터를 Fetch한 다음 페이지 렌더링하기
2. async/await를 활용하여 데이터를 Fetch한 다음 페이지 렌더링하기

이를 위해서 필요한 json 데이터는 예제를 위한 간단한 json을 제공해주는 [jsonplaceholder](https://jsonplaceholder.typicode.com/) 사이트를 활용하기로 했다.

## 3. Promise로 데이터 Fetch하기

외부 데이터를 가져오기 위해서 일반적으로 axios 라이브러리를 사용하지만 가장 기본적으로는 `fetch()`함수를 활용한다.
기존에 `fetch()` 함수를 다루면서 가장 문제가 되었던 점은 API 데이터를 Fetch하는 로직을 다른 파일로 분리하고 GET으로 받은 데이터를
어떻게 다른 로직에 전달할까? 였다. 이러한 문제가 발생하는 원인은 위에서 설명한대로 `fetch()` 함수가 Promise 형태로 동작하기 때문에 데이터만 따로 반환할 수가 없었다.

~~~javascript

export function getUsers() {
  const url = `https://jsonplaceholder.typicode.com/users`;

  let getData;
  fetch(url)
    .then((response) => response.json())
    .then((data) => getData = data) 

  return getData; // 이러한 형식이 불가능하다.
}

~~~

이러한 문제를 해결하기 위해 구글링을 해본 결과 Promise 내 데이터는 Promise 함수 자체로 return하여 활용한다는 [답변](https://stackoverflow.com/questions/37533929/how-to-return-data-from-promise)을 찾게되었다.
따라서 위 문제를 해결하기 위해 `fetch()` 함수 자체를 return하고 데이터를 사용하는 부분에서 이렇게 전달받은 Promise 함수를 `.then()`으로 활용하는 방식으로 진행하기로 했다.

~~~javascript

// Fetch Data
export function getUsers() {
  const url = `https://jsonplaceholder.typicode.com/users`;

  // fetch 함수 자체를 Promise 함수 형태로 반환한다.
  return new Promise((resolve, reject) =>
    fetch(url)
      .then((response) => response.json())
      .then((data) => resolve(data))
      .catch((error) => reject(error))
  );
}

// Use Data
export function usersRender() {
    const $app = document.getElementById("app");

    // fetch Promise 함수를 활용해 HTMLElement를 생성하여 렌더링한다.
    getUsers()
        .then((listData) => {
            let $el = `
            <h1>Users</h1>
            <ul>
                ${listData.map((data) => `<li>${data.name}</li>`).join("")}
            </ul>
            `;
            $app.innerHTML = $el;
        });   
}

~~~

### 구현 결과

위와 유사한 방식으로 이전에 진행한 SPA 프로젝트 내에 데이터를 렌더링하였다.
핵심적으로 살펴볼 부분은 Promise 형태를 사용하여 데이터를 렌더링하는 부분이다.
URL을 파싱하여 사용하는 것은 이전 포스팅을 참고하도록 한다.

> ✅ 이와 별개로, 데이터 대기 로직을 어디에서 처리하고 있는지도 주의깊게 볼 필요가 있다!

<iframe src="https://codesandbox.io/embed/ecstatic-orla-6k60o?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="ecstatic-orla-6k60o"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

## 4. async/await로 데이터를 Fetch하기

async/await는 ES2017부터 새롭게 등장한 비동기 함수 정의 키워드이다.
기존 프로그래밍 과제에서 async와 await를 사용하여 비동기 함수를 실행시키고 싶었는데 그럴때마다 데이터가 아닌 Promise 객체가 반환되어 사용할 때 애를 먹었던 경험이 있다.
이러한 문제를 자세히 관찰한 결과, 데이터를 Fetch하는 곳은 문제가 없지만 데이터를 사용하는 곳에서 async/await로 지정하지 않아 Pending 상태의 Promise 객체가 반환되는 문제가 발생하는 것이었다.
따라서 이 문제는 비동기 데이터를 사용하는 곳까지도 async/await 로 비동기처리를 진행해주면 깔끔하게 해결된다.
이러한 상황을 다르게 살펴보면 async/await 로 선언된 비동기 함수는 Promise 형태로도 활용할 수 있다는 말이므로 해당 함수를 `.then()`으로 연결해서 사용할 수 있다.

이러한 개념을 가지고 async/await를 사용해 데이터를 Fetch 하였다.
위의 Promise랑 비교해서 훨씬 코드가 깔끔해졌다는 것을 느낄 수 있었다.

~~~javascript

// Fetch Data
export async function getUsers() {
  const url = `https://jsonplaceholder.typicode.com/users`;

  const response = await fetch(url);
  const data = await response.json();

  return data;
}

// Use Data
export async function usersRender() {
  const $app = document.getElementById("app");
  const users = await getUsers(); // 앞에서 설명한대로 이 부분에도 비동기 처리를 해주어야한다.
  const $el = `
    <h1>Users</h1>
    <ul>
      ${users.map((data) => `<li>${data.name}</li>`).join("")}
    </ul>
  `;

  $app.innerHTML = $el;
}

~~~

### 구현 결과

<iframe src="https://codesandbox.io/embed/async-awaitro-deiteoreul-fetchhan-hu-peiji-rendeoring-retm9?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="async/await로 데이터를 Fetch한 후 페이지 렌더링"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>


&lt;참고자료&gt;

1. [jsonplaceholder](https://jsonplaceholder.typicode.com/)
2. [MDN Fetch API](https://developer.mozilla.org/ko/docs/Web/API/Fetch_API)
3. [How to return data from promise](https://stackoverflow.com/questions/37533929/how-to-return-data-from-promise)
4. [MDN async function](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Statements/async_function)