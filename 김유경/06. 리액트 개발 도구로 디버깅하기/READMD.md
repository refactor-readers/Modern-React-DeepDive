# 6.1 리액트 개발 도구 설치

리액트 애플리케이션의 내부 구조를 파악하고 성능을 측정하기 위해서는 브라우저 확장 프로그램인 React Developer Tools가 필수적

## 6.1.1 설치

- 크롬, 파이어폭스, 엣지 등 주요 브라우저의 확장 프로그램 스토어에서 설치 가능
- 설치 후 개발자 도구(F12)를 열면 Components와 Profiler 탭이 추가되어 있음

# 6.2 리액트 개발 도구 활용하기

## 6.2.1 Components 탭

현재 리액트 페이지의 컴포넌트 트리 구조를 확인할 수 있음

- 컴포넌트 트리 : 단순한 DOM 구조가 아닌, 리액트 컴포넌트 단위의 계층 구조를 보여줌
- Props 및 State : 선택한 컴포넌트가 받은 props, 내부 state, 사용 중인 hooks, 그리고 context 정보를 실시간으로 확인하고 직접 수정하여 UI 변화를 테스트할 수 있음
- 함수/클래스 컴포넌트 식별 : `memo`나 `forwardRef`를 사용한 경우에도 원래의 컴포넌트 이름을 확인할 수 있도록 도와줌

## 6.2.2 Profiler 탭

리액트 애플리케이션이 렌더링되는 과정에서 발생하는 성능 병목을 분석하는 도구

- 녹화 기능 : 특정 조작을 하는 동안의 렌더링 과정을 기록
- Flamegraph(플레임그래프): 커밋별로 각 컴포넌트가 렌더링되는 데 걸린 시간을 막대 형태로 표시
  - 노란색에 가까울수록 렌더링 시간이 길었음을 의미
- Ranked(랭크드) : 해당 커밋에서 렌더링 시간이 오래 걸린 순서대로 컴포넌트를 나열하여 최적화 대상을 쉽게 찾게 해줌
- Interactions (실험적 기능) 어떤 사용자 이벤트가 렌더링을 유발했는지 추적

### displayName 설정

고차 컴포넌트(HOC)나 익명 함수를 사용할 때 개발 도구에서 컴포넌트 이름을 명확히 표기하기 위해 사용

```tsx
const MyComponent = React.memo(() => {
  return <div>내용</div>;
});

// 설정하지 않으면 개발 도구에 'Memo'로 표시될 수 있음
MyComponent.displayName = "CustomInputComponent";
```

### Profiler API (코드 수준의 측정)

개발 도구 없이 특정 컴포넌트의 트리 성능을 직접 프로그래밍적으로 측정하고 싶을 때 사용

```tsx
import { Profiler } from "react";

function onRenderCallback(
  id,
  phase,
  actualDuration,
  baseDuration,
  startTime,
  commitTime
) {
  // 렌더링 성능 데이터를 서버로 보내거나 로그를 남길 수 있음
  console.log(`${id} [${phase}]: ${actualDuration}ms`);
}

function App() {
  return (
    <Profiler id="AppRoot" onRender={onRenderCallback}>
      <MyLargeComponent />
    </Profiler>
  );
}
```

### Call Stack & Error Boundary : 에러 발생 시 리액트가 어떻게 호출 스택을 추적하고, `componentDidCatch` 등을 통해 에러를 격리할까요?

#### 1. 리액트의 에러 추적 : Call Stack과 Components Stack

리액트에서 에러가 발생하면 브라우저는 기본적으로 자바스크립트의 Call Stack(호출 스택)을 보여줍니다. 하지만 리액트 개발 도구는 여기서 한 걸음 더 나아가 Components Stack(컴포넌트 스택)을 제공합니다.

- Call Stack(JS 엔진 레벨) : 에러가 발생한 시점의 자바스크립트 함수 호출 경로. 난독화된 코드에서는 `a()` => `b()` => `c()` 처럼 파악하기 어렵게 표시될 수 있음
- Component Stack (리액트 레벨) : "어떤 컴포넌트 내부에서 에러가 시작되었는가" 를 보여줌
- 예 `<App> -> <Layout> -> <ProductList> -> <ProductItem>`
- 작동 원리 : 리액트는 개발 모드에서 렌더링 중에 각 컴포넌트의 경로를 추적하여 에러 발생 시 이를 콘솔에 함께 출력
  - 범인 찾기 가능

#### 2. `componentDidCatch`와 Error Boundary의 격리 원리

기존 자바스크립트의 `try-catch`는 명령형 코드(이벤트 핸들러 등)에는 작동하지만, 리액트의 선언적 렌더링 과정에서 발생하는 에러는 잡지 못합니다. 이를 해결하기 위해 등장한 것이 Error Boundary ! 두둥 !

1. 에러 격리 메커니즘
   Error Boundary가 없는 상태에서 자식 컴포넌트가 렌더링 중 에러를 던지면(Throw), 리액트는 전체 컴포넌트 트리를 Unmount(파괴)해 버립니다. 흰 화면(White-out)이 나오는 이유.....ㅠ
   Error Boundary는 이 에러가 상위로 전파되는 것을 중간에 낚아채서(캐치 !), 에러가 발생한 하위 트리만 숨기고 대신 미리 준비한 UI(Fallback)를 보여줌으로써 나머지 애플리케이션의 정상 동작을 보장합니다.

2. 핵심 메서드

- `static getDerivedStateFromError(error)`
  - 목적 : 에러 상태를 UI에 반영
  - 에러가 발생하면 `hasError : true` 같은 상태를 반환하여 다음 렌더링 때 Fallback UI를 보여주도록 합니다 (Render 단계에서 호출)
- `componentDidCatch(error, info)`
  - 목적 : 에러 로깅 및 부수 효과 처리
  - `info.componentStack`을 통해 위에서 설명한 컴포넌트 스택 정보를 얻을 수 있습니다. 이를 Sentry 같은 에러 트래킹 서비스로 전송하여 디버깅에 활용합니다. (Commit 단계에서 호출)

```tsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  // 1. 에러 발생 시 UI 상태 업데이트 (격리의 시작)
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  // 2. 에러 세부 정보 기록 (디버깅 정보 획득)
  componentDidCatch(error, info) {
    console.error("에러 발생 컴포넌트 스택:", info.componentStack);
    // logErrorToService(error, info);
  }

  render() {
    if (this.state.hasError) {
      // 에러 발생 시 보여줄 대체 UI
      return <h1>서비스 이용에 불편을 드려 죄송합니다.</h1>;
    }

    return this.props.children;
  }
}
```

### 연관 개념 퀴즈

#### 자바스크립트 관련

❓Quiz. 번들링 된 코드(`main.js`)에서 원본 소스 코드(`.ts`, `jsx`)를 찾아가며 디버깅할 수 있게 해주는 연결 고리는 무엇일까요?

#### 리액트 동작 원리 관련

❓Quiz. 프로파일러에서 측정하는 시간은 주로 렌더링 단계(재조정 과정)이며, 실제 DOM에 반영되는 커밋 단계와는 다릅니다 ! 이때 렌더링이 반영되는 두 개의 Phase가 있지요? 무엇일까요?
