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
You may use quotes to specify string literals as attributes:

const element = <div tabIndex="0"></div>;
You may also use curly braces to embed a JavaScript expression in an attribute:

const element = <img src={user.avatarUrl}></img>;
Don’t put quotes around curly braces when embedding a JavaScript expression in an attribute. You should either use quotes (for string values) or curly braces (for expressions), but not both in the same attribute.

Warning:

Since JSX is closer to JavaScript than to HTML, React DOM uses camelCase property naming convention instead of HTML attribute names.

For example, class becomes className in JSX, and tabindex becomes tabIndex.

Specifying Children with JSX
If a tag is empty, you may close it immediately with />, like XML:

const element = <img src={user.avatarUrl} />;
JSX tags may contain children:

const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
JSX Prevents Injection Attacks
It is safe to embed user input in JSX:

const title = response.potentiallyMaliciousInput;
// This is safe:
const element = <h1>{title}</h1>;
By default, React DOM escapes any values embedded in JSX before rendering them. Thus it ensures that you can never inject anything that’s not explicitly written in your application. Everything is converted to a string before being rendered. This helps prevent XSS (cross-site-scripting) attacks.

JSX Represents Objects
Babel compiles JSX down to React.createElement() calls.

These two examples are identical:

const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
React.createElement() performs a few checks to help you write bug-free code but essentially it creates an object like this:

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