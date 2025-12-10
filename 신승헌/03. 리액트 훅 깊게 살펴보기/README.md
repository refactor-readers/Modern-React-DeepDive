> 함수형 컴포넌트가 상태를 사용하거나 클래스형 컴포넌트 생명주기 메서드를 대체하는 등 다양한 작업을 목적으로 훅(hook)이 추가됨

### 훅의 등장배경

1. 상태(State) 로직 재사용의 어려움
2. 생명주기 메서드의 한계
3. 클래스 문법 자체의 어려움

---

# 3.1 리액트의 모든 훅 파헤치기

## 3.1.1 useState

리액트에서 hook 하면 생각나는 useState

함수형 컴포넌트 내부에 상태를 정의하고 관리 할 수 있게 함

## useState

```tsx
import { useState } from "react";

const [state, setState] = useState(initialState);
// const [상태명, 세터함수명] = useState(초기값);
```

- 리액트는 클로저를 이용해 useState를 구현함
- 함수 실행 종료된 이후에도, 지역변수인 state를 계속 참조할 수 있음

### 구현 예시

```tsx
// useState 내부 모습 흉내내기

const MyReact = function () {
  const global = {};
  let index = 0;

  function useState(initialState) {
    // useState가 호출되었을 때 초기화 로직

    // 최초 호출 시 global에 빈 배열로 초기화
    // state 값들을 관리하는 공간을 만듬
    if (!global.states) {
      global.states = [];
    }

    // 기존 상태값이 있는지 확인 || 초기값으로 할당
    const currentState = global.states[index] || initialState;
    // states 배열에서 현재 실행된 훅에 해당하는 값을 찾아서 업데이트
    global.states[index] = currentState;

    // 즉시실행함수 setter 만들기
    const setState = (function () {
      // 현재 실행중인 훅에 해당하는 index를 클로저로 가둠
      // 이후 setter를 실행했을 때 적절한 index에 접근할 수 있도록
      let currentIndex = index;
      return function (value) {
        global.states[currentIndex] = value;
        // (... 컴포넌트 렌더링하는 코드)
      };
    })();

    // useState를 사용할 때 마다 index 하나씩 증가
    // 하나의 state마다 index가 할당되어 있도록
    index = index + 1;

    return [currentState, setState];
  }

  // ...
  function Component() {
    const [value, setValue] = useState(0);
  }
};
```

- state가 어떻게 관리되고, 기존 state를 어떻게 유지하는지 동작만 파악하는 용도의 Toy React
- 실제 구현체와는 차이가 있음

### 실제로는

- 전역(Global)이 아니라, 각 컴포넌트(인스턴스)마다 개별 저장소(Fiber Node)를 따로 가지고 있음
- 복잡한 컴포넌트 트리 내에서 특정 자식 컴포넌트만 리렌더링 될 때에는 그 Fiber 안에서만 인덱스(커서)가 초기화 됨
- 실제 Fiber에서는 연결리스트 형태로 저장됨 ⇒ 호출 순서 보장

```tsx
// 리액트 내부 어딘가...

// 현재 렌더링 중인 컴포넌트의 Fiber(메모리 박스)를 가리키는 변수
let currentlyRenderingFiber = null;
// 그 Fiber 안에서 현재 처리 중인 Hook을 가리키는 포인터
let workInProgressHook = null;

function render(Fiber) {
  // 렌더링 시작 시, 포인터를 해당 Fiber의 첫 번째 Hook으로 리셋
  currentlyRenderingFiber = Fiber;

  // (Linked List의 Head)
  workInProgressHook = Fiber.memoizedState;

  // 컴포넌트 함수 실행
  const output = Fiber.type(Fiber.props);
  return output;
}

function useState(initialState) {
  // 현재 포인터가 가리키는 Hook 데이터를 가져옴
  const hook = workInProgressHook;

  // (... 상태 관리 로직)

  // 다음 Hook을 위해 포인터를 옆으로 한 칸 이동
  workInProgressHook = workInProgressHook.next;

  return [hook.state, setState];
}
```

## 게으른 초기화

useState의 파라미터로 함수를 넣어줄수도 있음

```tsx
const [value, setValue] = useState(() => {
	Number.parseInt(window.localStorage.getItem(cacheKey))
}
```

> 공식문서: 초깃값이 복잡하거나, 무거운 연산을 포함하고 있을때 사용해라

- **게으른 초기화**는 무조건 state가 **처음** 만들어질 때만 사용됨
- 리렌더링 시에는 함수의 실행이 무시됨

### 무거운 연산

- localStorage, sessionStorage에 대한 접근
- map, filter, find 같은 배열 접근 메서드
- 초깃값 계산으로 위해 함수 호출이 필요한 경우
- …

---

## 3.1.2 useEffect

> 보통 useEffect를 다음과 같이 설명함. 일부는 맞지만 정확하지는 않음

- 콜백, 의존성 배열 두 개의 인수를 받고 의존성 배열의 값이 변경되면 콜백을 실행함
  ⇒ 변경될 때 만이 아니라, 첫 렌더링 직후에는 무조건 한 번 실행됨
- 클래스형 컴포넌트의 생명주기 메서드와 유사한 작동을 구현할 수 있음
  의존성 배열을 빈 배열로 둬서 마운트될 때만 실행되도록 할 수 있음
  ⇒ 브라우저가 화면을 다 그린(paint) 이후에 비동기적으로 실행됨
- 클린업 함수를 반환할 수 있음. 컴포넌트가 언마운트될 때 실행됨
  ⇒ 리렌더링으로 인해 이펙트가 다시 실행되기 **직전**에도 실행됨

## useEffect란?

```tsx
// 일반적인 형태
function Component() {
  useEffect(() => {
    // ...
  }, [props, state]);
}
```

- 특별한 기능으로 값의 변화를 관찰하는게 아님
- 렌더링 할 때 마다 의존성에 있는 값을 보면서, 이전과 다른게 하나라도 있으면 콜백 실행함

### 클린업 함수

일반적으로 이벤트를 등록하고 지울 때 사용해야 한다고 알려짐

```tsx
function CounterEffect() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    function clicked() {
      console.log(`클릭 ${count}`);
    }
    window.addEventListner("click", clicked);

    return () => {
      console.log(`클린업 ${count}`);
      window.removeEventListener("click", clicked);
    };
  }, [count]); // count가 변할 때마다 실행됨

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>증가</button>
    </div>
  );
}

// 클린업 0
// 클릭 1

// 클린업 1
// 클릭 2

// 클린업 2
// 클릭 3
// ...
```

- 클린업 함수는 이전 state를 참조해 실행됨
- 렌더링 뒤에 실행되지만, 변경된 값이 아닌 정의됐을 당시의 값을 가지고 실행됨
- 특정 이벤트 핸들러가 무한히 추가되는 것을 방지할 수 있음
- 언마운트와는 조금 다른 개념, **리렌더링**을 기준으로 실행된다는 개념이 맞음

### 의존성 배열

- 빈 배열로 두면, 최초 렌더링 직후에만 실행됨 ⇒ 컴포넌트 렌더링 여부 확인 방법으로 주로 사용
- 두 번째 인수를 비워두면, 렌더링 될 때 마다 실행됨

### 의존성배열 없이 쓰는 이유

```tsx
function Component() {
  console.log("rendered");
}

function Component() {
  useEffect(() => {
    console.log("effect");
  });
}
```

1. useEffect는 클라이언트 사이드 실행을 보장해줌
   1. window 객체 접근에 의존하는 코드를 사용해도 됨
2. 컴포넌트 렌더링이 완료된 이후에 실행됨
   1. 반면 직접 실행은 렌더링 도중 실행됨 → 서버 사이드 실행 가능 → 컴포넌트 반환 지연 → 성능 악화

### useEffect의 구현

```tsx
function useEffect(callback, dependencies) {
  const hooks = global.hooks;

  let previousDependencies = hooks[index];

  let isDependenciesChanged = previousDependencies
    ? dependencies.some(
        (value, index) => !Object.is(value, previousDependencies[index])
      )
    : true;

  if (isDependenciesChanged) {
    callback();
  }

  hooks[index] = dependencies;
  index++;
}
```

### useEffect 사용 시 주의점

**#### eslint-disable-line react-hooks/exhaustive-deps 사용 자제**

- useEffect 내부 값이 의존성 배열에 포함되어 있지 않은 경우
- 의도치 않은 버그를 발생시킬 가능성이 높음
- 빈배열 의존성이 필요한 경우라면 확인해야 할 사항
  1. 정말 마운트 시점에 실행이 필요한지
  2. 적절한 위치에서 실행 중인지

**#### 콜백에 이름 붙여주기**

- 첫번째 인수로 전달하는 함수에 이름을 붙여 가독성, 유지보수성 향상이 가능함
- effect의 목적을 명확히 하고, 책임을 최소한으로 좁힐 수 있음

```tsx
useEffect(function foo() {}, []);
```

**#### 거대한 useEffect 만들지 마라**

- 의존성 변경 시 마다 부수효과 실행됨
- 성능에 악영향

**#### 불필요한 외부 함수 만들지 마라**

- useEffect 내에서 사용할 부수효과라면, 내부에서 정의해서 사용하는게 좋음
- 코드 간소화, 가독성 향상, 의존성 배열 최소화

### useEffect 콜백에 비동기 함수 사용이 불가한 이유

- 의존성 배열에 의해서 반복적으로 실행되는 콜백의 응답속도에 따라 결과가 예측 불가능해짐
- 경쟁 상태(race condition):
  - 이전 state 기반 응답이 10초 소요
  - 변경된 state 기반 응답이 1초 소요
  - 이전 state 기반 응답으로 최신화 됨
- 필요한 경우 useEffect 내부에서 선언 후 사용이 가능함

  - 클린업 함수를 통해 abortController 등의 이전 비동기 함수에 대한 처리를 추가하면 좋음

  ```tsx
  useEffect(() => {
  	let shouldIgnore = false

  	async function fetchData() {
  		const response = await fetch('https://api.com')
  		const result = await response.json()
  		if (!shouldIgnore) {
  			setData(result)
  		}
  	}
  	fetchData()

  	return () => {
  		// ...abortController
  		shouldIgnore = true
  	}
  ```

---

## 3.1.3 useMemo

**비용이 큰 연산에 대한 결과를 저장(메모이제이션)**

```tsx
const [value, setValue] = useState(0);

const memoizedValue = useMemo(() => superExpensive(value), [value]);
const MemoizedComponent = useMemo(
  () => <SuperExpensiveCompoent value={value} />,
  [value]
);
```

- 최적화에서 대표적인 훅
- 값 뿐만 아니라 컴포넌트도 반환 가능
- 의존성으로 선언된 값이 변경되지 않는 한 다시 계산되지 않음

---

## 3.1.4 useCallback

**인수로 넘겨받은 콜백 자체를 기억함, 특정 함수를 재사용**

```tsx
const ChildComponent = memo(({ status, onClick }) => {
	useEffect(() => {
		console.log('render')
	}

	return (
		<button onClick={onClick}>
			{status? "on" : "off"}
		</button>
	)
}

function App() {
	const [statusA, setStatusA] = useState(false)
	const [statusB, setStatusB] = useState(false)

	const toggleA = () => {
		setStatusA(!status)
	}

	const toggleB = () => {
		setStatusB(!status)
	}

	return (
		<>
			<ChildComponent status={statusA} onClick={toggleA} />
			<ChildComponent status={statusB} onClick={toggleB} />
		</>
}
```

- 특정 status 상태 변경 시 다른 ChildComponent는 영향 받지 않아야 함
- 함수가 리렌더링마다 재생성 되어서 ChildComponent의 onClick prop이 변경되었다고 판단됨
- 위 같은 상황에 함수의 재생성을 방지할 수 있음
- useEffect의 콜백과 동일하게 기명함수를 사용하면 디버깅이 편리함

### useMemo로 구현이 가능함

- 메모이제이션 대상(변수 / 함수)의 차이만 있을 뿐
- Javascript에서 함수는 값으로 표현할 수 있음

---

## 3.1.5 useRef

렌더링에 필요하지 않은 값을 참조할 수 있는 hook

```tsx
function Stopwatch() {
  const intervalRef = useRef(0);

  function handleStartClick() {
    const intervalId = setInterval(() => {
      console.log("interval");
    }, 1000);
    intervalRef.current = intervalId;
  }

  function handleStopClick() {
    const intervalId = intervalRef.current;
    clearInterval(intervalId);
  }
}
```

- useState와 유사하지만, 큰 차이점이 있음
  1. useRef의 반환값 객체 내부에 있는 current로 접근, 업데이트가 가능함
  2. useRef의 값 변화는 렌더링을 발생시키지 않음
- 컴포넌트 렌더링 시에만 생성되어, 메모리를 차지하지 않음
- 인스턴스마다 개별적으로 새롭게 생성됨

### 사용 예시

useRef의 특성과 useEffect의 실행 시점을 활용해서, usePrevious같은 훅을 구현할 수 있음

```tsx
export function usePrevious(value) {
  // 1. 값을 저장할 ref 생성 (리렌더링을 유발하지 않는 저장소)
  const ref = useRef();

  // 2. 렌더링이 다 끝난 후(useEffect)에 최신 값을 ref에 저장
  useEffect(() => {
    ref.current = value;
  }, [value]);

  // 3. 렌더링 될 때에는 아직 업데이트되지 않은(이전) 값을 반환
  return ref.current;
}

export default function App() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <h1>현재 값: {count}</h1>
      <h2>이전 값: {prevCount}</h2>
      <button onClick={() => setCount(count + 1)}>증가</button>
    </div>
  );
}
```

- 컴포넌트가 렌더링될 때 `ref.current`를 **먼저 리턴**해버림.(이전 값)
- 렌더링이 끝나고 나서야 `useEffect`가 `ref.current`를 **새 값으로 덮어씀**
- 시간차 공격으로 한박자 늦은 이전 값을 얻을 수 있음

---

## 3.1.6 useContext

### Context

- props drilling의 범위가 넒어짐에 따라 발생하는 불편함을 해소하기 위해 등장한 개념
- 명시적인 props 전달 없이도, 하위 컴포넌트 모두에서 자유롭게 원하는 값을 가져다 쓸 수 있음

### useContext

- context를 함수형 컴포넌트에서 사용할 수 있게 해주는 훅
- 상위 컴포넌트 어딘가에 선언된 <Context.Provider /> 에서 제공되는 값을 사용할 수 있음
- Bottom-up으로 탐색하며, 가장 가까운 Provider를 찾으면 탐색을 멈춤
- useContext 내부에서 해당 콘텍스트가 존재하는 환경인지 체크

```tsx
const MyContext = createContext<{ hello: string} | undefined>(undefined)

function ContextProvider({
	children,
	text,
}: PropsWithChildren<{ text: string }>) {
	return (
		<MyContext.Provider value={{ hello : text }}>{children}</MyContext.Provider>
	)
}

function useMycontext() {
	const context = useContext(MyContext)
	if (context === undefined) {
		throw new Error(
		'useMyContext는 ContextProvider내부에서만 사용할 수 있습니다.',
		)
	}
	return context
)

function ChildComponent() {
	// 타입이 명확히 설정돼 있어서 굳이 undefined 체크를 하지 않아도 됨
	// 이 컴포넌트가 Provider 하위에 없다면 애리 발생
	const { hello } = useMyContext()
	return <>{hello}</>
}

function ParentComponent() {
	return (
	<>
		<ContextProvider text="react">
			<childComponent/>
		</ContextProvider>
	</>
	)
}
```

### 주의점

- 컴포넌트 내부에 useContext가 있으면, Provider 하위로 재사용이 제한됨 → 보이지 않는 의존성 형성
- 콘텍스트는 단순히 **상태를 주입해주는 API, 상태를 관리하거나 최적화하지 못함**

---

## 3.1.7 useReducer

useState의 심화버전

상태를 미리 정의해놓은 시나리오에 따라 관리할 수 있음

### 파라미터

1. **reducer**: 기본 action을 정의하는 함수
2. **initialState**: useReducer의 초깃값
3. **init**: 초깃값을 지연해서 생성시키고 싶을 때 사용하는 함수(optional)

```tsx
type State = { count: number };
type Action = { type: "INCREMENT" | "DECREMENT" | "RESET"; payload?: State };
// 보통 type, payload로 많이 씀

const initialState: State = { count: 0 };

function reducer(state: State, action: Action) {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    case "DECREMENT":
      return { count: state.count - 1 };
    case "RESET":
      return initialState;
    default:
      throw new Error("알 수 없는 액션입니다.");
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <h2>Count: {state.count}</h2>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
      <button onClick={() => dispatch({ type: "RESET" })}>초기화</button>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
    </div>
  );
}

export default Counter;
```

### useReducer의 목적

- 상태 업데이트 시나리오를 제한하고, 변경을 빠르게 확인할 수 있게끔 하는것
  - 값에 대한 접근은 컴포넌트에서만 가능하게 함
  - 업데이트 방법은 컴포넌트 외부에 격리
- 복잡한 state를 효율적으로 관리하기 위해
  - 복잡한 state를 사전에 정의된 dispatcher로만 수정할 수 있게 함
  - 성격이 비슷한 state를 묶어 한번에 관리 함
  - 비즈니스 로직 분리

### useState ↔ useReducer

- 클로저를 활용해 값을 가둬서 state를 관리한다는 개념은 똑같다

```tsx
// useReducer로 useState 흉내내기
function reducer(prevState, newState) {
  return typeof newState === "function" ? newState(prevState) : newState;
}

function init(initialArg: Initializer) {
  return typeof initialArg === "function" ? initialArg() : initialArg;
}

function useState(initialArg) {
  return useReducer(reducer, initialArg, init);
}
```

```tsx
// useState로 useReducer 흉내내기
const useReducer = (reducer, initialArg, init) => {
	const [state, setState] = useState(
		init ? () => init(initialArg) : initialArg
	)

	const dispatch = useCallback((action ) => {
		setState((prev) => reducer(prev, action)),
		[reducer],
	}

	return useMemo(() => [state, dispatch], [state, dispatch])
}
```

---

## 3.1.8 useImperativeHandle

### forwardRef

- ref를 상위 컴포넌트에서 하위 컴포넌트로 전달하고 싶을 때 사용
- 자식 컴포넌트에 그냥 props로 전달하면, ref는 props로 쓸 수 없다는 경고가 나타남
- 예약어로 지정된 ref로 그대로 전달할 수 있게 해주는 리액트 API
- ref 전달에 일관성을 제공하기 위해 사용됨

\*_ 19버전 출시와 함께 곧 지원 중단 예정이라고 발표됨_

```tsx
function ChildComponent({ parentRef }) {
	useEffect(() => {
		console.log(parentRef)
	}, [parentRef])

	return <div>HI</div>
}

function ParentComponent() {
	const inputRef = useRef()

	return (
		<>
			<ChildComponent ref={inputRef} /> // WARNING⚠️
			<ChildComponent parentRef={inputRef} />
		</>
```

### useImperativeHandle

- 부모로부터 전달받은 ref를 수정할 수 있게 해줌
- 컴포넌트 외부에서 컴포넌트 내부의 영역을 조작할 수 있게 할 수 있음
- 컴파운드 패턴으로 컴포넌트 구현할 때 몇번 사용했던 것 같음

```tsx
// 자식 컴포넌트
const CustomInput = forwardRef((props, ref) => {
  const inputRef = useRef();

  // 부모에게 노출할 함수들을 정의 (리모컨 제작)
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
    clear: () => {
      inputRef.current.value = "";
    },
  }));

  return <input type="text" ref={inputRef} placeholder="여기에 입력하세요" />;
});

// 부모 컴포넌트
export default function App() {
  // 자식에게 전달할 ref
  const childRef = useRef();

  return (
    <div>
      <h1>부모가 자식을 조종하기</h1>
      <CustomInput ref={childRef} />

      <div style={{ marginTop: "10px" }}>
        {/* 버튼을 누르면 자식의 함수를 직접 실행할 수 있음 */}
        <button onClick={() => childRef.current.focus()}>포커스</button>
        <button onClick={() => childRef.current.clear()}>지우기</button>
      </div>
    </div>
  );
}
```

---

## 3.1.9 useLayoutEffect

> 이 함수의 시그니처는 useEffect와 동일하나, 모든 DOM의 변경 후에 동기적으로 발생한다.

- DOM의 변경 후 useLayoutEffect의 콜백함수 실행이 동기적으로 발생
  **⇒ 렌더링 이후, 브라우저에 변경사항이 반영되기 전**

### effect 실행 순서

1. 리액트가 DOM 업데이트
2. **useLayoutEffect 실행**
3. 브라우저에 변경 사항 반영
4. **useEffect 실행**

> `useLayoutEffect`는 브라우저가 화면을 다시 그리기 전에 실행되는 [`useEffect`](https://ko.react.dev/reference/react/useEffect)입니다. [_공식문서_](https://ko.react.dev/reference/react/useLayoutEffect)

### 주의할 점

- useLayoutEffect는 동기적으로 발생 ⇒ 종료될 때 까지 기다려야 함
- 정말 필요한 경우(스크롤 위치 제어, DOM요소 기반 애니메이션 등)에만 사용

> `useLayoutEffect`를 사용하면 성능이 저하될 수 있습니다. 가능하다면 [`useEffect`](https://ko.react.dev/reference/react/useEffect)를 사용하세요. [_공식문서_](https://ko.react.dev/reference/react/useLayoutEffect)

---

## 3.1.10 useDebugValue

- 프로덕션에서 쓰는 훅은 아님
- 커스텀훅 내부에 원하는 정보를 기록하고, 리액트 개발자 도구에서 볼 수 있음
- 다른 훅 내부에서만 실행 가능함

---

## 3.1.11 훅의 규칙

### rules-of-hooks

- 리액트에서 제공하는 훅은 규칙이 존재함(공식문서 [Rules of Hooks](https://react.dev/reference/rules/rules-of-hooks))
- ESLint의 [react-hooks/rules-of-hooks](https://react.dev/reference/eslint-plugin-react-hooks/lints/rules-of-hooks)도 있음

### 1.

> 최상위에서만 훅을 호출해야 한다. 반복문이나 조건문, 중첩된 함수 내에서 훅을 실행할 수 없다. 이 규칙을 따라야 컴포넌트가 렌더링 될 때마다 항상 동일한 순서로 훅이 호출되는것을 보장할 수 있다.

> It’s not supported to call Hooks (functions starting with use) in any other cases, for example:
>
> 🔴 Do not call Hooks inside conditions or loops.
> 🔴 Do not call Hooks after a conditional return statement.
> 🔴 Do not call Hooks in event handlers.
> 🔴 Do not call Hooks in class components.
> 🔴 Do not call Hooks inside functions passed to useMemo, useReducer, or useEffect.
> 🔴 Do not call Hooks inside try/catch/finally blocks.
> If you break these rules, you might see this error.

### 2.

> 훅을 호출할 수 있는 것은 리액트 함수형 컴포넌트, 혹은 사용자 정의 훅 두 가지 경우뿐이다. 일반 자바스크립트 함수에서는 훅을 사용할 수 없다.

> Don’t call Hooks from regular JavaScript functions. Instead, you can:
>
> ✅ Call Hooks from React function components.
> ✅ Call Hooks from custom Hooks.
>
> By following this rule, you ensure that all stateful logic in a component is clearly visible from its source code.

### 훅의 순서

- 리액트 훅은 파이버 객체 내에 연결리스트 형태로 저장됨
- 각 훅이 파이버 객체 내에서 순서(index 같은..)에 의존해 state나 effect의 결과에 대한 값을 저장하고 있기 때문
- 고정된 순서로 이전값에 대한 비교가 가능해짐
- 훅이 조건부로 실행되면 이 순서가 깨지고, 비교에 실패해 에러를 유발할 수 있음

---

## 3.1.12 정리

---

# 3.2 사용자 정의 훅과 고차 컴포넌트

일반적인 자바스크립트 코드를 재사용 가능하게 구성하는 것 외에

리액트에서 재사용 로직을 관리하는 두가지 방법이 있음

1. 사용자 정의 훅(custom hook)
2. 고차 컴포넌트(HOC, higher order component)

## 3.2.1 사용자 정의 훅

서로 다른 컴포넌트 내부에서 같은 로직을 공유할 때 주로 사용됨

사용자 정의 훅은 리액트에서만 사용할 수 있는 방식

반드시 `use`로 시작하는 이름으로 설정해야 함

```tsx
export function useInput(initialValue) {
  const [value, setValue] = useState(initialValue);

  const onChange = (e) => {
    setValue(e.target.value);
  };

  return [value, onChange];
}

// ...

export default function LoginForm() {
  const [id, onChangeId] = useInput("");
  const [password, onChangePassword] = useInput("");

  return (
    <div>
      <h1>Signin</h1>
      <input placeholder="Email" value={id} onChange={onChangeId} />
      <input
        type="password"
        placeholder="PW"
        value={password}
        onChange={onChangePassword}
      />
    </div>
  );
}
```

---

## 3.2.2 고차 컴포넌트

### HOC, Higher Order Component

- 컴포넌트 자체의 로직을 재사용하기 위한 방법
- 고차 함수(Higher Order Function)의 일종
- 자바스크립트 일급 객체, 일급 함수의 특징을 이용함
  ⇒ 리액트가 아닌 자바스크립트 환경에서 널리 쓰임
  e.g. `React.memo` 도 유명한 고차 컴포넌트임

### React.memo

- 렌더링에 앞서 props를 비교해, 이전과 같다면 렌더링 자체를 생략
- 이전에 memoization 해둔 컴포넌트를 반환함

_컴포넌트도 값이라는 관점에서 보면, useMemo로도 구현이 가능하긴 함_

### 자바스크립트의 고차 함수

대표적인 예로 `Array.prototype.map`이 있음

```tsx
const list = [1, 2, 3];
const doubledList = list.map((item) => item * 2);
```

- 함수를 인수로 받음
- 함수를 결과로 반환(return) 함

### 리액트의 고차 함수

```tsx
const setState = (function () {
  let currentIndex = index;
  return function (value) {
    global.states[currentIndex] = value;
  };
})();
```

- 위 같은 setState 구현체도 함수를 반환하기 때문에 고차 함수

### 고차 컴포넌트!

**_e.g. 로딩처리 컴포넌트_**

```tsx
// 고차 컴포넌트는 보통 with로 시작하는 이름을 씀
function withLoading(WrappedComponent) {
  // 새로운 컴포넌트를 반환 (함수형 컴포넌트)
  return function WithLoadingComponent({ isLoading, ...props }) {
    // 1. 로직 개입: 로딩 중이면 스피너를 보여주고 끝냄 (원래 컴포넌트 렌더링 안 함)
    if (isLoading) {
      return <div style={{ fontSize: "20px" }}>로딩 중입니다...</div>;
    }

    // 2. 로딩이 끝났다면: 원래 컴포넌트를 보여줌
    // ★ 중요: 나머지 props(...props)를 그대로 전달해야 데이터가 끊기지 않음!
    return <WrappedComponent {...props} />;
  };
}

export default withLoading;
```

```tsx
// 순수 컴포넌트 (UI만 담당)
const UserList = ({ users }) => {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}님 환영합니다!</li>
      ))}
    </ul>
  );
};

// HOC로 감싸서 업그레이드
const UserListWithLoading = withLoading(UserList);

export default function App() {
  const [loading, setLoading] = useState(true);
  const [users, setUsers] = useState([]);

  // 3초 동안 로딩상태 표시
  useEffect(() => {
    setTimeout(() => {
      setUsers([
        { id: 1, name: "민수" },
        // ...
      ]);
      setLoading(false);
    }, 3000);
  }, []);

  return (
    <div>
      <h1>사용자 목록</h1>
      <UserListWithLoading isLoading={loading} users={users} />
      {/* isLoading={true} -> "로딩 중입니다..." */}
      {/* isLoading={false} -> UserList 컴포넌트 표시 */}
    </div>
  );
}
```

- 컴포넌트의 결과물에 영향을 미칠 수 있는 다른 공통된 작업을 처리할 수 있음

### 주의할 점

- `with`로 시작하는 이름을 사용해야 함(강제는 아님)
- 고차컴포넌트의 부수 효과를 최소화 해야 함
  - 고차 컴포넌트 내에서 인수로 받는 컴포넌트의 props를 임의로 수정/추가/삭제 하면 안됨
  - 고차 컴포넌트의 결과를 예측하기 어려워짐
- 여러개 고차 컴포넌트로 감싸는 경우 복잡성이 크게 올라갈 수 있음
  - 복잡해짐
  - 결과 예측이 어려워짐

---

## 3.2.3 무엇을 써야 할까?

### 공통점

- 리액트 코드에서 어떤 로직을 공통화 할 수 있음
- 중복된 로직을 분리해 별도로 관리할 수 있음

### 사용자 정의 훅

- 단순히 리액트에서 제공되는 훅으로만 공통 로직 격리가 가능한 경우
  - `useEffect`, `useState`, `…`

> "사용자 정의 훅은 그 자체로는 렌더링에 영향을 미치지 못하기 때문에 사용이 제한적이므로 반환하는 값을 바탕으로 무엇을 할지는 개발자에게 달려있다.” p.249

- 렌더링에 영향을 미치지 못함 ⇒ 화면에 그려지는 UI(View)를 결정하지 못함
- 컴포넌트 내부에 미치는 영향 최소화 ⇒ 개발자가 훅을 원하는 방향으로 사용이 가능함

### 고차 컴포넌트

- “로그인 여부 확인 후 조건부 UI 렌더링”과 같은 상황처럼
- 특정 케이스에 모두 같은 컴포넌트를 반환해야 할 때
- 커스텀훅은 컴포넌트가 반환하는 결과물까지 영향을 미치기 어려워짐
- 따라서 고차 컴포넌트를 사용해 처리하는것이 좋음

### 정리

- 컴포넌트의 반환값에 영향을 미치지 않는 경우 ⇒ 커스텀 훅
- 컴포넌트의 반환값, 렌더링 결과물에 영향을 미치는 경우 ⇒ 고차 컴포넌트

> 사실 대부분의 경우 커스텀훅을 사용하는 상황이 훨씬 많을 것 같습니다.
> 다만, (특히) 에러바운더리나 로그인/권한 처리 등에서는 고차 컴포넌트를 사용하는건 거의 필수적일 것 같아요.
