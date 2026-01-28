# 9.1 Next.js로 리액트 개발 환경 구축하기

## 9.1.1 CRA 없이 하나씩 구축하기

## 9.1.2 tsconfig.json 작성하기

```jsx
{
	"$schema": "https://json.schemastore.org/tsconfig.json"
}
```

- [schemaStore](https://www.schemastore.org/)에서 제공해주는 정보
- JSON 최상단에 $schema 키와 올바른 값을 넣어주면 IDE에서 자동완성이 가능해짐
- `.eslinrc`, `.prettierrc` 등 JSON 방식 설정파일에 사용 가능함

> 몰랐어요..🤯
> 

## tsconfig 옵션

### compilerOptions

타입스크립트를 자바스크립트로 컴파일할 때 사용하는 옵션

1. target: 타입스크립트가 변환을 목표로 하는 언어의 버전
2. lib: 구형 브라우저 지원을 위해 target을 설정했지만, 최신 문법을 사용하고 싶을 때 타입스크립트에게 정보를 제공해주기 위한 옵션
    1. 주의할 점! Polyfill을 추가해주진 않음
3. strict: 타입스크립트 컴파일러의 엄격 모드를 제어, 켜두는게 좋
    1. alwaysStirct: 모든 JS파일에 use strict추가
    2. strictNullChecks: 엄격한 Null check 활성화, 값이 비어있을 가능성을 사전에 대처하도록 해줌
    
    **🚨 strictNullChecks 옵션이 중요한 이유**
    
    - 런타임 에러(Runtime Error) 방지
    Uncaught TypeError.. 코드 작성 단계에 발견할 수 있음
    - 명확한 데이터 설계 강제
    데이터가 있을 수도 있고 없을 수도 있음을 코드에 명시
    코드 안정성 향상
    - 의도치 않은 버그 차단
4. …더보

[📘 타입스크립트 컴파일 설정 - tsconfig 옵션 총정리](https://inpa.tistory.com/entry/TS-%F0%9F%93%98-%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-tsconfigjson-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0-%EC%B4%9D%EC%A0%95%EB%A6%AC#compileroptions)

## 9.1.3 next.config.js 작성하기

[next.js/packages/next/src/server/config-shared.ts at canary · vercel/next.js](https://github.com/vercel/next.js/blob/canary/packages/next/src/server/config-shared.ts)

## 9.1.4 ESLint와 Prettier 설정하기

- 자바스크립트는 정적 분석의 의존도가 높음

## 9.1.5 스타일 설정하기

## 9.1.6 애플리케이션 코드 작성

## 9.1.7 정리

- 깃헙으로 보일러플레이트 프로젝트를 만들어보자

---

# 9.2 깃허브 100% 활용하기

## 9.2.1 깃허브 액션으로 CI 환경 구축하기

### CI란

- 코드의 변화를 모으고 관리하는 과정에서 여러 기여자들의 코드를 지속적으로 빌드/테스트해 정합성을 확인하는 과정
- 코드 변화가 발생할 때 마다 확인하는게 중요

### 깃허브 액션

- **러너**: 깃허브 액션이 실행되는 서버
- **액션**: 러너에서 실행되는 작업 단위
- **이벤트**: 액션의 실행을 일으키는 이벤트
    - PR(pull request)
    - issues
    - push
    - schedule(cron)
- **잡**: 하나의 러너에서 실행되는 여러 스텝의 모음, 별도 설정이 없다면 각 잡은 병렬로 실행됨
- **스텝**: 잡 내부에서 일어나는 하나하나의 작업, 스텝은 동기적으로 실행됨

### 깃허브 액션 작성하기

- .github/workflows 폴더에 .yml 또는 .yaml로 작성
- yaml파일도 Prettier를 적용할 수 있음

## 9.2.2 유용한 액션과 깃허브 앱 가져다 쓰기

- github의 [Marketplaces](https://github.com/marketplace)에서 미리 만들어진 액션을 가져다쓸 수 있음
- 공식 액션 외에는 유지보수 여부가 불투명하니, 꼭 테스트를 거쳐 사용하자

### 깃허브 제공 기본 액션

- **actions/checkout:** 깃허브 저장소를 체크아웃하는 액션
    - 체크아웃 한다 ⇒ 내가 작업할 위치를 선택하고, 로컬의 상태를 업데이트 한다는 뜻
- **actions/setup-node**: Node.js 설치하는 액션
- **actions/dependency-review-action**: 의존성 변경 시, 의존성을 분석해 보안이나 라이선스 문제를 알려줌
- **github/codeql-action**: 깃허브의 코드 분석 솔루션을 이용해 내 코드의 취약점을 분석해줌, javascript로 설정하면 타입스크립트까지 검사해줌

### calibreapp/image-actions

- PR에 올라온 이미지를 sharp패키지로 거의 무손실 압축 후 다시 커밋해주는 액

### lirantal/is-website-vulnerable

- 특정 웹사이트를 방문해 라이브러리 취약점이 있는지 확인해줌
- devDependencies나 트리쉐이킹으로 배포에 포함되지 않은 의존성은 검사하지 않

### Lighthouse CI

- 프로젝트 URL을 방문해 라이트하우스 검사를 실행해주는 도구

## 9.2.3 깃허브 Dependabot으로 보안 취약점 해결하기

### Dependabot

- 깃허브에서 제공하는 기능
- 의존성에 문제가 있는 경우 알려주고, 가능한 경우 해결을 위한 PR까지 열어줌

### dependencies 이해하기

- 버전은 **“주.부.수”**로 구성됨
- 부 버전이 변경되는 경우 기능 추가나 스펙 변경이 이루어질 수 있음, 기존 기능은 유지됨
- 수 버전은 버그 수정시에만 변경됨
- 버전 작성 시 “^”는 해당 주 버전에 호환되는 부, 수 버전을 설치할 수 있다는 의미
- 버전 작성 시 “~”는 해당 주 버전에 호환되는 수 버전을 의미

🚨 npm은 위에서 설명한 유의적 버전 관리 방식을 보증하지 않음, 버전 변경시에는 문제가 없는지 확인이 필요하다!

### Dependabot으로 취약점 해결하기

- 깃허브 원격저장소에 푸시하면 Dependabot이 저장소의 의존성에 문제가 있다고 알려줌

## 9.2.4 정리

---

# 9.3 리액트 애플리케이션 배포하기

## 9.3.1 Netlify

## 9.3.2 Vercel

## 9.3.3 DigitalOcean

## 9.3.4 정리

---

# 9.4 리액트 애플리케이션 도커라이즈 하기

## 9.4.1 리액트 앱을 도커라이즈 하는 방법

### 도커란?

애플리케이션을 빠르게 배포할 수 있도록, “컨테이너” 단위로 패키징하고 

컨테이너 내부에서 애플리케이션이 실행될 수 있도록 함

컨테이너는 독립된 환경을 가지고, 애플리케이션이 항상 일관되게 실행될 수 있도록 함

### 도커 용어

- 이미지: 컨테이너를 만드는데 사용되는 템플릿. Dockerfile을 빌드해서 이미지를 만들 수 있음
- 컨테이너: 도커 이미지를 실행한 상태. 독립된 공간을 의미하며, 이미지 실행에 필요한 운영체제, 파일시스템과 각종 자원 등이 할당됨
- Dockerfile: 어떤 이미지 파일을 만들지 정의하는 파일
- 태그: 이미지를 식별할 수 있는 값. *e.g. ubuntu:latest*
- 리포지터리: 이미지를 모아두는 저장소
- 레지스트리: 리포지터리에 접근할 수 있게 해주는 서비스. *e.g. 도커 허브*

### Dockerfile 작성하기

- 루트에 Dockerfile 생성 후 아래 내용들을 추가
1. 운영체제 설정
2. Node.js 설치
3. npm ci: 빌드에 필요한 의존성 모듈 설치
4. 프로젝트 빌드
5. 프로젝트 실행

## 9.4.2 도커로 만든 이미지 배포하기

## 9.4.3 정리

- 쿠버네티스, 헬름차트 활용에 대해서도 더 공부해보면 좋을 것 같음