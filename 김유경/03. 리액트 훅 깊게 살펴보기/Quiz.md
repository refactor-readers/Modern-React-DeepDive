### useState를 활용하지 않고 함수 자체적으로 변수를 통해 상태를 관리할 때 동작하지 않는 이유를 리액트의 렌더링 발생 기준으로 설명해주세요.

```tsx
// 동작하지 않음
function Component() {
  let stater = "hello";

  function handleButtonClick() {
    state = "hi";
  }

  return (
    <>
            <h1>{state}</h1>     {" "}
      <button onClick={handleButtonClick}>hi</button>   {" "}
    </>
  );
}
```

리액트의 함수형 컴포넌트는 **"값이 바뀌었다고 해서 자동으로 렌더링되지 않는다"**

렌더링을 발생시키는 공식적인 트리거는 두 가지이다.

1. state 업데이트 (setState 호출)
2. props 변경
      즉, 리액트는 컴포넌트 내부의 일반 변수 변화는 감지하지 못한다.

### 내부 변수 변경은 렌더링을 트리거하지 않는다

```tsx
function Component() {
  let stater = "hello";

  function handleClick() {
    stater = "hi";
  }

  return (
    <>
            <h1>{stater}</h1>      <button onClick={handleClick}>change</button>
    </>
  );
}
```

- 버튼을 눌러도 UI가 변하지 않는 이유
    - stater = 'hi'. 는 단순한 지역 변수 재할당이라, 리액트는 이 변화를 관찰하지도, 저장하지도 않는다.
    - 렌더링을 다시 실행해야만 새로운 값이 화면에 보이는데 내부 변수 변경은 렌더링을 발생시키지 않는다

**리액트는 아무고토 몰라효**

### 리렌더링 시 컴포넌트 함수는 다시 실행된다

-> 내부 변수는 다시 초기화된다.

=> 리액트에서 렌더링이 발생하면 함수형 컴포넌트는 전체가 다시 호출된다.

```tsx
function Component() {
  let stater = "hello"; // 렌더링 때마다 항상 초기화
}
```

따라서 강제 렌더링으로 렌더링이 일어난다고 해도

- stater은 다시 'hello'로 초기화되어서, 이전에 hi를 넣어봤자 저장되지 않는다
- 내부 변수는 렌더링 사이에서 유지되지 않기 때문에 *상태(state)*로 쓸 수 없다.

## useState는 왜 값이 유지될까?

- 클로저 + 리액트가 가진 별도의 저장소(memoizedState) 덕분

React는 컴포넌트 외부(파이버 구조)에 state 저장소를 따로 두고 각 useState 호출 순서에 따라 값을 저장한다.

리렌더링 시에도 컴포넌트를 재실행하지만

- state는 함수 내부 변수로 저장되지 않고, React에서 관리하는 memoizedState 링크드 리스트에서 값을 다시 받아온다
    => 그래서 컴포넌트가 다시 실행돼도 값을 유지하는 것
