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

```jsx
ReactDOM.render(<h1>Hello, world!</h1>, document.getElementById("root"));
```

## 2. JSX 소개

아래의 변수 선언은 뭘까요?

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

개념적으로 Component는 JavaScript 함수와 유사합니다. `props`라고 하는 임의의 입력을 받은 후, 화면에 어떻게 표시되는지를 기술하는 React 엘리먼트를 반환합니다.

### 함수 Component와 클래스 Component

Component를 정의하는 가장 간단한 방법은 JavaScript 함수를 작성하는 것입니다.

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

이 함수는 데이터를 가진 하나의 `props` (props는 속성을 나타내는 데이터입니다) 객체 인자를 받은 후 React 엘리먼트를 반환하므로 유효한 React Component입니다. 이러한 Component는 JavaScript 함수이기 때문에 말 그대로 “함수 Component”라고 호칭합니다.

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

React가 사용자 정의 Component로 작성한 엘리먼트를 발견하면 JSX 어트리뷰트와 자식을 해당 Component에 단일 객체로 전달합니다. 이 객체를 `props`라고 합니다.

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
{: .notice--info}

React Component는 `state`를 통해 위 규칙을 위반하지 않고 사용자 액션, 네트워크 응답 및 다른 요소에 대한 응답으로 시간에 따라 자신의 출력값을 변경할 수 있습니다.

## 5. State and Lifecycle

위의 3. 엘리먼트 렌더링에서 작성했던 째깍거리는 시계 예시를 다시 살펴보겠습니다.  
렌더링 된 출력값을 변경하기 위해 root.render()를 매초 호출했습니다.

```jsx
const root = ReactDOM.createRoot(document.getElementById("root"));

function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  root.render(element);
}

setInterval(tick, 1000);
```

우선 Clock의 캡슐화를 아래와 같이 진행합니다.

```jsx
const root = ReactDOM.createRoot(document.getElementById("root"));

function Clock(props) {
  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {props.date.toLocaleTimeString()}.</h2>
    </div>
  );
}

function tick() {
  root.render(<Clock date={new Date()} />);
}

setInterval(tick, 1000);
```

그런데 여기서 Clock 이 타이머를 설정하고 매 초 UI를 업데이트 하는 것은 Clock 의 구현 세부사항이어야 합니다.

이상적으로 아래와 같은 한 번만 코드를 작성하고 Clock이 스스로 업데이트하도록 만들려고 합니다.

```jsx
root.render(<Clock />);
```

이것을 구현하기 위해서 Clock Component에 `state`를 추가해야 합니다.

state는 props와 유사하지만, 비공개이며 Component에 의해 완전히 제어됩니다.

### 함수에서 클래스로 변환하기

현재는 함수형 Component가 대세이긴 하지만 실질적인 동작 메카니즘을 설명하는데는 Class형이 더 명확하기 때문에 `Clock`을 Component 형태로 변경하겠습니다.

```jsx
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

render method는 업데이트가 발생할 때마다 호출되지만,  
`<Clock />`을 동일한 DOM 노드에 렌더링하는 한 Clock 클래스의 `단일 인스턴스만` 사용됩니다.  
이를 통해 로컬 상태 및 수명 주기 메서드와 같은 추가 기능을 사용할 수 있습니다.

### 클래스에 로컬 State 추가하기

세 단계에 걸쳐서 date를 props에서 state로 이동해 보겠습니다.

1.`render()` 메서드 안에 있는 `this.props.date`를 `this.state.date`로 변경합니다.

```jsx
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

2.초기 this.state를 지정하는 class constructor를 추가합니다.  
 클래스 Component는 항상 props로 기본 constructor를 호출해야 합니다.

```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = { date: new Date() };
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

3.`<Clock />` 요소에서 date prop을 삭제합니다.

```jsx
root.render(<Clock />);
```

결과는 다음 코드와 같습니다.

```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = { date: new Date() };
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<Clock />);
```

### 생명주기 메서드를 클래스에 추가하기

많은 Component가 있는 애플리케이션에서 Component가 삭제될 때 해당 Component가 사용 중이던 리소스를 확보하는 것이 중요합니다.  
`Clock`이 처음 DOM에 렌더링 될 때마다 타이머를 설정하려고 합니다. 이것은 React에서 `mounting`이라고 합니다.  
또한 `Clock`에 의해 생성된 DOM이 삭제될 때마다 타이머를 해제하려고 합니다. 이것은 React에서 `unmounting`이라고 합니다.

```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = { date: new Date() };
  }

  // componentDidMount() 메서드는 Component 출력물이 DOM에 렌더링 된 후에 실행됩니다.
  // 이 장소가 타이머를 설정하기에 좋은 장소입니다.
  componentDidMount() {
    this.timerID = setInterval(() => this.tick(), 1000);
  }

  // componentWillUnmount() 생명주기 메서드에서 타이머를 해제해 사용 중이던 리소스를 확보합니다.
  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  // state의 data 매초 업데이트합니다.
  tick() {
    this.setState({
      date: new Date(),
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```

React의 동작 순서를 요약을 하면 아래와 같습니다. 정말 중요합니다.
{: .notice--info}

1. `<Clock />`이 `root.render()`로 전달되었을 때 React는 Clock Component의 constructor를 호출합니다. `Clock`이 현재 시각을 표시해야 하기 때문에 현재 시각이 포함된 객체로 `this.state`를 초기화합니다. 나중에 이 state를 매초 업데이트할 것입니다.
2. React는 Clock Component의 `render()` 메서드를 호출합니다. 이를 통해 React는 화면에 표시되어야 할 내용을 알게 됩니다. 그 다음 React는 `Clock`의 렌더링 출력값을 일치시키기 위해 `DOM을 업데이트`합니다.
3. `Clock` 출력값이 DOM에 삽입되면, React는 `componentDidMount()` 생명주기 메서드를 호출합니다. 그 안에서 Clock Component는 매초 Component의 `tick()` 메서드를 호출하기 위한 타이머를 설정하도록 브라우저에 요청합니다.
4. 매초 브라우저가 `tick()` 메서드를 호출합니다. 그 안에서 `Clock` Component는 `setState()`에 현재 시각을 포함하는 객체를 호출하면서 UI 업데이트를 진행합니다. `setState()` 호출 덕분에 React는 state가 변경된 것을 인지하고 화면에 표시될 내용을 알아내기 위해 `render()` 메서드를 다시 호출합니다. 이 때 `render()` 메서드 안의 `this.state.date`가 달라지고 렌더링 출력값은 업데이트된 시각을 포함합니다. React는 이에 따라 DOM을 업데이트합니다.
5. `Clock` Component가 DOM으로부터 한 번이라도 삭제된 적이 있다면 React는 타이머를 멈추기 위해 `componentWillUnmount()` 생명주기 메서드를 호출합니다.

### State를 올바르게 사용하기

1.직접 State를 수정하지 마세요

```jsx
// Wrong : 이 코드는 Component를 다시 렌더링하지 않습니다.
this.state.comment = "Hello";

// Correct : this.state를 지정할 수 있는 유일한 공간은 바로 constructor입니다.
this.setState({ comment: "Hello" });
```

2.State 업데이트는 비동기적일 수도 있습니다.
React는 성능을 위해 여러 setState() 호출을 단일 업데이트로 한꺼번에 처리할 수 있습니다.  
this.props와 this.state가 비동기적으로 업데이트될 수 있기 때문에 다음 state를 계산할 때 해당 값에 의존해서는 안 됩니다.  
예를 들어, 다음 코드는 카운터 업데이트에 실패할 수 있습니다.

```jsx
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

이를 수정하기 위해 객체보다는 함수를 인자로 사용하는 다른 형태의 setState()를 사용합니다.  
그 함수는 이전 state를 첫 번째 인자로 받아들일 것이고, 업데이트가 적용된 시점의 props를 두 번째 인자로 받아들일 것입니다.

```jsx
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment,
}));

// 위의 arrow function은 일반적인 함수에서도 정상적으로 작동합니다.

// Correct
this.setState(function (state, props) {
  return {
    counter: state.counter + props.increment,
  };
});
```

### State 업데이트는 병합됩니다

setState()를 호출할 때 React는 제공한 객체를 현재 state로 병합합니다.  
예를 들어, state는 다양한 독립적인 변수를 포함할 수 있습니다.

```jsx
  constructor(props) {
    super(props);
    this.state = {
      posts: [],
      comments: []
    };
  }
```

별도의 setState() 호출로 이러한 변수를 독립적으로 업데이트할 수 있습니다.

```jsx
  componentDidMount() {
    fetchPosts().then(response => {
      this.setState({
        posts: response.posts
      });
    });

    fetchComments().then(response => {
      this.setState({
        comments: response.comments
      });
    });
  }
```

병합은 얕게 이루어지기 때문에 this.setState({comments})는 this.state.posts에 영향을 주진 않지만 this.state.comments는 완전히 대체됩니다.

### 데이터가 아래로 흐릅니다.

부모 Component나 자식 Component 모두 특정 Component가 `stateful`인지 `stateless`인지 알 수 없으며, 함수로 정의되었는지 클래스로 정의되었는지도 신경 쓰지 않아야 합니다.  
이것이 `state`를 로컬 또는 캡슐화라고 부르는 이유입니다. `state`를 소유하고 설정하는 Component 이외의 다른 Component에서 액세스할 수 없습니다.  
Component는 자신의 `state`를 자식 Component에 `props`로 내려줄 수 있습니다.

## 6. 이벤트 제어하기

- React의 이벤트는 소문자 대신 캐멀 케이스(camelCase)를 사용합니다.
- JSX를 사용하여 문자열이 아닌 함수로 이벤트 핸들러를 전달합니다.

```jsx
<button onClick={activateLasers}>Activate Lasers</button>
```

React를 사용할 때 일반적으로 DOM 요소가 생성된 후에 리스너를 추가하기 위해 addEventListener 를 호출할 필요가 없습니다. 대신 요소가 처음 렌더링될 때 리스너를 제공합니다.

- 기본동작을 방지하기 위해서는 `e.preventDefault()` 함수를 반드시 써줘야 합니다.
- 이벤트 핸들러는 구성 요소가 가질 수 있는 모든 하위 이벤트도 포착합니다. 전파를 중지하기 위해서는 `e.stopPropagation()` 함수를 써줘야합니다.

## 7. 조건부 렌더링

### 논리 && 연산자로 If를 인라인으로 표현하기

```jsx
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 && (
        <h2>You have {unreadMessages.length} unread messages.</h2>
      )}
    </div>
  );
}

const messages = ["React", "Re: React", "Re:Re: React"];

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<Mailbox unreadMessages={messages} />);
```

자바스크립트에서 true && expression 은 항상 expression 으로 평가되고, false && expression 은 항상 false 로 평가되기 때문에 이 코드는 동작합니다.  
따라서 && 뒤의 엘리먼트는 조건이 true일때 출력이 됩니다. 조건이 false라면 React는 무시하고 건너뜁니다.  
잘못된 표현식을 반환하면 여전히 && 뒤에 있는 표현식은 건너뛰지만 잘못된 표현식이 반환된다는 것에 주의해주세요. 아래 예시에서, `<div>0</div>`가 render 메서드에서 반환됩니다.

```jsx
render() {
  const count = 0;
  return (
    <div>
      {count && <h1>Messages: {count}</h1>}
    </div>
  );
}

```

### 조건부 연산자를 사용한 인라인 If-Else

인라인으로 요소를 조건부 렌더링하는 다른 방법은 자바스크립트의 조건부 연산자인 condition ? true : false 를 사용하는 것입니다.

### Component가 렌더링 되지 못하도록 방지

흔하지 않지만 Component가 다른 Component에 의해 렌더링 되었더라도 이를 숨길 수 있습니다. 이렇게 하려면 렌더 출력 대신 null 을 반환합니다.

Component의 render 메서드에서 null을 반환해도 Component의 라이프사이클 메서드 실행에는 영향을 미치지 않습니다. 예를 들어 `componentDidUpdate는` 여전히 호출됩니다.

## 8. 리스트와 Key

### Key

Key는 React가 어떤 항목을 변경, 추가 또는 삭제할지 식별하는 것을 돕습니다. key는 엘리먼트에 안정적인 고유성을 부여하기 위해 배열 내부의 엘리먼트에 지정해야 합니다.

항목의 순서가 바뀔 수 있는 경우 key에 인덱스를 사용하는 것은 권장하지 않습니다. 이로 인해 성능이 저하되거나 컴포넌트의 state와 관련된 문제가 발생할 수 있습니다.  
리스트 항목에 명시적으로 key를 지정하지 않으면 React는 기본적으로 인덱스를 key로 사용합니다.

키는 React에게 힌트를 제공하지만 Component로 전달되지는 않습니다. 만약 Component에 동일한 값이 필요하다면 명시적으로 다른 이름의 `props`으로 전달하길 바랍니다.

## 9. Form

HTML 폼 엘리먼트는 폼 엘리먼트 자체가 내부 상태를 가지기 때문에, React의 다른 DOM 엘리먼트와 다르게 동작합니다. 예를 들어, 순수한 HTML에서 이 폼은 name을 입력받습니다

```jsx
<form>
  <label>
    Name:
    <input type="text" name="name" />
  </label>
  <input type="submit" value="Submit" />
</form>
```

이 폼은 사용자가 폼을 제출하면 새로운 페이지로 이동하는 기본 HTML 폼 동작을 수행합니다. React에서 동일한 동작을 원한다면 그대로 사용하면 됩니다. 그러나 대부분의 경우, JavaScript 함수로 폼의 제출을 처리하고 사용자가 폼에 입력한 데이터에 접근하도록 하는 것이 편리합니다. 이를 위한 표준 방식은 “제어 컴포넌트 (controlled components)“라고 불리는 기술을 이용하는 것입니다.

### 제어 컴포넌트 (Controlled Component)

HTML에서 <input>, <textarea>, <select>와 같은 폼 엘리먼트는 일반적으로 사용자의 입력을 기반으로 자신의 state를 관리하고 업데이트합니다. React에서는 변경할 수 있는 state가 일반적으로 컴포넌트의 state 속성에 유지되며 setState()에 의해 업데이트됩니다.

우리는 React state를 “신뢰 가능한 단일 출처 (single source of truth)“로 만들어 두 요소를 결합할 수 있습니다. 그러면 폼을 렌더링하는 React 컴포넌트는 폼에 발생하는 사용자 입력값을 제어합니다. 이러한 방식으로 React에 의해 값이 제어되는 입력 폼 엘리먼트를 “제어 컴포넌트 (controlled component)“라고 합니다.

예를 들어, 이전 예시가 전송될 때 이름을 기록하길 원한다면 폼을 제어 컴포넌트 (controlled component)로 작성할 수 있습니다.

```jsx
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: "" };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({ value: event.target.value });
  }

  handleSubmit(event) {
    alert("A name was submitted: " + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input
            type="text"
            value={this.state.value}
            onChange={this.handleChange}
          />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

value 어트리뷰트는 폼 엘리먼트에 설정되므로 표시되는 값은 항상 this.state.value가 되고 React state는 신뢰 가능한 단일 출처 (single source of truth)가 됩니다. React state를 업데이트하기 위해 모든 키 입력에서 handleChange가 동작하기 때문에 사용자가 입력할 때 보여지는 값이 업데이트됩니다.

제어 컴포넌트로 사용하면, input의 값은 항상 React state에 의해 결정됩니다. 코드를 조금 더 작성해야 한다는 의미이지만, 다른 UI 엘리먼트에 input의 값을 전달하거나 다른 이벤트 핸들러에서 값을 재설정할 수 있습니다.

### textarea 태그

HTML에서 <textarea> 엘리먼트는 텍스트를 자식으로 정의합니다.

```jsx
<textarea>Hello there, this is some text in a text area</textarea>
```

React에서 <textarea>는 value 어트리뷰트를 대신 사용합니다. 이렇게하면 <textarea>를 사용하는 폼은 한 줄 입력을 사용하는 폼과 비슷하게 작성할 수 있습니다.

```jsx
class EssayForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      value: "Please write an essay about your favorite DOM element.",
    };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({ value: event.target.value });
  }

  handleSubmit(event) {
    alert("An essay was submitted: " + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Essay:
          <textarea value={this.state.value} onChange={this.handleChange} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

### select 태그

HTML에서 <select>는 드롭 다운 목록을 만듭니다. 예를 들어, 이 HTML은 과일 드롭 다운 목록을 만듭니다.

```jsx
<select>
  <option value="grapefruit">Grapefruit</option>
  <option value="lime">Lime</option>
  <option selected value="coconut">
    Coconut
  </option>
  <option value="mango">Mango</option>
</select>
```

selected 옵션이 있으므로 Coconut 옵션이 초기값이 되는 점을 주의해주세요. React에서는 selected 어트리뷰트를 사용하는 대신 최상단 select태그에 value 어트리뷰트를 사용합니다. 한 곳에서 업데이트만 하면되기 때문에 제어 컴포넌트에서 사용하기 더 편합니다. 아래는 예시입니다.

```jsx
class FlavorForm extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: "coconut" };

    this.handleChange = this.handleChange.bind(this);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleChange(event) {
    this.setState({ value: event.target.value });
  }

  handleSubmit(event) {
    alert("Your favorite flavor is: " + this.state.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Pick your favorite flavor:
          <select value={this.state.value} onChange={this.handleChange}>
            <option value="grapefruit">Grapefruit</option>
            <option value="lime">Lime</option>
            <option value="coconut">Coconut</option>
            <option value="mango">Mango</option>
          </select>
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

### file input 태그

HTML에서 <input type="file">는 사용자가 하나 이상의 파일을 자신의 장치 저장소에서 서버로 업로드하거나 File API를 통해 JavaScript로 조작할 수 있습니다.

```jsx
<input type="file" />
```

값이 읽기 전용이기 때문에 React에서는 비제어 컴포넌트입니다. 문서 뒷부분에서 다른 비제어 컴포넌트와 함께 설명하고 있습니다.

## 10. State 끌어올리기

React에서 state를 공유하는 일은 그 값을 필요로 하는 컴포넌트 간의 가장 가까운 공통 조상으로 state를 끌어올림으로써 이뤄낼 수 있습니다. 이렇게 하는 방법을 “state 끌어올리기”라고 부릅니다.

React 애플리케이션 안에서 변경이 일어나는 데이터에 대해서는 “진리의 원천(source of truth)“을 하나만 두어야 합니다. 보통의 경우, state는 렌더링에 그 값을 필요로 하는 컴포넌트에 먼저 추가됩니다. 그러고 나서 다른 컴포넌트도 역시 그 값이 필요하게 되면 그 값을 그들의 가장 가까운 공통 조상으로 끌어올리면 됩니다. 다른 컴포넌트 간에 존재하는 state를 동기화시키려고 노력하는 대신 하향식 데이터 흐름에 기대는 걸 추천합니다.

state를 끌어올리는 작업은 양방향 바인딩 접근 방식보다 더 많은 “보일러 플레이트” 코드를 유발하지만, 버그를 찾고 격리하기 더 쉽게 만든다는 장점이 있습니다. 어떤 state든 간에 특정 컴포넌트 안에서 존재하기 마련이고 그 컴포넌트가 자신의 state를 스스로 변경할 수 있으므로 버그가 존재할 수 있는 범위가 크게 줄어듭니다. 또한 사용자의 입력을 거부하거나 변형하는 자체 로직을 구현할 수도 있습니다.

어떤 값이 props 또는 state로부터 계산될 수 있다면, 아마도 그 값을 state에 두어서는 안 됩니다. 예를 들어 celsiusValue와 fahrenheitValue를 둘 다 저장하는 대신, 단지 최근에 변경된 temperature와 scale만 저장하면 됩니다. 다른 입력 필드의 값은 항상 그 값들에 기반해서 render() 메서드 안에서 계산될 수 있습니다. 이를 통해 사용자 입력값의 정밀도를 유지한 채 다른 필드의 입력값에 반올림을 지우거나 적용할 수 있게 됩니다.

UI에서 무언가 잘못된 부분이 있을 경우, React Developer Tools를 이용하여 props를 검사하고 state를 갱신할 책임이 있는 컴포넌트를 찾을 때까지 트리를 따라 탐색해보세요. 이렇게 함으로써 소스 코드에서 버그를 추적할 수 있습니다.

## 11. 합성 (Composition) vs 상속 (Inheritance)

React는 강력한 합성 모델을 가지고 있으며, 상속 대신 합성을 사용하여 컴포넌트 간에 코드를 재사용하는 것이 좋습니다.
{: .notice--info}

### 컴포넌트에서 다른 컴포넌트를 담기

어떤 컴포넌트들은 어떤 자식 엘리먼트가 들어올 지 미리 예상할 수 없는 경우가 있습니다. 범용적인 ‘박스’ 역할을 하는 Sidebar 혹은 Dialog와 같은 컴포넌트에서 특히 자주 볼 수 있습니다.

이러한 컴포넌트에서는 특수한 children prop을 사용하여 자식 엘리먼트를 출력에 그대로 전달하는 것이 좋습니다.

```jsx
function FancyBorder(props) {
  return (
    <div className={"FancyBorder FancyBorder-" + props.color}>
      {props.children}
    </div>
  );
}
```

이러한 방식으로 다른 컴포넌트에서 JSX를 중첩하여 임의의 자식을 전달할 수 있습니다.

```jsx
function WelcomeDialog() {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">Welcome</h1>
      <p className="Dialog-message">Thank you for visiting our spacecraft!</p>
    </FancyBorder>
  );
}
```

`<FancyBorder>` JSX 태그 안에 있는 것들이 FancyBorder 컴포넌트의 children prop으로 전달됩니다. FancyBorder는 {props.children}을 `<div>` 안에 렌더링하므로 전달된 엘리먼트들이 최종 출력됩니다.

흔하진 않지만 종종 컴포넌트에 여러 개의 “구멍”이 필요할 수도 있습니다. 이런 경우에는 children 대신 자신만의 고유한 방식을 적용할 수도 있습니다.

```jsx
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">{props.left}</div>
      <div className="SplitPane-right">{props.right}</div>
    </div>
  );
}

function App() {
  return <SplitPane left={<Contacts />} right={<Chat />} />;
}
```

`<Contacts />`와 <`Chat />`같은 React 엘리먼트는 단지 객체이기 때문에 다른 데이터처럼 prop으로 전달할 수 있습니다. 이러한 접근은 다른 라이브러리의 “슬롯 (slots)“과 비슷해보이지만 React에서 prop으로 전달할 수 있는 것에는 제한이 없습니다.

### 특수화

때로는 어떤 컴포넌트의 “특수한 경우”인 컴포넌트를 고려해야 하는 경우가 있습니다. 예를 들어, WelcomeDialog는 Dialog의 특수한 경우라고 할 수 있습니다.

React에서는 이 역시 합성을 통해 해결할 수 있습니다. 더 “구체적인” 컴포넌트가 “일반적인” 컴포넌트를 렌더링하고 props를 통해 내용을 구성합니다.

```jsx
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">{props.title}</h1>
      <p className="Dialog-message">{props.message}</p>
    </FancyBorder>
  );
}

function WelcomeDialog() {
  return (
    <Dialog title="Welcome" message="Thank you for visiting our spacecraft!" />
  );
}
```

합성은 클래스로 정의된 컴포넌트에서도 동일하게 적용됩니다.

```jsx
function Dialog(props) {
  return (
    <FancyBorder color="blue">
      <h1 className="Dialog-title">{props.title}</h1>
      <p className="Dialog-message">{props.message}</p>
      {props.children}
    </FancyBorder>
  );
}

class SignUpDialog extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
    this.handleSignUp = this.handleSignUp.bind(this);
    this.state = { login: "" };
  }

  render() {
    return (
      <Dialog
        title="Mars Exploration Program"
        message="How should we refer to you?"
      >
        <input value={this.state.login} onChange={this.handleChange} />
        <button onClick={this.handleSignUp}>Sign Me Up!</button>
      </Dialog>
    );
  }

  handleChange(e) {
    this.setState({ login: e.target.value });
  }

  handleSignUp() {
    alert(`Welcome aboard, ${this.state.login}!`);
  }
}
```

### 그렇다면 상속은?

Facebook에서는 수천 개의 React 컴포넌트를 사용하지만, 컴포넌트를 상속 계층 구조로 작성을 권장할만한 사례를 아직 찾지 못했습니다.

props와 합성은 명시적이고 안전한 방법으로 컴포넌트의 모양과 동작을 커스터마이징하는데 필요한 모든 유연성을 제공합니다. 컴포넌트가 원시 타입의 값, React 엘리먼트 혹은 함수 등 어떠한 props도 받을 수 있다는 것을 기억하세요.

## 12. React로 생각하기

[https://ko.reactjs.org/docs/thinking-in-react.html](https://ko.reactjs.org/docs/thinking-in-react.html)
