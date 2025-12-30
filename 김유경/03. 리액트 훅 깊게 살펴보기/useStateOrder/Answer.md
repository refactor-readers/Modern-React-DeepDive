### 첫 답변

valueC에 할당되는 값은 global.states[1]인 'B'를 가져와 valueC로 반환합니다.

이 것은 올바른 state의 업데이트가 아닙니다. global.state[1]은 useState('B')에서 관리하던 state 이기 때문입니다.

### 보완 답변

index가 밀리면서 valueC는 이전 렌더링에서 valueB가 사용했던 값인 B를 가져오게 되고, 이는 완전히 잘못된 상태를 참조하게 만듭니다.

이러한 현상을 **인덱스 불일치(Index Mismatch)** 라고 부르며, 상태의 **일관성(Consistency)** 이 깨지는 순간이라고 할 수 있습니다.
