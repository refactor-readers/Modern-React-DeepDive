# 2장 : 리액트 핵심 요소 깊게 살펴보기

## 2.1 JSX란?

XML과 유사한 내장형 구문

- 리액트에 종속적이지 않고, 독자적인 문법이다.
- 페이스북이 개발했고, ECMAScript 표준의 일부는 아니다.
- 반드시 트랜스파일을 거쳐야 의미있는 JavaScript가 된다.

#### 설계 목적

- 다양한 트랜스파일러에서 다양한 속성을 가진 트리구조를 토큰화해 ECMAScript로 변환하는데 초점이 맞춰져 있다.

=> JavaScript 내부에서 표현하기 까다로웠던 XML 스타일의 트리 구문을 작성하는데 도움을 주는 새로운 문법이다.

### 2.1.1 JSX의 정의

#### 구성

4가지 Components로 이루어져있다.

- JSXElement
- JSXAttributes
- JSXChildren
- JSXStrings

1. `JSXElement`

   HTML의 요소(element)와 비슷한 역할을 한다.

   - 되기 위한 형태의 조건

   ```tsx
    // JSXOpeningElement
    // JSXClosingELement가 동일한 요소로 같은 단계에서 선언돼있어야 함
    <JSXElement JSXAttributes(optional)>

    // JSXClosingElement
    // JSXOpeningElement가 종료됐음을 알리는 요소로 반드시 이와 쌍으로 사용되어야 함
    <JSXElement />

    // JSXSelfClosingElement
    // 요소가 시작되고, 스스로 종료되는 형태. 자식을 포함할 수 없는 형태를 의미
    <JSXElement JSXAttributes(optional) />

    // JSXFragment
    // 아무런 요소가 없는 형태로, JSXSelfClosingElement 형태를 띌 수는 없다.
    <>JSXChildren(optional)</>
   ```

#### JSX의 요소명이 대문자로 시작해야 하는 이유

: HTML 태그명과 사용자가 만든 컴포넌트 태그명을 구분 짓기 위해

2. JSXElementName
   JSXElement의 요소 이름으로 쓸 수 있는 것

- 이름으로 가능한 것
  - JSXIdentifier : JSX내부에서 사용할 수 있는 식별자를 의미. <br />
    => 자바스크립트 식별자 규칙과 동일
  - JSXNamespacedName: JSXIdentifier의 조합, 즉 `:`을 통해 서로 다른 식별자를 이어주는 것도 하나의 식별자로 취급된다.
    - `:`를 통해 묶을 수 있는 것은 한 개뿐이다.
  - JSXMemberExpression: JSXIdentifier.JSXIdentifier의 조합. 즉 `.`을 통해 서로 다른 식별자를 이어주는 것도 하나의 식별자로 취급된다.
    - `.`을 여러 개 이어서 하는 것도 가능하다.
    - JSXNamespacedName과 이어서 사용하는 것은 불가능하다.

3. JSXAttributes

   JSXElement에 부여할 수 있는 속성

   - 필수값 X, 존재하지 않아도 에러가 나지 않음
   - 종류
     - JSXSpreadAttributes: JavaScript의 전개 연산자와 동일한 역할 (모든 표현식을 전개할 수 있음\_AssignmentExpression)
     - JSXAttributes : 속성을 나타내는 키와 값으로 짝을 이루어서 표현한다. 키는 JSXattributeName, 값은 JSXAttributesValue
       - JSXAttributeName: 속성의 키 값. `:`로 키를 나타낼 수 있음
       - JSXAttributeValue: 속성의 키에 할당할 수 있는 값
         - 큰따옴표, 작은따옴표로 구성되거나
         - AssignmentExpression이거나 (표현식)
         - JSXElement

4. JSXChildren

   JSXElement의 자식 값

   - JSX는 트리구조를 나타내기 위해 만들어졌기 때문에 JSX로 부모와 자식을 나타낼 수 있고, 그 자식을 JSXChildren이라고 함

   - JSXChildren이 될 수 있는 것들
     - JSXChild: JSXChildren을 이루는 기본단위
       - JSXChildren은 JSXChild를 0개 이상 가질 수 있다.
       - JSXText : `{, <, >,}`을 제외한 문자열, 이는 다른 JSX 문법과 혼동을 줄 수 있기 때문에 이 문자열을 표현하고 싶다면 문자열로 표현하는 방법이 있다.
     - JSXElement
     - JSXFragment
     - { JSXChildExpression (optional)} : 이 JSXChildExpression은 자바스크립트의 AssignmentExpression을 의미한다. (즉시 실행 함수 형태)

5. JSXStrings

   HTML에서 가능한 문자열은 모두 JSXString에서도 가능하다.

   - 개발자가 HTML의 내용을 손쉽게 JSX로 가져올 수 있도록 **의도적으로 설계된 부분**이다.
   - 문자열의 정의는 작은따옴표, 큰따옴표로 구성된 문자열 또는 JSXText를 의미
   - 자바스크립트와의 차이점
     - `\`로 시작하는 이스케이프 형태소 사용법에 있어, 자바스크립트에서 `\`를 사용하기 위해서는 몇 가지 제약사항(\\ 처리)이 있지만 HTML은 아무런 제약없이 사용이 가능하다.

### 2.1.2 JSX 예제

//

### 2.1.3 JSX는 어떻게 자바스크립트로 변환될까?

이를 알기 위해서는 우선 `@babel/plugin-transform-react-jsx` 플러그인에 대해 알아야함

```jsx
const ComponentA = <A required={true}>Hello World</A>;

const ComponentB = <>Hello World</>;

const ComponentC = (
  <div>
    <span>hello world</span>
  </div>
);
```

변환 결과

```js
// 한마디로
// React.createElement(태그, props객체, children) 형식

var ComponentA = React.createElement(
  A,
  {
    required: true,
  },
  "Hello World"
);

var ComponentB = React.createElement(React.Fragment, null, "Hello World");

var ComponentC = React.createElement(
  "div",
  null,
  React.createElement("span", null, "hello world")
);
```

=> 리액트 17, 바벨 7.9.0 버전에서 추가된 자동 런타임으로 트랜스파일한 결과는 다르다

```js
"use strict";

var _jsxRuntime = require("custom-jsx-library/jsx-runtime");

var ComponentA = (0, _jsxRuntime.jsx)(A, {
  required: true,
  children: "Hello World",
});
var ComponentB = (0, _jsxRuntime.jsx)(_jsxRuntime.Fragment, {
  children: "Hello World",
});
var ComponentC = (0, _jsxRuntime.jsx)("div", {
  children: (0, _jsxRuntime.jsx)("span", {
    children: "hello world",
  }),
});
```

- 공통점
  - JSXElement를 첫 번째 인수로 선언해 요소를 정의한다.
  - 옵셔널인 JSXChildren, JSXAttributes, JSXStrings는 이후 인수로 넘겨주어 처리한다.

=> 이 점을 활용한다면 경우에 따라 다른 JSXElement를 렌더링할 때 굳이 요소 전체를 감싸지 않더라도 처리할 수 있다. - JSXElement만 다르고, 나머지 JSXAttributes, JSXChildren이 완전히 동일한 상황에서 중복 로직을 최소화할 수 있는 상황

```tsx
// props 여부에 따라 children 요소만 달라지는 경우
// 굳이 번거롭게 전체 내용을 삼항 연산자로 처리할 필요가 없다.
// 이 경우 불필요한 코드 중복이 일언난다.
function TextOrHeading({
  isHeading,
  children,
}): PropsWithChildren<{ isHeading: boolean }> {
  return isHeading ? (
    <h1 className="text">{children}</h1>
  ) : (
    <span className="text">{children}</span>
  );
}

// JSX가 변환되는 특성을 활용한다면 다음과 같이 간결하게 처리할 수 있다.
function TextOrHeading({
  isHeading,
  children,
}): PropsWithChildren<{ isHeading: boolean }> {
  return createElement(isHeading ? h1 : span, { className: "text" }, children);
}
```

#### 잘 사용하지 않는 JSX문법

- JSXNamespacedName
- JSXMemberExpression

=> 그럼에도 다양한 라이브러리들에서는 해당 문법들을 목적에 맞게 사용하고 있을 수 있으니 알아두어야 한다.

## 2.2 가상 DOM과 리액트 파이버

### 2.2.1 DOM과 브라우저 렌더링 과정

#### DOM(Document Object Model)이란? <br />

웹페이지에 대한 인터페이스로 브라우저가 웹페이지의 콘텐츠와 구조를 어떻게 보여줄지에 대한 정보를 담고 있다.

1. 브라우저가 사용자가 요청한 주소를 방문해 HTML 파일을 다운로드 한다.
2. 브라우저의 렌더링 엔진은 HTML을 파싱해 DOM 노드로 구성된 트리(DOM)을 만든다.
3. 파싱하는 과정에서 CSS 파일을 만나면 CSS도 파싱해 CSS노드로 구성된 CSSOM을 만든다.
4. DOM 트리의 노드를 순회하는데, 여기서는 모든 노드를 방문하는 것이 아닌 사용자 눈에 보이는 노드만 방문한다.
   - 해당 과정에서 display:none 되어 있는 노드는 방문하지 않는다. (조금이라도 더 빠르게 하기 위해)
5. 눈에 보이는 노드를 대상을 해당 노드에 대한 CSSOM 정보를 찾고, 여기서 발견한 CSS 스타일 정보를 이 노드에 적용한다.
   - 레이아웃(layout, reflow) : 각 노드가 브라우저 화면의 어느 좌표에 정확히 나타나야 하는지 계산하는 과정. 레이아웃을 거치면 리페인팅은 반드시 거치게 된다.
   - 리페인팅(repainting) : 레이아웃 단계를 거친 노드에 색과 같은 실제 유효한 모습을 그리는 과정

### 2.2.2 가상 DOM의 탄생배경

위에서 확인했던 브라우저 렌더링 과정은 상당한 비용이 들고, 최근들어 렌더링 뒤에 사용자와의 인터랙션으로 웹페이지가 변경되는 상황 또한 고려해야하는 상황이 되었다.

- SPA 애플리케이션에서는 하나의 페이지에서 계속해서 요소의 위치를 재계산해야해서 더욱 많은 작업을 진행하게 된다.
- DOM의 모든 변경사항을 관리한느 것이 아닌, 결과물만을 받아볼 수 있어 개발자의 입장에서 훨씬 편하다.

#### 가상 DOM 이란?

웹페이지가 표시해야할 DOM을 일단 메모리에 저장하고, 리액트가 실제 변경에 대한 준비가 완료됐을 때 실제 브라우저의 DOM에 반영한다.

- 가장 큰 오해 : 일반적인 DOM을 관리하는 브라우저보다 가상 DOM이 빠르다.

### 2.2.3 가상 DOM을 위한 아키텍처, 리액트 파이버

가상DOM과 렌더링 과정 최적화를 가능하게 해주는 것이 리액트 파이버(React Fiber)다.

#### 리액트 파이버란?

리액트에서 관리하는 평범한 자바스크립트 객체다.

- 파이버는 파이버 재조정자(fiber reconciler)가 관리하는데, 이는 가상 DOM과 실제 DOM을 비교해 변경사항을 수집하고, 만약 이 둘 사이에 차이가 있으면 변경에 관련된 정보를 가지고 있는 파이버를 기준으로 화면에 렌더링을 요청하는 역할을 한다.
  => 한마디로 어떤 부분을 새롭게 렌더링해야하는지 가상 DOM과 실제 DOM을 비교하는 알고리즘이다.

#### 리액트 파이버의 목표

리액트 웹 애플리케이션에서 발생하는 애니메이션, 레이아웃, 사용자 인터랙션에 올바른 결과물을 만드는 반응성 문제를 해결하는 것

- 작업을 작은 단위로 분할하고 쪼갠다음 우선순위를 매긴다.
- 이러한 작업을 일시 중단하고 나중에 다시 시작할 수 있다.
- 이전에 했던 작업을 다시 재사용하거나, 필요하지 않은 경우 폐기할 수 있다.

  => **이 모든 과정은 비동기로 일어난다.**

과거에는 리액트의 조정 알고리즘이 스택으로 되어 있어 렌더링에 필요한 작업들이 쌓이면 이 스택이 빌 때까지 동기적으로 작업이 이루어졌다.

=> 싱글 스레드인 자바스크립트에서는 이 동기 작업은 중단될 수 없었고, 이는 리액트의 비효율성으로 이어졌다.

#### 리액트 파이버는 어떻게 구현되어 있을까?

- 파이버는 일단 하나의 작업 단위로 구성돼 있다. 리액트는 이러한 작업 단위를 하나씩 처리하고 `finishedWork()`라는 작업으로 마무리한다.
- 이 작업을 커밋해 실제 브라우저 DOM에 가시적인 변경 사항을 만들어낸다.
  1. 렌더 단계에서 리액트는 사용자에게 노출되지 않은 모든 비동기 작업을 수행한다. 그리고 이 단계에서 앞서 언급한 파이버의 작업. 우선순위를 지정하거나 중지시키거나 버리는 등의 작업이 일어난다.
  2. 커밋 단계에서는 앞서 언급한 것처럼 DOM에 실제 변경 사항을 반영하기 위한 작업. `commitWork()`가 실행되는데, 이 과정은 앞서와 다르게 동기식으로 일어나고 중단될 수도 없다.

#### 파이버 객체

```js
function FiberNode(tag, pendingProps, key, mode) {
  // Instance
  this.tag = tag;
  this.key = key;
  this.elementType = null;
  this.type = null;
  this.stateNode = null;

  // Fiber
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;
  this.ref = null;
  this.refCleanup = null;

  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.dependencies = null;

  this.mode = mode;

  // Effects
  this.flags = NoFlags;
  this.subtreeFlags = NoFlags;
  this.deletions = null;

  this.lanes = NoLanes;
  this.childLanes = NoLanes;

  this.alternate = null;

  // 이하 프로파일러, __DEV__ 코드는 생략
}
```

- 리액트 요소는 렌더링이 발생할 때마다 새롭게 생성되지만, 파이버는 가급적이면 재사용된다.
- 이렇게 생성된 파이버는 state가 변경되거나 생명주기 메서드가 실행되거나 DOM의 변경이 필요한 시점 등에 실행된다.
  - 리액트가 파이버를 처리할 때마다 이러한 작업을 직접 바로 처리하기도 하고 스케줄링하기도 한다.

#### 리액트의 핵심 원칙

리액트 팀은 가상 DOM이 아닌 Value UI, 즉 값을 가지고 있는 UI를 관리하는 라이브러리라는 내용을 피력한 바 있다.

- UI를 문자열, 숫자, 배열과 같은 값으로 관리하는 것이 핵심 원칙.

=> 변수에 이러한 UI값을 보관하고, 리액트의 자바스크립트 코드 흐름에 따라 이를 관리하고, 표현하는 것이 리액트다.

#### 리액트 파이버 트리

리액트 내부에서 두 개가 존재한다.

- 현재의 모습을 담은 파이버 트리
- 작업 중인 상태를 나타내는 workInProgress 트리

=> 리액트 파이버의 작업이 끝나면 단순히 포인터만 변경해 workInProgress 트리를 현재 트리로 바꿔버리는 더블 버퍼링 기술을 사용한다.

    - 이 더블 버퍼링 기술은 커밋 단계에서 수행된다.
    - 먼저 현재 UI 렌더링을 위해 존재하는 트리인 current 를 기준으로 모든 작업이 시작된다.
        - 여기에서 업데이트가 발생하면 파이버는 리액트에서 새로 받은 데이터로 새로운 workInProgress 트리를 빌드하기 시작한다. 이 트리의 빌드 작업이 모두 끝나면 다음 렌더링에 이 트리를 사용하고 이 workInProgress 트리가 UI에 최종적으로 반영되면 해당 트리가 current가 된다.

\*\* 더블 버퍼링은 원래 그래픽 분야에서 사용하던 용어로, 그래픽을 통해 화면에 표시되는 것을 그리기 위해서는 내부적으로 처리를 거쳐야하는데, 이러한 처리를 거치게 되면 사용자에게 미처 다 그리지 못한 모습을 보여줄 수도 있어 다른 곳에서 미리 그리고 완성되면 현재 상태를 새로운 그림으로 바꾸는 기법을 의미한다.

#### 파이버의 작업 순서

1. 리액트는 `beginWork()` 함수를 실행해 파이버 작업을 수행하는데, 더 이상 자식이 없는 파이버를 만날 때까지 트 형식으로 시작된다.
2. 1번에서 작업이 끝난다면 그 다음 `completeWork()` 함수를 실행해 파이버 작업을 완료한다.
3. 형제가 있다면 형제로 넘어간다.
4. 2번, 3번이 모두 끝났다면 return으로 돌아가 자신의 작업이 완료됐음을 알린다.

```jsx
<A1>
  <B1>안녕하세요</B1>
  <B2>
    <C1>
      <D1 />
      <D2 />
    </C1>
  </B2>
  <B3 />
</A1>
```

- A1의 beginWork()가 수행된다.
- A1은 자식이 있으므로 B1으로 이동해 beginWork()를 수행한다.
- B1은 자식이 없으므로 completeWork()가 수행됐다. 자식은 없으므로 형제인 B2로 넘어간다.
- B2의 beginWork()가 수행된다. 자식이 있으므로 C1로 이동한다.
- C1의 beginWork()가 수행된다. 자식이 있으므로 D1로 이동한다.
- D1의 beginWork()가 수행된다.
- D1은 자식이 없으므로 completeWork()가 수행됐다. 자식은 없으므로 형제인 D2로 넘어간다.
- D2는 자식이 없으므로 completeWork()가 수행됐다.
- D2도 더 이상의 형제도 없으므로 위로 이동해 D1, C1, B2 순으로 completeWork()를 호출한다.
- B2는 형제인 B3로 이동해 beginWork()를 수행한다.
- B3의 completeWork()가 수행되면 반환해 상위로 타고 올라간다.
- A1의 completeWork()가 수행된다.
- 루트 노드가 완성되는 순간, 최종적으로 commitWork()가 수행되고 이 중에 변경 사항을 비교해 업데이트가 필요한 변경 사항이 DOM에 반영된다.

#### 이후 setState로 인해 업데이트가 발생한다면?

- 최초 렌더링 시에는 모든 파이버를 새롭게 만들어야 했지만, 이제는 파이버가 이미 존배하므로 업데이트된 props만 받아 파이버 내부에서 처리한다.
- 과거 동기식으로 처리했다는 작업이 바로 이 트리 업데이트 과정, 재귀적으로 하나의 트리를 순회해 새로운 트리를 만드는 작업인데 이는 동기식이고 중단될 수 없었다.

  - 하지만 현재에는 우선순위가 높은 다른 업데이트가 오면 현재 업데이트 작업을 중단하거나 새롭게 만들거나, 폐기할 수도 있다.
  - 또한 작업 단위를 나누어 우선순위를 할당하는 것 또한 가능하다.

    => 리액트는 이러한 작업을 파이버 단위로 나눠서 수행한다.

### 2.2.4 파이버와 가상 DOM

- 파이버 : 리액트 컴포넌트에 대한 정보를 1:1로 가지고 있는 것
  - 이 파이버는 리액트 아키텍처 내부에서 비동기로 이뤄진다.
  - 이러한 비동기 작업과 달리, 살제 브라우저 DOM에 반영하는 것은 동기적으로 일어나야 하고, 또 처리하는 작업이 많아 화면에 불완전하게 표시될 수 있는 가능성이 높으므로 이러한 작업을 가상에서, 즉 메모리상에서 먼저 수행해서 최적적인 결과물만 실제 브라우저 DOM에 적용하는 것이다.

## 2.3 클래스형 컴포넌트와 함수형 컴포넌트

함수형 컴포넌트는 생각보다 역사가 깊다.

- 초창기 함수형 컴포넌트는 상태 없이 단순 렌더링만을 위해 사용되었다. (정적)
- 16.8 버전에서 훅이 소개된 이후부터 함수형 컴포넌트가 각광받기 시작했다.

### 2.3.1 클래스형 컴포넌트

기본적으로 클래스형 컴포넌트를 만드려면 클래스를 선언하고 extends로 만들고 싶은 컴포넌트를 extends 해야 한다.

- extends 구문에 넣을 수 있는 클래스
  - React.Component
  - React.PureComponent

```tsx
import React from "react";

// props 타입을 선언한다.
interface SampleProps {
  required?: boolean;
  text: string;
}

// state 타입을 선언한다.
interface SampleState {
  count: number;
  isLimited?: boolean;
}

// Component에 제네릭으로 props, state를 순서대로 넣어준다.
class SampleComponent extends React.Component<SampleProps, SampleState> {
  // constructor에서 props를 넘겨주고, state의 기본값을 설정한다.
  private constructor(props: SampleProps) {
    super(props);
    this.state = {
      count: 0,
      isLimited: false,
    };
  }

  // render 내부에서 쓰일 함수를 선언한다.
  private handleClick = () => {
    const newValue = this.state.count + 1;
    this.setState({ count: newValue, isLimited: newValue >= 10 });
  };

  // render에서 이 컴포넌트가 렌더링할 내용을 정의한다.
  public render() {
    // props와 state 값을 this, 즉 해당 클래스에서 꺼낸다.
    const {
      props: { required, text },
      state: { count, isLimited },
    } = this;

    return (
        <h2>
            <div>{required ? '필수' ㅣ '필수아님'}</div>
            <div>문자 : {text}</div>
            <div>count : {count}</div>
            <button onClick={this.handleClick} disabled={isLimited}>증가</button>
        </h2>
    )
  }
}
```

#### 클래스형 컴포넌트의 생명주기 메서드

- 마운트(mount) : 컴포넌트가 마운팅(생성)되는 시점
- 업데이트(update) : 이미 생성된 컴포넌트의 내용이 변경(업데이트)되는 시점
- 언마운트(unmount) : 컴포넌트가 더 이상 존재하지 않는 시점

#### 1. `render()`

리액트 클래스형 컴포넌트의 유일한 필수 값으로, 항상 쓰인다.

- UI를 렌더링하기 위해서 사용되고, **마운트**와 **언마운트** 과정에서 일어난다.
- 항상 순수해야 하며 부수효과가 없어야 한다.
  - render() 내부에서 직접 state를 직접 업데이트 하는 this.setState를 호출해서는 안된다.
  - state를 변경하는 일은 클래스형 컴포넌트의 메서드나 다른 생명주기 메서드 내부에서 발생해야 한다.

#### 2. `componentDidMount()`

클래스형 컴포넌트가 마운트되고 준비가 됐다면 그 다음으로 호출되는 생명주기 메서드

- 컴포넌트가 마운트되고 준비되는 즉시 실행된다.
- render()와는 다르게 해당 함수 내부에서 state를 변경하는 것이 가능하다.
- 해당 함수 안에서 리렌더링이 되고 UI가 업데이트 되기 전에 수행되기 때문에 사용자는 눈치챌 수 없지만, 성능 문제를 일으킬 수 있으니 주의해야 한다.
  - 생성자에서 할 수 없는 API 호출 후 업데이트 또는 DOM에 의존적인 작업(이벤트 리스너 추가 등)을 하기 위해서 사용하는 것이 좋다.

#### 3. `componentDidUpdate()`

컴포넌트 업데이트가 일어난 이후 바로 실행된다.

- 일반적으로 state나 props의 변화에 따라 DOM을 업데이트하는 등에 쓰인다.
- this.setState 호출이 가능하지만, 적절한 조건문으로 감싸지 않는다면 무한 호출 가능성이 있으니 주의하자.

#### 4. `componentWillUnmount()`

컴포넌트가 언마운트되거나 더 이상 사용되지 않기 직전에 호출된다.

- 메모리 누수나 불필요한 작동을 막기 위한 클린업 함수를 호출하기 위한 최적의 위치다.
- 이 함수 내부에서는 this.setState를 호출할 수 없다.

#### 5. `shouldComponentUpdate()`

state나 props의 변경으로 리액트 컴포넌트가 다시 리렌더링 되는 것을 막고 싶을 때 사용하는 메서드

- 기본적으로 this.setState가 호출되면 컴포넌트는 리렌더링을 일으킨다. 하지만, 이 생명주기 메서드를 활용하면 컴포넌트에 영향을 받지 않는 변화에 대해 정의할 수 있다.
- 사실 state의 변화에 따라 컴포넌트가 리렌더링되는 것은 굉장히 자연스러운 일이므로 이 메서드를 사용하는 것은 특정한 성능 최적화 상황에서만 고려해야한다.
  (조건 정의)

#### PureComponent와 Component

- PureComponent는 state 값에 대해 얕은 비교를 수행해 결과가 다를 때만 렌더링을 수행한다.
  - state가 객체와 같이 복잡한 구조의 데이터 변경은 감지하지 못하기 때문에 제대로 동작하지 못할 수도 있다.
  - 대다수의 컴포넌트가 PureComponent로 구성돼 있다면 오히려 성능에 역효과를 미칠 수 있다.
    - why? 컴포넌트가 얕은 비교를 했을 때 일치하지 않는 일이 더 잦다면 당연히 이러한 비교는 무의미하기 때문에

#### 6. `static getDerivedStateFromProps()`

가장 최근에 도입된 생명주기 메서드 중 하나로, 이전에 존재했으나 이제는 사라진 `componentWillReceiveProps`를 대체할 수 있는 메서드이다.

- render()를 호출하기 직전에 호출된다.
  - static으로 선언되어 this에 접근이 불가능하다.
  - 여기서 반환되는 객체는 해당 객체의 내용이 모두 state로 들어가게 되지만, null을 반환하면 아무런 일도 일어나지 않는다.

```tsx
static getDerivedStateFromProps(nextProps:Props, prevState: State){
    // 이 메서드는 다음에 올 props를 바탕으로 현재의 state를
    // 변경하고 싶을 때 사용할 수 있다.

    if(props.name !== prev.name){
        // state가 이렇게 변경된다.
        return {
            name : props.name
        }
    }

    // state에 아무런 영향을 미치지 않는다.
    return null
}
```

#### 7. `getSnapshotBeforeUpdate()`

componentWillUpdate()를 대체할 수 있는 메서드다.

- DOM이 업데이트되기 직전에 호출된다.
- 여기서 반환되는 값은 componentDidUpdate로 전달된다.
- DOM에 렌더링 되기 전에 윈도우 크기를 조절하거나 스크롤 위치를 조정하는 등의 작업을 처리하는데 유용하다.

```tsx
getSnapshotBeforeUpdate(prevProps: Props, prevState: State){
    // props로 넘겨받은 배열의 길이가 이전보다 길어질 경우
    // 현재 스크롤 높이 값을 반환한다.
    if (prevProps.list.length < this.props.list.length){
        const list = this.listRef.current;
        return list.scrollHeight - list.scrollTop;
    }
    return null;
}

// 3번째 인수인 snapshot은 클래스 제네릭의 3번째 인수로 넣어줄 수 있다.
componentDidUpdate(prevProps:Props, prevState:State, snapshot:Snapshot){
    // getSnapshotBeforeUpdate로 넘겨받은 값은 snapshot에서 접근이 가능하다
    // 이 snapshot 값이 있다면 스크롤 위치를 재조정해 기존 아이템이 스크롤에서
    // 밀리지 않게 도와준다.
    if(snapshot !== null){
        const list = this.listRef.current;
        list.scrollTop = list.scrollHeight - snapshot;
    }
}
```

#### 8. `getDerivedStateFromError()`

정상적인 생명 주기에서 실행되는 메서드가 아니라 에러 상황에서 실행되는 메서드

- 이 메서드와 `getSnapshotBeforeUpdate`는 아지 리액트 훅으로 구현되어 있지 않아 반드시 클래스형 컴포넌트를 사용해야 한다.
- 자식 컴포넌트에서 에러가 발생했을 때 호출되는 에러 메서드이다.b
  > ErrorBoundary.tsx
