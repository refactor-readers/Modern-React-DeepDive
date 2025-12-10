# 3장 리액트 훅 깊게 살펴보기

## 3.1 리액트의 모든 훅 파헤치기

#### hook
hook의 등장으로 함수형 컴포넌트가 상태값을 사용하거나 클래스형 컴포넌트의 생명주기 메서드를 대체하는 등의 다양한 작업이 가능해졌다

### useState
```js
import {useState} from 'react'

const [state, setState] = useState(initalState)
```
useState의 인수로 state의 초기값을 넘겨줘야하고 값이 없다면 초기값은 자동으로 undefined이다


```js
function Component() {
  let state = 'hello'

  function handleButtonClick() {
    state = 'hi'
  }

  return (
    <>
      <h1>{state}</h1>
      <button onClick={handleButtonClick}>hi</button>
    </>
  )
}
```
위 코드는 리렌더링이 작동하지 않는다 왜냐하면 리액트에서 렌더링은 함수형 컴포넌트의 return과 클래스형 컴포넌트의 render 함수를 실행한 다음 이 결과를 이전의 리액트 트리와 비교해 리렌더링이 필요한 부분만 업데이트해 이뤄진다고 정리했었다

```js
function Component(){
  const [,triggerRender] = useState()
  let state = 'hello'

  function handleButtonClick() {
    state = 'hi'
    triggerRender()
  }

  return (
    <>
      <h1>{state}</h1>
      <button onClick={handleButtonClick}>hi</button>
    </>
  )
}
```
set함수를 통해 리렌더링을 발생시켰는데 값이 변경되지 않는 이유는
매번 렌더링이 발생될 때마다 함수는 다시 새롭게 실행되고, 새롭게 실행되는 함수에서 state는 매번 hello로 초기화되므로 변경되지 않는다

***이를 위해 useState는 클로저를 활용해서 useState가 호출된 이후에도 state값을 계속 참조할 수 있다***

아래는 useState의 내부를 흉내낸 코드이다
```js
const MyReact = function () {
  const global = {}
  let index = 0

  function useState(initialState) {
    if (!global.states) {
      // 애플리케이션 전체의 states 배열을 초기화한다.
      // 최초 접근이라면 빈 배열로 초기화한다.
      global.states = []
    }

    // states 정보를 조회해서 현재 상태값이 있는지 확인하고,
    // 없다면 초깃값으로 설정한다.
    const currentState = global.states[index] || initialState
    // states의 값을 위에서 조회한 현재 값으로 업데이트한다.
    global.states[index] = currentState

    // 즉시 실행 함수로 setter를 만든다.
    const setstate = (function () {
      // 현재 index를 클로저로 가둬놔서 이후에도 계속해서 동일한 index에
      // 접근할 수 있도록 한다.
      let currentIndex = index
      return function (value) {
        global.states[currentIndex] = value
        // 컴포넌트를 렌더링한다. 실제로 컴포넌트를 렌더링하는 코드는 생략했다.
      }
    })()

    // useState를 쓸 때마다 index를 하나씩 추가한다. 이 index는 setState에서 사용된다.
    // 즉, 하나의 state마다 index가 할당되었고 그 index가 배열의 값(global.states)을
    // 가리키고 필요할 때마다 그 값을 가져오게 한다.
    index = index + 1

    return [currentState, setState]
  }

  // 실제 useState를 사용하는 컴포넌트
  function Component() {
    const [value, setValue] = useState(0)
    // ...
  }
}
```

위 코드에서 알 수 있는 클로저를 사용한 모습을 볼 수 있고
추가로 왜 훅을 반복문이나 조건문안에 사용하면 안되는지 알 수 있다
> Q1. 이유가 뭘까요?

#### 게으른 초기화
useState를 사용할 때 초기값인수로 함수를 주면 state가 최초생성될 때만 실행하게된다
리액트팀에서는 무거운 연산이 요구될 때 사용하라고 권장한다

### useEffect
useEffect가 생명주기 메서드를 대체하기 위해 만들어진 훅이라는 오해가 있지만 그렇지 않다
useEffect는 애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 매커니즘이다.

즉, 렌더링 할때마다 의존성에 있는 값을 보면서 이 의존성의 값이 이전과 다른게 하나라도 있으면 부수 효과를 실행하는 함수다

useEffect는 콜백이 실행될 때마다 이전의 클린업 함수가 존재한다면 그 클린업 함수를 실행한 뒤에 콜백을 실행한다. 클린업 함수는 언마운트라기보다는 함수형 컴포넌트가 리렌더링됐을 때 의존성 변화가 있었을 당시 이전의 값을 기준으로 실행되는, 말 그대로 이전 상태를 청소해 주는 개념으로 보는 것이 옳다

#### 의존성 배열

값을 안넘기거나 빈배열 혹은 값을 넣을 수 있다

- 빈배열이라면 최초 렌더링 후 1번
- 값이 없다면 매 렌더링 이후
- 값이 있다면 해당 값이 변함을 감지하면 실행


#### useEffect를 사용할 때 주의할 점

1. eslint-disable-line react-hooks/exhaustive-deps 주석은 최대한 지양하라
> 일부 라이브러리에서는 제공하는 함수를 의존성배열에 넣지 말라고 권장하고있다

> **Q2. 라이브러리의 일부 함수는 의존성 배열에 왜 넣지 않기를 권장할까?**

2. useEffect의 첫 번째 인수에 함수명을 부여하라
> 그 의도를 파악하기 위함

3. 거대한 useEffect를 만들지 마라
> 만약 부득이하게 크게 만들어야 한다면 적은 의존성 배열로 분리하는것이 좋다

4. 불필요한 외부 함수를 만들지 마라
> useEffect내에서 사용할 부수효과라면 내부에서 만들어서 사용하는 편이 좋다

#### useEffect의 콜백 인수로 비동기 함수를 넣을 수 없다

비동기 useEffect는 state의 경쟁 상태를 야기할 수 있고 클린업 함수의 실행 순서도 보장할 수 없기 때문에 개발자의 편의를 위해 useEffect에서 비동기 함수를 인수로 받지 않는다고 볼 수 있다.


### useMemo

결과를 메모이제이션해두는 훅으로 최적화가 필요할 때 가장 먼저 언급되는 훅이다

어떠한 값을 계산할 때 해당 값을 연산하는 데 비용이 많이 든다면 사용해 봄 직하다

### useCallback

useMemo와 달리 인수로 넘겨받은 콜백 자체를 기억한다 즉, 함수를 새로 만들지 않고 재사용한다는 의미이다.

이로인해 함수의 참조값이 변하지 않으므로 해당 함수가 다른 의존성 배열에 들어갔을때 의도하지않은 동작을 막을 수 있다

### useRef

useState와의 차이
- useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있다.
- useRef는 그 값이 변하더라도 렌더링을 발생시키지 않는다.

추가로 컴포넌트가 렌더링될 때만 생성되면 컴포넌트 인스턴스가 여러개라도 별개의 값을 바라본다
> 일반적인 변수를 사용하면 렌더링되지 않았어도 메모리에 불필요한 값을 갖게된다

useRef를 사용하면 개발자가 원하는 시점의 값을 렌더링에 영향을 미치지 않고 보관해둘 수 있다

### useContext

#### Context란

props drilling을 방지하기 위해 등장한 개념이다

명시적인 props전달 없이도 하위 컴포넌트가 모두 자유롭게 사용할 수 있다

#### useContext 훅

useContext를 사용하면 상위 컴포넌트 어딘가에서 선언된 <Context.Provider />에서 제공한 값을 사용할 수 있게 된다 만약 Provider가 여러개라면 가장 가까운 Provider의 값을 가져오게 된다

context가 없을때 값을 사용하면 에러가 나느데 이럴땐 에러 방지 코드를 사용해야 한다

```js
const MyContext = createContext<{ hello: string } | undefined>(undefined)

function ContextProvider(
  {
    children,
    text,
  }: PropsWithChildren<{ text: string }>
) {
  return (
    <MyContext.Provider value={{ hello: text }}>{children}</MyContext.Provider>
  )
}

function useMyContext() {
  const context = useContext(MyContext)
  if (context === undefined) {
    throw new Error(
      'useMyContext는 ContextProvider 내부에서만 사용할 수 있습니다.',
    )
  }
  return context
}

function ChildComponent() {
  // 타입이 명확히 설정되어 있어서 굳이 undefined 체크를 하지 않아도 된다.
  // 이 컴포넌트가 Provider 하위에 없다면 에러가 발생할 것이다.
  const { hello } = useMyContext()

  return <>{hello}</>
}

function ParentComponent() {
  return (
    <ContextProvide text="react">
      <ChildComponent />
    </ContextProvider>
  )
}
```

다수의 Provider와 useContext를 사용할때, 특히 타입스트립스틑 사용하고 있다면 위와 같이 별도 함수로 감싸서 사용하는 것이 좋다


#### 주의할 점

- useContext가 선언돼 있으면 Provider에 의존성을 가지고 있어서 재활용하기 힘든 컴포넌트가 된다
- 상태 주입하는 것이라 렌더링 최적화에는 아무런 도움이 되지 않는다

### useReducer

useState의 심화버전이다
- 반환값은 useState와 동일하게 길이가 2인 배열이다
  - state: 현재 갖고있는 값
  - dispatcher: state를 업데이트하는 함수이다 setState는 값을 넘겨주지만 여기서는 action을 넘겨준다
- useState의 인수와 달리 2~3개의 인수를 필요로 한다
  - reducer: 기본 action을 정의하는 함수
  - initialState: 초기값
  - init:useState의 인수로 함수를 넘겨줄 때처럼 초깃값을 지연해서 생성시키고 싶을때 사용하며 필수값이 아니다

useReducer의 목적은 복잡한 형태의 state를 사전에 정의된 dispatcher로만 수정할 수 있게 만들어 줌으로써. state값을 변경하는 시나리오를 제한적으로 두고 이에 대한 변경을 빠르게 확인할 수 있게끔 하는것이 목적이다

### useImprerativeHandle
실제 개발 과정에서는 자주 볼 수 없다

#### forwardRef
Props에 ref를 넣어 HTMLElement에 접근하는 용도로 사용된다

#### useImprerativeHandle란
부모에게서 넘겨받은 ref를 원하는 대로 수정할 수 있는 훅이다
useImprerativeHandle를 사용하면 부모 컴포넌트에서 노출되는 값을 원하는 대로 바꿀 수 있다

### useLayoutEffect
공식문서에는 이와 같이 명시하고 있다

"이 함수의 시그니처는 useEffect와 동일하나, 모든 DOM의 변경 후에 동기적으로 발생한다"

즉 실행순서는 아래와 같다

1. 리액트가 DOM을 업데이트
2. useLayoutEffect를 실행
3. 브라우저에 변경 사항을 반영
4. useEffect를 실행

렌더링 전에 동작하기 때문에 성능에 문제가 있을 수 있으니 DOM요소 기반 애니메이션, 스크롤 위치 제어 등 반드시 필요한 상황에만 사용하는 것이 좋다

### useDebugValue

일반적으로 웹서비스에서 사용하는 훅은 아니고 디버깅하고 싶은 정보를 이 훅에다 사용하면 리액트 개발자 도구에서 볼 수 있게 해준다

```js
// 현재 시간을 반환하는 사용자 정의 훅
function useDate() {
  const date = new Date()
  // useDebugValue로 디버깅 정보를 기록
  useDebugValue(date, (date) => `현재 시간: ${date.toISOString()}`)
  return date
}

export default function App() {
  const date = useDate()
  const [counter, setCounter] = useState(0) // 렌더링을 발생시키기 위한 변수

  function handleClick() {
    setCounter((prev) => prev + 1)
  }

return (
  <div className="App">
    <h1>
      {counter} {date.toISOString()}
    </h1>
    <button onClick={handleClick}>+</button>
  </div>
)
}
```

### 훅의 규칙

rules-of-hooks라는 몇가지 규칙이 존재한다

1. 최상위에서만 훅을 호출해야 한다. 반복문이나 조건문, 중첩된 함수 내에서 훅을 실행할 수 없다. 이규칙을 따라야만 컴포넌트가 렌더링될 때마다 항상 동일한 순서로 훅이 호출되는 것을 보장할 수 있다.
2. 훅을 호출할 수 있는 것은 리액트 함수형 컴포넌트, 혹은 사용자 정의 훅의 두 가지 경우뿐이다. 일반 js함수에서는 훅을 사용할 수 없다.

훅에 대한 정보 저장은 리액트 어딘가에 있는 index와 같은 키를 기반으로 구현되어있다
(실제로는 객체기반 링크드 리스트에 더 가깝다)

즉, 링크드 리스트가 깨지면 안되기 때문에 순서가 중요하다

## 3.2 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

### 사용자 정의 훅
서로 다른 컴포넌트 내부에서 같은 로직을 공유하고자 할 때 주로 사용되는 것이 사용자 정의 훅이다

훅은 함수형 컴포넌트 내부 또는 사용자 정의 훅 내부에서만 사용해야하기 때문에 이름이 use로 시작해야한다

### 고차 컴포넌트

가장 대표적인 고차 컴포넌트로는 React.memo가 있다
> 컴포넌트 메모이제이션

아래는 고차컴포넌트 예시다(인증 관련 로직)

```js
interface LoginProps {
  loginRequired?: boolean
}

function withLoginComponent<T>(Component: ComponentType<T>) {
  return function (props: T & LoginProps) {
    const { loginRequired, ...restProps } = props

    if (loginRequired) {
      return <>로그인이 필요합니다.</>
    }

    return <Component {...(restProps as T)} />
  }
}

// 원래 구현하고자 하는 컴포넌트를 만들고, withLoginComponent로 감싸기만 하면 끝이다.
// 로그인 여부, 로그인이 안 되면 다른 컴포넌트를 렌더링하는 책임은 모두
// 고차 컴포넌트인 withLoginComponent에 맡길 수 있어 매우 편리하다.
const Component = withLoginComponent((props: { value: string }) => {
  return <h3>{props.value}</h3>
})

export default function App() {
  // 로그인 관련 정보를 가져온다.
  const isLogin = true
  return <Component value="text" loginRequired={isLogin} />
  // return <Component value="text" />;
}
```

#### 사용자 정의 훅이 필요한 경우

- 리액트에서 제공하는 훅으로 공통 로직을 격리할 수 있는 경우
- 단순히 컴포넌트 전반에 걸쳐 동일한 로직으로 값을 제공하거나 특정한 훅의 작동을 취하게 하고 싶은 경우

#### 고차 컴포넌트를 사용해야 하는 경우

- 애플리케이션 관점에서 컴포넌트를 감추고 공통 컴포넌트를 노출하는 것이 필요할 때(인증관련)

함수형 컴포넌트의 반환값, 즉 렌더링의 결과물에도 영향을 미치는 공통 로직이라면 고차컴포넌트를 사용하자