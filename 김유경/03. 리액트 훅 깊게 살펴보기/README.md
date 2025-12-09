# hook의 도입 배경

- 함수형 컴포넌트가 상태를 사용하거나 클래스형 컴포넌트의 생명주기 메서드를 대체하는 등의 다양한 작업을 하기 위해 훅이 추가 됨
- 훅을 활용하면 클래스형 컴포넌트가 아니더라도 리액트의 다양한 기능 활용이 가능

# 1. 리액트의 모든 훅 파헤치기 (낋겨보자)

리액트 함수형 컴포넌트에서 가장 중요한 개념이 바로 훅

- 클래스형 컴포넌트에서만 가능했던 state, ref 등 리액트의 핵심적인 기능을 함수에서도 가능하게 만듬
- 가장 중요한 것은 클래스형 컴포넌트보다 "간결"하게 작성할 수 있다는 것

## 1.1 useState

함수형 컴포넌트 내부에서 상태를 정의하고 이 상태를 관리할 수 있게 해주는 훅

**useState 구현 살펴보기**

```tsx
import { useState } from "react";

const [state, setState] = useState(initialState);
```

- useState의 인수로 초깃값을 전달, 없으면 undefined
- useState 훅의 반환 값은 배열
  - 첫 번째 원소는 state 값 자체
  - 두 번째 원소는 setState 함수를 사용해 해당 state의 값을 변경할 수 있음

**useState를 사용하지 않고 함수 내부에서 자체적으로 변수를 사용해 상태값을 관리할 때**

```tsx
function Component() {
  let stater = "hello";

  function handleButtonClick() {
    state = "hi";
  }

  return (
    <>
      <h1>{state}</h1>
      <button onClick={handleButtonClick}>hi</button>
    </>
  );
}
```

위 코드는 동작하지 않는다.

why?

- 위 코드는 리렌더링을 발생시키기 위한 조건을 전혀 충족하지 못하고 있다.

**리액트의 렌더링**

=> 리액트의 렌더링은 함수형 컴포넌트의 return과 클래스형 컴포넌트의 render 함수를 실행한 다음, 이 실행 결과를 이전의 리액트 트리와 비교해 리렌더링이 필요한 부분만 업데이트 한다.

```tsx
function Component() {
  const [_, triggerRender] = useState();

  let stater = "hello";

  function handleButtonClick() {
    state = "hi";
    triggerRender();
  }

  return (
    <>
      <h1>{state}</h1>
      <button onClick={handleButtonClick}>hi</button>
    </>
  );
}
```

- useState의 반환값의 두 번째 원소를 실행해 리액트에서 렌더링이 일어나게끔 변경했다.
- 그럼에도 여전히 버튼 클릭시 state의 변경된 값이 렌더링 되고 있지 않다.

why?

- 리액트의 렌더링은 함수형 컴포넌트에서 반환한 결과물인 return의 값을 비교해 실행되기 때문이다.
- 즉, 매번 렌더링이 발생할 때마다 함수는 새롭게 실행되고, 새롭게 실행되는 함수에서 state는 매번 hello로 초기화되기 때문에 아무리 state를 변경해도 다시 hello로 초기화되는 것이다.

**❓ 그렇다면 useState 훅의 결괏값은 어떻게 함수가 실행돼도 그 값을 유지하고 있을까?**

```tsx
function useState(initialValue) {
  let internalState = initialValue;

  function setState(newValue) {
    internalState = newValue;
  }

  return [internalState, setState];
}
```

쉽게 상상할 수 있는 형태의 useState는 위와 같지만 이는 원하는 대로 동작하지 않는다.

why?

- setValue로 값을 변경했음에도 이미 구조 분해할당으로 state의 값, 즉 value를 이미 할당해 놓은 상태이기 때문에 훅 내부의 setState를 호출하더라도 변경된 새로운 값을 반환하지는 못한 것
- 이를 해결하려면 먼저 state를 함수로 바꿔서 state의 값을 호출할 때마다 현재 state를 반환하게 하면 된다.

```tsx
function useState(initialValue) {
  let internalState = initialValue;

  function state() {
    return internalState;
  }

  function setState(newValue) {
    internalState = newValue;
  }

  return [internalState, setState];
}
```

리액트는 클로저를 활용하여 useState가 호출된 이후에도 계속해서 지역 변수인 state를 사용할 수 있도록 했다.

```tsx
const MyReact = function () {
  const global = {};
  let index = 0;

  function useState(initialState) {
    if (!global.states) {
      // 애플리케이션 전체의 states 배열을 초기화한다.
      // 최초 접근이면 빈 배열로 초기화한다.
      global.states = [];
    }

    // states 정보를 조회해서 현재 상태값이 있는지 확인하고, 없다면 초깃값으로 설정한다.
    const currentState = global.states[index] || initialState;
    // states의 값을 위해서, 조회한 현재 값으로 업데이트한다.
    global.states[index] = currentState;

    // 즉시 실행 함수로 setter를 만든다.
    const setState = (function () {
      // 현재 index를 클로저로 가둬놔서 이후에도 계속해서 동일한 index에 접근할 수 있도록 한다.
      let currentIndex = index;
      return function (value) {
        global.states[currentIndex] = value;
        // 컴포넌트를 렌더링한다. 실제로 컴포넌트를 렌더링 하는 코드는 생략했다.
      };
    })();
    // useState를 쓸 때마다 index를 하나씩 추가한다. 이 index는 setState에서 사용된다.
    // 즉, 하나의 state마다 index가 할당돼 있어 그  index가 배열의 값 (global.states)을
    // 가리키고 필요할 때마다 그 값을 가져오게 한다.
    index = index + 1;

    return [currentState, setState];
  }
};
```

#### 클로저

함수의 실행이 끝났음에도 함수가 선언된 환경을 기억할 수 있는 방법

- 매번 실행되는 함수형 컴포넌트에서 state의 값을 유지하고 사용하기 위해서 리액트는 클로저를 사용하고 있다.

#### `__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED` 잘리고 싶냐?

리액트에서 변수명을 이렇게 무섭게 지은 이유는 일반 사용자의 접근을 차단하고, 실제 프로덕션 코드에서 사용하지 못하게 하기 위함이다.

#### 게으른 초기화

useState의 기본값으로 `특정한 값을 반환하는 함수`를 넘기는 것을 보고 게으른 초기화라고 한다.

- 공식 문서에서는 useState의 초깃값이 복잡하거나 무거운 연산을 포함하고 있을 때 사용하라고 권장한다.
- 게으른 초기화 함수는 state가 처음 만들어질 때만 사용되고, 이후에 리렌더링이 발생하면 무시된다.

게으른 초기화를 활용할 수 있는 방법

- `Number.parseInt(window.localStorage.getItem(cacheKey))`와 같이 한 번 실행되는 데 어느 정도 비용이 드는 값이 있다고 할 때
- useState의 인수로 이 값 자체를 사용하면 초깃값이 필요한 최초 렌더링, 초기값이 있어 더 이상 렌더링이 필요 없는 리렌더링 시에도 동일하게 계속 해당 값에 접근해서 낭비가 발생하기 때문에 이럴 때는 오히려 함수 형태로 인수로 전달하는 것이 경제적일 수 있다.
- `localStorage`, `sessionStorage`에 대한 접근
- map, filter, find 같은 배열에 대한 접근
- 초깃값 계산을 위해 함수 호출이 필요할 때

## 1.2 useEffect

일반적인 정의

- 두 개의 인수를 받는데, 첫 번째는 콜백, 두 번째는 의존성 배열이다. 이 두 번째 의존성 배열의 값이 변경되면 첫 번째 인수인 콜백을 실행한다.
- 클래스형 컴포넌트의 생명주기 메서드와 비슷한 작동을 구현할 수 있다. 두 번째 의존성 배열에 빈 배열을 넣으면 컴포넌트가 마운트될 때만 실행된다.
  => ❌ 생명 주기 메서드를 대체하기 위해 만들어진 훅도 아니다.
- useEffect는 클린업 함수를 반환할 수 있는데, 이 클린업 함수는 컴포넌트가 언마운트 될 때 실행된다.

=> 위의 정의들은 어느 정도 옳지만 정확하지는 않다.

#### 💡 정확한 정의

애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 메커니즘이다.

- 이 부수 효과가 '언제' 일어나는지보다 '어떤 상태값과 함께 실행되는지'를 보는 것이 중요하다.

#### useEffect란?

```tsx
// 일반적인 형태
function Component() {
  //...

  useEffect(() => {
    // do something
  }, [props, state]);
  // ...
}
```

- 첫 번째 인수로는 부수 효과가 포함된 함수
- 두 번째 인수로는 의존성 배열

- 두 번째 인수를 생략할 수도, 아주 긴 배열을 넣을 수도, 빈 배열을 넣을 수도 있다.

#### 의존성 배열이 변경될 때마다 useEffect의 콜백을 실행

그렇다면 useEffect는 어떻게 의존성 배열이 변경된 것을 알고 실행될까?

- ✏️ 함수형 컴포넌트는 매번 함수를 실행해 렌더링을 수행한다는 것을 기억해야 한다.

```tsx
// 일반적인 형태
function Component() {
  const [counter, setCounter] = useState(0);

  function handleClick() {
    setCounter((prev) => prev + 1);
  }

  return (
    <>
      <h1>{counter}</h1>
      <button onClick={handleClick}>+</button>
    </>
  );
}
```

- 버튼을 클릭하면 counter에 값을 1씩 올리는 평범한 컴포넌트다.
- 함수형 컴포넌트는 렌더링 시마다 고유의 state와 props를 갖고 있다. 여기에 useEffect가 추가된다면 아래와 같은 형태가 된다.

```tsx
// 일반적인 형태
function Component() {
  const [counter, setCounter] = useState(0);

  useEffect(() => {
    console.log(counter);
  });

  function handleClick() {
    setCounter((prev) => prev + 1);
  }

  return (
    <>
      <h1>{counter}</h1>
      <button onClick={handleClick}>+</button>
    </>
  );
}
```

useEffect는 자바스크립트의 proxy나 데이터 바인딩, 옵저버 같은 특별한 기능을 통해 값의 변화를 관찰하는 것이 아니고 렌더링 할 때마다 의존성에 있는 값을 보면서 이 의존성의 값이 이전과 다른 게 하나라도 있으면 부수 효과를 실행하는 평범한 함수라 볼 수 있다.

- 따라서 useEffect는 state와 props의 변화 속에서 일어나는 렌더링 과정에서 실행되는 부수 효과 함수라고 볼 수 있다.

#### 클린업 함수의 목적

```tsx
import { useState, useEffect } from "react";

export default function App() {
  const [counter, setCounter] = useState(0);

  function handleClick() {
    setCounter((prev) => prev + 1);
  }

  useEffect(() => {
    function addMouseEvent() {
      console.log(counter);
    }

    window.addEventListener("click", addMouseEvent);

    // 클린업 함수 (Cleanup function)
    return () => {
      console.log("클린업 함수 실행!", counter);
      window.removeEventListener("click", addMouseEvent);
    };
  }, [counter]);

  return (
    <>
      <h1>{counter}</h1>
      <button onClick={handleClick}>+</button>
    </>
  );
}
```

```
클린업 함수 실행! 0
1

클린업 함수 실행! 1
2

클린업 함수 실행! 2
3

클린업 함수 실행! 2
4

//
```

클린업 함수는 이전 counter 값, 즉 이전 state를 참조해 실행된다는 것을 알 수 있다.

- 클린업 함수는 새로운 값과 함께 렌더링된 뒤에 실행되기 때문에 위와 같은 메시지가 나타난다.

#### 💡 중요한 점

클린업 함수는 비록 새로운 값을 기반으로 렌더링 뒤에 실행되지만 이 변경된 값을 읽는 것이 아니라 함수가 정의됐을 당시에 선언됐던 이전 값을 보고 실행된다는 것이다.

```tsx
// 최초 실행
useEffect(() => {
  function addMouseEvent() {
    console.log(1);
  }

  window.addEventListener("click", addMouseEvent);

  // 클린업 함수
  // 그리고 이 클린업 함수는 다음 렌더링이 끝난 뒤에 실행된다.
  return () => {
    console.log("클린업 함수 실행!", 1);
    window.removeEventListener("click", addMouseEvent);
  };
}, [counter]);
//
// ...
// 이후 실행

useEffect(() => {
  function addMouseEvent() {
    console.log(2);
  }

  window.addEventListener("click", addMouseEvent);

  // 클린업 함수 (Cleanup function)
  return () => {
    console.log("클린업 함수 실행!", 2);
    window.removeEventListener("click", addMouseEvent);
  };
}, [counter]);
//
// ...
```

- 함수형 컴포넌트의 useEffect는 그 콜백이 실행될 때마다 이전의 클린업 함수가 존재한다면 그 클린업 함수를 실행한 뒤에 콜백을 실행한다. 따라서 이벤트를 추가하기 전에 이전에 등록했던 이벤트 핸들러를 삭제하는 코드를 클린업 함수에 추가하는 것이다.

=> 이를 통해 특정 이벤트의 핸들러가 무한히 추가되는 것을 방지할 수 있다.

- 언마운트 개념과는 차이가 있다.
  - 언마운트는 특정 컴포넌트가 DOM에서 사라진다는 것을 의미하는 클래스형 컴포넌트의 용어이다.
  - 클린업 함수는 언마운트라기보다는 함수형 컴포넌트가 리렌더링됐을 때 의존성 변화가 있었을 당시 이전의 값을 기준으로 실행되는, 말 그대로 이전 상태를 청소해주는 개념으로 보는 것이 옳다.

#### 의존성 배열

- 빈 배열

  - 리액트는 이 useEffect는 비교할 의존성이 없다고 판단해 최초 렌더링 ㅈ익후에 실행된 다음부터는 더 이상 실행되지 않는다.

- 아무런 값도 넘겨주지 않을 때

  - 의존성을 비교할 필요 없이 렌더링할 때마다 실행이 필요하다고 판단해 렌더링이 발생할 때마다 실행된다.
  - 컴포넌트가 렌더링 됐는지 확인하기 위한 방법으로 사용됨

- 사용자가 직접 원하는 값

❓ 의존성 배열이 없는 useEffect가 매 렌더링마다 실행된다면 그냥 useEffect 없이 써도 되는게 아닐까?

```tsx
// 1
function Component() {
  console.log("렌더링 됨");
}

// 2
function Component() {
  useEffect(() => {
    console.log("렌더링 됨");
  });
}
```

1. 서버 사이드 렌더링 관점에서 useEffect는 클라이언트 사이드에서 실행되는 것을 보장해준다. useEffect 내부에서는 window 객체의 접근에 의존하는 코드를 사용해도 된다.
2. useEffect는 컴포넌트 렌더링의 부수 효과, 즉 컴포넌트 렌더링이 완료된 이후에 실행된다.
   - 반면 직접 실행은 컴포넌트가 렌더링되는 도중에 실행된다.
   - 따라서 1번과는 달리 서버 사이드 렌더링의 경우에 서버에서도 실행된다.
   - 이 작업은 함수형 컴포넌트의 반환을 지연시키는 행위다. 즉, 무거운 작업일 경우 렌더링을 방해하므로 성능에 악영향을 미칠 수 있다.

useEffect의 effect는 컴포넌트의 사이드 이펙트, 즉 부수 효과를 의미한다는 것을 명심하자

useEffect는 컴포넌트가 렌더링된 후에 어떠한 부수 효과를 일으키고 싶을 때 사용하는 훅이다.

#### useEffect의 구현

```tsx
const MyReact = (function () {
  const global = {};
  let index = 0;

  function useEffect(callback, dependencies) {
    const hooks = global.hooks;

    // 이전 훅 정보가 있는지 확인한다.
    let previousDependencies = hooks[index];

    // 변경됐는지 확인
    // 이전 값이 있다면 이전 값을 얕은 비교로 비교해 변경이 일어났는지 확인한다.
    // 이전 값이 없다면 최초 실행이므로 변경이 일어난 것으로 간주해 실행을 유도한다.
    let isDependenciesChanged = previousDependencies
      ? dependencies.some(
          (value, idx) => !Object.is(value, previousDependencies[idx])
        )
      : true;

    // 변경이 일어났다면 첫 번째 인수인 콜백 함수를 실행한다.
    if (isDependenciesChanged) {
      callback();
    }

    // 현재 의존성을 훅에 다시 저장한다.
    hooks[index] = dependencies;

    // 다음 훅이 일어날 때를 대비하기 위해 index를 추가한다.
    index++;
  }

  return { useEffect };
})();
```

핵심은 의존성 배열의 이전 값과 현재 값의 얕은 비교다.

- 이전 의존성 배열과 현재 의존성 배열의 값에 하나라도 변경 사항이 있다면 callback으로 선언한 부수효과를 실행한다. 이것이 useEffect의 본질이다.

#### useEffect를 사용할 때 주의할 점

잘못 사용했을 때

- 예기치 못한 버그 발생 가능
- 조심하지 않으면 무한 루프에 빠져버린다구~!

1. eslint-disable-line react-hooks/exhaustive-deps 주석은 최대한 자제하라
   이 주석을 사용하여 ESLint의 경고를 무시하기도 한다.

- 해당 룰은 useEffect 인수 내부에서 사용하는 값 중 의존성 배열에 포함돼 있지 않은 값이 있을 때 경고를 발생시킨다.

보통 해당 주석을 사용하는 경우는

- 빈배열을 의존성으로 할 때, 즉 컴포넌트를 마운트하는 시점에만 무언가를 하고 싶다라는 의도로 작성

=> 클래스형 컴포넌트의 생명주기 메서드인 `ComponentDidMount`에 기반한 접근법으로 가급적이면 사용해선 안된다.

useEffect는 반드시 의존성 배열로 전달한 값의 변경에 의해 실행돼야 하는 훅이다.

- 그런데 의존성 배열을 넘기지 않은 채 콜백 함수 내부에서 특정 값을 사용한다는 것은, 이 부수 효과가 실제로 관찰해서 실행돼야 하는 값과는 별개로 작동하게 된다는 것이다.

useEffect에서 사용한 콜백 함수의 실행과 내부에서 사용한 값의 실제 변경 사이에 연결 고리가 끊어져 있는 것이다.

- 정말 의존성으로 []가 필요하다면 최초에 함수형 컴포넌트가 마운트됐을 시점에만 콜백 함수 실행이 필요한지를 다시 한 번 되물어보자
  - 정말 Yes 라면, 애초에 useEffect 내 부수 효과가 실행될 위치가 잘못됐을 가능성이 크다.

```tsx
function Component({ log }: { log: string }) {
  useEffect(() => {
    logging(log);
  }, []);
}
```

- log가 아무리 변하더라도 useEffect의 부수 효과는 실행되지 않고, useEffect의 흐름과 컴포넌트의 props.log의 흐름이 맞지 않게 된다.
- 해당 작업은 부모 컴포넌트에서 실행되는 것이 옳은 것이 아닐까?

#### useEffect의 첫 번째 인수에 함수명을 부여하라

보통 useEffect의 인수로 전달하는 콜백 함수는 익명 함수로 작성하지만 복잡해지고 많아질 수록 기명 함수로 바꾸는 것이 좋다.

```tsx
useEffect(function logActiveUser() {
  logging(user.id);
}),
  [user.id];
```

#### 거대 useEffect를 만들지 마라

useEffect는 의존성 배열을 바탕으로 렌더링 시 의존성이 변경될 때마다 부수 효과를 실행한다.
이 부수 효과의 크기가 커질 수록 애플리케이션 성능에 악영향을 미친다.

- 부득이하게 큰 useEffect를 만들어야 한다면 적은 의존성 배열을 사용하는 여러 개의 useEffect로 분리하는 것이 좋다.
- 여러 변수를 사용한다면, useCallback과 useMemo 등으로 사전에 정제한 내용들만 useEffect에 담아두는 것이 좋다.

#### 불필요한 외부 함수를 만들지 마라

```tsx
function Component({ id }: { id: string }) {
  const [info, setInfo] = useState<number | null>(null);
  const controllerRef = useRef<AbortController | null>(null);
  const fetchInformation = useCallback(async (fetchId: string) => {
    controllerRef.current?.abort();
    controllerRef.current = new AbortController();

    const result = await fetchInfo(fetchId, { signal: controllerRef.signal });
    setInfo(await result.json());
  }, []);

  useEffect(() => {
    fetchInformation(id);
    return () => controllerRef.current?.abort();
  }, [id, fetchInformation]);

  return <div>{/* 렌더링 */}</div>;
}
```

- props를 받아서 그 정보를 바탕으로 API 호출을 하는 useEffect를 가지고 있다.
  - but, useEffect 밖에서 함수를 선언하다보니, 불필요한 코드가 많아지고 가독성이 떨어졌다.

개선 뒤

```tsx
function Component({ id }: { id: string }) {
  const [info, setInfo] = useState<number | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    (async () => {
      const result = await fetchInfo(id, { signal: controller.signal });
      setInfo(await result.json());
    })();

    return () => controller.abort();
  }, [id]);

  return <div>{/* 렌더링 */}</div>;
}
```

### useEffect의 콜백 인수로 비동기 함수를 바로 넣을 수 없을까?

useEffect 내부에서 state를 결과에 따라 업데이트 하는 로직이 있다고 가정할 때, useEffect의 인수로 비동기 함수가 사용가능하다면 함수의 응답 속도에 따라 결과가 이상하게 나타날 수 있다.

- 이전 state 기반의 응답이 10초가 걸렸고, 이후 바뀐 state 기반의 응답이 1초 뒤에 왔다면 이전 state 기반으로 결과가 나와버리는 불상사가 생길 수 있다.

=> useEffect의 경쟁 상태 (race condition)

```tsx
useEffect(async () => {
  // useEffect에 async 함수를 넘겨주면 다음과 같은 에러가 발생한다.
  // // Effect callbacks are synchronous to prevent race conditions.
  // // Put the async function inside:

  const response = await fetch("http://some.data.com");
  const result = await response.json();

  setData(result);
}, []);
```

그렇다면 비동기 함수는 어떻게 실행하나요?

- useEffect의 인수로 비동기 함수를 지정할 수 없는거지, 비동기 함수 실행 자체가 문제가 되는 것은 아니다.
  - useEffect 내부에서 비동기 함수를 선언해 실행하거나, 즉시 실행 비동기 함수를 만들어서 사용하는 것은 가능하다.

```tsx
useEffect(() => {
  let shouldIgnore = false; // 플래그 변수 초기화

  async function fetchData() {
    const response = await fetch("http://some.data.com");
    const result = await response.json();

    // 요청이 완료된 후, 컴포넌트가 언마운트되지 않았는지 확인
    if (!shouldIgnore) {
      setData(result);
    }
  }

  fetchData();

  return () => {
    // 클린업 함수: 컴포넌트 언마운트 시 플래그를 true로 설정
    // // shouldIgnore를 이용해 useState의 두 번째 인수를 실행하는 것뿐만 아니라
    // // AbortController를 활용해 직전 요청 자체를 취소하는 것도 좋은 방법이 될 수 있다.
    shouldIgnore = true;
  };
}, []);
```

- 이렇게 사용할 거라면, 클린업 함수에서 이전 비동기 함수에 대한 처리를 추가하는 것이 좋다.
- fetch의 경우 abortController 등으로 이전 요청을 취소하는 것이 좋다.

# 1.3 useMemo

비용이 큰 연산에 대한 결과를 저장(메모이제이션)해 두고, 이 저장된 값을 반환하는 훅이다. 흔히 리액트에서 최적화를 떠올릴 때 가장 먼저 언급되는 훅이 바로 useMemo다.

```ts
import { useMemo } from "react";

const memoizedValue = useMemo(() => expensiveComputation(a, b), [a, b]);
```

- 첫 번째 인수 : 어떠한 값을 반환하는 생성 함수
- 두 번재 인수 : 해당 함수가 의존하는 값의 배열

=> 렌더링 발생 시 의존성 배열의 값이 변경되지 않았으면 함수를 재실행하지 않고 이전에 기억해 둔 해당 값을 반환하고, 의존성 배열의 값이 변경됐다면 첫 번째 인수의 함수를 실행한 후에 그 값을 반환하고 그 값을 다시 기억해 둘 것이다.

- 컴포넌트도 가능

```tsx
function ExpensiveComponent({ value }) {
  useEffect(() => {
    console.log("rendering!");
  });
  return <span>{value + 1000}</span>;
}

function App() {
  const [value, setValue] = useState(10);
  const [_, triggerRendering] = useState(false);

  // 컴포넌트의 props를 기준으로 컴포넌트 자체를 메모이제이션했다.
  const MemoizedComponent = useMemo(
    () => <ExpensiveComponent value={value} />,
    [value]
  );

  function handleChange(e) {
    setValue(Number(e.target.value));
  }

  function handleClick() {
    triggerRendering((prev) => !prev);
  }

  return (
    <>
      <input value={value} onChange={handleChange} />
      <button onClick={handleClick}>렌더링 발생!</button>
      <MemoizedComponent />
    </>
  );
}
```

# 1.4 useCallback

useMemo가 값을 기억했다면, useCallback은 인수로 넘겨받은 콜백 자체를 기억한다.

- 특정 함수를 새로 만들지 않고, 재사용한다는 것

// memo를 사용함에도 전체 자식 컴포넌트가 리렌더링되는 예제

```tsx
const ChildComponent = memo(( { name, value, onChange } ) => {
  // 렌더링이 수행되는지 확인하기 위해 넣어봤다.
  useEffect(() => {
    console.log('rendering!', name)
  })

  return (
    <>
      <h1>
        {name} {value ? '켜짐' : '꺼짐'}
      </h1>
      <button onClick={onChange}>toggle</button>
    </>
  )
})

function App() {
  const [status1, setStatus1] = useState(false)
  const [status2, setStatus2] = useState(false)
```

기본적으로 useCallback은 useMemo를 사용해서 구현할 수 있다.
// Preact에서의 useCallback 구현

```tsx
export function useCallback(callback, args) {
  currentHook = 8;
  return useMemo(() => callback, args);
}
```

# 1.5 useRef

useState와 동일하게 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장한다는 공통점이 있다.

but, 큰 차이점 두 가지가 있지요

- useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있다.
- useRef는 그 값이 변하더라도 렌더링을 발생시키지 않는다.

### ❓useRef가 왜 필요할까?

렌더링에 영향을 미치지 않는 고정된 값을 관리하기 위해서 useRef를 사용한다면 useRef를 사용하지 않고, 그냥 함수 외부에서 값을 선언해서 관리하는 것도 동일한 기능을 수행하는 거 아닌가?

```tsx
let value = 0;

function Component() {
  function handleClick() {
    value += 1;
  }

  // ...
}
```

- 컴포넌트 렌더링도 안됐는데 value라는 값이 기본적으로 존재해서 메모리 상에 낭비 발생
- 컴포넌트가 여러 번 생성되면 각 컴포넌트에서 가리키는 값이 모두 value로 동일하다.
  - 컴포넌트가 초기화되는 지점이 다르더라도 하나의 값을 봐야하는 경우가 많지 않다.

=> 이러한 방식의 단점을 보완한 리액트식 접근 !

- 컴포넌트가 렌더링될 때만 생성되고, 컴포넌트 인스턴스가 여러 개여도 각각 별개의 값을 바라본다.

### useRef를 사용한 DOM 접근

```tsx
function RefComponent() {
  const inputRef = useRef();

  // 이때는 미처 렌더링이 실행되기 전(반환되기 전)이므로 undefined를 반환한다.
  console.log(inputRef.current);

  useEffect(() => {
    console.log(inputRef.current); // <input type="text"></input>
  }, [inputRef]);

  return <input ref={inputRef} type="text" />;
}
```

- useRef는 최초에 넘겨받은 기본값을 가지고 있다.

💡 useRef의 최초 기본값은 return 문에 정의해둔 DOM이 아니고, useRef()로 넘겨받은 인수이다.

- useRef가 선언된 당시에는 아직 컴포넌트가 렌더링되기 전이라 return으로 컴포넌트의 DOM이 반환되기 전이므로 undefined이다.

### ❓useRef로 useState를 흉내 내도 렌더링되지 않는다.

// useRef를 사용한 간단한 코드

```tsx
function RefComponent() {
  const count = useRef(0);

  function handleClick() {
    count.current += 1;
  }

  return <button onClick={handleClick}>{count.current}</button>;
}
```

### useRef의 구현

```tsx
export function useRef(initialValue) {
  currentHook = 5;
  return useMemo(() => ({ current: initialValue }), []);
}
```

- 값이 변경돼도 렌더링되면 안된다는 점, 실제 값은 {current: value}와 같은 객체 형태로 되어 있다는 점을 떠올려보자.
- 렌더링에 영향을 미치면 안되기 때문에 useMemo에 의도적으로 빈배열을 선언 -> 각 렌더링마다 동일한 객체를 가리킬 것이다.
  - 객체의 값을 변경해도, 객체를 가리키는 주소가 변경되지 않는다는 것을 활용한 것으로 useMemo로 useRef를 구현할 수 있다.

# 1.6. useContext

#### Context란?

리액트 애플리케이션은 기본적으로 부모와 자식 컴포넌트로 이루어진 트리 구조를 갖기 때문에, 데이터를 넘겨주기 위해 props를 깊이 전달해야 하는 prop 내려주기(props drilling) 문제가 발생한다. 이를 극복하기 위해 등장한 개념이 콘텍스트(Context)다.

#### useContext 훅 사용하기

```tsx
const Context = createContext<{ hello: string } | undefined>(undefined);

function ParentComponent() {
  return (
    <>
      <Context.Provider value={{ hello: "react" }}>
        <Context.Provider value={{ hello: "javascript" }}>
          <ChildComponent />
        </Context.Provider>
      </Context.Provider>
    </>
  );
}

function ChildComponent() {
  const value = useContext(Context);
  // 가장 가까운 Provider의 값을 가져온다. 여기서는 'javascript'가 반환된다.
  return <>{value ? value.hello : ""}</>;
}
```

- useContext는 상위 컴포넌트에서 `<Context.Provider />`로 제공한 값을 함수형 컴포넌트 내부에서 사용할 수 있게 해준다
- 여러 개의 Provider가 있다면 가장 가까운 Provider의 값을 가져온다

#### ⚠️ useContext를 사용할 때 주의할 점

1. 재사용성 문제 : useContext가 선언된 컴포넌트는 Provider에 의존성을 갖게 되므로, 아무데서나 재활용하기 어려워진다.
2. 최적화 오해 : 컨텍스트는 상태를 주입해주는 API이지, 상태를 관리하거나 렌더링을 최적화하는 기능은 없다

#### 렌더링 시나리오와 최적화

```tsx
function ParentComponent() {
  const [text, setText] = useState("");
  // ... (text 변경 로직)
  return (
    <ContextProvider value={{ hello: text }}>
      <GrandChildComponent />
    </ContextProvider>
  );
}
```

- 위 코드에서 ParentComponent가 렌더링되면 GrandChildComponent는 useContext 사용 여부와 무관하게 부모가 렌더링되었으므로 같이 렌더링된다
- 이를 방지하려면 React.memo를 사용하여 props 변화가 없을 때 렌더링을 막아야 한다.

# useReducer

useState의 심화 버전으로, 복잡한 상태 값을 사전에 정의된 시나리오(action)에 따라 관리할 수 있다.

- 반환값 : [state, dispatcher] 형태의 배열
  - dispatcher: state를 업데이트하는 함수, action 객체를 인수로 받는다.
- 인수: (reducer, initialState, init)
  - reducer: action을 정의하는 함수
  - init 초깃값을 지연 생성하는 함수 (게으른 초기화)

#### useReducer 사용 예제

```tsx
// 1. state와 action 타입 정의
type State = { count: number };
type Action = { type: "up" | "down" | "reset"; payload?: State };

// 2. reducer 함수 정의
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case "up":
      return { count: state.count + 1 };
    case "down":
      return { count: state.count - 1 };
    case "reset":
      return { count: 0 }; // 게으른 초기화 함수(init) 결과 사용 가능
    default:
      throw new Error(`Unexpected action type ${action.type}`);
  }
}

// 3. 컴포넌트에서 사용
export default function App() {
  const [state, dispatcher] = useReducer(reducer, { count: 0 });

  function handleUpButtonClick() {
    dispatcher({ type: "up" });
  }

  return (
    <div>
      <h1>{state.count}</h1>
      <button onClick={handleUpButtonClick}>+</button>
    </div>
  );
}
```

- 장점 : 상태를 사용하는 로직(컴포넌트)과 상태를 관리하는 비즈니스(reducer)을 분리할 수 있다.

#### 💡 Preact 내부 구현을 통해 본 원리

useState는 useReducer로 구현되어 있고, 반대로 useReducer도 useState로 구현할 수 있다.

=> 결국 둘 다 클로저를 활용해 값을 가둬두고 관리하는 방식에는 변함이 없다.

# 1.8 useImperativeHandle

이 훅을 이해하려면 forwardRef에 대해 먼저 알아야 한다.

#### forwardRef

ref는 props와 다르게 리액트 컴포넌트의 예약어이기 때문에, 하위 컴포넌트로 ref를 전달하고 싶다면 forwardRef를 사용해야 한다.

```tsx
const ChildComponent = forwardRef((props, ref) => {
  return <div>안녕!</div>;
});

function ParentComponent() {
  const inputRef = useRef();
  return <ChildComponent ref={inputRef} />;
}
```

#### useImperativeHandle이란?

부모에게서 넘겨받은 ref를 원하는 대로 수정할 수 있는 훅이다.

```tsx
const Input = forwardRef((props, ref) => {
  useImperativeHandle(
    ref,
    () => ({
      alert: () => alert(props.value),
    }),
    [props.value]
  );

  return <input ref={ref} {...props} />;
});

function App() {
  const inputRef = useRef();
  // ...
  const handleClick = () => {
    // 부모 컴포넌트에서 자식의 커스텀 메서드(alert)를 실행할 수 있다.
    inputRef.current.alert();
  };
}
```

- 원래 ref는 DOM 노드(HTMLElement)를 가리키지만, 이 훅을 사용하면 부모에게 노출되는 값을 객체나 함수 등으로 커스터마이징 할 수 있다.

# 1.9 useLayoutEffect

공식 문서 정의 "이 함수의 시그니처는 useEffect와 동일하나, 모든 DOM의 변경 후에 동기적으로 발생한다"

#### 실행 순서

1. 리액트가 DOM을 업데이트 (렌더링)
2. useLayoutEffect 실행 (동기적)
3. 브라우저에 변경 사항을 반영 (Painting)
4. useEffect 실행 (비동기적)

#### 특징 및 사용처

- 동기적으로 발생한다는 것은 useLayoutEffect가 완료될 때까지 브라우저가 화면을 그리지 않는다는 뜻이다.
- 용도 : DOM은 계산됐지만 화면에 반영되기 전에 하고 싶은 작업이 있을 때 화면 깜빡임을 방지할 수 있다.
  - 스크롤 위치 제어, 애니메이션 등

# 1.10 useDebugValue

리액트 개발자 도구에서 사용자 정의 훅(Custom Hook)의 내부 상태를 디버깅할 때 유용한 정보를 제공한다.

```tsx
function useDate() {
  const date = new Date();
  // 개발자 도구에 "현재 시간: ..." 형태로 표시됨
  useDebugValue(date, (date) => `현재 시간: ${date.toISOString()}`);
  return date;
}
```

- 두 번째 인수로 포매팅 함수를 전달하면, 값이 변경되었을 때만 호출되어 성능 저하를 방지한다.
- 오직 다른 훅 내부에서만 실행할 수 있다.

# 1.11 훅의 규칙 (Rules of Hooks)

1. 최상위에서만 호출해야 한다. 반복문, 조건문, 중첩된 함수내에서 실행할 수 없다.
2. 리액트의 함수형 컴포넌트나 사용자 정의 훅 내부에서만 호출해야 한다.

❓ 왜 순서가 중요할까? 리액트 훅은 파이버 객체의 memoizedState에 링크드 리스트(Linked List)형태로 저장된다.

```tsx
// 훅은 순서에 아주 큰 영향을 받는다.
{
  memoizedState: 0, // useState 1
  next: {
    memoizedState: false, // useState 2
    next: {
       // useEffect ...
    }
  }
}
```

- 리액트는 훅을 호출된 순서대로 저장하고 가져오기 때문에, 조건문 등으로 순서가 바뀌거나 건너뛰어지면 에러가 발생한다.

# 2 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

재사용 로직을 관리하는 두 가지 방법인 사용자 정의 훅(Custom Hook)과 고차 컴포넌트(HOC)를 비교한다.

# 2.1 사용자 정의 훅 (Custom Hook)

- use로 시작하는 함수를 만들어 내부에서 다른 훅을 사용하는 방식
- 컴포넌트 내부에 미치는 영향을 최소화하면서 로직만 격리할 수 있다.

#### 예시 useFetch

```tsx
function useFetch<T>(url: string) {
  const [result, setResult] = useState<T>();

  useEffect(() => {
    // fetch logic...
  }, [url]);

  return { result };
}
```

# 2.2 고차 컴포넌트 (HOC)

- 컴포넌트 자체의 로직을 재사용하기 위한 방법으로 , 컴포넌트를 인수로 받아 새로운 컴포넌트를 반환하는 함수다.
- 가장 유명한 예 : `React.memo`

#### 예시 인증 처리 HOC

```tsx
function withLoginComponent(Component) {
  return function (props) {
    if (!props.loginRequired) {
      return <>로그인이 필요합니다.</>;
    }
    return <Component {...props} />;
  };
}
```

- with로 시작하는 이름을 사용하는 관습이 있다.
- 주의점 : props를 임의로 수정하거나 삭제하면 안되며, 여러 HOC로 감쌀 경우 복잡성이 커진다 (Wrapper Hell)

# 2.3 무엇을 선택해야 할까?

| 구분           | 추천 상황                                                              | 이유                                                                                   |
| -------------- | ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| 사용자 정의 훅 | "단순히 useEffect, useState 등 공통 로직을 격리할 때"                  | "컴포넌트 내부에 미치는 영향을 최소화하고, 개발자가 반환값을 유연하게 사용할 수 있음." |
| 고차 컴포넌트  | "렌더링 결과물에 영향을 미치는 공통 로직 (예: 로그인 체크, 에러 처리)" | 함수형 컴포넌트의 반환값(UI)을 변경하거나 컴포넌트를 감춰야 할 때 유용함               |
