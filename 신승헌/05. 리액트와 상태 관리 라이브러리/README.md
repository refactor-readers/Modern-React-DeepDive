# 5.1 상태 관리는 왜 필요한가?

### 웹 애플리케이션에서의 “상태”

- 어떠한 의미를 지닌 값
- 시나리오에 따라 지속적으로 변경될 수 있는 값
    1. **UI**
    : 다크/라이트, 노출 여부, hover/active/disabled 등
    2. **URL**
    : pathname, query params 등
    3. **Form**
    : isLoading, isSubmitting, disabled, validation 등
    4. **Data**
    : 서버에서 가져온 값, API 요청 등

> 💡 Tearing
하나의 상태에 따라 서로 다른 결과물을 보여주는 현상
상태 변화에 따라 모든 요소들이 변경되어 애플리케이션이 찢어지는 걸 말함
> 

## 5.1.1 리액트 상태 관리의 역사

**리액트에서 상태를 관리하는 방법은 개발자나 시간(시대)에 따라 많은 차이가 있다.**

- 리액트는 단순히 사용자 인터페이스를 만들기 위한 라이브러리일 뿐
- 그 이상의 기능을 제공하지 않는다

### Flux 패턴

MVC 패턴의 경우 모델과 뷰가 많아질수록 복잡도나 관리 난이도가 기하급수적으로 증가했음

⇒ 단방향 데이터 흐름을 채택한 상태 관리 방식

1. **액션(action)**
: 어떠한 작업을 처리할지와 작업에 포함시킬 데이터를 나타냄
이를 각각 정의해 디스패처로 보냄
2. **디스패처(Dispatcher)**
: 액션을 스토어에 보내는 역할
콜백 함수 형태로 앞서 액션이 정의한 타입, 데이터를 스토어로 보냄
3. **스토어(Store)**
: 실제 상태에 따른 값과 상태를 변경하는 메서드를 가짐
액션 타입에  따라 어떻게 변경할지 정의되어 있음
4. **뷰(View)**
: 리액트 컴포넌트에 해당하는 부분
스토어의 데이터를 가져와 화면에 보여주는 역할
사용자의 입력이나 행위에 따라 액션을 호출할 수 있음

### Reducer를 활용한 Flux 패턴

```tsx
type StoreState = {
	count: nuber
}

type Action = { type: 'add'; payload: number }

function reducer(prevState: StoreState, action: Action) {
	const { type; ActionType } = action;
	if (ActionType === 'add') {
		return {
			count: prevState.count + action.payload,
		}
	}
	
	throw new Error('Unexpected Action');
}

export default function App() {
	const [state, dispatcher] = useReducer(reducer, { count: 0 });
	
	function handleClick() {
		dispatcher({ type: 'add', payload: 1 });
	}
	
	return <button type='button' onClick={handleClick}>{state.count}</button>
}
```

1. 액션 ⇒ Action 타입 객체 `{ type: 'add', payload: 1 }`
2. 디스패처 ⇒ dispatcher 함수
3. 스토어 ⇒ reducer 함수 & state
4. 뷰 ⇒ button

### 리덕스의 등장!

- 최초에는 Flux 구조를 구현하기 위해 만들어짐
- Elm 아키텍처(model, view, update)를 도입, 단방향 데이터 흐름을 만듦
- 하나의 상태 객체를 스토어에 저장해두고, 객체를 업데이트 하는 작업을 디스패치 함
- connect를 통해 스토어에 접근할 수 있음
- 상태 객체를 전파할 수 있어서, props drilling으로 발생하는 문제를 해결함
- 보일러플레이트가 너무 많음 ⇒ 하나의 상태 변경에 따라오는 작업이 너무 많음 ⇒ 최근 많이 간소화됨

### Elm?

- 웹페이지를 선언적으로 작성하기 위한 언어
- model, update, view로 구성됨
    1. 모델
    : 애플리케이션의 상태를 의미함
    2. 뷰
    : 모델을 표현하는 HTML
    3. 업데이트
    : 모델을 수정하는 방식

### Context API와 useContext

- props drilling이 과도하게 발생하는 가독성, 유지보수성에 대한 문제를 개선하기 위해 출시됨(리액트 16.3)
- 전역 상태를 Context Provider를 통해 주입하는 형태
- 16.3 버전 이전에도 context는 존재했지만 단점이 몇 가지 있었음
    1. context를 불러오는 메서드(`getChildContext()`) 호출로 인한 불필요한 자식 컴포넌트 리렌더링
    2. context를 prop으로 전달해야 하고, 이로 인해 결합도가 높아짐

**Context API**는 상태 관리가 아닌 **“상태 주입”**을 도와주는 기능임을 기억하자!

### React Query와 SWR

- 훅과 state의 등장으로 state와 이를 변경하는 로직을 분리하고, 재사용할 수 있게되었고
이를 배경으로 React Query와 SWR 등이 등장함
- React Query나 SWR은 **HTTP요청에 특화된 상태 관리 라이브러리**라고 볼 수 있다.

### Recoil, Zustand, Jotai, Valtio, …

- 범용적으로 쓸 수 있는 상태 관리 라이브러리
- 훅을 활용해 상태를 가져오거나 관리할 수 있는 라이브러리
- Redux는 runtime에 모든 상태를 하나의 Store객체에 넣어서 관리하지만
- 위 상태관리 라이브러리들은 구독중인 store, atom 등의 변화에 따라서 리렌더링이 발생함
접근하는 페이지에서 사용하는 스토어만 메모리에 올림
- 렌더링 사이클이 훨씬 가벼워지고 효율적임

## 5.1.2 정리

> 어떤 상황에 무엇을 써야할지 결정할 수 있는게 중요할 것 같아요
> 

> 🙋‍♂️ 전역상태 라이브러리를 도입해본 경험과, 해당 라이브러리를 선정하게 되신 과정이 궁금해요.
> 

---

# 5.2 리액트 훅으로 시작하는 상태관리

리액트 훅의 등장과 함수형 컴포넌트의 패러타임에서 상태관리를 어떻게 해야할까

## 5.2.1 가장 기본: useState, useReducer

- useState, useReducer는 지역 상태 관리에 사용됨
- 동일한 로직을 훅으로 격리해서, 같은 내용(시나리오)을 재사용할 수 있도록 함
- 하지만! 훅을 사용할 때 마다 컴포넌트별로 초기화되어, 상태가 파편화 됨 → 지역 상태(local state)는 해당 컴포넌트 내에서만 유효함
- 부모 컴포넌트로 상태를 끌어올려 prop을 통해 전달해서, 상위 단위(전역)의 상태를 관리할 수 있음

## 5.2.2 지역 상태의 한계 돌파

> useState로 생성한 상태를 어딘가 멀리 떨어진 컴포넌트에서 관리해야 한다면..?
> 

외부 상태 참조, 리렌더링 발생에 필요한 조건

1. 컴포넌트 외부(window, global, 외부 어딘가..)에 상태를 두고 여러 컴포넌트가 공유
2. 상태를 사용하는 모든 컴포넌트가 외부 상태의 변화를 알아챌 수 있어야 함
    1. 최신 상태값 기준 리렌더링
3. 상태가 객체인 경우엔 참조하지 않는 값이 변하면 리렌더링이 발생하지 않아야 함

```tsx
// store 값이 단순 원시값인 경우 예시
type Initializer<T> = T extends any ? T | ((prev: T) => T) : never;

type Store<State> = {
  get: () => State; // 항상 최신값을 반환하기 위해 함수로 사용
  set: (action: Initializer<State>) => State;
  subscribe: (callback: () => void) => () => void;
};

export const createStore = <State extends unknown>( 
	initialState: Initializer<State>
): Store<State> => {
  let state =
    typeof initialState !== "function" ? initialState : initialState();
  // 게으른 초기화 가능

  const callbacks = new Set<() => void>(); // 중복 방지
  const get = () => state; // 최신값 반환을 위해 함수 형태로 작성
  const set = (nextState: State | ((prev: State) => State)) => {
    // 인수가 함수라면 함수를 실행해 새로운 값을 받고,
    // 아니라면 새로운 값을 그대로 사용
    state =
      typeof nextState === "function"
        ? (nextState as (prev: State) => State)(state)
        : nextState;

    // 값의 설정이 발생하면 콜백 목록을 순회하면서 모든 콜백을 실행
    callbacks.forEach((callback) => callback());

    return state;
  };

  const subscribe = (callback: () => void) => {
    // 받은 함수를 콜백 목록에 추가
    callbacks.add(callback);
    // 클린업
    return () => {
      callbacks.delete(callback);
    };
  };

  return { get, set, subscribe };
};

export const useStore = <State extends unknown>(store: Store<State>) => {
	// useState가 컴포넌트의 렌더링을 유도함
  const [state, setState] = useState<State>(() => store.get());
  
  useEffect(() => {
	  // createStore 내부의 값이 변경되는 경우, subscribe에 등록된 콜백이 실행됨
	  // store 내부의 값이 변경될 때 마다 현재의 state값이 변경되는 것을 보장받음
    const unsubscribe = store.subscribe(() => {
      setState(store.get());
    });
    return () => unsubscribe();  // 클린업
  }, [store]);
  
  return [state, store.set] as const;
};
```

## 5.2.3 useState와 Context를 동시에 사용해보기

- 훅을 사용하는 서로 다른 스코프에서, 동일한 스토어 구조로, 서로 다른 데이터를 사용해야 하는 경우
    1. 스토어를 여러개 만들기 → 필요할 때마다 스토어, 훅 생성
    2. Context로 스토어를 하위 컴포넌트에 주입
- 가장 가까운 Provider를 참조하는 특성(Shadowing)을 활용해, Provider로 자식 컴포넌트를 감싸서 스토어를 격리할 수 있음
- 각 Provider마다 별개의 스토어가 생성되고 관리됨
- 주의점⚠️ 각 Provider의 initialState(value)에 새로운 객체를 넣어줘야 함, 외부에서 기존에 생성된 스토어 객체를 그대로 넣어주면, 다 같은 값을 공유함

## 5.2.4 Recoil, Jotai, Zustand 살펴보기

- Recoil과 Jotai는 Context와 Provider, 훅을 기반으로 가능한 작은 상태를 효율적으로 관리하는데 초점
- Zustand는 리덕스와 비스무리, 하나의 큰 스토어를 기반으로 상태를 관리
    - 스토어가 가지는 클로저를 기반으로 생성됨
    - 상태가 변경되면 상태를 구독하고 있는 컴포넌트에 전파해서 리렌더링을 알림
- 더 알고싶으면 문서를 보자~

## Recoil

- 페이스북에서 만듬(React를 위해 만들어진 라이브러리)
- Atom 개념을 처음 선보임
- Bottom-up 방식, 컴포넌트들이 필요한 Atom만 골라서 구독하느 ㄴ방식
- Suspense 등 최신 기능과 통합이 자연스러움
- 상태 간 의존성이 복잡하거나 파생데이터가 많을 때, 리액트 최신 기능 활용이 필요할 때
- 복잡한 인터랙션 관리가 필요한 앱에 많이 씀
- 아직 정식 출시 이전, 놀랍게도 여전히 0.7.7 버전이 최신
- 프로덕션 환경에 사용하기에 위험이 있음

### Recoil의 Atom?

- 데이터를 잘게 쪼개서 관리함, 필요한 데이터만 선택적해서 사용이 가능함
- key(식별자), default(초기값)을 반드시 필요로 함
- 컴포넌트에서는 `useRecoilstate()`같은 Recoil에서 제공하는 훅으로 Atom을 사용할 수 있음

```tsx
import { atom } from 'recoil';

export const isDarkModeState = atom({
  key: 'isDarkModeState', // 고유한 ID (유니크해야 함)
  default: false,         // 초기값
});
```

## Jotai

- Recoil에 영감을 받아 만들어짐(Bottom-up)
- Atomic 패턴을 가져오고, 더 가볍게 만든 라이브러리
- Context의 문제점인 불필요한 리렌더링을 해결하기 위해 설계됨
- Next.js랑 호환성이 좋음
- 인터랙션이 많은 그래픽.. 대시보드.. 등 앱에서 많이 사용함
- Form 관리에도 편리함

### Jotai의 Atom

- 개념적인 원리는 Recoil에서 가져왔지만 조금 다름
- 생성 시 key가 필요 없음
- Provider 없이도 사용할 수 있고, Jotai의 Provider를 사용하면 마찬가지로 지역상태로 관리도 가능함
- `useAtomValue()`를 사용해 최신 값을 가져올 수 있음

## Zustand

- 간소화 된 Flux 패턴
- 리덕스의 장점을 가져오고, 복잡함을 조금 낮춘 형태
- Provider 필요 없음, 단순한 보일러플레이트..
- 번들사이즈 매우 작음
- 빠르고 간편한, 직관적인 전역 상태 세팅에 유리함

## 5.2.5 정리

현재 상황에 맞는 라이브러리를 적절하게 선택하되, 관리가 잘 되고 있는 라이브러리를 선택하는것도 중요하다!

---