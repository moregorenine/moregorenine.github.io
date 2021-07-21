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
{: .notice}

```js
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```
## 2. JSX 소개
아래의 변수 선언은 뭘까요?
{: .notice}

```js
const element = <h1>Hello, world!</h1>;
```
element는 string도 아니고 HTML도 아닙니다.  

- 이것을 JSX라고 합니다. javascript 구문의 확장입니다.
- JSX는 React의 "elements"를 생성합니다.

### why JSX
React에서는 본질적으로 렌더링 로직이 UI 로직(이벤트가 처리되는 방식, 시간에 따라 state가 변하는 방식, 화면에 표시하기 위해 데이터가 준비되는 방식 등)과 연결된다는 사실을 받아들입니다.  

마크업과 로직을 별도의 파일에 넣어 인위적으로 기술 을 분리하는 대신  
React 는 두 가지를 모두 포함하는 "components"라고 단위로 결합 합니다.

### JSX에 표현식 포함하기
JSX에서는 javascript변수를 `{}`묶어 사용할 수 있습니다.
```js
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;

ReactDOM.render(
  element,
  document.getElementById('root')
);
```
JSX의 중괄호 안에 유효한 `JavaScript 표현식`을 넣을 수 있습니다.  
예를 들어, `2 + 2`, `user.firstName`또는 `formatName(user)`모두 유효한 JavaScript 표현식입니다.

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

컴파일이 끝나면 JSX 표현식은 일반 JavaScript 함수 호출이 되어 JavaScript 객체로 인식됩니다.  

즉, JSX를 if 구문 및 for loop 안에 사용하고, 변수에 할당하고, 인자로서 받아들이고, 함수로부터 반환할 수 있습니다.
```js
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

### JSX 속성 정의
따옴표를 사용하여 문자열을 속성으로 지정할 수 있습니다
```js
const element = <div tabIndex="0"></div>;
```
속성에 JavaScript 표현식을 포함하기 위해 `{}`를 사용할 수도 있습니다:
```js
const element = <img src={user.avatarUrl}></img>;
```
어트리뷰트에 JavaScript 표현식을 삽입할 때 중괄호 주변에 따옴표를 입력하지 마세요. 따옴표(문자열 값에 사용) 또는 중괄호(표현식에 사용) 중 하나만 사용하고, 동일한 어트리뷰트에 두 가지를 동시에 사용하면 안 됩니다.

>**주의**  
JSX는 HTML보다 JavaScript에 더 가깝기 때문에
React DOM은 HTML 속성 대신 camelCase 명명 규칙을 사용합니다.  
예를 들어 class는 `className`, 그리고 tabindex는 `tabIndex`로 사용한다.

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
기본적으로 React DOM은 렌더링하기 전에 JSX에 포함된 모든 값을 `이스케이프` 합니다.
따라서 애플리케이션에 명시적으로 작성되지 않은 내용은 절대 주입할 수 없습니다.
모든 것은 렌더링되기 전에 문자열로 변환됩니다.
이는 `XSS(교차 사이트 스크립팅)` 공격 을 방지하는 데 도움이 됩니다 .

### JSX는 Objects를 표현합니다.
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
`React.createElement()` 버그 없는 코드를 작성하는 데 도움이 되는 몇 가지 검사를 수행하지만 기본적으로 다음과 같은 객체를 생성합니다.
```js
// Note: this structure is simplified
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```
이러한 객체를 “React elements”라고 하며, 화면에서 보고 싶은 것을 나타내는 표현이라 생각하면 됩니다. React는 이 객체를 읽어서, DOM을 구성하고 최신 상태로 유지하는 데 사용합니다.

## 엘리먼트 렌더링
엘리먼트는 React 앱의 가장 작은 단위입니다.
{: .notice}

엘리먼트는 화면에 표시할 내용을 기술합니다.
```js
const element = <h1>Hello, world</h1>;
```
브라우저 DOM 엘리먼트와 달리 React 엘리먼트는 일반 객체이며(plain object) 쉽게 생성할 수 있습니다. React DOM은 React 엘리먼트와 일치하도록 DOM을 업데이트합니다.

>**주의**  
더 널리 알려진 개념인 “컴포넌트”와 엘리먼트를 혼동할 수 있습니다. 다음 장에서 컴포넌트에 대해 소개할 예정입니다. 엘리먼트는 컴포넌트의 “구성 요소”이므로 이번 장을 읽고 나서 다음 장으로 넘어갈 것을 권합니다.

### DOM에 엘리먼트 렌더링하기
HTML 파일 어딘가에 `<div>`가 있다고 가정해 봅시다.
```js
<div id="root"></div>
```
이 안에 들어가는 모든 엘리먼트를 React DOM에서 관리하기 때문에 이것을 “루트(root)” DOM 노드라고 부릅니다.  

React로 구현된 애플리케이션은 일반적으로 하나의 루트 DOM 노드가 있습니다. React를 기존 앱에 통합하려는 경우 원하는 만큼 많은 수의 독립된 루트 DOM 노드가 있을 수 있습니다.  

React 엘리먼트를 루트 DOM 노드에 렌더링하려면 둘 다 `ReactDOM.render()`로 전달하면 됩니다.  

```js
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

위 코드를 실행하면 화면에 “Hello, world”가 보일 겁니다.

### 렌더링 된 엘리먼트 업데이트하기
React 엘리먼트는 `불변객체`입니다. 엘리먼트를 생성한 이후에는 해당 엘리먼트의 자식이나 속성을 변경할 수 없습니다. 엘리먼트는 영화에서 하나의 프레임과 같이 특정 시점의 UI를 보여줍니다.  

지금까지 소개한 내용을 바탕으로 하면 UI를 업데이트하는 유일한 방법은 새로운 엘리먼트를 생성하고 이를 `ReactDOM.render()`로 전달하는 것입니다.  

예시로 똑딱거리는 시계를 살펴보겠습니다.  
```js
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(element, document.getElementById('root'));
}
setInterval(tick, 1000);
```
위 함수는 `setInterval()` 콜백을 이용해 초마다 `ReactDOM.render()`를 호출합니다.

>**주의**  
실제로 대부분의 React 앱은 ReactDOM.render()를 한 번만 호출합니다. 다음 장에서는 이와 같은 코드가 `유상태 컴포넌트`에 어떻게 캡슐화되는지 설명합니다.

### 변경된 부분만 업데이트하기
React DOM은 해당 엘리먼트와 그 자식 엘리먼트를 이전의 엘리먼트와 비교하고 DOM을 원하는 상태로 만드는데 필요한 경우에만 DOM을 업데이트합니다.  

개발자 도구를 이용해 마지막 예시를 살펴보면 이를 확인할 수 있습니다.  

매초 전체 UI를 다시 그리도록 엘리먼트를 만들었지만 React DOM은 내용이 변경된 텍스트 노드만 업데이트했습니다.  

경험에 비추어 볼 때 특정 시점에 UI가 어떻게 보일지 고민하는 이런 접근법은 시간의 변화에 따라 UI가 어떻게 변화할지 고민하는 것보다 더 많은 수의 버그를 없앨 수 있습니다.