# 4.1 서버 사이드 렌더링이란?

## 4.1.1 싱글 페이지 애플리케이션의  세상

- 반대되는 개념, 서버 사이드 렌더링 ↔ 싱글 페이지 애플리케이션

### 싱글 페이지 애플리케이션(Single Page Application, SPA)

- 렌더링, 라우팅에 필요한 대부분 기능을 브라우저의 자바스크립트에 의존
- 첫 페이지에서 데이터(HTML, JS, CSS)를 모두 로드
- `history.pushState`, `history.replaceState`로 페이지 전환 작업

**⇒ 페이지 로드 이후 하나의 페이지에서 모든 작업을 처리함**

### 전통 애플리케이션 vs SPA

**네이버([홈](https://www.naver.com/) ↔ [스포츠](https://m.sports.naver.com/index))** VS **[GMAIL](https://mail.google.com/)**

- 서버 사이드에서 작동하던 기존의 방식은 **페이지 전환마다 새로운 HTML을 다운로드** 함
- SPA는 최초에 모든 리소스를 다운로드 하면, 이후 페이지 전환이 매끄러움

### 자바스크립트의 진화

- 보조적 수단 → 다양한 작업(복잡한 계산, 판단)으로 자바스크립트의 역할이 확장됨
- 코드 양이 많아지며 모듈화가 진행됨

**⇒ CommonJS, AMD(Asynchronous Module Definition)의 등장**

### 프레임워크의 등장

- 인터넷 환경, 기기 성능 발전으로 자바스크립트 영향력이 확장됨
- 자바스크립트 수준의 **MVx** 프레임워크 등장

**⇒ 다양한 자바스크립트 프레임워크, 라이브러리 등장**

### 싱글페이지 렌더링 방식의 유행

- 자바스크립트의 역할과 규모의 확장
- 페이지 렌더링에서부터 사용자 인터랙션까지 아우를 수 있는 싱글 페이지 렌더링이 인기를 얻게 됨

> 🤔 **CommonJS, AMD(Asynchronous Module Definition)**?
자바스크립트 파일을 어떻게 쪼개고(**Export**) 어떻게 불러올 것인지(**Import**)에 대한 서로 다른 해결책
가장 큰 차이는 **환경**과 **로딩 방식**에 있음

**1. CommonJS(CJS)**
Node.js를 위해서 태어난 놈. 자바스크립트를 브라우저 밖으로 꺼내려는 시도에서 시작됨.
서버 환경에서는 리소스와 가깝기 때문에 속도가 매우 빠름, 비동기가 필요 없음.
**동기적으로 실행**됨, 블로킹으로 브라우저에서는 못씀.
 `module.exports`, `require`를 사용함.

**2. AMD**
브라우저를 위한 놈. **비동기적 로딩**으로 동작함.
`define`, `require`를 사용함
Lazy Loading 지원, 브라우저 환경에서 성능이 좋음
문법이 지저분함(콜백지옥)

ESM 짱
> 

> 🤔 **MVx**?
MV..로 시작하는 디자인 패턴을 말함
데이터와 화면을 연결해주는 방식에 따라 MVC, MVP, MVVM으로 나뉨
M(Model): 데이터와 비즈니스 로직
V(View): 보이는 화면
…더보기
> 

### JAM 스택

- 기존의 LAMP 스택의 난이도로 인해 서버 확장이 어려웠음
*LAMP(Linux + Apache + MySQL + PHP)*
- 프레임워크의 등장으로 JAM 스택이 유행하게 됨
JAM(JS + API + Markup)
- 자바스크립트 역할 확대로 서버 부담이 줄어듦
- Express.js로 API도 자바스크립트로 구현하는 구조도 인기

### 웹 서비스의 발전

- 하드웨어 발전에 따라 자바스크립트 리소스도 커지면서 로딩 시간이 개선되지 못하고, 오히려 더 느려짐

---

## 4.1.2 서버 사이드 렌더링이란?

> 사용자에게 보여줄 페이지를 서버에서 렌더링해 빠르게 화면을 제공하는 방식
> 
- SPA의 한계를 개선하고자 사용됨
- 렌더링에 필요한 작업을 모두 서버에서 수행함

### SSR의 장점

1. 최초 페이지 진입이 빠름
    - FCP가 빨라짐
    - 서버에서 HTML을 문자열로 그려서 내려주는게 더 빠름
    - HTTP 요청이 많은 페이지거나 사이즈가 큰 경우 SSR이 유리함
2. 메타데이터 제공이 쉬움
    - 검색엔진 봇은 HTML을 다운로드 하고 자바스크립트는 실행하지 않음(정적 정보만 수집)
    - SPA의 경우 자바스크립트 의존성이 높아, 따로 조치해두지 않으면 메타정보 제공이 어려움
3. 누적 레이아웃 이동이 적음
    - 누적 레이아웃(CLS): 사이즈를 알 수 없는 요소로 인해 로딩 중 화면 덜그럭거림
    - 완성된 페이지를 제공하기 때문에 비교적 나음
4. 사용자 디바이스 성능에 비교적 자유로움
    - 자바스크립트 리소스는 디바이스에서만 실행됨
    - but, 인터넷 느리거나 트래픽으로 서버 부하가 늘어나면 느려지는건 똑같음
5. 보안에 좀 더 안전함
    - JAM 스택 애플리케이션은 모든 활동이 브라우저에 노출됨
    - API 호출, 인증같이 민감한 작업도 노출될 수 있음
    - 서버 사이드 렌더링은 민감한 작업을 서버에서 수행할 수 있음

### SSR의 단점

1. 서버 환경에 대한 고려
    - window, localStorage 등 브라우저 전역 객체에 접근이 불가하기 때문에
    서버사이드에서 실행되지 않도록 방지가 필요함
    - 그렇다고 클라이언트로 다 넘겨주는건 서버 사이드 렌더링 사용의 본질을 잃음
2. 적절한 서버 구축의 필요성
    - 렌더링을 수행할 적절한 사양의 서버 구축이 필요함
    - 서버에 대한 의존도가 늘어남
3. 서비스 지연에 따른 문제
    - 지연 작업에 대한 파급효과가 큼
    - 최초 로딩 중 지연 작업이 발생하면, 화면이 아예 보이지 않을 수 있음
    - 페이지 렌더링 작업이 끝나기 전에는 사용자에게 정보 제공이 어려움

---

## 4.1.3 SPA, SSR 모두 알아야 하는 이유

> Silver bullet …더보기
> 
- 서버로 모두 넘겨준다고 성능이 개선되는건 아니다
- 오히려 관리 포인트만 늘어날 수 있음

### 가장 뛰어난 SPA > 가장 뛰어난 MPA

- MPA는 아무리 최적화를 잘해도 페이지 이동 시 깜빡임은 불가피함
- SPA는 virtual DOM을 통한 부드러운 페이지 전환, 자연스러운 애니메이션, WebSocket 등으로 뛰어난 성능을 보여줄 수 있음 ⇒ 고점이 높음

### 평균 SPA < 평균 MPA

- MPA는 브라우저에 의존해 기본적인 기능은 균일한 성능을 보장함
- 최근 MPA의 라우팅 문제를 해결하기 위한 다양한 API가 브라우저에 추가됨
- SPA는 개발자가 자바스크립트로 통제해야 할 부분이 많음
- 번들사이즈 최적화, 브라우저와의 충돌, 메모리누수 등 고려해야할 부분이 많음

⇒ MPA는 고점 - 저점 간격이 작지만, SPA는 고점 - 저점 차이가 아주 큼

> 결론: 두 가지 모두 유효한 방법이므로, 잘 알아두어야 함
> 

### 현대의 서버 사이드 렌더링

- **LAMP에서의 서버 사이드 렌더링과는 조금 다름**
    - 모든 페이지를 서버에서 렌더링 함 → 초기 진입 빠름
    - 이후 라우팅 발생시에도 서버에 의존해야 함 → 라우팅이 느림
- **요즘 서버 사이드 렌더링은 LAMP와 SPA의 장점을 합침**
    - 최초 진입 시 SSR로 페이지를 제공하고
    - 이후 라우팅에서는 서버에서 내려받은 자바스크립트를 바탕으로 SPA처럼 동작함
    - *e.g. Next.js, Remix 등*

---

## 4.1.4 정리

- 좋은 사용자 경험을 위해서 두 가지 방법을 모두 이해해야 함
- 정답은 없음

---

# 4.2 SSR을 위한 React API

## 4.2.1 renderToString

- 인수로 넘겨받은 리액트 컴포넌트를 렌더링해 HTML 문자열로 반환 하는 함수
- SSR의 가장 기초적인 API
- 최초 페이지를 HTML로 렌더링할 때 사용됨
- 빠르게 브라우저가 렌더링할 수 있는 HTML 제공이 목적
- 자바스크립트 코드는 포함시키지 않음

### data-reactroot

- 리액트 컴포넌트 루트 엘리먼트를 식별하게 해주는 속성
- 하이드레이션에서 루트를 식별하는 기준점이 됨

---

## 4.2.2 renderToStaticMarkup

- renderToString과 유사하지만, 리액트 관련된 속성 없이 순수한 HTML 문자열을 반환함
- hydrate 없이 정적인 내용만 필요한 경우 유용함

---

## 4.2.3 renderToNodeStream

- renderToString과 동일한 결과물
- 차이점
    1. renderToStream은 브라우저에서 사용이 불가능함. (Node.js 환경에 의존)
    2. 문자열이 아닌 Node.js의 **ReadableStream**을 반환함

### ReadableStream

- utf-8로 인코딩 된 바이트 스트림
- Node.js 환경에서만 사용할 수 있음
- 브라우저에서도 사용할 수는 있지만, 만드는 과정이 브라우저에서 불가능함

### 스트림이란?

- 큰 데이터를 청크(chunk) 단위로 분할해 조금씩 가져오는 방식을 말함
- 큰 크기의 데이터를 한번에 응답하는건 서버 메모리에 부담

---

## 4.2.4 renderToStaticNodeStream

- hydrate 가 필요 없는 순수 HTML 결과물이 필요할 때 사용

---

## 4.2.5 hydrate

- renderToString, renderToNodeStream으로 생성된 HTML에 자바스크립트를 붙이는 역할
- 단순히 서버에서 렌더링한 HTML 결과물은 사용자와 인터랙션이 불가능 함
- 정적으로 생성된 HTML에 이벤트와 핸들러를 붙여서 결과물을 완성 함

### render 메서드

```tsx
import * as ReactDOM from 'react-dom'
import App from './App'

const rootElement = document.getElementById('root')

ReactDOM.render(<App />, rootElement)
```

- 컴포넌트와 HTML요소를 인수로 받음
- HTML 요소에 컴포넌트를 렌더링
- 이벤트 핸들러를 붙이는 작업까지 한번에 수행함
- 클라이언트에서만 실행되며 웹페이지 만드는데 필요한 작업을 모두 수행함

### hydrate

```tsx
import * as ReactDOM from 'react-dom'
import App from './App'

// containerId를 가리키는 element는 서버에서 렌더링된 HTML의 특정 위치를 의미
const element = document.getElementById(containerId)

// 해당 element를 기준으로 리액트 이벤트 핸들러를 붙임
ReactDOM.hydrate(<SomeComponent />, element)
```

- 컴포넌트와 HTML요소를 인수로 받는것은 동일함
- hydrate은 이미 렌더링된 HTML이 있는 것을 가정하고 작업이 수행됨
- 이벤트를 붙이는 작업만 함
- 리액트 관련 정보가 없는 순수한 HTML 정보를 넘겨주면, 경고 출력
    - renderToStaticMarkup 등으로 생성한 HTML
- 서버가 제공해준 HTML이 클라이언트의 결과물과 같을 것이라는 가정 하에 실행됨
- `element` 내부에는  `<SomeComponent />`가 이미 렌더링 되어있어야 함
- 경고가 떠도 리액트가 페이지를 만들어 주긴 함 → 서버/클라이언트에서 두 번 렌더링 하는 것 → 비효율
- 불가피하게 불일치가 발생하는 곳에는 `suppressHydrationWarning`속성을 추가해 경고를 끌 수 있음
    - 굳이 그럴바에 클라이언트에서 useEffect로 처리하는게 나을수도 있다..

### 정리

|  | **render()** | **hydrate()** |
| --- | --- | --- |
| **사용 환경** | **CSR (Client Side Rendering)** | **SSR (Server Side Rendering)** |
| **초기 상태** | **빈 껍데기** (`<div id="root"></div>`) | **내용이 꽉 찬 HTML** (서버가 보내줌) |
| **하는 일** | HTML 태그를 하나하나 **생성(Create)하고 배치함** | 이미 있는 HTML 태그에 **이벤트(클릭 등)만 연결(Attach)함** |
| **속도** | 초기 로딩이 느림 (화면이 늦게 뜸) | 초기 로딩이 빠름 (화면은 바로 뜸) |
| **부하** | 브라우저가 모든 걸 다 해야 함 | 브라우저는 '연결' 작업만 하면 됨 |

---

## 4.2.6 서버사이드 렌더링 예제

…

---

## 4.2.7 정리

- 서버사이드 렌더링 구현에는 아주 복잡한 코드가 필요함
- 리액트 팀에서 적절한 프레임워크 사용을 추천하는 이유가 있음
- 서버사이드 렌더링을 사용하려면 서버, 번들링 최적화, 캐싱 등 고려할 관리포인트가 많아지는 건 사실

---

# 4.3 Next.js

## 4.3.1 Next.js 란?

- Vercel에서 만든 리액트 기반 **서버 사이드 렌더링 프레임워크**
- PHP의 대용품으로 사용되기 위해 만들어짐

---

## 4.3.2 Next.js 시작하기

### package.json

- 프로젝트 구동에 필요한 모든 명령어, 의존성
- 프로젝트 대략적인 모습을 확인하는데 유용함

### next.config.js

- Next.js 프로젝트의 환경 설정을 담당
- `reactStrictMode`: 리액트의 Strict 모드와 관련된 옵션, 리액트 애플리케이션 내부 잠재적 문제를 알림
- `swcMinify`: SWC 기반 코드 최소화 작업 여부를 설정하는 속성, 바벨보다 훨씬 빠름
    - SWC는 Rust 기반 번들러/컴파일러로 한국인이 만들었다 ㄷㄷ

### _document.tsx

- Next.js 13 버전 이상(App Router)부터는 `_document.tsx`와 `_app.tsx`가 했던 일을 `app/layout.tsx` (Root Layout) 하나가 모두 통합해서 처리함

---

## 서버 라우팅 vs 클라이언트 라우팅

### **<a>**

- <a> 태그는 Full page reload
- 기존 페이지를 버리고, 새 페이지를 처음부터 다시 로드함

### **<Link>**

- <Link>는 Client side navigation
- 바뀌어야 할 부분만 가져와서 화면을 갈아끼움

### 결론

- <a> 대신 <Link> 사용하기
- `window.location.push` 대신 `router.push` 사용하기

---

## 4.3.3 Data Fetching

### getStaticPaths

- /pages/post[id] 형태로 접근 가능한 주소를 미리 정의할 수 있음

```tsx
export const getStaticPaths: GetStaticPaths = async () => {
	return {
		paths: [{ params: { id: "1" } }, { params: { id: "2" } }],
		fallback: false,
	}
}
// post/foo : Error!
```

- App Router에서는 `generateStaticParams` 함수 사용

```tsx
export async function generateStaticParams() {
  // API나 DB에서 데이터 가져오기
  const posts = await fetch('https://api.example.com/posts').then(res => res.json());

  // 반드시 객체의 배열 형태로 반환해야 함
  // !! 객체의 키 이름('id')은 폴더명 '[id]'와 똑같아야 함 !!
  return posts.map((post) => ({
    id: post.id.toString(), // 문자열로 변환
  }));
}

export default async function Page({ params }) {
  const { id } = params;
  // ... 
  return <h2>Post No. {id}</h2>;
}
```

### getStaticProps

- getStaticPaths로 정의한 페이지로 요청이 왔을 때, 제공할 props를 반환하는 함수

```tsx
export const getStaticProps: GetStaticProps = async ({ params }) => {
	const { id } = params
	const post = await fetchPost(id)
	
	return {
		props: { post },
	}
}

export default function Post(( Post }: { post: Post }) {
	// post로 페이지를 렌더링
)
// 무조건 같은 파일에서 써야함
```

- App Router에서는 그냥 컴포넌트가 직접 데이터 로드

```tsx
async function getData() {
  const res = await fetch('...');
  return res.json();
}

export default async function Page() {
  const data = await getData(); // 컴포넌트 안에서 바로 호출
  return <h2>{data.title}</h2>;
}
```

### getServerSideProps

- 페이지 진입 전 서버에서 무조건 실행되는 함수
- 페이지 컴포넌트에 props를 반환해주거나, 페이지 진입 전 리다이렉트 시킬 수도 있음

```tsx
export const getServersideProps: GetServersideProps = async (context) => {
	const {
		query: { id = '' },
	} = context;
	const post = await fetchPost(id.toString());
	return {
		props : { post},
	};
}

export default function Post({ post }: { post: Post }) {
	// ...
}
```

- App Router는 RSC(React Server components)로 대체되어, 컴포넌트 자체를 async 함수로 선언할 수 있게 됨

```tsx
async function getData(id: string) {
  // cache: 'no-store' 옵션으로 이전 SSR 방식처럼 매번 새로운 데이터를 로드함
  const res = await fetch(`https://api.example.com/post/${id}`, { cache: 'no-store' });
  return res.json();
}
 
export default async function Post({params}: {params: Promise<{id: string}>}) {
	const postId = await params.id
  const data = await getData();
  return <h2>{data.title}</h2>;
}
```

### getInitialProps

- getStaticProps, getServerSideProps 이전 유일한 페이지 데이터 불러오기 수단
- 서버와 클라이언트 모두에서 실행 가능한 메서드 ⇒ 사용 시 실행되는 환경을 반드시 고려해야 함

```tsx
// 어디서 실행되는지 확인하려면...
Post.getlnitialProps = async (context) => {
	const isServer = context.req;
	console.log(`${isServer ? '서버' : '클라이인트'}에서 실행됨`)
// ...
}
```

---

## 4.3.4 스타일 적용하기

### 전역 스타일

- CSS Reset(기본 제공 스타일 초기화) 등 앱 전체에 적용하고 싶은 스타일은 _app.tsx에 적용하면 됨
- import로 스타일 관련 리소스를 불러오면 됨 `import '../global.css'`

### 컴포넌트 레벨 CSS

- `[name].module.css`의 네이밍 규칙으로 컴포넌트 레벨의 CSS 추가가 가능함
- 다른 컴포넌트의 스타일과 충돌이 일어나지 않도록 고유한 클래스명으로 변환됨
(*e.g. .Button_active__62TGU*)

```tsx
// Button.module.css
.active {
	color: green;
	font-weight: 900;
}

// Button.tsx
import style from './button.module.css'
export function Button() {
	return <button type="button" className={style.active}>버튼</button>
}
```

### SCSS와 SASS

### CSS-in-JS

- 자바스크립트 내부에 스타일 시트를 삽입하는 방식
- 직관성과 편의성에 이점이 있음

---

## 4.3.5 _app.tsx 응용하기

- getInitialProps와 <Link> 또는 router를 이용해서 서비스 최초 접근시에만 원하는 로직을 실행해줄 수 있음
- 비용이 발생하거나 중복 실행되면 안되는 로직에 사용
    - *e.g. userAgent 확인, 사용자 정보 수집*

```tsx
MyApp.getInitialProps = async (context: AppContext) => {
	 const {
		 ctx: { req },
		 router: { pathname },
	 } = context;
	 
	 if (
		 req && // req가 있다면 서버로 오는 요청
		 !req.url?.startsWith('/_next') && // 클라이언트 렌더링 여부 확인
		 !['/500', '/404', '/error'].includes(pathname) 
		 // 에러페이지가 아닌 정상적인 페이지 접근 여부 확인, 에러페이지는 통계에서 제외시킴
	 ) {
		 doOnlyOnce()
	 }
	 
	 return appProps
 }
```

- App Router의 Root Layout에서 실행하면 항상 서버에서 실행되는 것을 보장받을 수 있음

---

## 4.3.6 next.config.js

- Next.js 실행에 필요한 설정을 추가할 수 있는 파일
- @type 구문으로 타입의 도움을 받을 수 있음

### 실무에서 자주 사용되는 설정

### **basePath**

- 서비스가 시작되는 주소
- <Link>나 router를 사용하는 경우 따로 추가하지 않아도 알아서 붙여줌
- <a>나 window.location.push 등에는 안붙여줘서 따로 써줘야함

### swcMinify

- swc 이용해서 코드 압축할 지 나타냄
- 기본값 true

### poweredByHeader

- 응답 헤더에 X-Power-by: Next.js 정보를 제공해줌
- 보안관련 솔루션에서는 이 헤더를 취약점으로 분류함 ㅋㅋ
    - 보안 결함 발견 시 악성 봇이 쉽게 탐지 가능
    - 기술스택을 알려서 최적화된 공격 받기 쉬움
    - 공격 힌트 같은..

### redirects

- 특정 주소를 다른 주소로 보내고싶을때 사용
- 정규식도 사용 가능

```tsx
/** @type {import('next').NextConfig} */
const nextConfig = {
  async redirects() {
    return [
      {
        source: '/old-path',
        destination: '/new-path',
        permanent: true, // true: 308 (영구 이동), false: 307 (임시 이동)
      },
    ]
  },
}

module.exports = nextConfig
```

### reactStrictMode

- 리액트에서 제공하는 엄격 모드를 설정할지 여부 표시
- true로 설정하자!

### assetPrefix

- 빌드 결과물을 다른 CDN에 업로드 하는 경우
- static 리소스들을 해당 주소에 있다고 가정하고, 해당 주소로 요청함

---

## 4.3.7 정리

…