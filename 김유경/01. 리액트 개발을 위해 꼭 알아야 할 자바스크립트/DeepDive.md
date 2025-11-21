# 태스크 큐와 매크로 태스크 큐가 같은 것일까?

### 관련 용어 정리

| 용어            | 의미                                             |
| --------------- | ------------------------------------------------ |
| Task            | setTimeout, click 이벤트 등 큰 단위 실행 단위    |
| Macrotask       | Task와 동일한 의미로 사용됨                      |
| Task Queue      | 여러 종류의 Macrotask Queue 전체를 통칭하는 용어 |
| Microtask       | Promise 후속 처리 등 더 높은 우선순위의 작업     |
| Microtask Queue | microtask들을 저장하는 전용 큐 (딱 1개)          |

## Quiz1. 브라우저 이벤트 루프 관점에서 볼 때, 리액트의 상태 업데이트(setState)에 의한 리렌더링은 마이크로 태스크인가요? 매크로 태스크인가요?

> 재성님 PR의 퀴즈

### 질문을 보고 떠올린 답변

setState는 단순히 인자를 받아서 객체를 업데이트해주는 역할이라고 생각해서, 태스크 큐라고 생각했다.

### chatGPT와 함께 다이브

결론은 `setState` 호출 자체는 마이크로 태스크도, 매크로 태스크도 아니다.
하지만 React의 상태 업데이트 과정은

```md
"현재 실행 중인 JS 실행 컨텍스트가 완료된 뒤, React의 내부 스케줄러가 별도 큐를 통해 처리한다."
```

=> 이벤트루프의 micro/macro task큐와 직접적으로 1:1 대응되지 않는다.

- React가 자체적으로 만든 업데이트 큐 (**Concurrent Mode**에서는 **Fiber** 스케줄러)에 의해 처리되기 때문이다.

# 이벤트 루프 기본 정리

JS 런타임은 아래 순서로 동작한다.

1. 현재 실행 컨텍스트 실행
2. microtask queue 비우기
3. macrotask 1개 실행
4. 다시 microtask queue 비우기
5. 반복...

## Microtask 예시

- Promise.then / catch / finally
- MutationObserver
- queueMicrotask
- async/await의 후속 처리

## Macrotask 예시

- setTimeout, setInterval
- I/O
- DOM 이벤트 콜백
- messageChannel

### setSTate의 동작 과정

1. setSTate 호출 순간

- setState는 동기 함수 호출이다.
  - 즉, "바로 실행 컨텍스트 안에서 실행되는 일반 JS 함수"일 뿐이다.
  - 이 때 React는 "업데이트가 필요하다는 사실"만 업데이트 큐(Fiber 업데이트 큐)에 넣고 바로 렌더링을 실행하지 않는다.
    => 따라서 setState는 microtask도, macrotask도 아니다. 이벤트 루프 큐에 들어가지 않기 떄문이다.

#### 🤔 그렇다면 상태 업데이트는 언제 일어나는가?

```md
현재 실행 컨텍스트가 끝난 이후, React 스케줄러에 의해 비동기로 처리된다.
```

1. JS에서 setState를 호출한다. (동기)
2. React는 "업데이트 요청"을 자체 updateQueue에 넣는다.
3. 현재 실행 컨텍스특가 끝난 뒤(= 이벤트 루프가 한 틱 종료된 뒤)
4. React 스케줄러가 동작하면서 업데이트 큐를 하나씩 처리하고
5. Virtual DOM -> diffing -> commit 단계로 이어진다.

=> 이 과정은 microtask queue/ macrotask queue 어디에도 존재하지 않는다.
=> 대신 React가 자체 구현한 Fiber 스케줄러와 우선순위 기반 Task Queue에서 처리된다.

# 질문에 대한 총 정리

우선 처음에 내가 했던 생각이 크게 다르지는 않지만, 리액트 렌더링이 별도의 비동기 업데이트로 이루어진다는 사실을 처음 알았고, 다음 챕터를 진행하면서 이를 기억하면서 차근차근 더 딥다이브해봐도 좋을 것 같다고 생각했다.

## 정리

- setState 함수 호출 자체는 micro/macro task가 아니다.
- 이벤트 루프의 microtask queue나 macrotask queue에서 처리되지 않는다.
- React는 자체 스케줄러(Fiber)와 내부 update queue에서 비동기로 업데이트를 처리한다.
- **렌더링 역시 브라우저 task queue가 아니라 React 스케줄러에 의해 우선순위 기반으로 실행된다.**
- **setState의 비동기성 때문에 클로저 문제가 발생할 수 있으면 이를 함수형 업데이트로 해결한다.**

세 번째 네 번째 내용을 완전히 이해하려면 관련된 여러 개념들에 대한 이해가 더욱 필요할 것으로 보인다.
