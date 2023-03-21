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
ReactDOM.render(<h1>Hello, world!</h1>, document.getElementById("root"));
```

## 2. JSX 소개

아래의 변수 선언은 뭘까요?
{: .notice}

```jsx
const element = <h1>Hello, world!</h1>;
```

JSX는 `JavaScript XML`의 약자로, React에서 사용되는 자바스크립트 확장 문법입니다. JSX는 HTML과 유사한 구문을 사용하여, UI 구성 요소를 정의하고 렌더링할 수 있도록 해줍니다.

JSX는 일반적으로 React Component의 render 메소드에서 사용되며, React 엘리먼트를 생성하는 데 사용됩니다. JSX 구문은 일반적인 자바스크립트 코드와 함께 사용할 수 있으며, 컴파일러를 사용하여 일반 자바스크립트 코드로 변환됩니다.

React에서 JSX를 사용하면 개발자는 더욱 직관적이고 가독성 높은 코드를 작성할 수 있으며, 컴파일러가 코드를 자동으로 최적화하여 성능을 향상시킬 수도 있습니다.

### JSX에 표현식 포함하기

JSX에서는 javascript변수를 `{}`묶어 사용할 수 있습니다.

```jsx
const name = "Josh Perez";
const element = <h1>Hello, {name}</h1>;
```

### JSX 속성 정의

따옴표를 사용하여 문자열을 속성으로 지정하거나  
JavaScript 표현식을 포함하기 위해 `{}`를 사용할 수도 있습니다

```jsx
const element = <div tabIndex="0"></div>;
const element = <img src={user.avatarUrl}></img>;
```

이 때, 어트리뷰트에 JavaScript 표현식을 삽입할 때 중괄호 주변에 따옴표를 입력하지 마세요. 따옴표(문자열 값에 사용) 또는 중괄호(표현식에 사용) 중 하나만 사용하고, 동일한 어트리뷰트에 두 가지를 동시에 사용하면 안 됩니다.

> **주의**  
> JSX는 HTML보다 JavaScript에 더 가깝기 때문에
> React DOM은 HTML 속성 대신 camelCase 명명 규칙을 사용합니다.  
> 예를 들어 class는 `className`, 그리고 tabindex는 `tabIndex`로 사용한다.

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
const element = <h1 className="greeting">Hello, world!</h1>;
```

```jsx
const element = React.createElement(
  "h1",
  { className: "greeting" },
  "Hello, world!"
);
```

`React.createElement()` 버그 없는 코드를 작성하는 데 도움이 되는 몇 가지 검사를 수행하지만 기본적으로 다음과 같은 객체를 생성합니다.

```jsx
// Note: this structure is simplified
const element = {
  type: "h1",
  props: {
    className: "greeting",
    children: "Hello, world!",
  },
};
```

이러한 객체를 “React elements”라고 하며, 화면에서 보고 싶은 것을 나타내는 표현이라 생각하면 됩니다. React는 이 객체를 읽어서, DOM을 구성하고 최신 상태로 유지하는 데 사용합니다.

## 3. 엘리먼트 렌더링

엘리먼트는 React 앱의 가장 작은 단위입니다.
{: .notice}

엘리먼트는 화면에 표시할 내용을 기술합니다.

### DOM에 엘리먼트 렌더링하기

HTML 파일 어딘가에 `<div>`가 있다고 가정해 봅시다.

```jsx
<div id="root"></div>
```

이 안에 들어가는 모든 엘리먼트를 React DOM에서 관리하기 때문에 이것을 “루트(root)” DOM 노드라고 부릅니다.

React로 구현된 애플리케이션은 일반적으로 하나의 루트 DOM 노드가 있습니다. React를 기존 앱에 통합하려는 경우 원하는 만큼 많은 수의 독립된 루트 DOM 노드가 있을 수 있습니다.

React 엘리먼트를 렌더링 하기 위해서는 우선 DOM 엘리먼트를 ReactDOM.createRoot()에 전달한 다음, React 엘리먼트를 root.render()에 전달해야 합니다.

```jsx
const root = ReactDOM.createRoot(document.getElementById("root"));
const element = <h1>Hello, world</h1>;
root.render(element);
```

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
  ReactDOM.render(element, document.getElementById("root"));
}
setInterval(tick, 1000);
```

위 함수는 `setInterval()` 콜백을 이용해 초마다 `ReactDOM.render()`를 호출합니다.

> **주의**  
> 실제로 대부분의 React 앱은 ReactDOM.render()를 한 번만 호출합니다. 다음 장에서는 이와 같은 코드가 `stateful Component`에 어떻게 캡슐화되는지 설명합니다.

### 변경된 부분만 업데이트하기

React DOM은 해당 엘리먼트와 그 자식 엘리먼트를 이전의 엘리먼트와 비교하고 DOM을 원하는 상태로 만드는데 필요한 경우에만 DOM을 업데이트합니다.

개발자 도구를 이용해 마지막 예시를 살펴보면 이를 확인할 수 있습니다.

매초 전체 UI를 다시 그리도록 엘리먼트를 만들었지만 React DOM은 내용이 변경된 텍스트 노드만 업데이트했습니다.

경험에 비추어 볼 때 특정 시점에 UI가 어떻게 보일지 고민하는 이런 접근법은 시간의 변화에 따라 UI가 어떻게 변화할지 고민하는 것보다 더 많은 수의 버그를 없앨 수 있습니다.

## 4. Components and Props

개념적으로 Component는 JavaScript 함수와 유사합니다. “props”라고 하는 임의의 입력을 받은 후, 화면에 어떻게 표시되는지를 기술하는 React 엘리먼트를 반환합니다.

### 함수 Component와 클래스 Component

Component를 정의하는 가장 간단한 방법은 JavaScript 함수를 작성하는 것입니다.

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

이 함수는 데이터를 가진 하나의 “props” (props는 속성을 나타내는 데이터입니다) 객체 인자를 받은 후 React 엘리먼트를 반환하므로 유효한 React Component입니다. 이러한 Component는 JavaScript 함수이기 때문에 말 그대로 “함수 Component”라고 호칭합니다.

또한 ES6 class를 사용하여 Component를 정의할 수 있습니다.

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

React의 관점에서 볼 때 위 두 가지 유형의 Component는 동일합니다.

### Component 렌더링

이전까지는 React 엘리먼트를 DOM 태그로 나타냈습니다.

```jsx
const element = <div />;
```

React 엘리먼트는 사용자 정의 Component로도 나타낼 수 있습니다.

```jsx
const element = <Welcome name="Sara" />;
```

React가 사용자 정의 Component로 작성한 엘리먼트를 발견하면 JSX 어트리뷰트와 자식을 해당 Component에 단일 객체로 전달합니다. 이 객체를 “props”라고 합니다.

다음은 페이지에 “Hello, Sara”를 렌더링하는 예시입니다.

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(element, document.getElementById("root"));
```

이 예시에서는 다음과 같은 일들이 일어납니다.

1. `<Welcome name="Sara" />` 엘리먼트로 `ReactDOM.render()`를 호출합니다.
2. React는 `{name: 'Sara'}`를 props로 하여 `Welcome` Component를 호출합니다.
3. Welcome Component는 결과적으로 `<h1>Hello, Sara</h1>` 엘리먼트를 반환합니다.
4. React DOM은 `<h1>Hello, Sara</h1>` 엘리먼트와 일치하도록 DOM을 효율적으로 업데이트합니다.
   > **주의 : Component의 이름은 항상 대문자로 시작합니다.**  
   > React는 소문자로 시작하는 Component를 DOM 태그로 처리합니다.  
   > 예를 들어 `<div />`는 HTML div 태그를 나타내지만,  
   > `<Welcome />`은 Component를 나타내며 범위 안에 Welcome이 있어야 합니다.

### props는 읽기 전용입니다.

React는 매우 유연하지만 한 가지 엄격한 규칙이 있습니다.

모든 React Component는 자신의 props를 다룰 때 반드시 순수 함수처럼 동작해야 합니다.
{: .notice--success}

React Component는 `state`를 통해 위 규칙을 위반하지 않고 사용자 액션, 네트워크 응답 및 다른 요소에 대한 응답으로 시간에 따라 자신의 출력값을 변경할 수 있습니다.

## 5. State and Lifecycle

Clock 이 타이머를 설정하고 매 초 UI를 업데이트 하는 것은 Clock 의 구현 세부사항이어야 합니다.

많은 컴포넌트를 가진 어플리케이션에서, 컴포넌트가 제거될 때 리소스를 풀어주는 건 아주 중요한 일입니다.

### 데이터가 아래로 흐릅니다.

부모 컴포넌트나 자식 컴포넌트는 특정 컴포넌트의 state 유무를 알 수 없으며 해당 컴포넌트가 함수나 클래스로 선언되었는 지 알 수 없습니다.

이는 state가 로컬이라고 부르거나 캡슐화된 이유입니다. 컴포넌트 자신 외에는 접근할 수 없습니다.

컴포넌트는 자신의 state를 자식 컴포넌트에 props 로 내려줄 수 있습니다.
