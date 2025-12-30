# 4장. 서버 사이드 렌더링

## 4.1 서버 사이드 렌더링이란?

### 싱글 페이지 애플리케이션의 세상(SPA)

html코드를 서버에서 렌더링 하지 않고 자바스크립트로 작성하는 형식


자바스크립트 모듈화 + 사용자 기기 성능의 향상 + 인터넷 속도발달

3가지의 결과로 다양한 부문에서 자바스크립트를 사용하게됨

LAMP -> JAM(js, api, markup)으로 넘어감

MERN(mongoDB, express.js, react, node.js)도 있음

위에 언급했듯 사용자의 하드웨어성능과 인터넷 속도의 발달로 리소스부담을 일정량 사용자에게 전가해도 되는 시대가 되었지만
js로 처리해야하는 코드의 양이 증가함에 따라 컨텐츠 로딩 속도가 날이 갈수록 늘어 날 수 밖에없는 리스크가 존재하게 되었다

장점
- UI/UX가 좋다(화면깜빡임 적음)
- 서버 부담이 적다

단점
- 최초 렌더링이 느리다
- SEO최적화가 힘들다

### 서버 사이드 렌더링이란?

최초에 사용자에게 보여줄 페이지를 서버에서 렌더링해 빠르게 사용자에게 화면을 제공하는 방식을 의미한다

즉, 렌더링의 부담을 사용자의 기기가 아닌 서버가 부담한다

장점
- 최초 페이지 진입이 빠르다
- SEO최적화에 유리하다
- 레이아웃 시프트가 적다
- 사용자 디바이스의 성능에 무관하다
- 보안이 안전하다

단점
- 서버를 고려해야한다
- 적절한 스케일의 서버가 구축되어있어야 한다
- 지연이 발생하면 유저에게 정보제공이 힘들다

### SPA와 SSR을 모두 알아야 하는 이유

- 가장 뛰어난 싱글 페이지 애플리케이션은 멀티페이지 애플리케이션보다 낫다
- 평균적인 싱글페이지 애플리케이션은 평균적인 멀티 페이지 애플리케이션보다 느리다
  - 멀티페이지 어플리캐이션에서 발생하는 라우팅으로 인한 문제해결을 위한 다양한 브라우저api가 있다
    - 페인트 홀딩(paint holding): 같은 출처에서 라우팅이 일어날 경우 화면을 잠깐 하얗게 띄우는 대신 이전 페이지의 모습을 잠깐 보여주는 기법
    - back forward cache(bfcache): 브라우저 앞으로 가기, 뒤로가기 실행 시 캐시된 페이지를 보여주는 기법
    - shared element transitions: 페이지 라우팅이 일어났을 때 두페이지에 동일 요소가 있다면 해당 콘텍스트를 유지해 부드럽게 전환되게 하는 기법

즉, 둘 다 장단점이 있고 어느 하나가 완벽하다고 볼 수 없다.

요즘엔 최초 진입시에는 SSR로, 이후에는 SPA처럼 동작하게해서 더 나은 경험을 안겨준다

## 4.2 서버 사이드 렌더링을 위한 리액트 API 살펴보기

### renderToString

인수로 넘겨받은 컴포넌트를 렌더링해 HTML문자열로 반환하는 함수다

```js
import ReactDOMServer from 'react-dom/server'

function ChildrenComponent({ fruits }: { fruits: Array<string> }) {
  useEffect(() => {
    console.log(fruits)
  }, [fruits])

  function handleClick() {
    console.log('hello')
  }

  return (
    <ul>
      {fruits.map((fruit) => (
        <li key={fruit} onClick={handleClick}>
          {fruit}
        </li>
      ))}
    </ul>
  )
}

function SampleComponent() {
  return (
    <>
      <div>hello</div>
      <ChildrenComponent fruits={['apple', 'banana', 'peach']} />
    </>
  )
}

const result = ReactDOMServer.renderToString(
  React.createElement('div', { id: 'root' }, <SampleComponent />),
)
```

결과는 이러하다

```js
<div id="root" data-reactroot="">
  <div>hello</div>
  <ul>
    <li>app1e</li>
    <li>banana</li>
    <li>peach</li>
  </ul>
</div>
```

특징은 useEffect나 handleClick같은 이벤트 핸들러는 결과물에 포함되지 않았다는 것이다
이는 의도된 것으로 빠르게 브라우저가 렌더링할 수 있는 HTML을 제공하는 데 목적이 있는 함수일 뿐이기 때문이다

마지막으로 div#root에 존재하는 속성인 data-reactroot다. 이 속성은 컴포넌트의 root엘리먼트가 뭔지 식별하는 역할을 한다. 이 속성은 이후에 js를 실행하기 위한hydrate함수에서 루트를 식별하는 기준점이 된다. 리엑트로 만들어진 애플리케이션을 보면 리액트의 루트 엘리먼트에 data-reactroot속성이 있는 것을 확인할 수 있다.

### renderToStaticMarkup

renderToString과 동일하지만 리액트 관련코드인 data-reactroot가 사라진 완전 순수한 HTML 문자열이 반환된다는 것을 확인할 수 있다.

이처럼 정적인 HTML을 만들때만 사용된다.

### renderToNodeStream

renderToString과 동일한 결과물을 만들지만 2가지 차이점이 있다
- 브라우저 환경에서도 사용가능한 renderToString와 다르게 node.js환경에서만 사용할 수 있다
- 결과물이 문자열인 renderToString과 다르게 Node.js의 readableStream타입이 결과물이다

renderToNodeStream은 데이터를 스트림으로 처리해야할 때 주로 사용하는데
> 스트림은 큰 데이터를 다룰 때 데이터를 청크(작은 단위)로 분할해 조금씩 가져오는 방식을 의미한다

대부분 널리 알려진 SSR프레임워크는 모두 renderToString대신 renderToNodeStream을 채택하고 있다.

### renderToStaticNodeStream

마찬가지로renderToNodeStream과 결과물을 동일하지만 리액트 속성이 없는 순수 HTML을 만들어낸다

### hydrate

앞에 살펴본 renderToString과 renderToNodeStream으로 생성된 HTML콘텐츠에 js핸들러나 이벤트를 붙이는 역할을 하는 함수이다

```js
import * as ReactDOM from 'react-dom'
import App from './App'

// containerId를 가리키는 element는 서버에서 렌더링된 HTML의 특정 위치를 의미한다.
const element = document.getElementById(containerId)

// 해당 element를 기준으로 리액트 이벤트 핸들러를 붙인다.
ReactDOM.hydrate(<App />, element)
```
비슷하게 동작하지만 클라이언트에서만 실행되는 render함수와 차이점은 hydrate는 기본적으로 이미 렌더링된 HTML이 있다는 가정하에 작업이 수행되고, 이 렌더링된 HTML을 기준으로 이벤트를 붙이는 작업만 실행한다는 것이다

두번째 인수로 리액트 관련 정보가 없는 순수한 HTML정보를 넘기면

```	Warning:	Expected server HTML to contain	a	matching```

경고가 발생한다

이렇게 되면 정상적으로 웹페이지를 만들긴 하지만 하이드레이션 과정의 렌더링 불일치 (Mismatch)가 발생하며 불일치가 발생해도 리액트는 강제로 렌더링을 수행하여 웹페이지를 정상적으로 만듭니다. 하지만 이는 사실상 두 번 렌더링하는 것과 같아 서버 사이드 렌더링의 장점을 포기하는 행위이므로 반드시 고쳐야 할 문제이다

### 정리

직접 SSR을 구현하긴 복잡하니 react팀에서도 권장하는 적절한 프레임워크를 사용하도록 하자

## 4.3 Next.js 톺아보기

Vercel이라는 미국 스타트업에서 만들었으며 리액트 기반 SSR프레임워크다 PHP에 영감을 받아 만들어졌고 실제로 PHP의 대용품으로 사용되기 위해 만들었다고 한다

### next.config.js

Next.js프로젝트의 환경 설정을 담당하는 파일이다
```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
}

module.exports = nextConfig
```
여기에 swcMinify는 바벨에 대안으로 rust로 작성되어서 바벨보다 빠르기에 사용을 권장한다

### pages/_app.tsx

_app.tsx, 그리고 내부에 있는 default export로 내보낸 함수는 전체 페이지의 시작점이다
이러한 특징 때문에 공통설정을 보통 여기에서 실행한다
- 에러 바운더리를 사용해 전역에서 발생하는 에러 처리
- reset.css같은 전역 css선언
- 모든 페이지에 공통으로 사용 또는 제공해야 하는 데이터 제공 등

_app.tsx의 render()내부에 콘솔을 찍으면 브라우저 개발자도구 콘솔이 아닌 next.js의 터미널에 기록되는걸 볼 수 있다

### pages/_document.tsx

_app.tsx가 애플리케이션 페이지 전체를 초기화하는 곳이라면, _document.tsx는 애플리케이션의 HTML을 초기화하는 곳이다 차이점이라면

- ```<html>```이나 ```<body>```에 DOM 속성을 추가하고 싶다면 _document.tsx를 사용한다.

- _app.tsx는 렌더링이나 라우팅에 따라 서버나 클라이언트에서 실행될 수 있지만 _document는 무조건 서버에서 실행된다. 따라서 이 파일에서 onClick과 같은 이벤트 핸들러를 추가하는 것은 불가능하다. 이벤트를 추가하는 것은 클라이언트에서 실행되는 hydrate의 몫이기 때문이다.

- Next.js에는 두 가지 ```<head>```가 존재하는데 하나는 next/document에서 제공하는 head이고, 다른 하나는 next/head에서 기본적으로 제공하는 head가 있다. 이름에서 알 수 있듯이 브라우저의 ```<head/>```와 동일한 역할을 하지만 next/document는 오직 document.tsx에서만 사용할 수 있다. next/head는 페이지에서 사용할 수 있으며, SEO에 필요한 정보나 title 등을 담을 수 있다. 또한 next/document의 ```<Head/>``` 내부에서는 ```<title/>```을 사용할 수 없다. 만약 이 태그를 사용하면 @next/next/no-title-in-document-head라는 경고가 발생한다. 웹 애플리케이션에 공통적인 제목이 필요하면 _app.tsx에, 페이지별 제목이 필요하다면 페이지 파일 내부에서 후자를 사용하면 된다.

- 이후에 설명할 getServerSideProps, getStaticProps 등 서버에서 사용 가능한 데이터 불러오기 함수는 여기에서 사용할 수 없다.

_document.tsx에서만 할 수 있는 작업은 CSS-in-JS(emotion, styled-component)의 스타일을 서버에서 모아 HTML로 제공하는 작업이다.

요약하자면  _app.tsx는 next.js를 초기화하는 파일이고 _document.tsx는 뼈대가 되는 HTML설정과 관련된 코드를 추가하는 곳이다

### pages/_error.tsx

500에러를 처리하는 파일이다

### pages/404.tsx

404 페이지를 정의하는 파일이다

### pages/500.tsx

서버에서 발생하는 에러를 핸들링하는 페이지이다 _error.tsx와 함께 있다면 500.tsx가 우선적용된다

### pages/index.tsx

next.js에서 제공하는 예약어로 관리되는 페이지라면 지금부터는 개발자가 자유롭게 명칭을 지정해 만들 수 있는 페이지다.

- /pages/index.tsx: 웹사이트의 루트이며, localhost:3000과 같은 루트 주소를 의미한다.

- /pages/hello.tsx: /pages가 생략되고 파일명이 주소가 된다. 즉, 여기서는 /hello이며 localhost:3000/hello로 접근할 수 있다.

- /pages/hello/world.tsx: localhost:3000/hello/world로 접근 가능하다. 디렉터리의 깊이만큼 주소를 설정할 수 있다.
  - 주의사항: hello/index.tsx와 hello.tsx 모두 같은 주소를 바라보므로 필요에 따라 선택해서 사용한다.

- /pages/hello/[greeting].tsx: []의 의미는 여기에 어떠한 문자도 올 수 있다는 뜻이다.
  - 사용자가 접속한 주소명이 greeting이라는 변수에 담겨 서버 사이드로 전달된다.
  - 예를 들어 localhost:3000/hello/1, localhost:3000/hello/greeting 모두 유효하며, 변수 값은 각각 1, greeting이 된다.
  - 이미 정의된 주소(예: /pages/hello/world.tsx)가 있다면 미리 정의된 주소가 우선한다.

- /pages/hi/[...props].tsx: 자바스크립트의 전개 연산자와 동일하게 작동한다.
/hi 하위의 모든 주소가 여기로 온다. (예: localhost:3000/hi/hello/world/foo 등)
이때 [...props] 값은 props라는 이름의 변수에 배열 형태로 전달된다.

```js
const { query: { props } } = useRouter()
```
이와 같이 []안에 값을 추출할 수 있다

- /hi/1 → ['1']
- /hi/1/2 → ['1', '2']
- /hi/1/2/3 → ['1', '2', '3']
- /hi/my/name/is → ['my', 'name', 'is']

### 서버 라우팅과 클라이언트 라우팅의 차이

```<a>``` vs ```<Link>```

- ```<a>```: 링크 이동시 필요한 모든 리소스를 다시 받아온다
  - 서버에서 렌더링을 수행하고, 클라이언트에서 hydrate하는 과정에서 한 번 더 실행된다

- ```<Link>```: 페이지 이동에 필요한 리소스만 받아온다
  - 서버 사이드 렌더링이 아닌, 클라이언트에서 필요한 js만 불러온 뒤 라우팅하는 클라이언트 라우팅/렌더링 방식으로 작동한다

그로인해 next.js의 장점을 살리기 위해 다음과 같은 규칙을 지켜야한다

- ```<a>```대신 ```<Link>```를 사용한다
- window.location.push 대신 router.push를 사용한다.

### getServerSideProps를 제거하면 어떻게 될까

제거하면 페이지는 Static으로 취급되어 빌드시 미리 트리쉐이킹이 일어난다

즉, 제거하면 ssr로 작동하지 않는다

### pages/api/hello.ts

다른 pages파일과 다르게 HTML요청을 하는게 아니라 단순 서버 요청을 주고받게 된다

당연히 서버에서만 실행되고 BFF형태로 활용하거나 완전한 풀스택 애플리케이션일때 사용한다 혹은 CORS를 우회하고자 할때 사용한다

### DATA Fetching
예약어로 지정되어 반드시 정해진 함수명으로 export를 사용해 SSR를 구현할 수 있다

### getstaticPaths와 getStaticProps

이 두 함수는 어떠한 페이지를 정적인 페이지로 보여주고자 할 때 사용된다

```js
import { GetStaticPaths, GetStaticProps } from 'next'

// 빌드 시 생성할 경로를 정의합니다.
export const getStaticPaths: GetStaticPaths = async () => {
  return {
    paths: [{ params: { id: '1' } }, { params: { id: '2' } }],
    fallback: false,
  }
}

// 빌드 시 데이터를 불러옵니다.
export const getStaticProps: GetStaticProps = async ({ params }) => {
  const { id } = params
  const post = await fetchPost(id)

  return {
    props: { post },
  }
}

// 불러온 데이터를 사용하여 페이지를 렌더링합니다.
export default function Post({ post }: { post: Post }) {
  // post로 페이지를 렌더링한다.
}
```

- getstaticPaths는 /pages/post/[id]가 접근 가능한 주소를 정의하는 함수도 paths를 배열로 반환한다

- getStaticProps는 앞에서 정의한 페이지를 기준으로 요청이 오면 제공할 props를 반환하는 함수다

- post는 getStaticProps가 반환한 post를 렌더링한다

이처럼 정적인 페이지는 빌드시 사용자는 미리 다 완성돼 있는 페이지를 받기만 하면 되기 때문에 매우 빠르다

### getServerSideProps

앞서 두 함수는 정적인 페이지 제공이였다면 getServerSideProps는 서버에서 실행되는 함수다
페이지 진입시 무조건 실행하며 props를 반환할 수도, 다른 페이지로 리다이렉트시킬 수도 있다.

단, getServerSideProps에서는 반드리 JSON.stringify로 직렬화할 수 있는 값만 제공해야 하고, 값에 대해 가공이 필요하다면 실제 페이지나 컴포넌트에서 하는것이 옳다

getServerSideProps에서는 다음과 같은 제약이 있다

- window.document와 같이 브라우저에서만 접근할 수 있는 객체에는 접근할 수 없다
- API호출시 /api/some/path와 같이 protocol과 domain 없이 fetch요청을 할 수 없다. 브라우저와 다르게 서버는 자신의 호스트를 유추할 수 없기 때문이다. 반드시 완전한 주소를 제공해야 fetch가 가능하다
- 여기에서 에러가 발생한다면 500.tsx와 같이 미리 정의해 둔 에러 페이지로 리다이렉트된다

### 스타일 적용하기

#### 전역 스타일
CSS Reset이라 불리는, 이른바 브라우저에서 기본으로 제공되고 있는 스타일을 초기화하는 등 공동으로 적용하고 싶은 스타일이 있다면 _app.tsx를 활용하면 된다

#### 컴포넌트 레벨 CSS
[name].module.css와 같은 명명 규칙만 준수하면 컴포넌트 레벨의 CSS를 추가할 수 있다

```css
.alert {
  color: red;
  font-size: 16px;
}
```
```js
import styles from './Button.module.css'

export function Button() {
  return (
    <button type="button" className={styles.alert}>
      경고!
    </button>
  )
}
```

위와같이 적용하면 실제 프로덕션 빌드시에 스타일 태그가 아닌 별도 css파일로 생성되어 적용된다

#### SCSS와 SASS
sass와 scss도 css를 사용했을 때와 마찬가지로 동일한 방식으로 사용할 수 있다. sass 패키지를 npm install --save-dev sass와 같은 명령어로 설치하면 별도의 설정 없이 바로 동일하게 스타일을 사용할 수 있다. scss에서 제공하는 variable을 컴포넌트에서 사용하고 싶다면 export 문법을 사용하면 된다.

```js
// primary 변수에 blue라는 값을 넣었다.
$primary: blue;

:export {
  primary: $primary;
}

import styles from "./Button.module.scss";

export function Button() {
  return (
    {/* styles.primary 형태로 꺼내올 수 있다. */}
    <span style={{color: styles.primary}}>
      안녕하세요
    </span>
  );
}

```

#### CSS-in-JS
최근에는 js내부에 스타일시트를 삽입하는 CSS-in-JS방식의 스타일링이 각광받고 있다
대표적으로는 Emotion, styled-component등이 있다

사용하려면 _document.tsx에 특정 코드를 넣어줘야 하는데 

리액트 트리 내부에서 사용하고 있는 styled-components의 스타일을 모두 모은 다음, 이 각각의 스타일에 유니크한 클래스명을 부여해 스타일이 충돌하지 않게 클래스명과 스타일을 정리해 이를 _document.tsx가 서버에서 렌더링할 때 React.Context 형태로 제공하는 것이다. 이렇게 CSS-in-JS의 스타일을 서버에서 미리 모은 다음 서버 사이드 렌더링에서 한꺼번에 제공해야 올바른 스타일을 적용할 수 있다. 만약 이런 과정을 거치지 않는다면 스타일이 브라우저에서 뒤늦게 추가되어 **FOUC(flash of unstyled content)**라는, 스타일이 입혀지지 않은 날것의 HTML을 잠시 사용자에게 노출하게 된다. 이는 다른 CSS-in-JS도 마찬가지로, 코드는 약간씩 다르지만 모두 서버에서 스타일을 모아 서버 사이드 렌더링 시에 일괄 적용하는 과정은 동일하게 거치게 된다. 따라서 CSS-in-JS를 Next.js와 같은 서버 사이드 렌더링 프레임워크에서 사용할 때는 반드시 이런 초기화 과정을 서버에서 거쳐야 한다.


#### next.config.js 살펴보기

[공식링크](https://nextjs.org/docs/13/app/api-reference/next-config-js)
