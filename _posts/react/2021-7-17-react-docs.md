---
title: "React docs"
excerpt: "React docs 공식 홈페이지 정리"
categories:
    - react
tags:
    - react
last_modified_at: 2021-8-16T14:00:00+09:00
toc: true
toc_sticky: true
---

# Main Concepts
## 1. Hello World
아래는 화면에 Hello, world!를 출력하는 가장 간단한 예제입니다.
{: .notice}

```jsx
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```
## 2. JSX 소개
아래의 변수 선언은 뭘까요?
{: .notice}

```jsx
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
```jsx
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;

ReactDOM.render(
  element,
  document.getElementById('root')
);
```
JSX의 중괄호 안에 유효한 `JavaScript 표현식`을 넣을 수 있습니다.  
예를 들어, `2 + 2`, `user.firstName`또는 `formatName(user)`모두 유효한 JavaScript 표현식입니다.

```jsx
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
```jsx
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  return <h1>Hello, Stranger.</h1>;
}
```

### JSX 속성 정의
따옴표를 사용하여 문자열을 속성으로 지정할 수 있습니다
```jsx
const element = <div tabIndex="0"></div>;
```
속성에 JavaScript 표현식을 포함하기 위해 `{}`를 사용할 수도 있습니다:
```jsx
const element = <img src={user.avatarUrl}></img>;
```
어트리뷰트에 JavaScript 표현식을 삽입할 때 중괄호 주변에 따옴표를 입력하지 마세요. 따옴표(문자열 값에 사용) 또는 중괄호(표현식에 사용) 중 하나만 사용하고, 동일한 어트리뷰트에 두 가지를 동시에 사용하면 안 됩니다.

>**주의**  
JSX는 HTML보다 JavaScript에 더 가깝기 때문에
React DOM은 HTML 속성 대신 camelCase 명명 규칙을 사용합니다.  
예를 들어 class는 `className`, 그리고 tabindex는 `tabIndex`로 사용한다.

### JSX로 자식 지정하기
태그가 비어 있으면 XML과 같이 `/>`태그를 사용하여 즉시 닫을 수 있습니다 .
```jsx
const element = <img src={user.avatarUrl} />;
```
JSX 태그에는 하위 항목이 포함될 수 있습니다.
```jsx
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

### JSX는 주입 공격을 방지합니다.
JSX에 사용자 입력을 포함하는 것이 안전합니다.
```jsx
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
```jsx
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```
```jsx
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```
`React.createElement()` 버그 없는 코드를 작성하는 데 도움이 되는 몇 가지 검사를 수행하지만 기본적으로 다음과 같은 객체를 생성합니다.
```jsx
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

## 3. 엘리먼트 렌더링
엘리먼트는 React 앱의 가장 작은 단위입니다.
{: .notice}

엘리먼트는 화면에 표시할 내용을 기술합니다.
```jsx
const element = <h1>Hello, world</h1>;
```
브라우저 DOM 엘리먼트와 달리 React 엘리먼트는 일반 객체이며(plain object) 쉽게 생성할 수 있습니다. React DOM은 React 엘리먼트와 일치하도록 DOM을 업데이트합니다.

>**주의**  
더 널리 알려진 개념인 “컴포넌트”와 엘리먼트를 혼동할 수 있습니다. 다음 장에서 컴포넌트에 대해 소개할 예정입니다. 엘리먼트는 컴포넌트의 “구성 요소”이므로 이번 장을 읽고 나서 다음 장으로 넘어갈 것을 권합니다.

### DOM에 엘리먼트 렌더링하기
HTML 파일 어딘가에 `<div>`가 있다고 가정해 봅시다.
```jsx
<div id="root"></div>
```
이 안에 들어가는 모든 엘리먼트를 React DOM에서 관리하기 때문에 이것을 “루트(root)” DOM 노드라고 부릅니다.  

React로 구현된 애플리케이션은 일반적으로 하나의 루트 DOM 노드가 있습니다. React를 기존 앱에 통합하려는 경우 원하는 만큼 많은 수의 독립된 루트 DOM 노드가 있을 수 있습니다.  

React 엘리먼트를 루트 DOM 노드에 렌더링하려면 둘 다 `ReactDOM.render()`로 전달하면 됩니다.  

```jsx
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

위 코드를 실행하면 화면에 “Hello, world”가 보일 겁니다.

### 렌더링 된 엘리먼트 업데이트하기
React 엘리먼트는 `불변객체`입니다. 엘리먼트를 생성한 이후에는 해당 엘리먼트의 자식이나 속성을 변경할 수 없습니다. 엘리먼트는 영화에서 하나의 프레임과 같이 특정 시점의 UI를 보여줍니다.  

지금까지 소개한 내용을 바탕으로 하면 UI를 업데이트하는 유일한 방법은 새로운 엘리먼트를 생성하고 이를 `ReactDOM.render()`로 전달하는 것입니다.  

예시로 똑딱거리는 시계를 살펴보겠습니다.  
```jsx
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

## 4. Components and Props
컴포넌트를 통해 UI를 재사용 가능한 개별적인 여러 조각으로 나누고,
각 조각을 개별적으로 살펴볼 수 있습니다. 이 페이지에서는 컴포넌트의 개념을 소개합니다.
자세한 컴포넌트 API 레퍼런스는 [여기](https://ko.reactjs.org/docs/react-component.html)에서 확인할 수 있습니다.
{: .notice}
      
개념적으로 컴포넌트는 JavaScript 함수와 유사합니다. “props”라고 하는 임의의 입력을 받은 후, 화면에 어떻게 표시되는지를 기술하는 React 엘리먼트를 반환합니다.
      
### 함수 컴포넌트와 클래스 컴포넌트
컴포넌트를 정의하는 가장 간단한 방법은 JavaScript 함수를 작성하는 것입니다.
```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```      
이 함수는 데이터를 가진 하나의 “props” (props는 속성을 나타내는 데이터입니다) 객체 인자를 받은 후 React 엘리먼트를 반환하므로 유효한 React 컴포넌트입니다. 이러한 컴포넌트는 JavaScript 함수이기 때문에 말 그대로 “함수 컴포넌트”라고 호칭합니다.
      
또한 ES6 class를 사용하여 컴포넌트를 정의할 수 있습니다.
```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```      
React의 관점에서 볼 때 위 두 가지 유형의 컴포넌트는 동일합니다.
      
### 컴포넌트 렌더링
이전까지는 React 엘리먼트를 DOM 태그로 나타냈습니다.
```jsx
const element = <div />;
```
React 엘리먼트는 사용자 정의 컴포넌트로도 나타낼 수 있습니다.
```jsx
const element = <Welcome name="Sara" />;
```
React가 사용자 정의 컴포넌트로 작성한 엘리먼트를 발견하면 JSX 어트리뷰트와 자식을 해당 컴포넌트에 단일 객체로 전달합니다. 이 객체를 “props”라고 합니다.  

다음은 페이지에 “Hello, Sara”를 렌더링하는 예시입니다.
```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```
이 예시에서는 다음과 같은 일들이 일어납니다.
      
1. `<Welcome name="Sara" />` 엘리먼트로 `ReactDOM.render()`를 호출합니다.
2. React는 `{name: 'Sara'}`를 props로 하여 `Welcome` 컴포넌트를 호출합니다.
3. Welcome 컴포넌트는 결과적으로 `<h1>Hello, Sara</h1>` 엘리먼트를 반환합니다.
4. React DOM은 `<h1>Hello, Sara</h1>` 엘리먼트와 일치하도록 DOM을 효율적으로 업데이트합니다.
>**주의:컴포넌트의 이름은 항상 대문자로 시작합니다.**  
React는 소문자로 시작하는 컴포넌트를 DOM 태그로 처리합니다. 예를 들어 `<div />`는 HTML div 태그를 나타내지만,
`<Welcome />`은 컴포넌트를 나타내며 범위 안에 Welcome이 있어야 합니다.
      
### 컴포넌트 합성
컴포넌트는 자신의 출력에 다른 컴포넌트를 참조할 수 있습니다. 이는 모든 세부 단계에서 동일한 추상 컴포넌트를 사용할 수 있음을 의미합니다. React 앱에서는 버튼, 폼, 다이얼로그, 화면 등의 모든 것들이 흔히 컴포넌트로 표현됩니다.  
      
예를 들어 `Welcome`을 여러 번 렌더링하는 `App` 컴포넌트를 만들 수 있습니다.
```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```
      
일반적으로 새 React 앱은 최상위에 단일 App 컴포넌트를 가지고 있습니다. 하지만 기존 앱에 React를 통합하는 경우에는 Button과 같은 작은 컴포넌트부터 시작해서 뷰 계층의 상단으로 올라가면서 점진적으로 작업해야 할 수 있습니다.

### 컴포넌트 추출
컴포넌트를 여러 개의 작은 컴포넌트로 나누는 것을 두려워하지 마세요.  

다음 Comment 컴포넌트를 살펴봅시다.
```jsx
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
이 컴포넌트는 `author`(객체), `text`(문자열) 및 `date`(날짜)를 props로 받은 후 소셜 미디어 웹 사이트의 코멘트를 나타냅니다.  

이 컴포넌트는 구성요소들이 모두 중첩 구조로 이루어져 있어서 변경하기 어려울 수 있으며, 각 구성요소를 개별적으로 재사용하기도 힘듭니다. 이 컴포넌트에서 몇 가지 컴포넌트를 추출하겠습니다.  

먼저 `Avatar`를 추출하겠습니다.
```jsx
function Avatar(props) {
  return (
    <img className="Avatar"
      src={props.user.avatarUrl}
      alt={props.user.name}
    />
  );
}
```
`Avatar` 는 자신이 `Comment` 내에서 렌더링 된다는 것을 알 필요가 없습니다. 따라서 props의 이름을 `author`에서 더욱 일반화된 `user`로 변경하였습니다.  

props의 이름은 사용될 context가 아닌 컴포넌트 자체의 관점에서 짓는 것을 권장합니다.  

이제 `Comment` 가 살짝 단순해졌습니다.
```jsx
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <Avatar user={props.author} />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
다음으로 `Avatar` 옆에 사용자의 이름을 렌더링하는 `UserInfo` 컴포넌트를 추출하겠습니다.
```jsx
function UserInfo(props) {
  return (
    <div className="UserInfo">
      <Avatar user={props.user} />
      <div className="UserInfo-name">
        {props.user.name}
      </div>
    </div>
  );
}
```
`Comment` 가 더욱 단순해졌습니다.
```jsx
function Comment(props) {
  return (
    <div className="Comment">
      <UserInfo user={props.author} />
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
처음에는 컴포넌트를 추출하는 작업이 지루해 보일 수 있습니다. 하지만 재사용 가능한 컴포넌트를 만들어 놓는 것은 더 큰 앱에서 작업할 때 두각을 나타냅니다. UI 일부가 여러 번 사용되거나 (Button, Panel, Avatar), UI 일부가 자체적으로 복잡한 (App, FeedStory, Comment) 경우에는 별도의 컴포넌트로 만드는 게 좋습니다.

### props는 읽기 전용입니다.
함수 컴포넌트나 클래스 컴포넌트 모두 컴포넌트의 자체 props를 수정해서는 안 됩니다. 다음 `sum` 함수를 살펴봅시다.
```js
function sum(a, b) {
  return a + b;
}
```
이런 함수들은 `순수 함수`라고 호칭합니다. 입력값을 바꾸려 하지 않고 항상 동일한 입력값에 대해 동일한 결과를 반환하기 때문입니다.

반면에 다음 함수는 자신의 입력값을 변경하기 때문에 순수 함수가 아닙니다.
```js
function withdraw(account, amount) {
  account.total -= amount;
}
```
React는 매우 유연하지만 한 가지 엄격한 규칙이 있습니다.  

모든 React 컴포넌트는 자신의 props를 다룰 때 반드시 순수 함수처럼 동작해야 합니다.
{: .notice--success}

물론 애플리케이션 UI는 동적이며 시간에 따라 변합니다. 다음 장에서는 `“state”`라는 새로운 개념을 소개합니다. React 컴포넌트는 `state`를 통해 위 규칙을 위반하지 않고 사용자 액션, 네트워크 응답 및 다른 요소에 대한 응답으로 시간에 따라 자신의 출력값을 변경할 수 있습니다.
