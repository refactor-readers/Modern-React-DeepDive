useState 관련해서 클로저와 호출 순서 보장의 중요성에 대해서 알아보다가 문제를 한 개 더 내보았습니다.

```js
// MyReact 내부의 상태는 렌더링이 끝나도 유지됨
// global.states= []
// index = 0

function Component() {
  // 첫 번째 훅
  const [valueA, setValueA] = useState("A");

  // 조건부 훅
  if (someCondition) {
    const [valueB, setValueB] = useState("B");
  }

  // 세 번째 훅
  const [valueC, setValueC] = useState("C");
}
```

#### 1단계 : 최초 렌더링 (`someCondition = true`)

1. `useState('A')` 실행 -> `global.states[0]`에 'A' 저장 -> `index = 1`
2. `useState('B')` 실행 -> `global.states[1]`에 'B' 저장 -> `index = 2`
3. `useState('C')` 실행 -> `global.states[2]`에 'C' 저장 -> `index = 3`
   => `global.state`는 `['A', 'B', 'C']` 입니다. (정상)

#### 2단계 : 다음 렌더링 (`someCondition = false`)

valueA를 업데이트하는 setValueA 를 호출하여 리렌더링이 시작되었고, 이번에는 someCondition이 false가 되었습니다.

1. `useState('A')` 실행
   - 현재 index는 0 입니다
   - `global.states[0]`에 저장된 A를 가져와 `valueA`로 변환합니다 (정상)
   - index 는 1이 됩니다.
2. 조건문이 거짓이므로, 두 번째 훅(`valueB`)은 건너뛰어집니다.
3. `useState('C')` 실행
   - 현재 index는 1 입니다
   - 이 훅은 `global.states[1]`에 접근하여 상태값을 가져오려고 합니다.

### Q. 이 시점에서 valueC에 할당되는 값은 무엇이 되며, 왜 문제가 발생했다고 할 수 있을까요?
