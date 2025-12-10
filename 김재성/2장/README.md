# 2장 리액트 핵심 요소 깊게 살펴보기

## 2.1 JSX란?

메타에서 만든 새로운 문법
- 트랜스파일러를 거쳐야 js 런타임이 이해할 수 있는 js코드로 변환된다
- 리액트에 종속되지 않는다

### JSX

아래 4가지 컴포넌트를 기반으로 구성되어 있다

**JSXElement**
- JSXOpeningElement: ```<JSXEIement JSXAttributes(optional)>```
- JSXClosingEIement:: ```<JSXEIement />```
- JSXSelfClosingEIement:	: ```<JSXEIement JSXAttributes(optional) />```
- JSXFragment: ```<>JSXEIement JSXAttributes(optional)</>```
> 반드시 대문자로 시작하는 이유? HTML태그과 구분짓기위해서

**JSXAttributes**
- JSXAttribute: 속성을 나타내는 key와value 그리고 value에는 JSXElement가 올 수도 있다
- JSXSpreadAttributes: js의 전개 연산자와 동일

**JSXChildren**
- JSXChild: 기본 단위 이며 optional이다

**JSXStrings** 

html에서도 사용 가능한 모든 문자열 이며 js보다 좀 더 제약사항이 없음

### JSX는 어떻게 자바스크립트에서 변환될까

```js
const ComponentA = <A required={true}>Hello World</A>
const ComponentB = <>Hello World</>

const ComponentC = (
  <div>
    <span>Hello World</span>
  </div>
)
```
를 babel로 변환하면

```js
'use strict'

var	ComponentA	=	React.createEIement(
  A,
  {
    required: true,
  },
  'Hello World',
)
var	ComponentB	=	React.createEIement(React.Fragment,	null,	'Hello World')
var	ComponentC	=	React.createEIement(
  'div',
  null,
  React.createEIement('span',null,'hello world'),
)
```
특징으로는
- JSXElement를 첫 번째 인수로 선언해 요소를 정의한다
- 옵셔널인 JSXChildren, JSXAttributes, JSXStrings는 이후 인수로 넘겨주어 처리한다

이를 이용하면 이렇게 리펙토링 할 수 있다
```js
// props 여부에 따라 전체 children 요소만 달라지는 경우
// 굳이 번거롭게 전체 내용을 삼항 연산자로 처리할 필요가 없다.
// 이 경우 불필요한 코드 중복이 일어난다.
function TextOrHeading({
  isHeading,
  children,
}: PropsWithChildren<{ isHeading: boolean }>) {
  return isHeading ? (
    <h1 className="text">{children}</h1>
  ) : (
    <span className="text">{children}</span>
  )
}

// JSX가 변환되는 특성을 활용한다면 다음과 같이 간결하게 처리할 수 있다.
import { createElement } from 'react'

function TextOrHeading({
  isHeading,
  children,
}: PropsWithChildren<{ isHeading: boolean }>) {
  return createElement(
    isHeading ? 'h1' : 'span',
    { className: 'text' },
    children,
  )
}
```

<br/>

## 2.2 가상 DOM과 리액트 파이버

**브라우저가 화면을 그리는 과정**
1. url에 접근해 html과 css 다운
2. 각각 DOM과 CSSOM 생성
3. DOM을 순회하며 해당되는 CSSOM정보를 찾아서 결합한다
4. 레이아웃과 페인팅 과정을 적용

**가상DOM**

react-dom에서 메모리에 저장하여 관리하는 DOM트리

**리액트 파이버(React Fiber)**

파이버 재조정자가 관리하는 자바스크립트 객체 이며 가상DOM과 실제DOM을 비교해 변경이 있으면 파이버 기준으로 재조정(알고리즘)한다
- 작업을 작은 단위로 분할하고 쪼갠 다음, 우선순위를 매긴다
- 이러한 작업을 일시 중지하고 나중에 다시 시작할 수 있다
- 이전에 했던 작업을 다시 재사용하거나 필요하지 않은 경우에는 폐기할 수 있다

이 모든 과정은 **비동기**로 일어난다

과거 조정 알고리즘은 스택이였고 비효율성으로 인해 파이버(아키텍처)라는 개념이 탄생
- fiberNode를 생성해서 하나의 element에 1:1로 매칭하여 관련 정보를 보관
- child, sibling, return 이 3개의 속성을 이용한 연결리스트 구조
  - index: 자신의 위치
  - pendingProps: 아직 작업을 처리하지 못한 props
  - memoizedProps: pendingProps를 기준으로 렌더링이 완료된 이후에 pendingProps를 memoizedProps로 저장해 관리한다.
  - updateQueue: 상태 업데이트, 콜백 함수, DOM 업데이트 등 필요한 작업을 담아두는 큐
  - memoizedState: 함수형 컴포넌트의 훅 목록이 저장된다. 여기에는 단순히 useState뿐만 아니라 모든 훅 리스트가 저장된다.
  - alternate: 반대편 트리파이버를 가리킴

**리액트 파이버 트리**

2개의 파이버트리로 더블 버퍼링을 활용해서 완전히 그려진 UI를 교체하는 방식을 사용한다
> 더블 버퍼링이란? 컴퓨터 그래픽분야에서 활용하는 용어이며 그림을 미리그리고 완성되면 교체하는 방식

**파이버 노드의 생성 흐름**
1. beginWork()함수로 자식이 없는 파이버를 만날 때까지 트리 형식으로 순환
2. 작업이 끝나면 completeWork()로 작업 완료
3. 형제가 있다면 형제로 넘어간다
4. 모두 끝나면 return으로 돌아가 자신의 작업이 완료됐음을 알린다.
5. 최종적으로 commimtWork()가 수행되어 변경사항을 비교해 업데이트가 필요한 변경 사항이 DOM에 반영

> Q1. setState등으로 업데이트가 발생하면 파이버 트리에는 무슨일이 일어날까?

<br/>

## 2.3 클래스형 컴포넌트와 함수형 컴포넌트

### 클래스형 컴포넌트

**클래스형 컴포넌트의 생명주기 메서드**

실행되는 시점 3가지
- 마운트(mount): 컴포넌트가 마운팅(생성)되는 시점
- 업데이트(update): 이미 생성된 컴포넌트의 내용이 변경(업데이트)되는 시점
- 언마운트(unmount): 컴포넌트가 더 이상 존재하지 않는 시점

생명주기 메서드
- render: UI렌더링용이며 순수해야한다
- componentDidMount: 컴포넌트가 마운트된 시점에 실행
- componentDidUpdate: 컴포넌트 업데이트가 일어난 직후 실행
- componentWillUnmount: 컴포넌트가 언마운드 되거나 더 이상 사용되지 않기 직전에 실행
- shouldComponentUpdate: state나 props의 변경으로 리렌더링되는것을 제어 하고 싶을때 사용
  - ReactPureComponent는 state나 props를 얕은 비교를 통해 변해야만 리렌더링되고 ReactComponent는 항상 리렌더링 된다
- getSnapShotBeforeUpdate: DOM이 업데이트되기 직전에 호출
- getDerivedStateFromError: 에러 상황에서 실행
- componentDidCatch: getDerivedStateFromError에서 에러잡고 state를 결정한 이후 실행


**클래스형 컴포넌트의 한계**

- 데이터 흐름추적이 어렵다
- 로직 재사용이 어렵다
- 기능과 컴포넌트 크기가 비례한다
- 번들 크기가 크다
- 핫 리로딩에서 불리하다

### 함수형 컴포넌트 vs 클래스형 컴포넌트

- 생명주기 매서드의 부재
  - 함수형은 useEffect로 비슷하게 구현
- 함수형은 랜더링된 값을 고정한다
  - 클래스형은 props의 값을 항상 this로부터 가져오기때문에 불변이 아니라 생명주기 메서드가 필요하다

<br/>

## 2.4 렌더링은 어떻게 일어나는가?

**리액트의 렌더링이란?**

리액트 애플리케이션 트리 안에 있는 모든 컴포넌트들이 현재 자신들이 가지고 있는 props와 state의 값을 기반으로 어떻게 UI를 구성하고 이를 바탕으로 어떤 DOM 결과를 브라우저에 제공할 것인지 계산하는 일련의 과정을 의미한다

**리액트에서 렌더링이 발생하는 시나리오**
- 최초 렌더링: 사용자가 처음 애플리케이션에 진입하면 보여질 결과물
- 리렌더링
  - 클래스) setState가 실행되는 경우
  - 클래스) forceUpdate가 실행되는 경우
  - 함수) useState의 두 번째 배열 요소인 setter가 실행되는 경우
  - 함수) useReducer의 두 번째 배열 요소인 dispatch가 실행되는 경우
  - 컴포넌트의 key props가 변경되는 경우
    >  key는 리렌더링이 발생하는 동안 형제 요소들 사이에서 동일한 요소를 식별하는 값이다
  - props가 변경되는 경우
  - 부모 컴포넌트가 렌더링될 경우
>함수형 컴포넌트에서는 memo를 사용하면 props가 변경되지 않으면 렌더링을 생략할 수 있다

**렌더와 커밋**

렌더
- 컴포넌트를 렌더링하고 변경 사항을 계상하는 모든 작업
- 컴포넌트를 실행해서 이전 가상DOM과 비교
- 비교하는건 크게 3가지이며, type, props, key이다

커밋
- 렌더 단계의 변경 사항을 실제 DOM에 적용해 유저에게 보여주는 과정
- DOM을 커밋 단계에서 업데이트한다면 만들어진 모든 DOM 노드 및 인스턴스를 가리키도록 리액트 내부의 참조를 업데이트한다
- 렌더단계에서 변경사항을 못찾으면 커밋 단계는 생략가능

> 리액트의 렌더링이 일어난다고 해서 무조건 DOM 업데이트가 일어나는것은 아니다

<br/>

## 2.5 컴포넌트와 함수의 무거운 연산을 기억해 두는 메모이제이션

### 필요할때만 메모이제이션 하자 vs 모든곳에 메모이제이션 하자

> Q2. 본인의 생각과 그 이유는?