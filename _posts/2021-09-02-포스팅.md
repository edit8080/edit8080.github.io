---
title:  "JavaScript로 계산기 만들기"
excerpt: "계산기 미니 프로젝트"

categories:
 - JavaScript
tags:
 - [project]

toc: true
toc_sticky: true

date: 2021-09-02
last_modified_at: 2021-09-02

published: true
---

## JavaScript로 계산기 만들기

이번 시간에는 언어를 활용하는데 가장 기본적인 프로젝트로 잘 알려져있는 계산기 구현을 JavaScript를 활용하여 제작하기로 하였다.
이 프로젝트에서 발생할 수 있는 예외 사항을 최대한 해결하고, 알고리즘을 활용하여 입력받은 수식에 대해 정확한 계산을 진행할 수 있도록 목표하였다.
계산 알고리즘은 일반적인 수식 계산 방법으로 잘 활용되고 있는 중위 표기식을 후위 표기식으로 변환하고 후위 표기식을 연산하는 알고리즘을 적용하기로 하였다.
위 알고리즘 과정은 Crocus 님 블로그를 참고하여 작성하였다.

> [참고 자료] [중위 표기법을 후위 표기법으로 변환후 계산하기](https://www.crocus.co.kr/1703)

## 계산기 구현 계획과 예외 사항

JavaScript로 계산기를 만드는 과정은 다음과 같이 계획하였다.

1. 키보드 입력과 HTML과 CSS로 제작한 계산기를 클릭하는 것으로 중위 표기식을 입력받는다.
2. 중위 표기식을 후위 표기식으로 변환한다.
3. 변환한 후위 표기식을 연산한다.

위 과정에서 생각한 예외 사항은 다음과 같았다.

1. 연산자를 연속으로 입력했을 경우 (ex : 12+/34)
2. 연산자로 입력을 시작하거나 끝낸 경우 (ex : +345, 345*)
3. 소수점을 이미 찍었지만 또 찍은 경우 (ex : 12.3.4+56)
4. 연산이 불가능한 경우 (ex : 123 / 0)

## 계산기 구현 과정

### 1. 입력처리

가장 먼저 이벤트를 통한 계산기 입력을 통해 중위 표기식을 위한 숫자와 연산자 입력, 초기화, 수식 계산을 처리하는 함수를 구성하였다.
입력한 숫자와 연산자는 `expression`이라는 변수에 문자열로 추가하고, 생각한 예외 사항에서 1번과 3번을 처리하기 위해
각각의 상황을 `befOper`와 `isDouble` 이라는 boolean 변수로 체크했다.

~~~JavaScript

const opers = ["+", "-", "*", "/"]; // 연산자
let expression = ""; // 입력한 수식
let befOper = false; // 이전에 사칙 연산자가 나왔는지 확인
let isDouble = false; // 현재 값이 소수인지 확인

// 입력에 대해 계산기 동작
const calculate = (value) => {
  // 사칙 연산자 입력
  if (opers.find((oper) => oper === value)) {
    if (befOper) expression = expression.substring(0, expression.length - 1);
    befOper = true;
    isDouble = false;
    expression += value;
  }
  // 초기화 입력
  else if (value.toUpperCase() === "C" || value === "Escape") {
    expression = "";
    befOper = false;
    isDouble = false;
  }
  // 수식 계산
  else if (value === "=" || value === "Enter") {
    const res = result(); // 연산 시작
    expression = res === 0 ? "" : res;
  }
  // 숫자, 소수점 입력
  else {
    // 소수점 처리
    if (value === ".") {
      if (isDouble) return;
      else isDouble = true;
    }
    befOper = false;
    expression += value;
  }
};

~~~

### 2. 중위 표기식을 후위 표기식으로 변환

`=` 기호를 클릭하거나 키보드로 엔터를 눌렀을 때 입력받은 중위표기식에 대한 연산을 시작한다.
먼저 중위 표기식을 후위 표기식으로 변환할 때 사용할 사칙연산자에 대해 연산자 우선순위를 지정하였다.
그 다음, 피연산자와 연산자를 파싱하고 연산자 스택을 활용하여 후위 표기식을 구성해주었다.
피연산자는 숫자, 소수점이 나오면 문자열 형태로 `num` 변수에 저장했다가 연산자를 만나게되면 해당 피연산자 값을 `postfix`라는 후위 표기식 배열에
추가하였다.

이 과정에서 2번 예외를 처리할 수 있었는데 피연산자를 입력할 때 `num` 변수가 빈 문자열로 되어있다가 연산자를 만나서 이를 Number로 변환하면
0이 후위 표기식으로 추가되었다.

~~~JavaScript

const priority = {
  // 연산자 우선순위
  "+": 0,
  "-": 0,
  "*": 1,
  "/": 1,
};

// 입력받은 중위 표기식을 후위 표기식으로 변환
const makePostfix = () => {
  let num = ""; // 피연산자
  let postfix = []; // 후위표기식
  let operStack = []; // 연산자 스택

  for (let i = 0; i < expression.length; i++) {
    // 연산자 판단
    if (opers.find((oper) => oper === expression[i])) {
      // 피연산자는 후위표기식에 조건 없이 push한다.
      postfix.push(Number(num));

      // 현재 연산자보다 연산자 스택의 우선순위보다 크면 pop하여 후위표기식에 저장 (스택 빌 때까지)
      while (
        operStack.length !== 0 &&
        priority[expression[i]] <= priority[operStack[operStack.length - 1]]
      ) {
        postfix.push(operStack.pop());
      }
      // 이후 현재 연산자를 연산자 스택에 저장
      operStack.push(expression[i]);
      num = "";
    }
    // 피연산자 판단
    else {
      num += expression[i];
    }
  }
  // 남아있는 연산자, 피연산자 모두 후위표기식에 저장
  postfix.push(Number(num));
  while (operStack.length !== 0) {
    postfix.push(operStack.pop());
  }

  return postfix;
};

~~~

### 3. 후위 표기식 계산

이어서 변환한 후위 표기식을 피연산자 스택을 사용하여 연산하였다.
피연산자를 만나면 피연산자 스택에 저장했다가 연산자를 만나면 상위 2개의 피연산자를 뽑아 계산한 후 결과를 다시 피연산자 스택에 넣어주면 된다.
위 과정을 마무리하면 최종 결과만 피연산자 스택에 남게 된다.

상위 2개의 피연산자를 뽑을 때 num1과 num2의 위치를 주의하여 연산자 계산을 처리한다.
또한 피연산자를 제공할 때 Number로 형변환을 하여 넘겨주면 실수 타입으로 제공되므로 부동소수점 연산으로 처리된다.
예외 4번을 처리하기 위해 몫 연산에 대해 제수가 0인 상황과 그 외에 발생할 수 있는 연산 문제를 try~catch 문으로 확인하고 예외 발생시 `calError` 라는 boolean 변수로 확인한다.

~~~JavaScript

let calError = false; // 연산 예외 발생

// 연산자 처리
const operator = (num1, num2, oper) => {
  try {
    switch (oper) {
      case "+":
        return num2 + num1;
      case "-":
        return num2 - num1;
      case "*":
        return num2 * num1;
      case "/":
        if (num1 === 0) {
          calError = true;
          return;
        }
        return num2 / num1;
    }
  } catch (e) {
    calError = true;
  }
};


// 입력받은 중위표기식을 후위표기식으로 변환한 후 수식 계산
const result = () => {
  let postfix = makePostfix();
  let num = [];

  for (let i = 0; i < postfix.length; i++) {
    // 연산자 처리 - 피연산자 배열에서 상위 2개의 수를 뽑아 계산
    if (opers.find((oper) => oper === postfix[i])) {
      num.push(operator(num.pop(), num.pop(), postfix[i]));
    }
    // 피연산자 처리 - 피연산자(num) 배열에 저장
    else {
      num.push(postfix[i]);
    }
  }

  return num[0];
};

~~~

## 계산기 구현 완성

최종적으로 구성한 기능에 대해 이벤트 함수와 DOM 조작을 통해 계산기가 동작하는 모습을 확인할 수 있도록 만들었다.
JS 구성 자체는 어렵지 않았지만 알고리즘의 구조와 예외 처리를 고민해야하는 점에서 재미있는 미니 프로젝트였다.
이외에도 괄호를 추가하거나 다른 공학 연산자를 추가하는 등 다양한 방법으로 기능을 확장하여 구성하는 것도 기능 구현 역량 강화에 도움이 될 것이라고 생각했다.

<iframe src="https://codesandbox.io/embed/sad-tesla-pojvo?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="sad-tesla-pojvo"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>
