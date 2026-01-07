상태 관리 라이브러리를 거의 필수적으로 사용하고 계실텐데요, 여러분들은 몇 프로 정도 활용하고 있다고 생각하시나요?

저는 80프로 정도 이해하고 사용하고 있는 것 같아요

20프로 깎인 이유는, 이전에 생각 없이 습관적으로 사용했던 부분이 있구나 하고 반성하고 있기 때문입니다.
그래서 사용하던 일부 로직은 상태 관리 라이브러리를 조금 덜어내려고 합니다.

# 5.1 상태 관리는 왜 필요한가?

상태(State)는 애플리케이션의 시나리오에 따라 지속적으로 변경될 수 있는 값을 의미하며, 현대 웹에서는 관리해야 할 상태가 폭발적으로 증가

- 상태의 분류
  - UI: 다크/라이트 모드, 알림창 노출 여부, 각종 입력값 등
  - URL : 브라우저 라우팅에 따른 파라미터 상태
  - 폼(Form) : 로딩, 제출 여부, 유효성 등
  - 서버 데이터: API 요청을 통해 가져온 값

## 5.1.1 리액트 상태 관리의 역사

리액트는 UI 라이브러리일 뿐, 상태 관리 방식을 강제하지 않아 다양한 시도가 이어져 옴

- Flux 패턴의 등장 (2014년)

  - 기존 MVC 패턴의 양방향 데이터 바인딩이 주는 복잡성을 해결하기 위해 단방향 데이터 흐름을 제안
  - 구조 : Action -> Dispatcher -> Store -> View

- 리덕스(Redux)의 시장 지배

  - Flux 패턴에 Elm 아키텍처(Model, Update, View)를 도입
  - 하나의 전역 상태 객체(Store)를 두어 Props Drilling 문제를 해결했으나, 보일러플레이트가 많다는 단점이 있음

- Context API와 useContext (v.16.8+)

  - 상태를 하위 컴포넌트에 주입하는 표준 방식을 제공하여 간단한 전역 상태 공유가 가능해짐

- Hooks의 탄생과 React Query/SWR
  - `useState`, `useReducer`의 등장으로 로직 재사용이 쉬워졌으며, HTTP 요청 상태 관리에 특화된 라이브러리들이 주목받기 시작

## 5.1.2 지역 상태의 한계를 벗어나보자: 외부 스토어 모델

`useState`는 해당 컴포넌트 내부에서만 유효한 지역 상태. 이를 컴포넌트 외부 자바스크립트 스코프에 두고 공유하는 방식이 현대 라이브러리의 핵심이다

### 소스코드로 보는 모델

#### VanillaJS 스토어 모델

컴포넌트 외부에서 상태를 관리하고, 변경 시 등록된 콜백(구독자)들에게 알리는 기본 구조

```tsx
// 컴포넌트 외부의 스토어 생성 함수 (Zustand 등 라이브러리의 모태)
export const createStore = (initialState) => {
  let state = initialState;
  const callbacks = new Set(); // 상태 변경 시 실행할 리스너들

  const get = () => state;
  const set = (nextState) => {
    state = typeof nextState === "function" ? nextState(state) : nextState;
    callbacks.forEach((cb) => cb()); // 모든 구독자에게 알림
  };
  const subscribe = (cb) => {
    callbacks.add(cb);
    return () => callbacks.delete(cb); // 구독 해제 로직
  };

  return { get, set, subscribe };
};
```

#### useStore를 이용한 리액트 컴포넌트와의 연결

외부 스토어의 변화를 리액트의 리렌더링으로 연결해주는 커스텀 훅

❓Quiz. 리액트의 리렌더링이 언제 일어나나요?

> 반복해서 기억할 필요가 있을 것 같아 막간 퀴즈 넣었습니다

```tsx
function useStore(store) {
  // 외부 스토어의 값을 리액트 로컬 상태에 동기화
  const [state, setState] = useState(() => store.get());

  useEffect(() => {
    // 스토어가 변경될 때마다 리액트의 setState를 호출하여 리렌더링 유도
    const unsubscribe = store.subscribe(() => {
      setState(store.get());
    });
    return unsubscribe;
  }, [store]);

  return state;
}
```

### 연관 개념 퀴즈

#### 자바스크립트 관련

❓Quiz. 외부 스토어가 상태를 유지하고 은닉할 수 있는 자바스크립트의 기본 원리는 무엇일까요?

❓Quiz. Reducer나 상태 업데이트 시 기존 객체를 수정하지 않고 새 객체를 반환해야 하는 이유는 무엇일까요?

❓Quiz. `subscribe`를 통해 상태 변화를 통지 받는 방식의 디자인 패턴은 무엇일까요?

#### 리액트 동작 원래 관련

❓Quiz. 하나의 상태에 대해 서로 다른 결과물이 사용자에게 보여지는 UI 불일치 현상을 뭐라고 부를까요?

❓Quiz. 리액트 18에서 도입된 훅으로, 외부 스토어를 안전하게 구독하여 테어링 현상을 방지하는 훅이 무엇일까요?

❓Quiz. 상태를 필요한 곳까지 전달하기 위해 중간 컴포넌트들을 거쳐야하는 문제를 뭐라고 하나요? 이런 문제가 상태 관리 라이브러리의 개념과 어떤 관련이 있나요?

❓Quiz. 스토어에서 관리하는 상태가 객체일 때, 필요한 부분만 업데이트 하여 관련 없는 상태 변화에는 리렌더링이 발생하지 않도록 제어하는 기법이 무엇인가요?

# 5.2 상태 관리 라이브러리 Recoil, Jotai, Zustand 살펴보기

## 5.2.1 페이스북이 만든 상태 관리 라이브러리 : Recoil

Recoil은 리액트 훅의 개념을 상태 관리로 확장하여, 리액트스러운(React-ish) 방식으로 전역 상태를 관리

- RecoilRoot: 리액트의 Context를 활용해 스토어를 생성하며, 상태를 전파하는 `notifyComponents`를 통해 하위 컴포넌트 리렌더링 유발
- Atom: 상태의 최소 단위로, 고유한 `key` 필수
- Selector : 하나 이상의 Atom 값을 바탕으로 새로운 값을 조립하거나 비동기 작업을 처리하는 파생 상태(Derived State)를 만듬

```ts
// Recoil의 Atom 및 Selector 예시
const counterAtom = atom({
  key: "counter", // 필수 식별자
  default: 0,
});

const isEvenSelector = selector({
  key: "isEven",
  get: ({ get }) => {
    const count = get(counterAtom);
    return count % 2 === 0;
  },
});
```

## 5.2.2 리덕스를 대체할 작고 강한 라이브러리 : Jotai

Jotai는 Recoil에서 영감을 받았지만, `key`를 직접 관리해야 하는 번거로움을 해결하고 메모리 효율을 높인 라이브러리

- Bottom-up 접근 : 작은 단위의 Atom을 쌓아 올려 전체 상태를 구성
- Key 생략: 자바스크립트의 `WeakMap`을 활용해 객체 참조 자체를 키로 사용하므로, 별도의 고유 키를 문자열로 선언할 필요 없음
- Provider-less: Context 없이도 기본 스토어를 루트에 생성하여 사용 가능

## 5.2.3 작고 빠르며 확장성이 뛰어난 : Zustand

Zustand는 Flux 패턴을 따르면서도 보일러플레이트를 파격적으로 줄인 라이브러리

- 간결한 API : 리덕스와 유사하게 스토어를 중심으로 움직이지만, 하나의 함수(`create`)로 상태 정의와 업데이트 로직을 한 번에 처리
- 외부 스토어 모델 : 내부적으로는 리액트에 의존하지 않는 바닐라 자바스크립트로 스토어를 만들고, 리액트에서는 이를 구도(Subscribe)하는 방식 사용
- usSyncExternalStore: 리액트 18의 기능을 활용해 외부 상태를 리액트의 렌더링 시스템과 안전하게 동기화

```tsx
// Zustand 스토어 생성 및 사용 예시 [cite: 275]
const useStore = create((set) => ({
  count: 0,
  increase: () => set((state) => ({ count: state.count + 1 })),
}));

function Counter() {
  const count = useStore((state) => state.count); // Selector 활용
  return <h1>{count}</h1>;
}
```

### 연관 개념 퀴즈

#### 자바스크립트 관련

❓Quiz. Jotai가 별도의 키 없이 상태를 관리할 수 있게 해주는 핵심 자료구조는 무엇일까요? 이 자료 구조를 사용했을 때의 이점도 설명해주세요

❓Quiz. Zustand나 Recoil의 Selector가 리렌더링 여부를 결정할 때 사용하는 기본 원리가 무엇일까요?

#### 리액트 동작 원리 관련

❓Quiz. Context API vs Global Store의 차이점은 무엇일까요?

> Recoil은 내부적으로 Context를 쓰고, Zustand는 리액트 외부의 변수를 구독하는 방식을 취합니다

❓Quiz. 상태 원본을 가공하여 보여주는 개념으로, 불필요한 중복 상태 선언을 줄이고 데이터 무결성을 유지하는 중요한 기법은 무엇일까요?
