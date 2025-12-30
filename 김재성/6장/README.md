# 6장. 리액트 개발 도구로 디버깅하기

## 6.1 리액트 개발 도구란?

리액트 팀은 개발도구인 react-dev-tools를 만들어 제공하고 있다. 이 개발 도구는 리액트로 만들어진 다양한 애플리케이션을 디버깅하기 위해 만들어졌으며, 리액트웹 분만 아니라 리액트 네이티브 등 다양한 플레폼에서 사용할 수 있다.

## 6.2 리액트 개발 도구 설치

웹환경은 브라우저 확장도구로 설치해야한다

## 6.3 리액트 개발 도구 활용하기

리액트 개발 도구가 설치되었다면 크롬 개발자 도구에 Components와 profiler가 추가된것을 볼 수 있다

### 컴포넌트

현재 리액트 애플리케이션의 컴포넌트 트리를 확인할 수 있다. 단순히 컴포넌트의 구조뿐만 아니라 props와 내부 hooks등 다양한 정보를 확인할 수 있다.

![컴포넌트 트리](/김재성/6장/image1.png "컴포넌트 트리")

기명 함수로 선언되어 컴포넌트명을 알 수 있다면 해당 컴포넌트명을 보여주고, 만약 익명 함수로 선언돼 있다면 Anonymous라는 이름으로 컴포넌트를 보여준다

```js
// App.tsx
import AnonymousDefaultComponent from './Component3'

function Component1() { return <>Component1</> }
const Component2 = () => { return <>Component2</> }
const MemoizedComponent = memo(() => <>MemoizedComponent</>)

// 고차 컴포넌트 (HOC)
const withSampleHOC = (Component: ComponentType) => {
  return function () {
    return <Component />
  }
}
const HOCComponent = withSampleHOC(() => <>HOCComponent</>)

export default function App() {
  return (
    <div className="App">
      <Component1 />
      <Component2 />
      <AnonymousDefaultComponent />
      <MemoizedComponent />
      <HOCComponent />
    </div>
  )
}

// Component3.tsx
export default () => { return <>Component3</> }
```

이 컴포넌트를 렌더링해서 리액트 개발 도구에서 확인하면 다음과 같다

![컴포넌트 트리 결과](/김재성/6장/image2.png "컴포넌트 트리 결과")

함수 선언식과 함수 표현식으로 생성한 컴포넌트는 모두 정삭적으로 함수명을 표시하고 있는 것을 확인할 수 있다. 그러나 함수 선언식과 함수 표현식으로 선언되지 않는 컴포넌트는 다음과 같은 문제를 확인할 수 있다

- 익명 함수를 default로 export한 AnonymousDefaultComponent의 경우 AnonymousDefaultComponent는 코드 내부에서 사용되는 이름일 뿐, 실제로 default export로 내보낸 함수의 명칭은 추론할 수 없다. 따라서 _default로 표시된다.

- memo를 사용해 익명 함수로 만든 컴포넌트를 감싼 경우, 함수명을 명확히 추론하지 못해서 Anonymous로 표시됐다. 추가로 memo 라벨을 통해 memo로 감싸진 컴포넌트임을 알 수 있다.

- 고차 컴포넌트인 withSampleHOC로 감싼 HOCComponent의 경우 두 가지 모두 Anonymous로 선언돼 있다. 이 또한 고차 컴포넌트의 명칭을 제대로 추론하지 못했기 때문이다.

```js
const MemoComponent = memo(funtion(){
  return <>memo</>
})

MemoComponent.displayName = '메모 컴포넌트입니다'
```
위와 같이 displayName을 설정하면 리액트 개발자 도구에서 컴포넌트명을 확인할 수 있다.

#### 컴포넌트명과 props

![컴포넌트명과 props](/김재성/6장/image3.png "컴포넌트명과 props")

1. 컴포넌트명과 key
    - 컴포넌트 명칭과 해당 컴포넌트를 나타낸다
2. 컴포넌트 도구
    - 첫번째 눈 아이콘을 누르면 크롬 개발자 도구처럼 해당 컴포넌트가 HTML의 어디에서 렌더링 됐는지 확인할 수 있다.
    - 두번째 벌레 아이콘은 해당 컴포넌트의 정보가 콘솔탭에 기록된다
    - 세번째 소스코드 아이콘을 누르면 해당 컴폰너트의 소스를 볼 수 있다
3. 컴포넌트 props
    - 해당 컴포넌트가 받은 props를 확인할 수 있다. 단순 원시값뿐만 아니라 함수도 포함돼 있다
4. 컴포넌트 hooks
    - 컴포넌트에서 사용중인 훅 정보를 확인할 수 있다
5. 컴포넌트를 렌더링한 주체, rendered by
    - 해당 컴포넌트를 렌더링한 주체가 누구인지 확인할 수 있다. 프로덕션 모드에서는 react-dom의 버전만 확인할 수 있지만 개발 모드에서는 해당 컴포넌트를 렌더링한 부모 컴포넌트까지 확인할 수 있다

### 프로파일러

컴포넌트 메뉴는 정적인 현재 리액트 컴포넌트 트리의 내용을 디버깅하기 위한 도구라면 프로파일러는 리액트가 렌더링하는 과정에서 발생하는 상황을 확인하기 위한 도구다.

#### 프로파일링

프로파일링을 시작하고 웹에서 테스트하면 정보들을 확인할 수 있다

- Flamegraph
  - 렌더 커밋별로 어떠한 작업이 일어났는지 나타낸다
- Ranked
  - 해당 커밋에서 렌더링하는 데 오랜 시간이 걸린 컴포넌트를 순서대로 나열한 그래프다

#### 타임라인

시간이 지남에 따라 컴포넌트에서 어떤 일이 일어났는지를 확인할 수 있다

### 정리

지금까지는 state변경을 최소 컴포넌트 단위로 분리하는 게 좋다는 사실을 이론적으로만 숙지하고 이해했지만 프로파일러 도구를 활용해 명확히 확인할 수 있게 됐다.

> 여러분은 리액트 개발자 도구를 적극 활용하고 계신가요?