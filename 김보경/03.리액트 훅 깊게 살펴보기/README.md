# 03. 리액트 훅 깊게 톺아보기

> 함수형 컴포넌트가 상태를 사용하거나 클래스형 컴포넌트의 생명주기 메서디를 대체하는 등의 다양한 작업을 하기 위해 Hooks가 도입되었습니다.
> 훅을 활용하면 클래스형 컴포넌트가 아니더라도 리액트의 다양한 기능을 활용할 수 있습니다.
> 리액트에서 사용가능한 훅에 대해서 어떻게 쓰이고, 사용할 때의 주의할 점은 무엇인지 톺아보겠습니다.

## 03.1. 리액트의 모든 훅 파헤치기

### 03.1.1. useState

#### useState 구현 살펴보기

```ts
const [state setState] = useState(initialState)
```

- useState의 인수로는 사용할 state의 초깃값을 넘겨줍니다. 아무런 값을 넘겨주지 않으면 초깃값은 undefined입니다.
- useState 훅의 반환값은 배열이며, 배열의 첫 번째 원소로 state값 차제를 사용할 수 있고, 두 번째 원소인 setState 함수를 사용해 해당 state의 값을 변경할 수 있습니다.

```tsx
function Component() {
  let state = "hello"; // ❌ 일반 변수 (상태 변경 감지 불가)

  function handleButtonClick() {
    state = "hi"; // ❌ 직접 수정 (리렌더링 안됨)
  }

  return (
    <>
      <h1>{state}</h1>
      <button onClick={handleButtonClick}>hi</button>
    </>
  );
}
```

- 위 코드가 동작하지 않는 이유가 무엇일까요?
  - 리액트에서는 렌더링이 어떻게 일어나는지 다시 상기해보겠습니다.
    - 리액트에서 렌더링은 함수형 컴포넌트의 `return` 과 클래스형 컴포넌트의 `render` 함수를 실행한 다음, 이 실행 결과를 이전의 리액트 트리와 비교하여 리렌더링이 필요한 부분만 업데이트해 이루어집니다.
  - 위 코드는 리액트에서 리렌더링을 발생시키기 위한 조건을 전혀 충족하지 못하고 있습니다.

```tsx
let state = "hello";

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
```

- 위 코드에서는 state가 업데이트 되고 있지만 왜 렌더링이 일어나지 않을까요?
  - 이유는 리액트의 렌더링은 함수형 컴포넌트에서 반환한 결과물인 `return`의 값을 비교해 실행되기 때문입니다.
  - 즉, 매번 렌더링이 발생될 때마다 함수는 다시 새롭게 실행되고, 새롭게 실행되는 함수에서 state는 매번 hello로 초기화되므로 아무리 state를 변경해도 다시 hello로 초기화 되는것 입니다.
- 함수형 컴포넌트는 매번 함수를 실행해 렌더링이 일어나고, 함수 내부의 값은 함수가 실행될 때마다 다시 초기화됩니다.
  - **그렇다면 useState훅의 결괏값은 어떻떳뜩떡게 함수가 실행되어도 그 결과값을 유지하고 있을까요?**
- 이를 해결하기 위해 리액트는 클로저를 이용했습니다.
  - 클로저를 사용함으로써 외부에 해당 값을 노출시키지 않고 함수형 컴포넌트가 매번 실행되더라도 useState에서 이전의 값을 정확하게 꺼내 쓸 수 있게 되었습니다.

#### 게으른 초기화

- 알반적으로는 useState에서 기본값을 선언하기 위해 `setState()` 인수로 원시값을 넣는 경우가 대부분일 것 입니다.
- 그러나 이 useState의 인수로 특정한 값을 넘기는 함수를 인수로 넣어줄 수도 있습니다.
- useState에 변수 대신 함수를 넘기는 것을 **게으른 초기화(lazy initialization)**라고 합니다.

```ts
// 일반적인 useState 사용
// 바로 값을 집어넣는다.
const [count, setCount] = useState(
  Number.parseInt(window.localStorage.getItem(cacheKey))
);

// 게으른 초기화
// 위 코드와의 차이점은 함수를 실행해 값을 반환하는 것이다.
const [count, setCount] = useState(() =>
  Number.parseInt(window.localStorage.getItem(cacheKey))
);
```

- 리액트 공식 문서에서 이러한 게으른 초기화는 useState의 초깃값이 복잡하거나 무거운 연산을 포함하고 있을 때 사용하라고 되어 있습니다.

  - 게으른 초기화 함수는 오로지 state가 처음 만들어질 때만 사용됩니다. 만약 이후에 리렌더링이 발생된다면 이 함수의 실행운 무시됩니다.

  ```tsx
  import { useState } from "react";

  export default function App() {
    const [state, setState] = useState(() => {
      console.log("복잡한 연산..."); // App 컴포넌트가 처음 구동될 때만 실행되고, 이후 리렌더링 시에는 실행되지 않는다.
      return 0;
    });

    function handleClick() {
      setState((prev) => prev + 1);
    }

    return (
      <div>
        <h1>{state}</h1>
        <button onClick={handleClick}>+</button>
      </div>
    );
  }
  ```

  - 리액트에서는 렌더링이 실행될 때마다 함수형 컴포넌트의 함수가 다시 실행된 다는 점을 명심해야 합니다.
    - 물론 함수형 컴포넌트의 useState의 값도 재실행됩니다.
    - 이미 앞선 내용으로 인해 내부에는 클로저가 존재하며, 클로저를 통해 값을 가져오며 초깃값은 최초에만 사용된다는 것을 알고 있습니다.
    - 만약 useState 인수로 자바스크립트에 많은 비용을 요구하는 작업이 들어가 있다면 이는 계속해서 실행될 위험이 존재할 것 입니다.
      - **그러나 우려와는 다르게 useState 내부에 함수를 넣으면 이는 최초 렌더링 이후에는 실행되지 않고 최초의 state값을 넣을때만 실행됩니다.**

- 다시 앞선 예제를 돌아본다면 `Number.parseInt(window.localStorage.getItem(cacheKey))`와 같은 비용이 많이 들어가는 값이 있다고 가정해보겠습니다.
  - useState의 인수로 이 값 자체를 사용한다면 초깃값이 필요한 최초 렌더링과, 초깃값이 있어 더 이상 필요 없는 리렌더링 시에도 동일하게 계속 해당 값에 접근해서 낭비가 발생합니다.
  - 따라서 이런 경우에는 함수 형태로 인수에 넘겨주어 게으른 초기화를 유도하는 편이 훨씬 경제적일 것 입니다.
- 그렇다면 게으른 최적화는 언제 사용하는 것이 좋을까요?
  - useState의 초깃값으로 무거운 연산이 들어가는 경우 사용하는게 좋을 것 같습니다. - localStorage 혹은 sessionStorage에 대한 접근 - map, filter, find 같은 배열에 대한 접근 - 초깃값 계산을 위한 함수 호출이 필요한 때와 같이 무거운 연산을 포함하여 실행 비용이 많이 드는 경우 - 등등

### 03.1.2. useEffect

> 이녀석은 리액트에서 아주 유명한 골칫거리입니다.

- 대부분의 개발자에게 useEffect의 정의에 대해 물어본다면 다음과 같은 답변을 들을 수 있을 것 입니다.

  - useEffect는 두 개의 인수를 받는데 첫 번째는 콜백, 두 번째는 의존성 배열 입니다. 이 두 번째 배열의 값이 변경되면 첫 번째 인수인 콜백을 실행합니다.
  - 클래스형 컴포넌트의 생명주기 메서드와 비슷한 동작을 구현할 수 있습니다. 두 번째 의존성 배열에 빈 배열을 넣으면 컴포넌트가 마운트될 때만 실행됩니다.
  - useEffect는 클린업 함수를 반환할 수 있는데, 이 클린업 함수는 컴포넌트가 언마운트될 때 실행됩니다.

- 이러한 useEffect에 대한 정의는 어느 정도 옳지만 완전히 정확하지는 않습니다. **그리고 useEffect는 자주 사용되지만 생각보다 사용하기 쉬운 훅이 아닙니다.**
- 그리고 알려진 것처럼 생명주기 메서드를 대체하기 위해 만들어진 훅도 아닙니다.
- **useEffect의 정의를 정확하게 내리자면 useEffect는 애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 메커니즘 입니다.**
  > 중요 포인트 ⭐️⭐️⭐️⭐️⭐️
- 그리고 이 부수 효과가 "언제" 일어나는지보다 어떤 상태값과 함께 실행되는지 파악하는 것이 중요합니다.

#### useEffect란? 두둥탁 - 암기 ⭐️⭐️⭐️⭐️

```ts
function Component() {
  //...
  useEffect(() => {
    // 대충 코드 생략
  }, [props, state]);

  //...
}
```

- useEffect는 첫 번째 인수로는 실행할 부수 효과가 포함된 함수를, 두 번째 인수로는 의존성 배열을 전달합니다.
  - 이 의존성 배열은 어느 정도 길이를 가진 배열일 수도, 아무런 값이 없는 빈 배열일 수도 있고, 배열 자체를 넣지 않고 생략할 수도 있습니다.
- useEffect를 살펴보기 이전에 중요한 포인트는 **함수형 컴포넌트는 매번 함수를 실행해 렌더링을 수행한다는 것입니다.**

  - 예시로 useState의 원리에 따라 동작하는 함수형 컴포넌트는 렌더링 시마다 고유의 `state`와 `props`값을 갖고 있습니다.
  - 여기에 useEffect가 추가된 형태의 예제코드를 살펴보겠습니다.

    ```tsx
    function Component() {
      const counter = 1;

      useEffect(() => {
        console.log(counter); // 1, 2, 3, 4....
      });

      //...
      return (
        <>
          <h1>{counter}</h1>
          <button onClick={handleClick}>+</button>
        </>
      );
    }
    ```

    - useEffect는 자바스크립트의 proxy 혹은 데이터바인딩, 옵저버 같은 특별한 기능을 통해 값의 변화를 관찰하는 것이 아니고, **렌더링**할 때마다 의존성에 있는 값을 보면서 이 의존성의 값이 이전과 다른 게 하나라도 있으면 부수 효과를 실행하는 평범한 함수라 볼 수 있습니다.
    - **따라서 useEffect**는 state와 props의 변화 속에서 일어나는 렌더링 과정에서 실행되는 부수 호과 함수라고 볼 수 있습니다.

#### 클린업 함수의 목적

- 일반적으로 useEffect내에서 반환되는 클린업 함수는 이벤트를 등록하고 삭제할 때 사용해야 한다고 알려져 있습니다.

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

      // 클린업 함수
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

  - 위 코드를 실행하면 다음과 같은 결과를 얻을 수 있습니다.
    > 클린업 함수 실행! 0
    > 1
    >
    > 클린업 함수 실행! 1
    > 2
    >
    > 클린업 함수 실행! 2
    > 3
    >
    > 클린업 함수 실행! 3
    > 4
    > // ...
  - 로그를 살펴보면 클린업 함수는 이전 counter 값, 즉 이전 state를 참조해 실행된다는 것을 알 수 있습니다.
  - 클린업 함수는 새로운 값과 함께 렌더링된 뒤에 실행되기 때문에 위와 같은 메세지가 나타납니다.

    - 여기서 중요한 것은, 클린업 함수는 비록 새로운 값 기반으로 렌더링 뒤에 실행되지만 이 변경된 값을 읽는 것이 아니라 함수가 정의되었을 당시에 선언되었던 이전 값을 보고 실행된다는 것 입니다.
    - 이해하기 쉽게 코드를 직관적으로 표현해보겠습니다.

      ```ts
      // 최초 실행
      useEffect(() => {
        function addMouseEvent() {
          console.log(1);
        }

        window.addEventListener("click", addMouseEvent);

        // 클린업 함수
        // 크리고 이 클린업 함수는 다음 렌더링이 끝나 위에 실행된다.
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

        // 클린업 함수
        return () => {
          console.log("클린업 함수 실행!", 2);
          window.removeEventListener("click", addMouseEvent);
        };
      }, [counter]);
      ```

      - 위 사실을 종합해보면 왜 useEffect에 이벤트를 추가했을 때 클린업 함수에서 지워야 하는지 알 수 있습니다.
        - 함수형 컴포넌트의 useEffect는 그 콜백이 실행될 때마다 이전의 클린업 함수가 존재한다면 그 클린업 함수를 실행한 뒤 콜백을 실행합니다.
        - 따라서 이벤트를 추가하기 전에 이전에 등록했던 이벤트 핸들러를 삭제하는 코드를 클린업 함수에 추가하는것 입니다.
        - 클린업 함수를 구성함으로써 특정 이벤트의 핸들러가 무한히 추가되는 것을 방지할 수 있습니다.

- 이처럼 클린업 함수는 생명주기 메서드의 언마운트 개념과는 조금 차이가 있는 것을 볼 수 있습니다.
  - 언마운트는 특정 컴포넌트가 DOM에서 사라진다는 것을 의미하는 클래스형 컴포넌트의 용어입니다.
  - 클린업 함수는 언마운트라기 보다는 함수형 컴포넌트가 리렌더링 되었을 때 의존성 변화가 있었을 당시 이전의 값을 기준으로 실행되는, 말 그대로 이전 상태를 청소해 주는 개넘으로 보는 것이 옳습니다.
    > 언마운트는 DOM에서 삭제를 의미하는 클래스형 컴포넌트 용어, 클린업 함수는 리렌더링 과정에서 이전의 값을 기준으로 실행되는 이전 상태 혹은 이벤트핸들러를 청소해주는 개념

#### 의존성 배열

```ts
useEffect(() => {
  console.log("컴포넌트 렌더링 끼얏호우!");
});
```

- 해당 예제는 아주 유우명한 컴포넌트 리렌더링 확인 방법입니다.
- 여기서 의문인 점은 의존성 배열이 없는 useEffect가 매 렌더링마다 실행된다면 그냥 useEffect가 없는 코드를 써도 되는게 아닌가에 주목해야 합니다.

  ```ts
  // 1
  function Component() {
    console.log("렌더링");
  }

  // 2
  function Component() {
    useEffect(() => {
      console.log("렌더링");
    });
  }
  ```

  - 두 코드는 명백한 차이점을 가지고 있습니다.
    1. 이후 소개할 SSR 관점에서 useEffect는 클라이언트 사이드에서 실행되는 것을 보장해줍니다. useEffect 내부에서는 window 객체의 접근에 의존하는 코드를 사용해도 됩니다.
    2. useEffect는 컴포넌트 렌더링의 부수효과, 즉 컴포넌트의 렌더링이 완료된 이후에 실행됩니다.
       반면 직접 실행은 컴포넌트가 렌더링 되는 도중에 실행됩니다.
       따라서 1번 방식과는 달리 SSR의 경우 서버에서도 실행됩니다. 그리고 이 작업은 함수형 컴포넌트의 반환을 지연시키는 행위입니다. 즉, 무거운 작업일 경우 렌더링을 방해하므로 성능에 악영향을 미칠 수 있습니다.

- useEffect의 effect는 컴포넌트의 사이드 이펙트(부수 효과)를 의미한다는 것을 명심해야합니다. useEffect는 컴포넌트가 렌더링된 후에 어떠한 부수 효과를 일으키고 싶을 때 사용하는 훅 입니다.

#### useEffect의 구현

- useEffect의 핵심은 의존성 배열의 이전 값과 현재 값의 얕은 비교입니다.
  - 앞서 1장에서 언급한 것 처럼 리액트는 값을 비교할 때 `Object.is`를 기반으로 하는 얕은 비교를 수행합니다.
  - **이전 의존성 배열과 현재 의존성 배열의 값에 하나라도 변경 사항이 있다면 callback으로 선언한 부수 효과를 실행합니다. 이것이 useEffect의 본질입니다.**

#### useEffect를 사용할 때 주의할 점

> useEffect는 리액트 코드를 작성할 때 가장 많이 사용하는 훅이면서 가장 주의해야 할 훅이기도 합니다. 오싹오싹

##### 1. eslint-disable-line react-hooks/exhaustive-deps 주석은 최대한 자제하라

> 솔직히 이부분 많이 뜨끔하네요 와하하하하

- 정말로 필요한 때에는 사용할 수도 있지만 대부분의 경우에는 의도치 못한 버그를 만들 가능성이 큰 코드입니다.
- 해당 eslint off를 진행하는 이유는 컴포넌트를 마운트 하는 시점에만 무언가를 하고 싶다라는 의도로 작성하고는 합니다.
  - **그러나 해당 방법은 클래스형 컴포넌트의 생명주기 메서드인 componentDidMount에 기반한 접근법으로, 함수형 컴포넌트에서는 가급적이면 사용해서는 안됩니다.**
- useEffect는 반드시 의존성 배열로 전달한 값의 변경에 의해 실행되어야 하는 훅 입니다.
  - 그러나 의존성 배열을 넘기지 않은 채 콜백 함수 내부에서 특정 값을 사용한다는 것은, 이 부수 효과가 실제로 관찰해서 실행되어야 하는 값과는 별개로 동작한다는 것을 의미합니다.
  - 즉, 컴포넌트의 state, props와 같은 어떤 값의 변경과 useEffect의 부수 효과가 별개도 동작하게 된다는 뜻 입니다.
  - useEffect에서 사용한 콜백 함수의 실행과 내부에서 사용한 값의 실제 변경 사이에 연결고리가 끊어져 있는 것 입니다.
- 따라서 정말로 의존성으로 `[]` 빈 배열이 필요하다면 최초에 함수형 컴포넌트가 마운트 되었을 시점에만 콜백 함수 실행이 필요한지를 다시 한번 되물어봐야 합니다.
  - 그리고 정말로 필요하다고 판단되면 useEffect 내 부수 효과가 실행될 위치가 잘못되었을 가능성이 큽니다.
- 마지막으로 정리헤보겠읍니다.
  - useEffect에 빈 배열을 넘기기 전에는 useEffect의 부수 효과가 컴포넌트의 상태과 별개도 동작해야 하는지 혹은 현재 위치에서(depth) 호출하는게 최선인지 검토가 필요합니다.
  - 빈 배열이 아닐때에도 마찬가지로 특정 값을 사용하지만 해당 값의 변경 시점을 피할 목적이라면 메모이제이션을 적절히 활용하여 해당 값의 변화를 막거나 알맞은 실행 위치를 다시 한번 고민해 보는 것이 좋습니다.

##### 2. useEffect의 첫 번째 인수에 함수명을 부여하라.

> 앞서 나왔던 꿀팁이네요!

- useEffect의 인수를 익명 함수가 아닌 적절한 이름을 사용한 기명 함수로 변경해 주면 해당 useEffect의 목적을 파악하기 쉬워집니다.
  ```ts
  useEffect(function 리액트스터디() {
    console.log("useEffect 공부중");
  }, [매주 수요일]);
  ```
  - 함수명을 부여하는 것이 어색할 수 있지만 목적을 명확히 하고 그 책임을 최소한으로 좁한다는 점에서 굉장히 유용합니다.

##### 3. 거대한 useEffect를 만들지 마라샹궈

- useEffect는 의존성 배열을 바탕으로 렌더링 시 의존성이 변경될 때마다 부수효과를 실행합니다.
- 이 부수효과의 크기가 커질수록 애플리케이션 성능에 악영향을 미칠 수 있습니다.
- 가능하면 적은 의존성 배열을 사용하는 useEffect로 분리하는 것이 좋습니다.
- 만약 불가피하게 여러 변수가 들어가야 하는 상황이라면 최대한 useCallback과 useMemo 등으로 사전에 정제한 내용들만 useEffect에 담아두는 것이 좋습니다.

##### 4. 블필요한 외부 함수를 만들지 마라탕

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

- 가독성이 아주 디스거스팅하죠?

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

- 외부 함수를 내부로 가져왔더니 훨씬 간결하죠?
- 불필요한 의존성 배열도 줄일 수 있었죠?
- 무한루프에 빠지기 위해 넣었던 코드인 useCallback 또한 삭제 가능했죠?
- useEffect 내에서 사용할 부수 효과라면 내부에서 만들어 정의하는 편이 훨씬 도움될겁니다

> ###### 아하모먼트 😇
>
> 왜 useEffect의 콜백 인수로 비동기 함수를 바로 넣을 수 없을까요?
> 그 이유는 useEffect에서 비동기로 함수를 호출할 경우 경쟁 생태가 발생할 수 있기 때문입니다.
> 경쟁상태 Race condition
> 추가적으로 비동기 함수가 내부에 존재하게 되면 useEffect 내부에서 비동기 함수가 생성되고 실행되는 것을 반복하므로 클린업 함수에서 이전 비동기 함수에 대한 처리를 추가하는 것이 좋습니다.

> 경쟁 상태 (Race Condition) 😮
> 멀티스레딩 환경에서 동시에 여러 프로세스나 스레드가 공유된 자원에 접근하려고 할 때 발생할 수 있는 문제입니다. 이러한 > 상황에서 실행 순서나 타이밍을 예측할 수 없게 되어 프로그램 동작이 원하지 않는 방향으로 흐를 수 있습니다.

### 03.1.3. useMemo

- useMemo는 비용이 큰 연산에 대한 결과를 저장(메모이제이션)해 두고, 이 저장된 값을 반환하는 훅입니다. 흔히 리액트에서 최적화를 떠올일 때 가장 먼저 언급되는 훅이 바로 useMemo 입니다.
- useMemo는 렌더링 발생 시 의존성 배열의 값이 변경되지 않았으면 함수를 재실행하지 않고 이전에 기억해 둔 해당 값을 반환합니다.
  - 만약 의존성 배열의 값이 변경되었다면 첫 번째 인수로 받는 함수를 실행한 후 값을 반환하고 해당 결괏값을 다시 기억해 둘 것입니다.
- 이러한 메모이제이션은 단순히 값뿐만 아니라 컴포넌트도 가능합니다. 물론 React.memo를 사용하는 것이 더 현명합니다.

### 03.1.4. useCallback

- useMemo가 값을 기억했다면, useCallback은 인수로 넘겨받은 콜백 자체를 기억합니다.
  - 쉽게 이야기해보면, useCallback은 특정 함수를 새로 만들지 않고 다시 재사용한다는 의미입니다.
- 어떻게 특정 함수를 재생성하지 않는다는 것인지 한번 톺아보겠읍니다.

  ```tsx
  const ChildComponent = memo(({ name, value, onChange }) => {
    // 렌더링이 수행되는지 확인하기 위해 넣었다.
    useEffect(() => {
      console.log("rendering!", name);
    });

    return (
      <>
        <h1>
          {name} {value ? "켜짐" : "꺼짐"}
        </h1>
        <button onClick={onChange}>toggle</button>
      </>
    );
  });

  function App() {
    const [status1, setStatus1] = useState(false);
    const [status2, setStatus2] = useState(false);

    const toggle1 = () => {
      setStatus1(!status1);
    };

    const toggle2 = () => {
      setStatus2(!status2);
    };

    return (
      <>
        <ChildComponent name="1" value={status1} onChange={toggle1} />
        <ChildComponent name="2" value={status2} onChange={toggle2} />
      </>
    );
  }
  ```

  - memo를 사용해서 컴포넌트를 메모이제이션 했지만 App의 자식 컴포넌트 전체가 렌더링되고 있습니다.
    - 사실 위 예제에서는 하위 컴포넌트에 memo를 사용해 props의 값을 모두 기억하고, 이 값이 변경되지 않았을 때는 렌더링되지 않도록 작성된 코드입니다.
    - 정상적인 흐름이라면 하나의 value 변경이 다른 컴포넌트에 영향을 미쳐서는 안되고 클릭할 때마다 하나의 컴포넌트만 렌더링되어야 합니다.
    - 하지만 어느 한쪽을 클릭해도 모든 컴포넌트가 렌더링되는 것을 알 수 있습니다.
      - 그 이유는 state값이 바뀌면서 App 컴포넌트가 리렌더링되고, 그때마다 매번 onChange로 넘기는 함수가 재생성되고 있기 때문입니다.

- 값의 메모이제이션을 위해 useMemo를 사용했다면, 함수의 메모이제이션을 위해 사용하는 것이 useCallback입니다.
- 함수의 재생성을 막아 불필요한 리소스 소모 또는 리렌더링을 방지하고 싶을 때 useCallback을 사용해 볼 수 있습니다.
- useMemo와 useCallback의 유일한 차이는 메모이제이션을 하는 대상이 변수냐 함수냐일 뿐입니다.
  - 자바스크립트에서는 함수 또한 값으로 표현될 수 있으므로 매우 자연스러운 코드라고 볼 수 있습니다.
  - 다만 useMemo로 useCallback을 구현하는 경우 불필요하게 코드가 길어지고 혼동을 야기할 수 있으므로 리액트에서 별도의 훅으로 제공하는 것으로 추측해볼 수 있습니다.

### 03.1.5. useRef

- useRef는 useState와 동일하게 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장한다는 공통점이 있습니다.
  - 그러나 useState와 구별되는 큰 차이점 두 가지를 가지고 있습니다.
    1. useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있습니다.
    2. useRef는 그 값이 변하더라도 렌더링을 발생시키지 않습니다.

> 신입 입사 초기에 useState와 useRef의 사용을 혼동해서 안좋은 리뷰를 받았던 기억이 올라오네요. 😇

- useRef에 대해 고민해볼 포인트가 몇가지 존재합니다.

  - 렌더링에 영향을 미치지 않는 고정값을 관리하기 위해 useRef를 사용한다면 그냥 함수 외부에서 값을 선언해서 관리하는 것도 가능하지 않을까요?
  - 해당 방법은 몇 가지 단점이 존재합니다.
    - 먼저 컴포넌트가 실행되어 렌더링되지 않았음에도 value값이 기본적으로 존재하게 되어 메모리에 불필요한 값을 갖게 만듭니다.
    - 만약 컴포넌트가 여러 번 생성된다면 각 컴포넌트에서 가리키는 값이 모두 value로 동일합니다. 대부분의 경우에는 컴포넌트 인스턴스 하나당 하나의 값을 필요로 하는 경우가 일반적입니다.
  - useRef는 이러한 문제들을 모두 극복할 수 있는 리액트식 접근법입니다.
    - useRef value는 컴포넌트가 렌더링될 때만 생성되며, 컴포넌트 인스턴스가 여러 개라도 각각 별개의 값을 바라봅니다.

- useRef의 가장 일반적인 사용 예시는 바로 요소의 DOM에 접근할 때의 사용일 것입니다.

```tsx
function RefComponent() {
  const inputRef = useRef();

  // 이때는 미처 렌더링이 실행되기 전(반환되기 전)이므로 undefined를 반환한다.
  console.log(inputRef.current); // undefined

  useEffect(() => {
    console.log(inputRef.current); // <input type="text"></input>
  }, [inputRef]);

  return <input ref={inputRef} type="text" />;
}
```

**이 코드가 보여주는 중요한 개념:**

**실행 순서:**

```
1. RefComponent 렌더링 시작
2. const inputRef = useRef() → { current: undefined }
3. console.log(inputRef.current) → undefined (DOM이 아직 생성 안 됨!)
4. JSX 반환 (return)
5. 실제 DOM 생성 & 마운트
6. useEffect 실행
7. console.log(inputRef.current) → <input type="text"></input> (이제 DOM이 있음!)
```

- useRef는 최초에 넘겨받은 기본값을 가지고 있습니다.
  - **중요 포인트는 useRef의 최초 기본값은 return문에 정의해 둔 DOM이 아닌`useRef()`로 넘겨받은 인수**라는 것 입니다.
  - useRef가 선언된 당시에는 아직 컴포넌트가 렌더링되기 전이라 return으로 컴포넌트의 DOM이 반환되기 이전이므로 undefined 입니다.

### 03.1.6. useContext

- useContext에 대해 이해하려면 먼저 리액트의 Context에 대해 알아야 합니다.

#### Context란?

- 리액트 애플리케이션은 기본적으로 부모 컴포넌트와 자식 컴포넌트로 이루어진 트리 구조를 갖고 있기 때문에 부모가 가지고 있는 데이터를 자식에도 사용하고 싶다면 props로 데이터를 넘겨주는 것이 일반적입니다.
  - 그러나 전달해야 하는 데이터가 있는 컴포넌트와 전달받아야 하는 컴포넌트의 깊이가 깊어질수록 props drilling이 발생하기 쉽습니다.
  - 또한 전달받기 위한 경로에서도 사용하지 않는 데이터를 전달하기 위해 props가 열려있어야 하는 등의 번거로움이 발생합니다.
- 이러한 prop 내려주기를 극복하기 위해 등장한 개념이 바로 Context 입니다.
- Context를 사용하면 명시적인 Props 전달 과정이 없이도 선언한 하위 컴포넌트에서 모두 자유롭게 원하는 값을 사용할 수 있습니다.

#### Context를 함수형 컴포넌트에서 사용할 수 있게 해주는 useContext 훅

- useContext는 상위 컴포넌트에서 만들어진 Context를 함수형 컴포넌트에서 사용할 수 있도록 만들어진 훅입니다.
- useContext를 사용하면 상위 컴포넌트에 선언되어있는 `<Context.Provider />`에서 제공한 값을 사용할 수 있게 됩니다.
- 만약 여러 개의 provider가 존재한다면 가장 가까운 provider의 값을 가져오게 됩니다.

#### useContext를 사용할 때 주의할 점

- useContext를 함수형 컴포넌트 내부에서 사용할 때는 항상 컴포넌트 재사용이 어려워 진다는 점을 염두에 두어야 합니다.
- useContext가 선언되어 있으면 Provider에 의존성을 가지고 있게 되므로 Provider 바깥 트리에서는 재활용하기 어려운 컴포넌트가 됩니다.
- **마지막으로 일부 리액트 개발자들이 Context와 useContext를 상태 관리를 위한 리액트의 API로 오해하고 있다는 점 입니다.**
  - 엄밀히 따지면 콘텍스트는 상태를 주입해주는 API입니다.
  - 상태관리 라이브러리가 되기 위해서는 최소한 다음 두 가지 조건을 만족해야 합니다.
    1. 어떠한 상태를 기반으로 다른 상태를 만들어 낼 수 있어야 합니다.
    2. 필요헤 따라 이러한 상태 변화를 최적화 할 수 있어야 합니다.

> 상태관리 스토어와의 차이점을 인지하지 못하고 있었습니다. 😇
> 리액트 팀에서 의도한 설계 철학을 알아야 적절히 사용할 수 있는 부분일 것 같아요

### 03.1.7. useReducer

- useReducer는 아주 유우명한 useState의 심화 버전입니다.
- useReducer에서 사용되는 용어를 먼저 살펴보겠습니다.
  - 반환값은 useState와 동일하게 길이가 2인 배열이다.
    - state: 현재 useReduecer가 가지고 있는 값을 의미합니다. useState와 마친가지로 배열을 반환하는데, 동일하게 첫 번째 요소가 이 값입니다.
    - dispatcher: state를 업데이트하는 함수입니다. useReducer가 반환하는 배열의 두 번째 요소입니다.
      setState는 단순히 값을 넘겨주지만 여기서는 action을 넘겨준다는 점이 다릅니다. action이란 state를 변경할 수 있는 action을 의미합니다.
  - useState의 인수와 달리 2개에서 3개의 인수를 필요로 합니다.
    - reducer: useReducer의 기본 action을 정의하는 함수입니다. reducer는 useReducer의 첫 번째 인수로 넘겨주어야 합니다.
    - initialState: 두 번째 인수로, useReducer의 초깃값을 의미합니다.
    - init: useState의 인수로 함수를 넘겨줄 때처럼 초깃값을 지연해서 생성시키고 싶을때 사용하는 함수입니다.
      옵셔널한 인수이며, 이곳에 인수로 넘겨주는 함수가 존재한다면 useState와 동일하게 게으른 초기화가 일어나면 initialState를 인수로 init함수가 실행됩니다.

### 03.1.8. useImperativeHandle

- 이게 뭔가 싶겠지만 일단 적어보겠습니다.
- useImperativeHandle을 이해하기 위해서는 먼저 forwardRef에 대해 알아야 합니다.

> React 19에서는 ref를 일반 prop으로 취급하도록 변경되었습니다. 그러므로 이후 내용은 생략해보겠습니다 😇

### 03.1.9. useLayoutEffect

- 공식 문서에 따르면 useLayoutEffect를 다음과 같이 정의하고 있습니다.

  ```
  이 함수의 시그니처는 useEffect와 동일하나, 모든 DOM의 변경 후에 동기적으로 발생한다.
  ```

  - 먼저 함수 시그니처가 동일하다는 의미는 두 훅의 형태나 사용 예제가 동일하다는 것을 의미합니다.

- useLayoutEffectr를 이해하기 위한 중요한 사실은 **모든 DOM의 변경 후에 useLayoutEffect의 콜백 함수 실행이 동기적으로 발생**한다는 점입니다.
  - 여기서 이야기하는 DOM 변경이란 렌더링입니다.
    - 브라우저에서 실제로 해당 변경 사항이 반영되는 시점을 의미하는 것은 아닙니다.
    - 즉, 실행순서는 다음과 같습니다.
      1. 리액트가 DOM을 업데이트
      2. useLayoutEffect를 실행
      3. 브라우저에 변경 사항을 반영
      4. useEffect를 실행
  - 동기적으로 실행된다는 것은 리액트에서는 useLayoutEffect의 실행이 종료될 때까지 기다린 다음 화면을 그린다는 것을 의미합니다.
  - 즉, 리액트 컴포넌트는 useLayoutEffect가 완료될 때까지 기다리기 때문에 컴포넌트가 잠시 동안 일시 중지되는 것과 같은 일이 발생합니다.
    - 이러한 동작 방식으로 인해 애플리케이션 성능에 문제가 발생할 수 있습니다.
- 언제 사용하는게 좋을까요?
  - useLayoutEffect의 특징상 DOM은 계산되었지만 이것이 화면에 반영되기 전에 하고 싶은 작업이 있을때 등 반드시 필요한 경우만!
  - 특정 요소에 따라 DOM 요소를 기반으로 한 애니메이션, 스크롤 위치를 제어하는 등 화면에 반영되기 전에 하고 싶은 작업에 사용한다면 useEffect보다 더 자연스러운 UX를 제공할 수 있게 됩니다.

#### 얼마전 TS공부중에 궁금해서 정리했던 내용을 다시 가져와보았습니다.

- useLayoutEffect는 DOM 변경 완료 후, 브라우저가 화면을 그리기 이전(paint)전에 실행됩니다.
- useLayoutEffect를 사용하는 실제 활용사례를 나열해보겠습니다.

  - 툴팁 혹은 팝오버 위치 계산
  - 스크롤 위치 복원
  - FLIP 애니메이션
  - 포커스 관리
  - DOM 크기 측정 후 조건부 렌더링

- 핵심 판단 기준 useLayoutEffect를 사용해야 하는 경우:

  - ✅ DOM 측정이 필요하고
  - ✅ 그 결과로 즉시 다른 DOM을 업데이트해야 하며
  - ✅ 사용자가 중간 상태를 보면 안 되는 경우

### 03.1.10. useDebugValue

- useDebugValue는 일반적으로 프로덕션 웹서비스에서 사용하는 훅이 아닙니다.
- 이 훅은 개발과정에서 사용되는대, 디버깅하고 싶은 정보를 이 훅에 사용하면 리액트 개발자 도구에서 볼 수 있습니다.
- useDebugValue는 커스텀훅 내부에 어떠한 정보를 남길 수 있는 훅입니다.
  - useDebugValue를 사용할 때에는 오직 훅 내부에서만 실행할 수 있음에 주의합시다!
  - 컴포넌트 레벨에서는 동작하지 않습니다

### 03.1.11. 훅의 규칙

#### 리액트에서 훅을 안전하게 사용하기 위해서는 두 가지 규칙을 준수해야 합니다.

1. 훅은 항상 최상위 레벨에서 호출되어야 합니다. 반복문, 조건문, 중첩함수, 클래스 등의 내부에서는 호출하지 않아야 합니다.
   반환문으로 함수 컴포넌트가 종료되거나, 조건문 또는 변수에 따라 반복문 등으로 훅의 호출 여부가 결정되어서는 안됩니다.
   이렇게 해야 useState 또는 useEffect가 여러 번 호출되더라도 훅의 상태를 올바르게 유지할 수 있습니다.
2. 훅은 항상 컴포넌트나 커스텀 훅 등의 리액트 컴포넌트 내에서만 호출되어야 합니다. 일반 자바스크립트 함수에서는 훅을 호출할 수 없습니다.

이러한 규칙이 필요한 이유는 리액트에서 훅은 호출 순서에 의존하기 때문이빈다. 모든 컴포넌트 렌더링에서 훅의 순서가 항상 동일하게 유지되어야 하며, 이를 통해 항상 동일한 컴포넌트 렌더링이 보장되어야 합니다.

- 리액트에서는 훅에 대한 정보 저장은 리액트 내부 어딘가에 있는 index와 같은 키를 기반으로 구현되어 있습니다.(실제로는 객체 기반 링크드 리스트에 더 가깝습니다.)
- 즉, 실행 순서에 아주 큰 영향을 받는다는 의미입니다.

  ```ts
  function Component() {
    const [count, setCount] = useState(0);
    const [required, setRequired] = useState(false);

    useEffect(() => {
      // do something...
    }, [count, required]);
  }
  ```

  - 이 컴포넌트는 파이버에서 다음과 같이 저장됩니다.

    ```ts
    {
      memoizedState: 0, // setCount 값
      baseState: 0,
      queue: { /* ... */ },
      baseUpdate: null,
      next: {
        // setRequired 값
        memoizedState: false,
          baseState: false,
        queue: { /* ... */ },
        baseUpdate: null,
        next: {
          // useEffect 값
          memoizedState: {
            tag: 192,
            create: () => {},
            destroy: undefined,
            deps: [0, false],
            next: { /* ... */ }
          },
        baseState: null,
        queue: null,
        baseUpdate: null,
      }
    }
    ```

    - 코드에서 확인할 수 있듯이 리액트 훅은 파이버 객체의 링크드 리스트의 호출 순서에 따라 저장됩니다.
      - 그 이유는 각 훅이 파이버 객체 내에서 순서에 의존해 state나 effect의 결과에 대한 값을 저장하고 있기 때문입니다.
    - 이렇게 고정된 순서에 의존해 훅과 관련된 정보를 저장함으로써 이전 값에 대한 비교와 실행이 가능해집니다.

- 조건이나 다른 이슈로 인해 훅의 순서가 깨지거나 보장되지 않을 경우 링크드 리스트가 깨지며 리액트 코드는 에러를 발생시키게 됩니다.

## 03.2. 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?

> HOC는 지양해야 한다고 생각합니다. forwardRef가 React19에서는 더이상 사용되지 않는 것 처럼 사용하기에 어려움과 번거로움 그리고 displayName등의 처리로 인한 복잡함이 불필요하게 소요된다고 생각합니다.
> 또한 책에서 언급하듯이 컴포넌트가 또 다른 컴포넌트를 감싸는 구조로 되어있어, 반복적인 고차컴포넌트 사용시 복잡성이 매우 커지며, 예측할 수 없는 사이드 이펙트의 위험 또한 커지게 됩니다.

### 03.2.2. 고차 컴포넌트 HOC

#### React.memo 란?

- memo는 아주 유우명한 React 고차 컴포넌트 중 하나입니다.
- React.memo를 이해하기 위해 렌더링에 대해 다시 떠올려보겠습니다.
  - 리액트 컴포넌트가 렌더링하는 조건 중 하나인 부모 컴포넌트가 새롭게 렌더링 되는 경우 자식 컴포넌트의 props 변경 여부와 관계없이 발생합니다.
  - 이 경우 props의 변화가 없음에도 컴포넌트의 렌더링을 방지하기 위해 만들어진 리액트의 고차 컴포넌트가 바로 React.memo입니다.
- React.memo는 렌더링 하기 이전에 props를 비교하여 이전과 달라진 점이 없다면 렌더링 자체를 생략하고 메모이제이션 된 컴포넌트를 반환하게 됩니다.
