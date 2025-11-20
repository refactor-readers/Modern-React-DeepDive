## Q.1 해당 코드에서 user.age가 변경되었을 때 일어날 수 있는 문제점과 이를 개선하기 위해 어떻게 변경해야하는지 말씀해주세요.

> 원시 타입과 객체 타입의 저장 방식과 렌더링 관점에서 설명해주세요.

```ts
import React, { useState } from "react";

type User = {
  name: string;
  age: number;
};

const UserInfo = React.memo(({ user }: { user: User }) => {
  console.log("UserInfo render");
  return (
    <div>
      {user.name} ({user.age})
    </div>
  );
});

export default function App() {
  const [user, setUser] = useState<User>({ name: "Yoo", age: 29 });

  const handleBirthday = () => {
    user.age += 1;
    setUser(user);
  };

  return (
    <div>
      <UserInfo user={user} />
      <button onClick={handleBirthday}>한 살 더 먹기</button>
    </div>
  );
}
```
