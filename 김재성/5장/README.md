# 5장. 리액트와 상태 관리 라이브러리

## 5.1 상태 관리는 왜 필요한가?

웹 개발에서 상태를 의미하는 대표적인 것들이 있다

- UI: 기본적으로 웹 애플리케이션에서 상태라 함은 상호 작용이 가능한 모든 요소의 현재 값을 의미한다. 다크/라이트 모드, 라디오를 비롯한 각종 input, 알림창의 노출 여부 등 많은 종류의 상태가 존재한다.
- URL: 브라우저에서 관리되고 있는 상태값으로, 여기에도 우리가 참고할 만한 상태가 존재할 수 있다. 예를 들어 query prameter등이 있겠다
- Form: 폼에도 상태가 존재한다. loading인지 submit인지 disabled인지 validation등 모두가 상태로 관리된다.
- 서버에서 가져온 값: 클라이언트에서 서버로 요청을 통해 가져온 값도 상태로 볼 수 있다. API요청 등

### 리액트 상태 관리의 역사

#### Flux 패턴의 등장
리액트팀에서는 비대해진 웹 환경에서의 양방향 데이터 통신이 상태관리를 매우 어렵게 한다고 판단해서 단방향으로 데이터 흐름을 변경하는 것을 제안하는데 이것이 바로 Flux패턴의 시작이다

각 용어를 정의하면

- 액션(action): 어떠한 작업을 처리할 액션과 그 액션 발생 시 함께 포함시킬 데이터를 의미한다. 액션 타입과 데이터를 각각 정의해 이를 디스패처로 보낸다.

- 디스패처(dispatcher): 액션을 스토어에 보내는 역할을 한다. 콜백 함수 형태로 앞서 액션이 정의한 타입과 데이터를 모두 스토어에 보낸다.

- 스토어(store): 여기에서 실제 상태에 따른 값과 상태를 변경할 수 있는 메서드를 가지고 있다. 액션의 타입에 따라 어떻게 이를 변경할지가 정의돼 있다.

- 뷰(view): 리액트의 컴포넌트에 해당하는 부분으로, 스토어에서 만들어진 데이터를 가져와 화면을 렌더링하는 역할을 한다. 또한 뷰에서도 사용자의 입력이나 행위에 따라 상태를 업데이트하고자 할 수 있을 것이다. 이 경우에는 다음 그림처럼 뷰에서 액션을 호출하는 구조로 구성된다.

![flux패턴](/김재성//5장//flux.png "flux패턴")

이러한 단방향 데이터 흐름 방식은 당연히 불편함도 존재한다.. 사용자의 입력에 따라 데이터를 갱신하고 화면을 어떻게 업데이트해야 하는지도 코드로 작성해야 하므로 코드의 양이 많아지고 개발자도 수고로워진다. 그러나 데이터의 흐름은 모두 액션이라는 한 방향(단방향)으로 줄어들므로 데이터의 흐름을 추적하기 쉽고 코드를 이해하기가 한결 수월해진다.

리액트는 대표적인 단방향 데이터 바인딩을 기반으로 한 라이브러리였으므로 이러한 단방향 흐름을 정의하는 Flux 패턴과 매우 궁합이 잘 맞았다.

#### 시장 지배자 리덕스의 등장

Redux는 최초에는 Flux 구조를 구현하기 위해 만들어진 라이브러리 중 하나였다. 이에 한 가지 더 특별한 것은 여기에 Elm 아키텍처를 도입했다는 것이다. Elm 아키텍처를 이해하려면 먼저 Elm이 뭔지를 알아야 한다 Elm은 웹페이지를 선언적으로 작성하기 위한 언어다

ELm은 model, update, view 이 3가지가 핵심 아키텍처이다

- model: 애플리케이션의 상태를 의미한다.
- view: 모델을 표현하는 HTML을 말한다.
- update: 모델을 수정하는 방식을 말한다

즉, Elm은 Flux와 마찬가지로 데이터 흐름을 세 가지로 분류하고, 이를 단방향으로 강제해 웹 애플리케이션의 상태를 안정적으로 관리하고자 노력했다. 그리고 리덕스는 이 Elm 아키텍처의 영향을 받아 작성되었다

리덕스는 하나의 상태 객체를 스토어에 저장해 두고, 이 객체를 업데이트하는 작업을 디스패치해 업데이트를 수행한다. 이러한 작업은 reducer함수로 발생시킬 수 있는데, 이 항수의 실행은 웹 애플리케이션 상태에 대한 완전히 새로운 복자본을 반환한 다음, 애플리케이션에 이 새롭게 만들어진 상태를 전파하게 된다.

이렇게 하나의 글로벌 상태 객체를 통해 이 상태를 하위 컴포넌트에 전파할 수 있기 때문에 prop 내려주기 문제를 해결할 수 있었고, 스토어가 필요한 컴포넌트라면 connect만 쓰면 접근이 가능했다

단, 단점은 하나의 상태를 바꾸고 싶을때 해야 할 일이 너무 많았다 즉 보일러플레이트가 너무 많다는 비판의 목소리가 있었다

#### Context API와 useContext

리액트가 처음 나온뒤 상태주입에 대한 고민은 계속 이어져왔다
redux는 보일러플레이트가 부담스러웠고 이때 리액트 16.3버전에서 새로운 Context API를 출시했고 props로 상태를 넘겨주지 않아도 Context API를 사용하면 원하는 곳에서 Context Provider가 주입하는 상태를 사용할 수 있게 된 것이다.

이전 getChildContext는 불필요한 렌더링이 일어난다는 점 및 context를 인수로 받아야하기때문에 컴포넌트 결합도가 높아지는 단점이 있었다

```js
type Counter = {
  count: number
}

const CounterContext = createContext<Counter | undefined>(undefined)

class CounterComponent extends Component {
  render() {
    return (
      <CounterContext.Consumer>
        {(state) => <p>{state?.count}</p>}
      </CounterContext.Consumer>
    )
  }
}

class DummyParent extends Component {
  render() {
    return (
      <>
        <CounterComponent />
      </>
    )
  }
}

export default class MyApp extends Component<{}, Counter> {
  state = { count: 0 }

  componentDidMount() {
    this.setState({ count: 1 })
  }

  handleClick = () => {
    this.setState((state) => ({ count: state.count + 1 }))
  }

  render() {
    return (
      <CounterContext.Provider value={this.state}>
        <button onClick={this.handleClick}>+</button>
        <DummyParent />
      </CounterContext.Provider>
    )
  }
}
```
이 코드를 보면 부모 컴포넌트인 MyApp에 상태가 선언돼 있고, 이를 Context로 주입하고 있는 것을 볼 수 있다. 그리고 Provider로 주입된 상태는 자식의 자식인 CounterComponent에서 사용하고 있음을 알 수 있다.

그러나 이전에 이야기한 것처럼 Context API는 상태 관리가 아닌 주입을 도와주는 기능이며, 렌더링을 막아주는 기능 또한 존재하지 않으니 사용할 때 주의가 필요하다.

#### 훅의 탄생, 그리고 React Query와 SWR

리액트 16.8버전에서 함수형 컴포넌트에 사용할 수 있는 다양한 훅 API를 추가했다.
이 훅 API는 기존에 무상태 컴포넌트를 선언하기 위해서만 제한적으로 사용됐던 함수형 컴포넌트가 클래스형 컴포넌트 이상의 인기를 구가할 수 있도록 많은 기능을 제공했다.

대표적으로 state를 매우 손쉽게 재사용 가능하도록 만들 수 있다는 것이다.
```js
function useCounter() {
  const [count, setCount] = useState(0)

  function increase() {
    setCount((prev) => prev + 1)
  }

  return { count, increase }
}
```
useCounter는 단순히 count state와 이를 1씩 올려주는 increase로만 구성돼 있지만 내부적으로 관리하고 있는 state도 있으며, 또 이를 필요한 곳에서 재사용할 수도 있게 됐다.

이러한 훅과 state의 등장으로 이전에는 볼 수 없던 방식의 상태가 등장하는데 바로 React Query와 SWR이다

두 라이브러리 모두 외부에서 데이터를 불러오는 fetch를 관리하는데 특화된 라이브러리지만, API호출에 대한 상태를 관리하고 있기 때문에 HTTP 요청에 특화된 상태 관리 라이브러리라 볼 수 있다.

SWR의 예시를 보면
```js
import React from 'react'
import useSWR from 'swr'

const fetcher = (url) => fetch(url).then((res) => res.json())

export default function App() {
  const { data, error } = useSWR(
    'https://api.github.com/repos/vercel/swr',
    fetcher,
  )

  if (error) return 'An error has occurred.'
  if (!data) return 'Loading...'

  return (
    <div>
      <p>{JSON.stringify(data)}</p>
    </div>
  )
}
```
useSWR의 첫번째 인수로 조회할 API주소를, 두번째로 fetch함수를 점겨준다, API주소를 키로도 사용되며, 이후에 다른 곳에서 동일한 키로 호출하면 재조회하는 것이 아니라 캐시의 값을 활용한다

#### Recoil, Zustand, Jotai, Valtio에 이르기까지

훅이라는 새로운 패러다임의 등장에 따라, 훅을 활용해 상태를 가져오거나 관리할 수 있는 다양한 라이브러리가 등장하게 된다 Recoil, Zustand, Jotai, Valtio 등이 있고

```js
// Recoil
const counter = atom({ key: 'count', default: 0 })
const todoList = useRecoilValue(counter)

// Jotai
const countAtom = atom(0)
const [count, setCount] = useAtom(countAtom)

// Zustand
const useCounterStore = create((set) => ({
  count: 0,
  increase: () => set((state) => ({ count: state.count + 1 })),
}))
const count = useCounterStore((state) => state.count)

// Valtio
const state = proxy({ count: 0 })
const snap = useSnapshot(state)
state.count++
```
이렇게 훅을 활용해 작은 크기의 상태를 효율적으로 관리한다는 것이다

이는 기존 상태 관리 라이브러리의 아쉬운 점으로 지적받던 전역 상태 관리 패러다임에서 벗어나 개발자가 원하는 만큼의 상태를 지역적으로 관리하는 것을 가능하게 만들었고, 훅을 지원함으로써 함수형 컴포넌트에서 손쉽게 사용할 수 있다는 장점 또한 가지고 있다.

## 5.2 리액트 훅으로 시작하는 상태 관리

### 가장 기본적인 방법: useState와 useReducer

```js
function useCounter(initCount: number = 0) {
  const [counter, setCounter] = useState(initCount)

  function inc() {
    setCounter((prev) => prev + 1)
  }

  return { counter, inc }
}

function Counter1() {
  const { counter, inc } = useCounter()

  return (
    <>
      <h3>Counter1: {counter}</h3>
      <button onClick={inc}>+</button>
    </>
  )
}

function Counter2() {
  const { counter, inc } = useCounter()

  return (
    <>
      <h3>Counter2: {counter}</h3>
      <button onClick={inc}>+</button>
    </>
  )
}
```
훅은 이렇게 함수형 컴포넌트 어디에서든 사용할 수 있게 구현이 가능하다

만약 훅이 아니라면 기능이 필요한 곳 각각의 컴포넌트에서 모두 위와 같은 내용을 구현해야만 했을 것이다.

...useReducer는 스킵

### 지역 상태의 한계를 벗어나보자: useState의 상태를 바깥으로 분리하기

useState의 한계는 클로저 내부에서 관리되어 지역 상태로 생성되기 때문에 해당 컴포넌트에서만 사용할 수 있다는 단점이 있다. 

- 일방적으로 getter와 setter를 외부로 빼서 관리하면 리액트의 리렌더링 규칙에 포함되지 않아서 작동은 하지만 화면이 변하지 않는다
- state를 내부로 두고 handleClick같은 이벤트를 외부로 두면 내외부 동일한 상태를관리해야해서 비효율적이다

컴포넌트 레벨의 지역 상태를 벗어나느 새로운 상태 관리 코드 예시를 볼건데 이를 store로 정의한다
```js
export const createStore = <State extends unknown>(
  initialState: Initializer<State>,
): Store<State> => {
  // useState와 마찬가지로 초깃값을 게으른 초기화를 위한 함수 또한
  // 그냥 값을 받을 수 있도록 한다.
  // state의 값은 스토어 내부에서 보관해야 하므로 변수로 선언한다.
  let state = typeof initialState !== 'function' ? initialState : initialState()

  // callbacks는 자료형에 관계없이 유일한 값을 저장할 수 있는 Set을 사용한다.
  const callbacks = new Set<() => void>()
  // 언제든 get이 호출되면 최신값을 가져올 수 있도록 함수로 만든다.
  const get = () => state
  const set = (nextState: State | ((prev: State) => State)) => {
    // 인수가 함수라면 함수를 실행해 새로운 값을 받고,
    // 아니라면 새로운 값을 그대로 사용한다.
    state =
      typeof nextState === 'function'
        ? (nextState as (prev: State) => State)(state)
        : nextState

    // 값의 설정이 발생하면 콜백 목록을 순회하면서 모든 콜백을 실행한다.
    callbacks.forEach((callback) => callback())

    return state
  }

  // subscribe는 콜백을 인수로 받는다.
  const subscribe = (callback: () => void) => {
    // 받은 함수를 콜백 목록에 추가한다.
    callbacks.add(callback)
    // 클린업	실행 시 이를 삭제해서 반복적으로 추가되는 것을 막는다.
    return ()=>{
      callbacks.delete(callback)
    }
  }
  return {get, set, subscribe}
}
```

단계별로 살펴보면

1. 먼저 store의 초깃값을 state 또는 게으른 초기화 함수를 받아 store의 기본값을 초기화할 수 있게 해뒀다.

2. 1번에서 받은 인수를 바탕으로 함수를 실행하거나 초깃값 그 자체를 할당해 state 초깃값을 할당한다.

3. 컴포넌트로 넘겨받는 콜백 함수를 저장하기 위해 callbacks를 Set으로 선언한다. Set은 원시값이나 객체에 관계없이 유일한 값을 저장할 수 있어 중복 없이 콜백 함수를 저장하는 용도로 유용하다.

4. get을 함수로 만들어 매번 최신값을 가져올 수 있게 만든다.

5. set을 만들어 새로운 값을 넣을 수 있도록 만든다. useState의 두 번째 인수와 마찬가지로 함수일 수도, 단순히 값을 받을 수도 있다. 그리고 값을 설정한 이후에 callbacks를 순회해 등록된 모든 콜백을 실행한다. set으로 값을 설정하는 순간 콜백을 모두 실행해 컴포넌트의 렌더링을 유도할 것이다.

6. subscribe는 callbacks Set에 callback을 등록할 수 있는 함수다. callbacks.add와 더불어, 반환값으로는 등록된 callback을 삭제하는 함수를 반환한다. 이는 callbacks에 callback이 무한히 추가되는 것을 방지하게 만들어져 있으며, useEffect의 클린업 함수와 동일한 역할을 한다.

7. 마지막으로 get, set, subscribe를 하나의 객체로 반환해 외부에서 사용할 수 있도록 한다.

요약하면 createStore는 자신이 관리해야 하는 상태를 내부 변수로 가진 다음, get함수로 해당 변수의 최신값을 제공하며, set 함수로 내부 변수를 최신화하며, 이 과정에서 등록된 콜백을 모조리 실행하는 구조를 띠고 있다.

useStore훅을 만들어 이 store의 변화를 감지하는 예제를 보면
```js
export const useStore = <State extends unknown>(store: Store<State>) => {
  const [state, setState] = useState<State>(() => store.get())

  useEffect(() => {
    const unsubscribe = store.subscribe(() => {
      setState(store.get())
    })
    return unsubscribe
  }, [store])

  return [state, store.set] as const
}
```

1. 먼저 훅의 인수로 사용할 store를 받는다.

2. 이 스토어의 값을 초깃값으로 하는 useState를 만든다. 이제 이 useState가 컴포넌트의 렌더링을 유도한다.

3. useEffect는 store의 현재 값을 가져와 setState를 수행하는 함수를 store의 subscribe로 등록해 두었다. createStore 내부에서 값이 변경될 때마다 subscribe에 등록된 함수를 실행하므로 useStore 내부에서는 store의 값이 변경될 때마다 state의 값이 변경되는 것을 보장받을 수 있다.

4. 마지막으로 useEffect의 클린업 함수로 unsubscribe를 등록해 둔다. useEffect의 작동이 끝난 이후에는 callback에서 해당 함수를 제거해 callback이 계속해서 쌓이는 현상을 방지했다.

이렇게 리액트 외부에서 관리되는 값에 대한 변경을 추적하고 이를 리렌더링까지 할 수 있는 useStore를 만들 수 있지만 이미 리액트팀에서 useSubscription이라는 훅을 만들어놨다

useStore와의 차이는 selector와 subscribe에 대한 비교도 추가 되었다는 것이다

### useState와 Context를 동시에 사용해 보기

훅을 사용하는 서로 다른 스코프에서 스토어의 구조는 동일하되, 여러 개의 서로 다른 데이터를 공유해 사용하고 싶다면 store와 Context를 함께 사용하면 된다

```js
// Context를 생성하면 자동으로 스토어도 함께 생성한다.
export const CounterStoreContext = createContext<Store<CounterStore>>(
  createStore<CounterStore>({ count: 0, text: 'hello' }),
)

export const CounterStoreProvider = ({
  initialState,
  children,
}: PropsWithChildren<{
  initialState: CounterStore
}>) => {
  const storeRef = useRef<Store<CounterStore>>()

  // 스토어를 생성한 적이 없다면 최초에 한 번 생성한다.
  if (!storeRef.current) {
    storeRef.current = createStore(initialState)
  }

  return (
    <CounterStoreContext.Provider value={storeRef.current}>
      {children}
    </CounterStoreContext.Provider>
  )
}
```
CounterStoreContext를 통해 먼저 어떠한 Context를 만들지 타입과 함께 정의해 뒀다. 이렇게 타입과 함꼐 정의된 Context를 사용하기 위해 CounterStoreProvider를 정의했다. 이 Provider에서는 storeRef를 사용해서 스토어를 제공하는데, 그 이유는 Provider로 넘기는 props가 불필요하게 변경돼서 리렌더링되는 것을 막기 위해서다.. 이렇게 useRef를 사용했기 때문에 CounterStoreProvider는 오직 최초 렌더링에서만 스토어를 만들어서 값을 내려주게 될 것이다.

> Q1. Provider를 여러번 사용할때의 장단점이 뭘까요?

이제 이 Context에서 내려주는 값을 사용하기 위해 Context에서 제공하는 스토어에 접근해야 하기 때문에 useContext를 사용해 스토어에 접근할 수 있는 새로운 훅이 필요하다
```js
export const useCounterContextSelector = <State extends unknown>(
  selector: (state: CounterStore) => State,
) => {
  const store = useContext(CounterStoreContext)

  const subscription = useSubscription(
    useMemo(
      () => ({
        getCurrentValue: () => selector(store.get()),
        subscribe: store.subscribe,
      }),
      [store, selector],
    ),
  )

  return [subscription, store.set] as const
}
```

이제 이 새로운 훅과 Context를 사용하는 예제를 보면
```js
const ContextCounter = () => {
  const id = useId()
  const [counter, setStore] = useCounterContextSelector(
    useCallback((state: CounterStore) => state.count, []),
  )

  function handleClick() {
    setStore((prev) => ({ ...prev, count: prev.count + 1 }))
  }

  useEffect(() => {
    console.log(`${id} Counter Rendered`)
  })

  return (
    <div>
      {counter} <button onClick={handleClick}>+</button>
    </div>
  )
}

const ContextInput = () => {
  const id = useId()
  const [text, setStore] = useCounterContextSelector(
    useCallback((state: CounterStore) => state.text, []),
  )

  function handleChange(e: ChangeEvent<HTMLInputElement>) {
    setStore((prev) => ({ ...prev, text: e.target.value }))
  }

  useEffect(() => {
    console.log(`${id} Counter Rendered`)
  })

  return (
    <div>
      <input value={text} onChange={handleChange} />
    </div>
  )
}
export default function App() {
  return (
    <>
      {/* 0 */}
      <ContextCounter />
      {/* hi */}
      <ContextInput />

      <CounterStoreProvider initialState={{ count: 10, text: 'hello' }}>
        {/* 10 */}
        <ContextCounter />
        {/* hello */}
        <ContextInput />

        <CounterStoreProvider initialState={{ count: 20, text: 'welcome' }}>
          {/* 20 */}
          <ContextCounter />
          {/* welcome */}
          <ContextInput />
        </CounterStoreProvider>
      </CounterStoreProvider>
    </>
  )
}
```

주석을 보면 알 수 있듯이 이렇게 Context와 Provider를 기반으로 각 store값을 격리해서 관리했다. 이렇게 작성한 코드는 스토어를 사용하는 컴포넌트가 해당 상태가 어느 스토어에서 온 상태인지 신경쓰지 않아도 된다. 단지 해당 스토어를 기반으로 어떤 값을 보여줄지만 고민하면 되므로 좀 더 편리하게 코드를 작성할 수 있다

또한 Context와 Provider를 관리하는 부모 컴포넌트의 입장에서는 자신이 자식 컴포넌트에 따라 보여주고 싶은 데이터를 Context로 잘 격리하기만 하면 된다

결국 상태 관리 라이브러리들의 작동방식은 다음과 같이 요약할 수 있다
- useState, useReducer가 가지고 있는 한계, 컴포넌트 내부에서만 사용할 수 있는 지역 상태라는 점을 극복하기 위해 외부 어딘가에 상태를 둔다. 이는 컴포넌트의 최상단 내지는 상태가 필요한 부모가 될 수도 있고, 혹은 격리된 자바스크립트 스코트 어딘가일 수도 있다.
- 이 외부의 상태 변경을 각자의 방식으로 감지해 컴포넌트의 렌더링을 일으킨다.

### 상태 관리 라이브러리 Recoil, Jotai, Zustand

Recoil과 Jotai는 Context과 Provider, 그리고 훅을 기반으로 가능한 작은 상태를 효율적으로 관리하는데 초점을 맞추고 있다. 그리고 Zustand는 리덕스와 비슷하게 하나의 큰 스토어를 기반으로 상태를 관리하는 라이브러리다. Recoil, Jotai와는 다르게 이 하나의 큰 스토어는 Context가 아니라 스토어가 가지는 클로저를 기반으로 생성되며, 이 스토어의 상태가 변경되면 이 상태를 구독하고 있는 컴포넌트에 전파해 리렌더링을 알리는 방식이다

***...Jotai가 Recoil과 유사하고 Recoil이 개발중단됨에 따라 스킵***

#### Recoil에서 영감을 받은, 그러나 조금 더 유연한 Jotai

```js
import { atom, useAtom, useAtomValue } from 'jotai'

// 컴포넌트 외부에서 상태 선언
const counterState = atom(0)

function Counter() {
  // useState와 비슷하게 사용
  const [, setCount] = useAtom(counterState)

  function handleButtonClick() {
    setCount((count) => count + 1)
  }

  return (
    <>
      <button onClick={handleButtonClick}>+</button>
    </>
  )
}
// 다른 atom의 값으로부터 파생된 atom 생성
const isBiggerThan10 = atom((get) => get(counterState) > 10)

function Count() {
  // useAtomValue를 통해 getter만 가져옴
  const count = useAtomValue(counterState)
  const biggerThan10 = useAtomValue(isBiggerThan10)

  return (
    <>
      <h3>{count}</h3>
      <p>count is bigger than 10: {JSON.stringify(biggerThan10)}</p>
    </>
  )
}

export default function App() {
  return (
    <>
      <Counter />
      <Count />
    </>
  )
}
```
먼저 Jotai에서 상태를 선언하기 위해서는 atom이라는 API를 사용하는데, 이 API는 리액트의 useState와는 다르게 컴포넌트 외부에서도 선언할 수 있다는 장점이 있다. 

또한 atom은 값뿐만 아니라 함수를 인수로 받을 수 있는데, 이러한 특징을 활용해 다른 atom의 값으로부터 파생된 atom을 만들 수도 있다. 그리고 이 atom은 컴포넌트 내부에서 useAtom을 활용해 useState와 비슷하게 사용하거나 useAtomValue를 통해 getter만 가져올 수 있다. 

이렇게 기본적인 API 외에도 localStorage와 연동해 영구적으로 데이터를 저장하거나, Next.js, 리액트 네이티브와 연동하는 등 상태와 관련된 다양한 작업을 Jotai에서 지원한다.

##### 특징
가장 먼저 Recoil의 atom개념을 도입하면서도 API가 간결하다는 점을 꼽을 수 있다. Recoil의 atom에서는 각 상태값이 모두 별도의 키를 필요로 하기 때문에 이 키를 별도로 관리해야 하는데, Jotai는 이러한 부분을 추상화해 사용자가 키를 관리할 필요가 없다. 

Jotai가 별도의 문자열 키가 없이도 각 값들을 관리할 수 있는 것은 객체의 참조를 통해 값을 관리하기 떄문이다. 
객체의 참조를 WeakMap에 보관해 해당 객체 자체가 변경되지 않는 한 별도의 키가 없이도 객체의 참조를 통해 값을 관리할 수 있다
> WeakMap은 Map과 다르게 키 객체가 다른 참조를 모두 잃으면 데이터도 함께 삭제된다

#### 작고 빠르며 확장에도 유연한 Zustand
Jotai가 Recoil의 영감을 받아 만들어졌다면, Zustand는 리덕스에 영감을 받아 만들어졌다. 즉, atom이라는 개념으로 최소 단위의 상태를 관리하는 것이 아니라 하나의 스토어를 중앙 집중형으로 활용해 스토어 내부에서 상태를 관리하고 있다

```js
import { create } from 'zustand'

const useCounterStore = create((set) => ({
  count: 1,
  inc: () => set((state) => ({ count: state.count + 1 })),
  dec: () => set((state) => ({ count: state.count - 1 })),
}))

function Counter() {
  const { count, inc, dec } = useCounterStore()
  
  return	(
    <div class="counter">
      <span>{count}</span>
      <button onClick={inc}>up</button>
      <button onClick={dec}>down</button>
    </div>
  )
}
```
Zustand의 create를 사용해 스토어를 만들고, 반환 값으로 이 스토어를 컴포넌트 내부에서 사용할 수 있는 훅을 받았다. 그리고 이 훅을 사용하면 스토어 내부에 있는 getter와 setter 모두에 접근해 사용할 수 있게 된다
```js
import { createStore, useStore } from 'zustand'

const counterStore = createStore((set) => ({
  count: 1,
  inc: () => set((state) => ({ count: state.count + 1 })),
  dec: () => set((state) => ({ count: state.count - 1 })),
}))

function Counter() {
  const { count, inc, dec } = useStore(counterStore)
  
  return	(
    <div class="counter">
      <span>{count}</span>
      <button onClick={inc}>up</button>
      <button onClick={dec}>down</button>
    </div>
  )
}
```
createStore를 사용하면 리액트와 상관없는 바닐라 스토어를 만들 수 있으며 이 바닐라 스토어는 useStore훅을 통해 접근해 리액트 컴포넌트 내부에서 사용할 수 있게 된다.

##### 특징

Zustand는 특별히 많은 코드를 작성하지 않아도 빠르게 스토어를 만들고 사용할 수 있다는 큰 장점이 있다. 스토어를 만들고 이 스토어에 파생된 값을 만드는 데 간 몇 줄의 코드면 충분하다

또한 Zustand는 2.9kB밖에 안되는 가벼운 크기이며 API가 복잡하지 않아 쉽게 접근할 수 있는 라이브러리다

## 정리
각 라이브러리별 특징을 잘 파악하고 현재 제품 상황과 철학에 맞는 상태 관리 라이브러리를 적절하게 성택해 사용하면 효율적인 제품을 만드는데 도움이 될것이다
> 여러분이 사용하는 상태 관리 라이브러리는 무엇인가요?

