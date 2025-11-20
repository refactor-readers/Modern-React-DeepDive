# 01. 리액트 개발을 위해 꼭 알아야 할 자바스크립트

> 본격적으로 리액트를 다루기 앞서, 리액트 코드의 기반이 되는 자바스크립트에 대해 먼저 알아보겠습니다.
> 단순 코드를 작성하는데 그치지 않고, 웹 애플리케이션이 동작하는 이면에서 리액트가 수행하는 작업을 이해하려면 자바스크립트의 개념을 다시금 알아볼 필요가 있습니다.

## 01.1. 자바스크립트의 동등 비교

- 리액트 커포넌트의 렌더링이 일어하는 이유 중 하나가 바로 Props의 동등 비교에 따른 결과 입니다.
  - props의 동등 비교는 객체의 같은 비교를 기반으로 이뤄지는데, 이 얕은 비교가 리액트에서 어떻게 작동하는지 이해하지 못하면 ~.~ 큰일나요~
    - 리액트의 Virtual DOM과 실제DOM의 비교
    - 리액트 컴포넌트가 렌더링할지를 판단하는 방법
    - 변수나 함수의 메미오제이션
    - 등등 모든 작업은 자바스크립트의 동등 비교를 기반으로 합니다.

### 자바스크립트의 데이터 타입

> 자바스크립트 데이터 타입 내용 스킵

```js
typeof null === "object"; // true?
```

- null이 가지고 있는 특별한 점 하나는 다른 원시값과 다르게 `typeof`로 null을 확인했을 때 해당 데이터타입이 아닌 "object"가 반환된다는 것입니다.
  - 이는 초창기 자바스크립트가 값을 표현하는 방식 때문에 발생한 문제로, 이후에 `typeof null === "null"`로 표현하고자 하는 시도가 있었으나 이전 코드의 호환성이 깨지는 변경사항(breaking change)이어서 받아들여지지 않았습니다.
    > undefined가 전역변수였던 시절이 있었다는데 js는 도대체...

#### 객체 타입

- 객체 차입은 참조를 전달한다고 해서 참조 타입으로도 불립니다.
  > 여기에서 우리가 알아둬야 할 자바스크립트 동등 비교의 특징이 나타납니다.

```js
typeof [] === "object"; // true
typeof {} === "object"; // true

function hello() {}
typeof hello === "function"; // true

const hello1 = function () {};
const hello2 = function () {};

// 객체인 함수의 내용이 육안으로는 같아 보여도 참조가 다르기 때문에 false가 반환된다.
hello1 === hello2; // false
```

### 값을 저장하는 방식의 차이

- 원시 타입과 객체 타입의 가장 큰 차이점이라고 한다면, 바로 값을 저장하는 방식의 차이 입니다.
  - 이 값을 저장하는 방식의 차이가 동등 비교를 할 때 차이를 만드는 원인이 됩니다.
  - 원시 타입은 불변 형태의 값으로 저장됩니다. 그리고 이 값은 변수 할당 시점에 메모리 영역을 차지하고 저장됩니다.
- 반면 객체는 프로퍼티를 삭제, 추가, 수정할 수 있으므로 원시 값과 다르게 변경 가능한 형태로 저장되며, 값을 복사할 때도 값이 아닌 참조를 전달하게 됩니다.

  - 객체는 값을 저장하는게 아니라 참조를 저장하기 때문에 동일하게 선언했던 객체라 하더라도 저장하는 순간 다른 참조를 바라보기 때문에 동등 비교에서 false를 반환하게 됩니다.
  - 즉, 값은 같을지언정 참조하는 곳이 다른 셈 입니다.
  - 반면 참조를 전달하는 경우에는 이전에 원시 값에서 했던 것과 같은 결과를 기대할 수 있습니다.

- 따라서 자바스크립트 개발자는 항상 객체 간에 비교가 발생하면, 우리가 이해하는 내부의 값이 같다 하더라도 결과는 대부분 true가 아닐 수 있다는 것을 인지해야 합니다.

### 자바스크스립트의 또 다른 비교 공식, Object.is

- Object.is는 또 다른 자바스크립트의 비교 방식입니다.
- Object.is는 두 개의 인수를 받으며, 이 인수 두 개가 동일한지 확인하고 반환하는 메서드 입니다.
- Object.is가 `==`나 `===`과 다른 점은 다음과 같습니다.

  - `==` vs Object.is
    - == 비교는 같음을 비교하기 이전에 양쪽이 같은 타입이 아니라면 비교할 수 있도록 강제 형변환 이후 비교를 진행합니다.
    - 하지만 Object.is는 이러한 작업을 하지 않습니다.
      - 즉, ===와 동일하게 타입이 다르면 그냥 false입니다.
  - `===` vs Object.is
    - 해당 방식 또한 차이가 있습니다.
    - 아래 예시를 확인해보면 Object.is가 조금 더 개발자가 기대하는 방식으로 정확히 비교를 진행합니다.

  ```js
  -0 === +0; // true
  Object.is(-0, +0); // false

  Number.NaN === NaN; // false
  Object.is(Number.NaN, NaN); //true
  NaN === 0 / 0; // false
  Object.is(NaN, 0 / 0); //true
  ```

  - 해당 예시와 같이 ==과 ===가 만족하지 못하는 특수한 케이스를 추가하기 위해, 나름의 알고리즘으로 Object.is가 동작하는 것을 알 수 있습ㄴ디ㅏ.
  - 하지만 주의해야 할 점은, Object.is를 사용해도 객체 비교에는 별 차이가 없다는 것 입니다.
    - 객체 비교는 앞서 이야기한 객체 비교 원리와 동등합니다.

### 리액트에서의 동등 비교

- 리액트에서 사용하는 동등 비교는 `==`나 `===`가 아닌 Object.is 메서드를 사용합니다.
- 리액트에서는 objectIs를 기반으로 하는 동등 비교를 수행하는 `shallowEqual`이라는 함수를 만들어서 사용합니다.
  - 해당 `shallowEqual`은 의존성 비교 등 리액트의 동등 비교가 필요한 다양한 곳에서 사용됩니다.

```ts
// 단순히 Object.is를 수행하는 것뿐만 아니라 객체 간의 비교도 추가돼 있다.
function shallowEqual(objA: mixed, objB: mixed): boolean {
  if (is(objA, objB)) {
    return true;
  }

  if (
    typeof objA !== "object" ||
    objA === null ||
    typeof objB !== "object" ||
    objB === null
  ) {
    return false;
  }

  // 각 키배열을 꺼낸다.
  const keysA = 0bject.keys(objA)
  const keysB = 0bject.keys(objB)

  // 배열의 길이가 다르다면 false
  if (keysA.length !== keysB.length) {
    return false
  }

  // A의 키를 기준으로, B에 같은 키가 있는지, 그리고 그 값이 같은지 확인한다
  for (let i = 0; i < keysA.length; i++) {
    const currentKey = keysA[i]
    if (
      !hasOwnProperty.call(objB,currentKey) ||
      !is(objA[currentKey],objB[currentKey])
    ) {
      return false
    }

  return true
}

export default shallowEqual
```

- 리액트에서의 비교를 요약해보겠습니다.
  1. Object.is로 먼저 비교를 수행한 다음
  2. Object.is에서 수행하지 못하는 비교인 객체 간 같은 비교를 한번 더 수행하는 것을 알 수 있습니다.
     > 객체 간 얕은 비교란 객체의 첫 번째 깊이에 존재하는 값만 비교하는 것을 의마합니다.
- 리액트에서는 객체의 얕은 비교까지만 구현한 이유는 무엇일까요?

  - 리액트에서 사용하는 JSX props는 객체이고, 여기에 있는 props만 일차적으로 비교하면 되기 때문입니다.

  ```tsx
  type Props = {
    hello: string;
  };

  function HelloComponent(props: Props) {
    return <h1>{hello}</h1>;
  }

  // ...

  function App() {
    return <HelloComponent hello="hi!" />;
  }
  ```

  - 위 코드에서 props는 객체입니다. 그리고 기본적으로 리액트는 props에서 꺼내온 값을 기준으로 렌더링을 수행하기 때문에 일반적인 케이스에서는 얕은 비교로 충분할 것 입니다.
  - 이런 내용을 기반으로 다시 이야기 해보면, props에 또 다른 객체를 넘겨준다면 리액트 렌더링이 예상치 못하게 작동한다는 것을 알 수 있습니다.
  - props의 depth가 깊어지는 예시를 확인해보겠습니다.

    ```tsx
    import { memo, useEffect, useState } from "react";

    type Props = {
      counter: number;
    };

    const Component = memo((props: Props) => {
      useEffect(() => {
        console.log("Component has been rendered!");
      });

      return <h1>{props.counter}</h1>;
    });

    type DeeperProps = {
      counter: {
        counter: number;
      };
    };

    const DeeperComponent = memo((props: DeeperProps) => {
      useEffect(() => {
        console.log("DeeperComponent has been rendered!");
      });

      return <h1>{props.counter.counter}</h1>;
    });

    export default function App() {
      const [, setCounter] = useState(0);

      function handleClick() {
        setCounter((prev) => prev + 1);
      }

      return (
        <div className="App">
          <Component counter={100} />
          <DeeperComponent counter={{ counter: 100 }} />
          <button onClick={handleClick}>+</button>
        </div>
      );
    }
    ```

    - 예시와 같이 props가 깊어지는 경우, 즉 한 객체 안에 또다른 객체가 있는 경우 React.memo는 컴포넌트에 실제로 변경된 값이 없음에도 불구하고 메모이제이션된 컴포넌트를 반환하지 못합니다.
      - 상위 컴포넌트인 App 에서 버튼을 클릭해서 강제로 렌더링을 일으킬 경우, shallowEqual을 사용하는 Component 함수는 위 로직에 따라 정확히 객체 간 비교를 수행해서 렌더링을 방지해 주었지만, depth가 하나 더 있는 DeeperComponent 함수는 제대로 비교하지 못해 memo가 동작하지 않는 모습을 볼 수 있습니다.

### 정리

- 자바스크립트에서 객체 비교의 불완전성은 스칼라나 하스켈 등의 다른 함수형 언어에서는 볼 수 없는 특징으로, 자바스크립트 개발자라면 반드시 기억해 두어야 합니다.
- 자바스크립트를 기반으로 한 리액트 함수형 프로그래밍 모델에서도 이러한 언어적인 한계를 뛰어남을 수 없으므로 얕은 비교만을 사용하여 비교를 수행하고, 필요한 기능을 구현하고 있습니다.
- 이러한 자바스크립트 특징을 잘 숙지해야 향후 함수형 컴포넌트에서 사용되는 훅의 의존성 배열 비교, 렌더링 방지를 넘어선 useMemo와 uesCallback의 필요성, 렌더링 최적화를 위해서 꼭 필요한 React.memo를 올바르게 작동시키기 위해 고려해야 할 것들을 쉽게 이해할 수 있을 것입니다.

## 01.2. 함수

### 함수란 무엇인가?

- 자바스크립트에서 함수란 작업을 수행하거나 값을 계산하는 등의 과정을 표헌하고, 이를 하나의 블록으로 감싸 실행 단위로 만들어 놓은 것을 의미합니다.
- 리액트에서는 컴포넌트를 만드는 함수도 이러한 기초적인 형태를 따르고 있습니다.

### 함수를 정의하는 4가지 방법

#### 함수 선언문

```js
function add(a, b) {
  return a + b;
}
```

- 함수 선언문은 표현식이 아닌 일반 문으로 분류됩니다.

#### 함수 표현식

- 함수 표현식에 대해 알아보기 전에 "일급 객체"라는 개념을 알고 있어야 합니다.
  > #### 일급 객체
  >
  > 일급 객체란 다른 객체들에 일반적으로 적용 가능한 연산을 모두 지원하는 객체를 의미합니다.
- 자바스크립트에서 함수는 일급 객체입니다.
  - 함수는 다른 함수의 매개변수가 될 수 있고, 반환값이 될 수도 있으며, 할당 또한 가능하므로 일급 객체가 되기 위한 조건을 모두 갖추고 있습니다.
- 앞서 함수가 일급 객체라고 했으니, 함수를 변수에 할당하는 것 또한 가능합니다.

```ts
cosnt sum = function add(a, b) {
  return a + b;
}

sum(10, 24) // 34
```

- 함수 표현식에서는 할당하려는 함수의 이름을 생략하는 것이 일반적입니다. 그 이유는 코드를 보았을 때 혼란을 방지하기 위함입니다.
  - 함수 표현식에서 함수에 이름을 주는 것은 함수 호출에 도움이 전혀 안되는, 코드를 읽는 데 방해가 될 수 있는 요소입니다.

##### 함수 표현식과 선언 식의 차이

> 두 방식의 가장 큰 차이점은 호이스팅 여부입니다.

- 함수의 호이스팅은 함수에 대한 선언을 실행 전에 미리 메모리에 등록하는 작업을 의미합니다.
  - 이러한 함수 호이스팅 특성으로 인해 함수 선언문이 미리 메모리에 등록되었고, 코드의 순서에 상관없이 함수를 호출할 수 있게 된 것입니다.
- 반면 함수 표현식은 함수를 변수에 할당합니다.
  - 그러나 함수 호이스팅과는 다르게 호이스팅 되는 시점에서 var로 선언된 식별자의 경우 undefined로 초기화 된다는 차이가 있습니다.
  - 이후 런타임 시점에 함수가 할당되어 동작하게 됩니다.

#### Function 생성자

> 왠만하면 쓰지말자.

#### 화살표 함수

- 화살표 함수는 겉보기와 다르게 앞서 언급한 함수 생성 방식과 몇 가지 큰 차이점이 있습니다.
- 먼저 화살표 함수에서는 constructor를 사용할 수 없습니다.

  - 즉, 생성자 함수로 화살표 함수를 사용하는 것은 불가능합니다.

  ```ts
  const Car = (name) => {
    this.name = name;
  };

  // 사용 불가능한 패턴입니다.
  const myCar = new Car("하이");
  ```

- 그리고 화살표 함수에서는 arguments가 존재하지 않습니다.

```ts
// 1. 일반 함수 - arguments 객체 사용 가능
function hello() {
  console.log(arguments);
}

// Arguments(3) [1, 2, 3, callee: ƒ, Symbol(Symbol.iterator): ƒ]
hello(1, 2, 3);

// 2. 화살표 함수 - arguments 객체 없음
const hi = () => {
  console.log(arguments);
};

// Uncaught ReferenceError: arguments is not defined
hi(1, 2, 3);
```

- 그리고 화살표 함수와 일반 함수의 가장 큰 차이점은 바로 this 바인딩 입니다.

  - 이 차이로 인해 후술할 클래스형 컴포넌트에서 이벤트에 바인딩 할 메서드 선언 시 화살표 함수로 했을 때와 일반 함수로 했을 때 서로 다르게 동작하게 됩니다.

  ```tsx
  class Component extends React.Component {
    constructor(props) {
      super(props);
      this.state = {
        counter: 1,
      };
    }

    // 일반 메서드 - this가 undefined
    functionCountUp() {
      console.log(this); // undefined
      this.setState((prev) => ({ counter: prev.counter + 1 }));
    }

    // 화살표 함수 - this가 class Component를 가리킴
    ArrowFunctionCountUp = () => {
      console.log(this); // class Component
      this.setState((prev) => ({ counter: prev.counter + 1 }));
    };

    render() {
      return (
        <div>
          {/* Cannot read properties of undefined (reading 'setState') */}
          <button onClick={this.functionCountUp}>일반 함수</button>

          {/* 정상적으로 작동함 */}
          <button onClick={this.ArrowFunctionCountUp}>화살표 함수</button>
        </div>
      );
    }
  }
  ```

  - 일반 함수에서의 this는 undefined를, 화살표 함수에서의 this는 우리가 원하는 대로 클래스의 인스턴스읜 this를 가리키는 것을 볼 수 있습니다.
    - 즉, 별도의 작업을 추가로 하지 않고 this를 접근할 수 있는 방법이 바로 화살표 함수인 것입니다.
  - 이처럼 화살표 함수의 this는 선언 시점에 결정된다는 일반 함수와 대비되는 큰 차이점이 있기 때문에 this를 사용할 수 밖에 없는 클래스형 컴포넌트 내부에서 각별한 주의가 필요합니다.

### 다양한 함수 살펴보기

- 리액트에서 자주 사용되는 형태에 대해서만 알아보겠습니다.

#### 즉시 실행 함수

#### 고차 함수 (HOF)

### 함수를 만들 때 주의해야 할 사항

- 번수 선언하는 것만큼 자주 함수를 만들게 됩니다.
- 습관적으로 함수를 만들다 보면 함수 생성과 사용에 있어 중요한 부분을 놓칠 수 있는데, 좋은 함수는 무엇이고 또 함수를 만들 때 무엇을 조심해야 하는지 살펴보겠습니다.

#### 함수의 부수 효과를 최대한 억제하라

- 함수의 부수 효과 (side effect)란 함수 내의 작동으로 인해 함수가 아닌 외부에 영향을 끼치는 것을 의미합니다.
  - 이러한 부수 효과가 없는 함수를 순수 함수라고 하고, 부수 효과가 존재하는 함수를 비순수 함수라고 합니다.
- 즉, 함수는 부수 효과가 없으며, 언제 어디서나 어떠한 상황에서든 동일한 인수를 받으면, 동일한 결과를 반환해야 합니다.
- 그리고 이러한 동작 와중에 외부에 어떠한 영향도 미쳐서는 안됩니다.

##### 그렇다면 어떻게 해서든 항상 순수 함수로만 작성해야 할까요?

- 사이드 이펙트는 웹 애플리케이션 개발에 있어 피할 수 없는 요소중 하나입니다.
- 피할 수 없지만 이러한 사이드 이펙트를 최대한 억제할 수 있는 방향으로 함수를 설계해야 합니다.
- 리액트의 관점에서 본다면 부수 효과를 처리하는 훅인 useEffect의 작동을 최소화하는 것이 그 일환이라고 볼 수 있습니다.
  - useEffect의 사용은 피할 수 없지만 최소한으로 줄임으로써 함수의 역할을 좁히고, 버그를 줄이며, 컴포넌트의 안정성을 높일 수 있습니다.

> 따라서 자바스크립트 함수에서는 가능한 한 부수 효과를 최소화하고, 함수의 실행 결과를 최대한 예측 가능하도록 설계해야 합니다.

### 가능한 함수를 작게 만들어라

### 누구나 이해할 수 있는 이름을 붙여라

- 꿀팁! Terser를 사용하면 한글로 변수명이나 함수명을 작성해도 최종 결과물에는 크게 문제가 없다!
  > 다들 한글 변수명 자주 사용하시나요?
- 또한 리액트에서 useEffect혹은 useCallback등의 훅에서 넘겨주는 콜백 함수에 네이밍을 붙여준다면 가독성에 도움이 됩니다.
  ```ts
  useEffect(function apiRequest() {
    // 이거 꿀팁인듯요!
  });
  ```

## 01.3. 클래스

### 클래스란 무엇인가 띠용

- 자바스크립트의 클래스란 특정 객체를 만들기 위한 일종의 템플릿과 같은 개념으로 볼 수 있습니다.
  - 즉, 특정한 형태의 객체를 반복적으로 만들기 위해 사용되는 것이 바로 클래스입니다.

#### constructor

- constructor는 생성자로, 이름에서 알 수 있는 것처럼 객체를 생성하는 데 사용하는 특수한 메서드 입니다. 최초에 생성할 때 어떤 인수를 받을지 결정할 수 있으며, 객체를 초기화하는 용도로도 사용합니다.
- 단 하나만 존재할 수 있으며, 생략 또한 가능합니다.

#### 프로퍼티

- 프로퍼티란 클래스로 인스턴스를 생성할 때 내부에 정의할 수 있는 속성값을 의미합니다.

  ```ts
  class Car {
    constructor(name) {
      // 값을 받으면 내부에 프로퍼티로 할당됩니다.
      this.name = name;
    }
  }

  const myCar = new Car("자동차");
  ```

  - 기본적으로 인스턴스 생성 시 constructor 내부에는 빈 객체가 할당되어 있는데, 바로 이 빈 객체에 프로퍼티의 키와 값을 넣어서 활용할 수 있게 도와줍니다.

#### getter와 setter

- getter란 클래스에서 무언가 값을 가져올 때 사용됩니다. getter를 사용하기 위해서는 get을 앞에 붙여야 하고, 뒤이어서 getter의 이름을 선언해야 합니다.

  ```ts
  class Car {
    constructor(name) {
      this.name = name;
    }

    // Getter - 메서드처럼 동작하지만 속성처럼 접근
    get firstCharacter() {
      return this.name[0];
    }
  }

  const myCar = new Car("자동차");
  myCar.firstCharacter; // '자'
  ```

- 반대로 setter란 클래스 필드에 값을 할당할 때 사용합니다. 마찬가지고 set이라는 키워드를 먼저 선언하고 그 뒤를 이어서 이름을 붙이면 됩니다.

  ```ts
  class Car {
    constructor(name) {
      this.name = name;
    }

    // Getter - 첫 글자 반환
    get firstCharacter() {
      return this.name[0];
    }

    // Setter - 첫 글자 변경
    set firstCharacter(char) {
      this.name = [char, ...this.name.slice(1)].join("");
    }
  }

  const myCar = new Car("자동차");

  myCar.firstCharacter; // '자'

  // '자'를 '차'로 변경하다.
  myCar.firstCharacter = "차";

  console.log(myCar.firstCharacter, myCar.name); // '차', '차동차'
  ```

#### 인스턴스 메서드

- 클래스 내부에서 선언한 메서드를 인스턴스 메서드라고 합니다.
- 인스턴스 메서드는 실제로 자바스크립트의 prototype에 선언되므로 프로토타입 메서드로 불리기도합니다.
- prototype에 선언된다는 의미가 무엇인지 다음 예시를 통해 확인해보겠습니다.

  ```ts
  class Car {
    constructor(name) {
      this.name = name;
    }

    // 인스턴스 메서드 정의
    hello() {
      console.log(`안녕하세요, ${this.name}입니다.`);
    }
  }

  const myCar = new Car("테슬라");
  myCar.hello(); // 안녕하세요, 테슬라입니다.
  ```

  - 직접 객체에서 선언하지 않았음에도 프로토타입에 있는 메서드를 찾아 실행을 도와주는 것을 프로토타입 체이닝이라고 합니다.
  - 모든 객체는 프로토타입을 가지고 있는데, 특정 속성을 찾을 때 자기 자신부터 시작해서 프로토타입을 타고 최상위 객체인 Object까지 훑습니다.
  - 결론적으로 프로토타입과 프로토타입 체이닝 특성 덕분에 생성한 객체에서도 직접 선언하지 않은, 클래스에 선언한 메서드를 호출할 수 있고, 메서드 내부에서 this도 접근해 사용할 수 있게 됩니다.

#### 정적 메서드

- 정적 메서드는 특이하게 클래스의 인스턴스가 아닌 이름으로 호출할 수 있는 메서드입니다.

  ```ts
  class Car {
    static hello() {
      console.log("안녕하세요!");
    }
  }

  const myCar = new Car();

  myCar.hello(); // TypeError: myCar.hello is not a function

  Car.hello(); // 안녕하세요!
  ```

  - 정적 메서드 내부의 this는 클래스로 생성된 인스턴스가 아닌, 클래스 자신을 가리키기 때문에 다른 메서드에서 일반적으로 사용하는 this를 사용할 수 없습니다.
    - 이러한 이유로 리액트 클래스형 컴포넌트 생명주기 메서드인 `static getDerivedStateFromProps(props, state)`에서는 this.state에 접근할 수 없습니다.
  - 정적 메서드는 비록 this에 접근할 수 없지만 인스턴스를 생성하지 않아도 사용 가능하며, 생성하지 않아도 접근할 수 있기 때문에 객체를 생성하지 않아도 여러 곳에서 재사용이 가능하다는 장점이 있습니다.

#### 상속

- 생략

### 클래스와 함수의 관계

- 클래스가 작동하는 방식은 자바스크립트의 프로토타입을 활용하는 것이라고 볼 수 있습니다.

### 정리

- 클래스를 이해하면 클래스형 컴포넌트에서 어떻게 생명주기를 구현할 수 있었는지, 왜 클래스형 컴포넌트 생성을 위해 React.Component나 React.PureComponent를 상속하는지, 메서드가 화살표 함수와 일반 함수일 때 어떤 차이가 있는지 등을 이해할 수 있을것 입니다.

## 01.4. 클로저

- 리액트에서 함수형 컴포넌트에 대한 이해는 클로저에 달려있습니다.
- 함수형 컴포넌트의 구조와 작동 방식, 혹의 원리, 의존성 배열 등 함수형 컴포넌트의 대부분의 기술이 모두 클로저에 의존하고 있기 때문에 함수형 컴포넌트 작성을 위해서는 클로저에 대해 이해하는 것이 필수 입니다.

### 클로저 정의

- 클로저란 함수와 함수가 선언된 렉시컬 스코프의 조합 입니다.

### 클로저의 활용

- 리액트에서는 리액트가 관리하는 내부 상태 값을 리액트가 별도로 관리하는 클로저 내부에서만 접근 가능합니다.
- 클로저를 사용하면 전역 스코프의 사용을 막고, 개발자가 원하는 정보만 개발자가 의도한 방향으로 노출시킬 수 있다는 장점이 있습니다.
- 리액트에서는 useState의 변수를 저장해두고, useState의 변수 접근 및 수정 또한 클로저 내부에서 확인이 가능하여 값이 변하면 렌더링 함수를 호출하는 등의 작업이 이루어 집니다.

#### 리액트에서의 클로저

- 링액트 함수형 컴포넌트의 훅에서 클로저는 어떻게 사용될까요?
- 클로저의 원리를 사용하고 있는 대표적인 것 중 하나가 바로 useState 입니다.

  ```tsx
  function Component() {
    const [state, setState] = useState(0);

    function handleClick() {
      // useState 호출은 위에서 끝났고,
      // setState는 게속 내부의 최신값(prev)을 알고 있다.
      // 이는 클로저를 활용했기 때문에 가능하다.
      setState((prev) => prev + 1);
    }

    return <button onClick={handleClick}>증가</button>;
  }
  ```

  - useState 함수의 호출은 Component 내부의 첫 줄에서 종료되었는대, setState는 useState 내부의 최신 값을 어떻게 확인할 수 있을까요?
    - 외부함수(useState)가 반환환 내부함수(setState)는 외부함수의 호출이 끝났음에도 자신이 선언된 외부함수가 선언된 환경을 기억하기 때문에 계속해서 state값을 사용할 수 있는것 입니다.
