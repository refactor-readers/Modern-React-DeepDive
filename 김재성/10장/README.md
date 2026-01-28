# 10장 리액트 17과 18의 변경 사항 살펴보기

## 10.1 리액트 17버전 살펴보기

점진적인 업그레이드를 지원하기 위해 일부 컴포넌트 변경이 리액트 17의 업데이트 주요 변경 사항중 하나다

만약 애플리케이션이 너무 거대해서 한번에 버전업이 부담스러운 상황이라면 16과17을 함께 사용하면서 점진적으로 업그레이드하는 시나리오도 고려해볼 수 있다

### 이벤트 위임 방식의 변경

리액트는 이벤트 핸들러를 해당 이벤트 핸들러를 추가한 각각의 DOM 요소에 부탁하는 것이 아니라, 이벤트 타입당 하나의 핸들러를 루트에 부착한다. 이를 이벤트 위임이라고한다

1. 캡쳐: 트리 최상단부터 내려가는것
2. 타깃: 타깃노드에 도달하는단계
3. 버블링: 이벤트 발생 요소부터 최상위 까지 올라가는것

리액트 17부터는 이러한 이벤트 위임이 모두 document가 아닌 이랙트 컴포넌트 최상단 트리 즉, 루트 요소로 바뀌였다 (점진적 업그레이드를 지원하기 위함)

### import React from 'react'가 더이상 필요 없다: 새로운 JSX transform

17부터 바벨과 협력해 이러한 import 구문 없이도 JSX를 변환할 수 있게 됐다

이는 불필요한 import구문을 삭제해 번들링 크기를 줄이는 효과과 컴포넌트 작성을 더욱 간결하게 해준다는 장점이 있다

### 그외 변경사항

- 이벤트 풀링 제거
- useEffect 클린업 함수의 비동기 실행
- 컴포넌트의 undefined 반환에 대한 일관적인 처리

## 10.2 리액트 18버전 살펴보기

### 새로 추가된 훅 살펴보기

- useId
  - 고유한 id생성
- useTransition
  - UI변경을 가로막지 않고 상태를 업데이트할 수 있는 훅
  - 동시성을 다룰 수 있음
- useDeferredValue
  - 컴포넌트 트리에서 리렌더링이 급하지 않는 부분을 지연할 수 있게 도와주는 훅
  - 디바운스와 유사함
- useSyncExternalStore
  - UI가 찢어지는 테어링(tearing)현상을 해결하기 위한 훅
- useInsertionEffect
  - useSyncExternalStore가 상태 관리 라이브러리를 위한 훅이라면 useInsertionEffect는 CSS-in-js라이브러리를 위한 훅이다

### react-dom/client

#### createRoot
기존의 render를 대체할 새로운 메서드다.

#### hydrateRoot
SSR에서 하이드레이션을 하기 위한 메서드다

### react-dom/server

#### renderToPipeableStream
리액트 컴포넌트를 HTML로 렌더링하는 메서드다 이름처럼 스트림을 지원하는 메서드로 점진적 렌더링중 script를 삽입하는 등의 작업을 할 수 있다

#### renderToReadableStream
renderToPipeableStream이 Node.js환경에서 렌더링을 위해 사용된다면 이건 웹 스트림을 기반으로 작동한다

### 자동 배치
리액트가 여러 상태 업데이트를 하나의 리렌더링으로 묶어서 성능을 향상시키는 방법이다

### 더욱 엄격해진 엄격 모드

- 더이상 안전하지 않은 특정 생명주기를 사용하는 컴포넌트에 대한 경고
- 문자열 ref 사용 금지
- findDOMNode에 대한 경고 출력
- 구 Context API 사용 시 발생하는 경고
- 예상치 못한 부작용(side-effects)검사

### Suspense 기능 강화

- 아직 마운트되기 직전임에도 effect가 빠르게 실행되는 문제가 수정됐다. 이제 컴포넌트가 실제로 화면에 노출될 때 effect가 실행된다.

- Suspense로 인해 컴포넌트가 보이거나 사라질 때도 effect가 정상적으로 실행된다. 이전에는 컴포넌트 스스로가 Suspense에 의해 현재 보여지고 있는지, 숨겨져 있는지 알 방법이 없었다. 그러나 이제 Suspense에 의해 노출이 된다면 useLayoutEffect의 effect(componentDidMount)가, 가려진다면 useLayoutEffect의 cleanUp(componentWillUnmount)이 정상적으로 실행된다.

- Suspense를 이제 서버에서도 실행할 수 있게 된다. 앞의 예제와 같이 CustomSuspense를 구현하지 않더라도 정상적으로 Suspense를 사용할 수 있다. 서버에서는 일단 fallback 상태의 트리를 클라이언트에 제공하고, 불러올 준비가 된다면 자연스럽게 렌더링된다.

- Suspense 내에 스로틀링이 추가됐다. 화면이 너무 자주 업데이트되어 시각적으로 방해받는 것을 방지하기 위해 리액트는 다음 렌더링을 보여주기 전에 잠시 대기한다. 즉, 중첩된 Suspense의 fallback이 있다면 자동으로 스로틀되어 최대한 자연스럽게 보여주기 위해 노력한다.

### IE지원 중단에 따른 추가 폴리필 필요
pass