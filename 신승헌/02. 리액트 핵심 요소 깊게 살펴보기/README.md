# 2장. 리액트 핵심 요소 깊게 살펴보기

# 2.1 JSX란

> JSX는 리액트의 전유물이 아니다

- JSX는 리액트 등장으로 메타에서 소개한 새로운 구문
- 반드시 리액트에서만 써야되는건 아님

### **JSX**

- XML유사한 내장형 구문
- Javascript 표준(ECMAScript)을 따르진 않음
- **반드시!** 트랜스파일러를 거쳐야 유효한 JS코드로 변환됨

### **JSX의 목적**

- HTML이나 XML을 JS 내부에 쓰려는게 아님
- 코드를 간결하고 친숙하게 쓸 수 있도록 설계되어 있음
- 다양한 Attributes를 가진 트리 구조를 토큰화 해서 JS로 변환하는 데 초점이 있음
- 트랜스파일링으로 JS(ECAMScript)로 변경하는게 주 목적

### **XML?**

- e**X**tensible **M**arkup **L**anguage
- 계층 구조로 데이터를 표현
- JSON 이전 데이터 교환의 표준

```html
<user>
  <name>Adam</name>
  <age>20</age>
</user>
```

## 2.1.1 JSX의 정의

### 1️⃣ JSXElement / JSXElementName

- React에서 HTML 외 컴포넌트 이름은 무조건 대문자 ⇒ 기존 태그와 구별하기 위함

### JSXElement

- JSX의 가장 기본 요소
- 다음과 같은 형태를 만족해야 함

1. **JSXOpeningElement**
   1. <JSXElement JSXAttribute(optional)>
   2. `<button type="button">`
2. **JSXClosingElement**
   1. </ JSXElement>
   2. `</button>`
3. **JSXSelfClosingElement**
   1. <JSXElement JSXAttribute(optional) />
   2. `<button type=”button” />`
4. **JSXFragment**

   1. <>JSXChildren(optional)</>

   ```jsx
   <>
     <span />
     <button />
   </>
   ```

### JSXElementName

- JSXElement의 요소 이름으로 쓸 수 있는것

1. **JSXIdentifier**

   1. **JS 식별자 규칙**과 동일

      > 식별자는 대소문자를 구별하며 유니코드 글자, $, \_, 숫자(0-9)로 구성할 수 있지만, **숫자로 시작할 수는 없음**
      >
      > [](data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==)
      >
      > [](data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==)
      >
      > [](data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==)
      >
      > [](data:image/gif;base64,R0lGODlhAQABAIAAAP///wAAACH5BAEAAAAALAAAAAABAAEAAAICRAEAOw==)

   2. “$”, “\_” 외에 다른 특수문자(숫자 등)로 시작하는 이름은 안됨
   3. `<$></$>`, `<_></_>`

2. **JSXNamespacedName**
   1. `[JSXIdentifier]:[JSXIdentifier]` 형태로 사용이 가능,
      “:”(콜론)은 하나만 쓸 수 있음
   2. `<foo:bar></foo:bar>`
3. **JSXMemberExpression**
   1. `[JSXIdentifier].[JSXIdentifier]` 형태로 사용이 가능
   2. 여러개 이어서 쓸 수 있음
   3. NamespacedName과 함께 쓰는건 불가능
   4. `<foo.bar></foo.bar>`

> **🤔 JSXNamespacedName를 쓰긴 하는건가?**
> 레거시 또는 SVG, XML 라이브러리 등 아주 특수한 경우에만 씀

- HTML에서 JSXNamespacedName 형태로 태그를 쓰면 그냥 커스텀 태그(빈 div 태그)로 취급함

### 2️⃣ JSXAttributes

**JSXElement에 부여할 수 있는 속성을 의미**

1. **JSXSpreadAttributes**: 자바스크립트 전개 연산자와 동일한 역할
   1. `{ ...AssignmentExpression }`
2. **JSXAttribute**: “키 = 값” 형태로 표현
   1. JSXAttributeName
   - JSXIdentifier, JSXNamespacedName 사용할 수 있음
   2. JSXAttributeValue
   - 문자열, AssignmentExpression, JSXElement, JSXFragment 등 쓸 수 있음
     1. `<Child attribute={<div>heUo</div>} />` 처럼 `{}`로 감싸는건 prettier 규칙…
     2. _eslint-react의 [no-children-prop](https://www.eslint-react.xyz/docs/rules/no-children-prop) rule 같은것도 있음 → children은 Attribute말고 태그로 감싸자_

### 3️⃣ JSXChildren

**JSXElement의 자식값을 나타냄, 속성을 가진 트리구조를 나타내기 위해 부모-자식 관계를 나타냄**

- **JSXChild:** JSXChildren의 기본 단위, 없어도 됨(0개 이상)
  - JSXText: `{, }, <, >` 외의 문자열, JSX 문법과 혼동 방지
    ⇒ `return <>{”{ } <>”}</>` 문자열로는 써줄 수 있음
  - JSXElement
  - JSXFragment
  - JSXChildExpression: 표현식 형태로도 쓸 수 있음
    ⇒ `return<>{(()=>'foo')()}</>`

### 4️⃣ JSXstrings

**HTML ↔ JSX 간 내용 전환 편리하도록 의도적으로 설계됨**

- HTML에서 사용 가능한 문자열은 JSXStrings에서도 가능함
- 문자열 ⇒ `“ “`, `‘ ‘`, JSXText
- JS와는 달리 `“\”` (이스케이프)를 표현하려면 `“\\”`로 써야하지만
  HTML에서는 그냥 써도 됨

```jsx
<span>안녕\n하세요</span>
// 안녕\n하세요
```

```jsx
console.log("안녕\n하세요");
// 안녕
// 하세요
```

---

## 2.1.2 JSX 예제

---

## 2.1.3 JSX → Javascript

### @babel/plugin-transform-react-jsx

JSX 구문을 자바스크립트가 이해할 수 있는 형태로 변환해주는 플러그인

```jsx
"use strict";

const Component = (
  <div>
    <button type="button">안녕하세요</button>
  </div>
);
// 변환
var Component = React.createElement(
  "div",
  null,
  React.createElement(
    "button",
    {
      type: "button",
    },
    "안녕하세요"
  )
);
```

바벨 7.9.0 이후 추가된 자동 런타임(automatic runtime) 트랜스파일

```jsx
'use strict'

var _jsxRuntime = require('custom-jsx-1ibrary/jsx-runtime')

var ComponentC = (0, _jsxRuntime.jsx)('div', {
	children: (0, _jsxRuntime.jsx)('button', {
		type: "button",
		children: "안녕하세요",
	)),
})
```

### 리팩터링 전략

트랜스파일 결과를 살펴보면

- JSXElement를 첫번째 인수로 선언, 요소를 정의한다.
- JSXChildren, JSXAttributes, JSXStrings등 optional 값은 인수로 넘겨서 처리

```jsx
// JSX 트랜스파일링의 특성을 활용, 코드 리팩터링이 가능
import { createElement, PropsWithChildren } from "react";

function TextOrHeading({
  isHeading,
  children,
}: PropsWithChildren<{ isHeading: boolean }>) {
  return createElement(
    isHeading ? "h1" : "span",
    { className: "text" },
    children
  );
}
```

---

## 2.1.4 정리

> 리액트에서 JSX의 모든 구문을 다 활용할 필요는 없음

- JSXNamespacedName
- JSXMemberExpression

위와 같은 문법은 리액트에서 잘 사용하지 않는다.

Preact, SolidJS, Nano JSX 등 JSX를 채택하고 위 문법들을 사용하는 경우도 있음

> _🤔 JSXMemberExpression은 UI라이브러리 등에서 간혹 쓰는 것 같기도…_

---

# 2.2 가상 DOM & 리액트 파이버

## 2.2.1 DOM과 브라우저 렌더링

### DOM이란?

- Document Object Model
- 브라우저가 웹페이지의 컨텐츠와 구조를 어떻게 보여줄지에 대한 정보를 담고 있음

### 브라우저 렌더링 파이프라인

```markdown
1. 사용자가 요청한 주소 방문
2. HTML 파일 다운로드
3. HTML 파싱 → DOM 트리 생성
4. CSS 있는 경우 다운로드, 파싱 → CSSOM 트리 생성
5. DOM 노드 순회
   - 사용자 눈에 보이는 노드만 (e.g. **`displace: none` 제외**)
   - 트리 분석 성능 향상을 위함
6. 해당 노드에 대한 CSSOM 정보 검색
7. CSS 적용 - 두 가지 과정으로 나눌 수 있음
   a. 레이아웃(layout, reflow): 각 노드가 브라우저 어디에 나타나야 하는지
   b. 페인팅(painting): 레이아웃 이후 노드에 색상, opacity 등 실제 모습 적용
   _레이아웃이 발생하면 **페인팅**도 무조건 발생함_
```

---

## 2.2.2 가상 DOM의 탄생 배경

- 렌더링 과정은 매우 복잡하고 많은 비용이 발생함
- 요즘 앱은 인터랙션으로 변경이 잦음

### 변경에 대한 비용

- 색상이 변경되는 경우, 페인팅만 발생 ⇒ 비용 낮음
- 노출 여부나 사이즈 변경되는 경우, 레이아웃 발생 ⇒ 리페인트 발생 ⇒ 비용 높음

### 일반 웹 vs SPA

- 일반적인 웹페이지는 페이지 이동 시 HTML을 새로 받아 렌더링 과정을 거침
- SPA는 하나의 페이지에서 요소의 위치를 재계산 하며 렌더링 함

### SPA 동작방식

- 라우팅 변경 시
  1. 대부분의 요소 삭제
  2. 다시 삽입
  3. 위치 재계산
  4. …
- 페이지 깜빡임이 적음
- DOM 관리 부담, 비용 증가

### 가상 DOM

> “모든 DOM의 변경을 추적하기보단, 인터랙션에 따른 최종 DOM 결과물만 확인함”

- 웹페이지가 표시해야 할 DOM을 메모리에 저장함
- 리액트(react-dom)가 실제 변경에 대한 준비가 완료됐을 때 실제 브라우저의 DOM에 반영함
- 메모리에서 계산 후 브라우저에 적용하는 방식으로 렌더링 과정을 최소화 할 수 있음

> **“가상 DOM이 무조건 일반적인 DOM 관리 방식보다 무조건 빠르진 않음”**

- 그냥 애플리케이션 만들 정도로 “충분히” 빠름
- DX를 위해 가상 DOM이 탄생했고, 합리적인 성능으로 채용되었다고 보는게 맞음

---

## 2.2.3 가상 DOM을 위한 아키텍쳐, 리액트 파이버

### 리액트 파이버?

- 리액트에서 관리하는 평범한 자바스크립트 객체
- 파이버 재조정자 (fiber reconciler)에 의해 관리됨

> **파이버 재조정자의 역할**
>
> 1. 가상 DOM, 실제 DOM 비교 및 변경사항 수집
> 2. **변경 관련 정보를 가지고 있는 파이버를 기준**으로 렌더링 요청
>
> 재조정(reconciliation)이란 리액트에서 어떤 부분을 새롭게 렌더링 할 지
> 가상 ↔ 실제 DOM을 비교하는 작업(알고리즘)

### 파이버의 역할

- 아래 작업들은 모두 비동기로 진행됨

1. 작업을 작은 단위로 분할, 우선순위 배정
2. 작업 일시중지 및 재개
3. 이전 작업 재사용 또는 폐기

### 파이버 특징

- 과거의 리액트 조정 알고리즘이 스택 알고리즘으로 이루어져 스레드 블로킹이 발생했음
- 파이버는 하나의 작업 단위로 구성되어 있고
- 작업단위를 하나씩 처리한 후 `finishedWork()`로 마무리 함
- 이 작업을 커밋해서 실제 DOM에 변경사항을 만들어냄

1. 렌더 단계
   1. 사용자에게 노출되지 않는 모든 비동기 작업 수행
   2. 파이버의 작업, 우선순위 선정, 중지 등 작업 발생
2. 커밋 단계
   1. `commitWork()`
   2. DOM에 실제 변경사항을 반영하기 위한 작업
   3. 동기식으로 일어남

- 작업들을 작은 단위로 나누거나, 애니메이션 처럼 우선 순위가 높은 착업을 우선 처리하는 등 유연하게 처리됨

> 리액트의 실체는 사실 가상 DOM이 아닌 Value UI
> => 즉, 값을 가진 UI를 관리하는 라이브러리다

### 파이버의 구현

- 파이버는 최초 마운트 시점에 생성되어, 가급적이면 재사용됨
- 하나의 React Element 마다 하나의 FiberNode(Fiber 작업단위)가 할당됨
- React Element처럼 Fiber도 트리 구조로 구성됨

### 주요 속성

- **tag**: 파이버의 1:1 매칭 정보를 가지고 있음
- **stateNode**: 파이버 자체에 대한 참조 정보, 참조를 바탕으로 파이버 관련 상태에 접근함
- **child, sibling, return**: 파이버 간 관계를 나타냄, 트리 형식을 구성하는데 필요한 정보
  - child(하나)만 존재함
  - 형제 노드는 sibling 속성으로 나타냄
  - 작업완료 후 복귀할 상위 태그를 return 속성으로 표시
- **index**: 속성으로 형제들 사이 순서를 표현
- **pendingProps**: 렌더링 완료 후 memoizedProps로 저장해 관리함
- **updateQueue**: 상태업데이트, 콜백함수, DOM 업데이트 등 필요한 작업을 담아두는 큐
- **memoizedState**:

  - 함수형 컴포넌트의 훅 목록 저장, useSatae 뿐만 아니라 모든 훅 리스트를 저장함
  - 컴포넌트 내부에서 호출한 훅들을 연결리스트 형태로 저장함

  ```jsx
  function Component() {
    const [a, setA] = useState(1);
    const [b, setB] = useState(2);
    const ref = useRef(null);
    useEffect(() => {}, []);

    return null;
  }

  // Hook(useState a)
  //    ↓
  // Hook(useState b)
  //    ↓
  // Hook(useRef)
  //    ↓
  // Hook(useEffect)
  //    ↓
  // null
  ```

- **alternate**: 반대편 파이버 트리를 가리킴

### 더블 버퍼링

리액트 파이버 트리는 리액트 내부에 사실 **두 개** 존재하는데

1. **fiber 트리**: **현재 모습**을 담고 있음
2. **workInProgress** 트리: 작업 중인 상태를 나타냄
3. 파이버의 작업이 완료되면, 포인터만 변경해서 WIP트리를 현재 트리로 바꿈

⇒ **더블 버퍼링!**

- 더블 버퍼링은 사실 CG에서 사용하는 용어
- 미처 다 그리지 못한 모습을 보여주지 않기 위해, 보이지 않는곳에서 완성 후 상태를 바꿔서 한번에 보여줌

### 파이버 작업 순서

1. 리액트가 `beginWork()` 함수 실행
2. 파이버 작업 수행 - 더 이상 자식이 없는 파이버 만날 때 까지(트리 형식)
3. 작업 완료 후 `completeWork()` 함수 실행, 파이버 작업 완료
4. 형제가 있는 경우 형제로 이동
5. 완료 후 return으로 돌아감, 작업 완료 알림

### 렌더링 이후 setState가 발생하면?

- 업데이트 요청을 받아, workInProgress 트리를 다시 빌드함
- 최초 렌더링 시에는 모든 파이버를 새로 만들어야 하지만,
- 이미 존재하는 파이버에서 업데이트된 props를 받고, 파이버 내부에서 처리함
- 객체를 새로 만들기 보다는 속성값만 초기화 하거나 바꿔서 트리 업데이트

---

## 2.2.4 파이버와 가상 DOM

### 파이버와 가상 DOM의 관계

- 파이버의 비동기 작업과 달리, **실제 DOM에 반영**하는 것은 **동기적**으로 일어나야 함
- 처리량이 많아 화면에 불완전한 상태로 표시될 가능성이 높아서, 메모리(가상)에서 먼저 수행 후
  ⇒ 최종결과물만 실제 브라우저 DOM에 적용함

### 파이버 ≠ 가상 DOM

- 가상 DOM은 웹 애플리케이션에만 통용되는 개념
- 리액트 네이티브와 같은 브라우저 외의 환경에서도 리액트 파이버는 사용됨
- 리액트와 RN의 렌더러가 다르지만, 파이버로 조정되는 과정은 동일함
  ⇒ 동일한 재조정자(fiber reconciler)로 관리될 수 있음

---

## 2.2.5 정리

### 가상 DOM과 파이버의 탄생 이유

- 단순히 빠르다는 이유로 만들어진게 아님
- 개발자가 직접 수동으로 DOM을 조작(변경)하는 수고와 리스크를 줄이기 위함
- 효율적인 유지보수와 편리한 관리를 위해

### 리액트와 가상 DOM의 핵심

- 단순히 빠른 업데이트를 하는 것이 아니라
- **값으로 UI를 표현** 하는게 목적

### 값으로 UI를 표현/관리하는 것

1. 함수의 반환값으로 UI를 만듦
2. 조건문으로 UI를 조합하고 제어함
3. 배열로 UI를 반복 렌더링
4. 이전 값과 새로운 값(UI)를 비교(diffing) 할 수 있음

> 즉, 값으로 다룸으로써 가상 DOM에서 diffing 같은 작업이 가능해질 수 있음

---

# 2.3 클래스형 컴포넌트와 함수형 컴포넌트

- 함수형 컴포넌트는 리액트 0.14버전에 만들어짐
- “stateless functional component”라 해서, 상태 없이 정적 렌더링이 목적

```jsx
var Aquarium = (props) => {
  var fish = getFish(props.species);
  return <Tank>{fish}</Tank>;
};
var Aquarium = ({ species }) => <Tank>{getFish(species)}</Tank>;
```

- 현재는 함수형 컴포넌트가 대다수이지만, 레거시를 이해하기 위해 클래스형 컴포넌트도 알아두자

## 2.3.1 클래스형 컴포넌트

- 레거시나 오래된 라이브러리에서 주로 사용됨

### 만드는법

```jsx
import React from "react";

class Samplecomponent extends React.Component {
  render() {
    return <h2>sample Component</h2>;
  }
}
```

- 클래스를 선언
- extends로 만들 컴포넌트를 확장
  1. React.Component
  2. React.PureComponent
     ⇒ 위 두 가지는 **shouldComponentUpdate**를 다루는 데 차이점이 있음

### props, state 정의

```tsx
import React from "react";

// props 타입
interface SampleProps {
  required?: boolean;
  text: strin9;
}
// state 타입
interface SampleState {
  count: number;
  isLimited?: boolean;
}

// Component에 제네릭으로 props, state를 순서대로 넣어준다.
class SampleComponent extends React.component<SampleProps, SampleState> {
  // constructor에서 props룰 넘겨추고, State의 기본값을
  private constructor(props: SampleProps) {
    super(props);
    this.state = {
      count: 0,
      isLimited: false,
    };
  }

  // render 내부에서 쓰일 함수를 선언한다.
  private handleClick = () => {
    const newValue = this.stats.count + 1;
    this.setState({ count: newValue, isLimited: newValue >= 10 });
  };

  public render() {
    const {
      props: { required, text },
      state: { count, isLimited },
    } = this;

    return (
      <h2>
        <div>{required ? "필수" : ""}</div>
        <div>{text}</div>
        <div>{count}</div>
        <button onClick={this.handleClick} disabled={isLimited}>
          increase
        </button>
      </h2>
    );
  }
}
```

- **props**: 컴포넌트에 속성을 전달하는 용도
  - `<SampleComponent text=”안녕하세요” />`
- **state**: 클래스형 컴포넌트 내부에서 관리하는 값
- **메서드**: 렌더링 함수 내부에서 사용되는 함수, 주로 DOM에서 발생하는 이벤트와 함께 사용됨

  1. constructor에서 this 바인딩

     ```tsx
     // .bind(x) : 이 함수가 실행될 때 내부의 this를 x로 고정함

     this.handleClick.bind(this);
     // this(인스턴스)를 this로 고정한 새로운 함수 생성

     this.handleClick = this.handleClick.bind(this);
     // 새로운 함수를 다시 인스턴스에 덮어씀
     ```

     **리액트 내부에서 함수를 호출**

     ```tsx
     <button onClick={this.handleClick} />

     // 여기서 onClick은 함수 참조만 넘겨줌
     // 이벤트 발생 시 넘겨준 함수를 React코드가 직접 호출
     ```

     **리액트 내부**

     ```tsx
     function dispatchEvent(e) {
       // 이벤트 감지
       handler(e); // 넘겨준 this.handleClick 호출
     }
     ```

     - 위와같은 구조로 인해 this(호출 컨텍스트)가 날아갈 수 있어서
     - `.bind(x)`로 컨텍스트를 고정함

  2. 화살표 함수 쓰기
     - `<button onClick={() => this.handleClick()} />`
     - 하지만 매번 렌더링마다 새로운 함수가 생성되기 때문에 최적화가 어려움
     - 지양하는 것이 좋다~

### 클래스형 컴포넌트의 생명주기 메서드

클래스형 컴포넌트의 많은 코드가 생명주기 메서드에 의존함

생명주기 메서드가 실행되는 시점은 크게 3가지

- **마운트**: 컴포넌트가 생성되는 시점
- **업데이트**: 이미 생성된 컴포넌트의 내용이 변경되는 시점
- **언마운트**: 컴포넌트가 더 이상 존재하지 않는 시점

### **1. `render()`**

- 리액트 클래스형 컴포넌트 유일한 필수 값
- UI를 렌더링 하기 위해 사용
- 마운트, 업데이트 과정에 렌더링이 발생함
- `render()` 함수는 항상 순수해야 하고 부수 효과가 없어야 함(no side-effects)
- 같은 입력값(props, state)에 항상 같은 결과를 반환해야 함
  - render 내부에서 this.setState 호출하면 안됨
  - state 변경은 다른 메서드나 생명주기에서..

### 2. `componentDidMount()`

- 마운트 이후 호출되는 생명주기 메서드
- 컴포넌트가 마운트되고 준비되는 즉시 실행됨
- setState 가능하지만, 성능 문제에 주의해야 함
- 무한루프 일으키는건 아닌가? ⇒ 마운트 이후 딱 한 번만 실행됨
- state 조작은 생성자에서 하는게 좋음
- API호출 후 업데이트나 이벤트리스너 추가 등 꼭 필요한 경우에만 확인 해 보고 사용하자

### 3. `componentDidUpdate()`

- 컴포넌트 업데이트가 일어난 이후 바로 실행
- state, props 의 변화에 따라 DOM 업데이트에 주로 쓰임
- setState 가능하지만, 무한루프 위험 ⇒ 적절한 조건문 배치가 필요함

### 4. `componentWillUnmount()`

- 언마운트 되거나 사용되지 않기 직전에 호출된다.
- 메모리 누수나 불필요한 작동 방지를 위한 클린업 호출에 최적의 타이밍
- setState안됨
- 이벤트 삭제, API호출 취소, 타이머 삭제 등 작업 실행

### 5. `shouldComponentUpdate()`

- state, props의 변경으로 인한 리렌더링을 막고싶을 때 사용
- 보통 setState 호출 시 리렌더링이 일어나지만
- 이 메서드는 컴포넌트에 영향받지 않는 변화를 정의할 수 있음
- 리렌더링은 자연스러운 일이므로, 특정한 최적화 상황에만 사용해야 함

### Component vs PureComponent

```tsx
import React from "react";

interface State {
  count: number;
}

type Props = Record<string, never>;

export class ReactComponent extends React.Component<Props, State> {
  private renderCounter = 0;

  private constructor(props: Props) {
    super(props);
    this.state = {
      count: 1,
    };
  }
  private handleClick = () => {
    this.setState({ count: 1 });
  };
  public render() {
    console.log("ReactComponent", ++this.renderCounter);
    return (
      <h1>
        ReactComponent: {this.state.count}{" "}
        <button onClick={this.handleClick}>+</button>
      </h1>
    );
  }
}

export class ReactPureComponent extends React.PureComponent<Props, State> {
  private renderCounter = 0;

  private constructor(props: Props) {
    super(props);
    this.state = {
      count: 1,
    };
  }
  private handleClick = () => {
    this.setState({ count: 1 });
  };
  public render() {
    console.log("ReactPureComponent", ++this.renderCounter);
    return (
      <h1>
        ReactPureComponent: {this.state.count}{" "}
        <button onClick={this.handleClick}>+</button>
      </h1>
    );
  }
}

export default function CompareComponent() {
  return (
    <>
      <h2>React.Component</h2>
      <ReactComponent />
      <h2>React.PureComponent</h2>
      <ReactPureComponent />
    </>
  );
}

// 각 버튼 3회씩 클릭 시
// ReactComponent 1
// ReactComponent 2
// ReactComponent 3
// ReactPureComponent 1
```

### 6. `static getDerivedStateFromProps()`

- 18 버전 기준 최근에 도입된 생명주기 메서드
- 과거의 `componentWiillRecieveProps`를 대체할 수 있는 메서드
- `render()` 직전에 매번 호출됨
  - props의 변경
  - setState로 state 변경 시
  - 부모 리렌더링 시
    ⇒ 자주 호출되기 때문에 신중한 사용이 필요함
- static 으로 선언되어, this에 접근할 수 없음
- 반환값은 state로 들어가게 됨
- `null` 을 반환하면 아무 일도 일어나지 않음

> 기본적으로 React는 props와 state의 분리를 권장하지만
> props로 들어온 값을 state에 복사해야 하는 경우와 같이 특수한 상황에 사용하는 고-급 API
> _e.g. “props로 들어온 값으로 내부에서 가공된 state를 만들어야 할 때”_

### 7. `getSnapshotBeforeUpdate()`

- 18 버전 기준 최근에 도입된 생명주기 메서드
- `componentWillUpdate()`를 대체할 수 있는 메서드
- DOM 업데이트 직전에 호출됨
- 반환값은 `componentDidUpdate()`로 전달됨
- DOM 렌더링 전 윈도우 크기를 조절하거나 스크롤 위치를 조정하는 등 작업에 유용

```jsx
// props로 받은 배열의 길이가 길어진 경우
// 스크롤 높이 값을 반환
getSnapshotBeforeUpdate(prevProps: Props, prevState: State) {
	if (prevProps.list.length < this.props.list.length) {
		const list = this.listRef.current;
		return list.scrollHeight - list.scrollTop;
	}
	return null;
}

// snapshot 값이 있다면, 스크롤 위치를 재조정
// 기존 아이템이 스크롤에서 밀리지 않도록 해줌
componentDidUpdate(prevProps: Props, prevState: State, snapshot: Snapshot) {
	if (snapshot !== null) {
		const list = this.listRef.current;
		list.scrollTop = list.scrollHeight - snapshot;
	}
}
```

### 생명주기 메서드 다이어그램

...

### 에러상황에 실행되는 메서드

```jsx
// ErrorBoundary.tsx
import React, { PropsWithChildren } from "react";

type Props = PropsWithChildren<{}>;
type State = { hasError: boolean, errorMessage: string };

export default class ErrorBoundary extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, errorMessage: "" };
  }

  // static 메서드, error를 인수로 받음(하위 컴포넌트에서 발생한 error)
  // 렌더링 과정에서 호출되는 메서드로, 부수 효과를 발생시키면 안됨
  // 필자의 의견으로는 해도 되긴 하는데.. 렌더링 과정을 불필요하게 방해하게 되므로 하지말라 함
  // 부수효과 => state 반환 외 모든 작업, e.g. console.error
  static getDerivedStateFromError(error: Error): State {
    // 반드시 미리 정의해둔 state를 반환
    return { hasError: true, errorMessage: error.message };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    // 여기서 주로 에러 로깅
    console.error("ErrorBoundary caught an error", error, info);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h1>Something went wrong.</h1>
          <p>{this.state.errorMessage}</p>
        </div>
      );
    }

    // 일반적인 상황에 JSX를 렌더링함
    return this.props.children;
  }
}

// ...

// 사용 예시
function App() {
  return (
    <ErrorBoundary>
      <MyComponent />
    </ErrorBoundary>
  );
}

function MyComponent() {
  const [error, setError] = React.useState(false);

  const handleClick = () => {
    setError((prev) => !prev);
  };

  if (error) {
    throw new Error("An unexpected error occurred!");
  }

  return <button onClick={handleClick}>Trigger Error</button>;
}
```

아래 두 메서드와 `getSnapshotBeforeUpdate()`는 리액트 훅으로 구현되어있지 않기 때문에
필요한 경우 반드시 클래스형 컴포넌트로 사용해야 한다.

> 19버전에서 따로 관련 업데이트는 없는것 같다..

### 8. `getDerivedStateFromError()`

- 자식 컴포넌트에서 에러가 발생했을 때 호출되는 에러메서드
- 적절한 에러처리 로직에 필요함

### 9. `componentDidCatch()`

- 자식 컴포넌트에서 에러가 발생했을 때 실행됨
- `getDerivedStateFromError()`에서 에러를 잡고, state 결정 이후 실행됨
- 두 개의 인수를 받음
  1. `error` : 위에서와 동일한 error
  2. `info`: 어떤 컴포넌트가 에러를 발생시켰는지에 대한 정보
- 위에서 못했던 부수효과 수행이 가능함 ⇒ 커밋 단계에 실행되기 때문!

### ErrorBoundary

- 위 두 메서드는 `ErrorBoundary` 구성에 많이 사용됨
- 앱 전역에서 처리되지 않는 에러 핸들링을 위해 사용됨
- 물론 만능은 아님
- 컴포넌트별로 다른 ErrorBoundary를 적용해서 에러가 발생한 부분만 별도로 처리
  ⇒ 애플리케이션 전체에 에러 전파를 방지할 수 있음

```jsx
function App() {
  return (
    <ErrorBoundary name="parent">
      <ErrorBoundary name="child">
        <MyComponent />
      </ErrorBoundary>
    </ErrorBoundary>
  );
}
```

> 📣 주의할 점
> `componentDidCatch`는 개발 모드와 프로덕션 모드에서 다르게 작동함
> 개발모드에서는 에러발생 시 window까지 전파됨
> 프로덕션에서는 `componentDidCatch`로 잡히지 않은 에러만 window까지 전파됨
> ⇒ “개발모드에서 에러 발생을 확실히 확인시켜주기 위한 역할로 보인다” - 필자

> `componentDidCatch`의 두 번째 인수 `errorInfo`의 `componentStack`은 에러 추적을 위해 만들어짐
> 표시되는 이름은 `Function.name` 또는 `displayName`을 따름
> 원활한 디버깅을 위해 기명함수 또는 `displayName`을 적극 활용하자!

### 클래스형 컴포넌트의 한계

- 데이터 흐름을 **추적하기 어려움**
- 내부 로직의 **재사용이 어려움**
- 컴포넌트 내부 로직이 많아질수록, 크기가 **기하급수적으로 커짐**
- 함수에 비해 **높은 난이도**
- 코드 사이즈 **최적화가 어려움**
  - 번들링 시에 이름 최소화(minified)나 트리쉐이킹이 되지 않음
- 핫 리로딩(hot reloading)에 **불리함**
  - 클래스형 컴포넌트는 최초 렌더링 시에 instance를 생성하고, 내부에서 state를 관리함
  - instance 내부 render의 수정사항을 반영하려면 instance를 새로 만들어야 함
  - instance가 새로 만들어지면, 결국 값도 초기화 됨
  - 함수형 컴포넌트의 클로저로 함수가 다시 실행되어도 해당 state를 잃지 않고 다시 보여줄 수 있음

```jsx
class Counter extends React.Component {
  state = { count: 0 };

  render() {
    return (
      <div>
        <p>{this.state.count}</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          증가
        </button>
      </div>
    );
  }
}

// 1. render 내부 텍스트 수정
// 2. Webpack HMR(핫 리로드)이 새로운 코드로 컴포넌트를 다시 로드
// 3. 클래스 컴포넌트는 인스턴스를 새로 만들어야 함
//     state는 인스턴스 안에서 관리되므로
// 새로운 인스턴스 newInstacne.state = { count: 0 }이 됨
```

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>증가</button>
    </div>
  );
}

// 1. 함수 코드만 다시 실행됨
// 2. 컴포넌트의 state는 Fiber Node에 저장됨
// 3. hot reload 되어도 state의 값은 유지됨
```

---

## 2.3.2 함수형 컴포넌트

- React 16.8 에서 함수형 컴포넌트에서 사용 가능한 훅이 등장함
- 코드가 간결해짐
- render 내부에 묶일 필요가 없고, this 바인딩을 신경쓰지 않아도 됨
- state는 객체가 아니라 각각 원시값으로 관리됨
- return문 에서도 this 없이 props, state에 접근이 가능해짐

---

## 2.3.3 함수형 vs 클래스형

### 생명주기 메서드의 부재

- 함수형 컴포넌트는 useEffect 훅으로 생명주기 메서드와 비슷한 구현이 가능함
- “비슷”이지 똑같은건 아님

### 함수형 컴포넌트와 렌더링된 값

> 함수형 컴포넌트는 렌더링된 값을 고정하고, 클래스형 컴포넌트는 그렇지 못하다
> \- 댄 아브라모프, (리액트 개발자)

### 클래스형 컴포넌트를 공부해야 할까?

- 클래스형 컴포넌트가 사라지지는 않을 것
  - 수많은 커뮤니티와 레거시..
- 충분히 준비되어 있다면, 함수형 컴포넌트로의 이관을 고려해볼만 함
- 일부 클래스 컴포넌트의 메서드, 에러처리 등을 위해 어느정도 지식은 필요하다!

---

## 2.3.4 정리

---

# 2.4 렌더링은 어떻게 일어나는가?

> **브라우저**의 렌더링 ⇒ HTML + CSS ⇒ UI 그리기

> **리액트**의 렌더링 ⇒ DOM 트리 생성

## 2.4.1 리액트의 렌더링이란?

- 리액트와 브라우저의 렌더링은 다름

### 리액트에서의 렌더링

> 리액트 애플리케이션 트리 안에 있는 모든 컴포넌트들이 현재 자신들이 가지고 있는 props와 state의 값을 기반으로 어떻게 UI를 구성하고, 이를 바탕으로 어떤 DOM 결과를 브라우저에게 제공할 것인지 계산하는 일련의 과정을 의미함

⇒ props와 state에 따라 UI가 어떻게 생겼을지 계산하고, 가상 DOM트리를 만들어서 실제 돔에 반영(커밋)

---

## 2.4.2 리액트의 렌더링이 일어나는 이유

### 1. 최초 렌더링

- 사용자가 처음 애플리케이션에 진입할 때, 당연히 뭘 보여줘야 함
- 브라우저에 보여줄 정보를 제공하기 위해 최초 렌더링 수행함

### 2. 리렌더링(이 발생하는 경우)

**클래스형 컴포넌트**

- setState, state 변화 ⇒ 리렌더링
- forceUpdate 실행

**함수형 컴포넌트**

- `useState()`의 `setter` 실행
- `useReducer()`의 `dispatch` 실행
- `key` props 변경 - 모든 컴포넌트에서 사용할 수 있는 특수한 props
- props 변경
- 부모 컴포넌트 리렌더링 ⇒ 무조건 자식 컴포넌트 리렌더링

### `key` props

- 형제 요소들 사이 동일한 요소 식별하기 위한 값
- current 트리와 workInProgress 트리 사이 변경을 구별 할 때 필요함
- 리렌더링 최소화를 위해 필요함
- 반대로, 강제로 리렌더링을 일으키기도 가능함

---

## 2.4.3 리액트의 렌더링 프로세스

- 컴포넌트 루트 → 아래쪽으로 내려가면서 업데이트가 필요한 컴포넌트를 탐색함
- 발견하면 `render()` 또는 `FunctionComponent()` 호출하고 결과물 저장
- 결과(컴파일 된 JSX의 반환값)를 수집해서 가상 DOM과 비교(diffing)하고 변경할 사항을 수집함
- 위와 같은 과정을 재조정(Reconciliation)이라 함

---

## 2.4.4 렌더와 커밋

```markdown
Render Phase
↓
Commit Phase
↓
DOM 업데이트 적용, ref 연결
↓
useLayoutEffect 실행
↓
브라우저 페인트
↓
componentDidMount, componentDidUpdate 실행
```

### 렌더 단계(Render Phase)

- 컴포넌트를 렌더링하고 변경사항을 계산하는 모든 작업
- `render()` 결과 또는 함수형 컴포넌트 반환값과 이전 가상 DOM 비교
- 변경이 필요한 컴포넌트 확인하는 단계

**비교대상**

1. **type**
2. **props**
3. **key**

### 커밋 단계(Commit Phase)

- 변경 사항을 실제 DOM에 적용해 사용자에게 보여주는 과정
- 이 단계가 끝나야 비로소 브라우저의 렌더링이 발생함

### 커밋 이후

- 리액트가 먼저 커밋 단계에서 DOM을 업데이트 하고,
- 이후 DOM노드 및 인스턴스를 가리키도록 리액트 내부 참조 업데이트가 이루어짐
- `componentDidMount`, `componentDidUpdate`, `useLayoutEffect` 실행됨

### 리액트 렌더링 ≠ DOM 업데이트 발생

> 리액트 렌더링이 일어난다고 해서 무조건 DOM 업데이트가 일어나는건 아님
> 렌더링 수행했으나 계산된 변경사항이 없다면(커밋단계가 필요없다면) 생략될 수 있음

- 리액트의 렌더링은 꼭 가시적인 변경이 일어나지 않아도 발생함
- 커밋 단계가 생략될 수 있음

---

## 2.4.5 일반적인 렌더링 시나리오

```jsx
import { useState } from "react";

export default function A() {
  return (
    <div className="App">
      <h1>Hello React!</h1>
      <B />
    </div>
  );
}

function B() {
  const [count, setCount] = useState(0);

  function handleButtonClick() {
    setCount((previous) => previous + 1);
  }

  return (
    <>
      <label>
        <C number={counter} />
      </label>
      <button onClick={handleButtonClick}>+</button>
    </>
  );
}

function C({ number }) {
  return (<div>{number} <D /></div>)
}

function D = memo(() {
  return (<>안녕하세요!</>)
})
```

> 사용자가 B 컴포넌트의 버튼을 눌러서 counter 변수를 업데이트 한다고 가정

1. `B` 컴포넌트의 setState 호출
2. `B` 컴포넌트의 리렌더링 작업이 렌더링 큐에 들어감
3. ~~리액트는 트리 최상단에서부터 렌더링 경로를 검사함~~
   ⇒ 업데이트가 발생한 FiberNode를 루트로 그 하위만 렌더링
4. `A` 컴포넌트는 리렌더링 필요 표시가 없으므로 따로 작업 안함
5. `B` 는 업데이트 필요, 리렌더링
6. `B` 리렌더링으로 반환되는 `C` 리렌더링
7. `D`는 memoization으로 리렌더링 방지

> **🤔 Props 변경 ≠ 리렌더링**
>
> 1. 부모가 리렌더링됨
> 2. 부모에서 자식에게 새로운 props 전달됨
> 3. 자식은 새로운 props 비교 후
>    - React.memo 없음 → 무조건 리렌더
>    - React.memo 있음 → 비교해서 같으면 스킵
>
> 즉, 리렌더링 원인은 항상 “해당 Fiber가 업데이트 대상으로 표시되었는가”로 결정됨

> 리렌더의 원인은 아래 중 하나:
>
> - 해당 컴포넌트 Fiber에 state 업데이트 발생
> - 해당 컴포넌트 Fiber에 부모의 렌더링으로 인해 새 element가 생성됨
> - context value 변경
> - reducer 업데이트
> - forceUpdate (클래스)
> - Suspense boundary 활동
> - error boundary 동작 등
>
> props 자체는 단독으로 리렌더링을 발생시키지 않음

---

## 2.4.6 정리

> 리액트 렌더링 시나리오를 이해하면
> 컴포넌트 트리구조 개선이나 불필요한 리렌더링 감소로 최적화에 도움이 된다.

---

# 2.5 메모이제이션

리액트에서 제공되는 API 중 useMemo, useCallback 훅과 memo는 렌더링을 줄이기 위해 제공됨

최적화 기법에 사용은 되지만, 사용은 하지만 정확히 언제 써야할까?

메모이제이션 최적화는 오래된 논쟁 중 하나

> 19 버전에 컴파일러 출시로 쉬워졌지만, 여전히 모든 상황에 완벽히 적용되진 않음

> 자동 최적화가 “개발자 노-력 없이 성능 확보”를 보장해주진 않음
> → 상황에 따라 기존 방식의 판단과 적용이 필요함

## 2.5.1 섣최독(섣부른 최적화는 독이다!)

- 필요한 곳을 신중하게 골라서 메모이제이션 해야한다는 의견
- 메모이제이션도 비용이 드는 작업
  1. 값 비교, 렌더링/재계산 여부 확인 작업
  2. 저장 비용(메모리)
- **“premature optimization”** 라고 한다고 함
- 메모이제이션이 **“silver bullet”** 은 아니다

> useMemo를 사용하지 않고도 작동할 수 있도록 코드를 작성하고, 그것을 추가해 성능을 최적화 하세요
> [공식문서](https://ko.react.dev/reference/react/useMemo)

---

## 2.5.2 모메(모조리 메모이제이션)

- 렌더링 비용은 비싸다는 의견
- 규모의 증가로 현실적으로 최적화나 성능 향상에 시간 투자가 어려움
- 일단 memo로 감싸고 보자
- 리액트는 어차피 다름 렌더링과 구별을 위해 결과를 저장해둠
- memo로 인해 발생하는 props에 대한 얕은 비교로 발생하는 비용 감수하고 memo로 전부 감싸자

### memo 하지 않았을 때 잠재적 위험

- 렌더링 비용
- 컴포넌트 내부 복잡한 로직 재실행
- 모든 자식 컴포넌트 리렌더링
- 리액트의 트리 비교 비용

---

## 2.5.3 결론

### 리액트에 대한 학구열과 시간이 있다면, 지양하면서 꼼꼼하게 살펴보자

### 현업에서 다루거나 시간적 여유가 없다면, 의심되는 곳에 먼저 다 적용해버리자

---
