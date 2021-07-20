---
title: "React docs"
excerpt: "React docs 공식 홈페이지 정리"
categories:
    - react
tags:
    - react
last_modified_at: 2021-7-17T14:00:00+09:00
toc: true
toc_sticky: true
---

# Main Concepts
## 1. Hello World
아래는 화면에 Hello, world!를 출력하는 가장 간단한 예제입니다.
```js
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```
## 2. Introducing JSX
아래의 변수 선언은 뭘까요?
```js
const element = <h1>Hello, world!</h1>;
```
element는 string도 아니고 HTML도 아닙니다.  

- 이것을 JSX라고 합니다. javascript 구문의 확장입니다.
- JSX는 React의 "elements"를 생성합니다.

### why JSX
React는 렌더링 로직이 본질적으로 다른 UI 로직과 결합되어 있다는 사실을 받아들입니다.  
이벤트 처리 방식, 시간 경과에 따른 상태 변경 방식, 데이터 표시 준비 방식 등입니다.  

마크업과 로직을 별도의 파일에 넣어 인위적으로 기술 을 분리하는 대신  
React 는 두 가지를 모두 포함하는 "components"라고 단위로 결합 합니다.

### Embedding Expressions in JSX
JSX에서는 javascript변수를 `{}`묶어 사용할 수 있습니다.
```js
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;

ReactDOM.render(
  element,
  document.getElementById('root')
);
```
JSX의 중괄호 안에 유효한 JavaScript 표현식을 넣을 수 있습니다.  
예를 들어, 2 + 2, user.firstName또는 formatName(user)모두 유효한 JavaScript 표현식입니다.

```js
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!
  </h1>
);

ReactDOM.render(
  element,
  document.getElementById('root')
);
```

### JSX 표현식

컴파일 후 JSX 표현식은 일반 JavaScript 함수 호출이 되어 JavaScript 객체로 평가됩니다.  
즉, if명령문 및 for루프 내에서 JSX를 사용 하고 , 변수에 할당하고, 인수로 수락하고, 함수에서 반환할 수 있습니다.
```js
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

### JSX 속성 지정
따옴표를 사용하여 문자열을 속성으로 지정할 수 있습니다
```js
const element = <div tabIndex="0"></div>;
```
속성에 JavaScript 표현식을 포함하기 위해 `{}`를 사용할 수도 있습니다:
```js
const element = <img src={user.avatarUrl}></img>;
```
속성에 JavaScript 표현식을 포함할 때 중괄호 주위에 따옴표를 사용하지 마십시오. 따옴표(문자열 값의 경우) 또는 중괄호(표현식의 경우)를 사용해야 하지만 동일한 속성에서 둘 다 사용해서는 안 됩니다.

>Warning:  
JSX는 HTML보다 JavaScript에 더 가깝기 때문에
React DOM은 HTML 속성 대신 camelCase 명명 규칙을 사용합니다.  
예를 들어 class는 className, 그리고 tabindex는 tabIndex로 사용한다.

### JSX로 자식 지정하기
태그가 비어 있으면 XML과 같이 `/>`태그를 사용하여 즉시 닫을 수 있습니다 .
```js
const element = <img src={user.avatarUrl} />;
```
JSX 태그에는 하위 항목이 포함될 수 있습니다.
```js
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

### JSX는 주입 공격을 방지합니다.
JSX에 사용자 입력을 포함하는 것이 안전합니다.
```js
const title = response.potentiallyMaliciousInput;
// This is safe:
const element = <h1>{title}</h1>;
```
기본적으로 React DOM은 렌더링하기 전에 JSX에 포함된 모든 값을 이스케이프 합니다.
따라서 애플리케이션에 명시적으로 작성되지 않은 내용은 절대 주입할 수 없습니다.
모든 것은 렌더링되기 전에 문자열로 변환됩니다.
이는 XSS(교차 사이트 스크립팅) 공격 을 방지하는 데 도움이 됩니다 .

### JSX Represents Objects
Babel은 JSX를 `React.createElement()`호출 로 컴파일 합니다.  
이 두 예는 동일합니다.
```js
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```
```js
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```
React.createElement() performs a few checks to help you write bug-free code
but essentially it creates an object like this:
`React.createElement()` 버그 없는 코드를 작성하는 데 도움이 되는 몇 가지 검사를 수행하지만 기본적으로 다음과 같은 객체를 생성합니다.
// Note: this structure is simplified
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
These objects are called “React elements”. You can think of them as descriptions of what you want to see on the screen. React reads these objects and uses them to construct the DOM and keep it up to date.

We will explore rendering React elements to the DOM in the next section.

Tip:

We recommend using the “Babel” language definition for your editor of choice so that both ES6 and JSX code is properly highlighted.