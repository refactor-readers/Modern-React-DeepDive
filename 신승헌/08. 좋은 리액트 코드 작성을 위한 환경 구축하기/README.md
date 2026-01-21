# 8.1 ESLint를 활용한 정적 코드 분석

## 8.1.1 ESLint 살펴보기

### ESLint는 어떻게 코드를 분석할까

1. 자바스크립트 코드를 문자열로 읽음
2. JS코드를 분석할 수 있는 parser로 코드를 구조화
3. 위 parsing의 결과물인 AST(Abstract Syntax Tree)를 규칙과 대조
4. 위반한 코드를 알려주거나 수정함

### espree

- 자바스크립트를 분석하는 파서의 한 종류
- 코드를 JSON형태로 구조화 함
- [AST Explorer](https://astexplorer.net/)
    - JSON 트리를 통해 다양한 정보를 알 수 있음
    - 변수/함수 여부, 변수/함수명 뿐만 아니라
    - 코드의 정확한 위치 등
    - 이를통해 ESLint나 Prettier같은 도구가 코드를 정확하게 파악할 수 있음

### 용어정리

- ESLint가 분석결과를 바탕으로 어떤 코드가 잘못되었고, 어떻게 수정해야하는지 정함
- 이를 **rules**라고 하며, 특정 규칙의 모음들을 **plugins**라고 함

## 8.1.2 eslint-plugin과 eslint-config

- 모두 ESLint관련 npm 패키지이지만 역할이 다르다

### eslint-plugin

- 위 접두사로 시작하는 플러그인은 규칙을 모아놓은 패키지
- 위에서 설명한 것 처럼 플러그인 = 규칙모음

### eslint-config

- eslint-plugin을 묶어서 세트로 제공하는 패키지
- 여러 프로젝트에서 동일한 ESLint 설정을 사용할 수 있도록

### 대표적인 라이브러리

- [eslint-config-airbnb](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb)
- [triple-config-kit](https://github.com/titicacadev/triple-config-kit)
    - 규칙 변경에 대비한 테스트코드가 있음
    - npm 라이브러리 구축 및 관리를 위한 시스템이 잘 구축되어있음
    - 참고용으로 좋음
- [eslint-config-next](https://www.npmjs.com/package/eslint-config-next)
    - Next.js 분석에 최적화 되어있음

## 8.1.3 나만의 ESLint 규칙 만들기

## 8.1.4 주의할 점

### Prettier와의 충돌

- 코드 포매팅 과정에서 줄바꿈, 들여쓰기, 따옴표, 글자수, … 등으로 인해 충돌하는 규칙이 생길 수 있음
- 역할을 잘 나눠서 규칙을 잘 짜거나
- 각 도구가 관여하는 파일을 물리적으로 분리할 수 있음
    - JS/TS는 ESLint에 그 외에는 Prettier에 맡기고
    - 추가적으로 필요한 규칙은 모두 eslint-plugin-prettier 사용하기
    - 하지만 이렇게 해도 충돌 날 수도 있음
- eslint-config-prettier를 사용하면 ESLint와 Prettier에 충돌할만한 규칙을 모두 꺼버릴수 있음

```json
{
  "extends": [
    "eslint:recommended",
    "plugin:prettier/recommended"
  ]
}
```

### 규칙에 대한 예외처리

`eslint-disable-line no-console`, `eslint-disable-line no-exhaustive-deps` 등..

- 적절한 이유로 사용했는지 검토하고 고민해볼 필요가 있음

### ESLint 버전충돌

- 현재 프로젝트와 설치하고자 하는 plugin, config에서 지원하는 버전을 확인하자

## 8.1.5 정리

---

# 8.2 리액트 팀이 권장하는 리액트 테스트 라이브러리

- 테스트란 코드가 **의도대로 작동하는지 확인**하는 일련의 작업을 의미함
- 프론트엔드는 일반적인 사용자와 동일하거나 유사한 환경에서 테스트를 수행함
- 디자인 요소 뿐 아니라, 인터랙션 의도치 않은 작동 등 다양한 시나리오를 고려해야 함
- 프론트엔드의 테스트는 까다롭다

## 8.2.1 React Testing Library

- DOM Testing Library를 기반으로 만들어진 테스팅 라이브러리

### DOM Testing Library

- jsdom을 기반으로 하고 있음
- 자바스크립트만 존재하는 Node.js같은 환경에서 HTML과 DOM을 사용할 수 있도록 해줌

## 8.2.2 자바스크립트 테스트의 기초

### 테스트 과정

1. 테스트할 함수/모듈 선정
2. 함수/모듈의 반환 기대값 작성
3. 실제 반환값 확인

### Node.js의 assert

- assert라는 모듈을 활용해 테스트를 간결하게 작성하고 결과를 확인할 수 있음
- 단순 동등 비교(equal)외에도 객체비교(deepEqual), 에러여부(throw) 등 다양함

### 좋은 테스트코드란?

- 단순히 테스트 코드를 작성하고 통과하는것 뿐만 아니라
- 어떤 테스트가 무엇을 테스트하는지 보여주는것도 중요함

### 테스팅 프레임워크

- 테스트의 기승전결을 완성해줌
- Assertion(주장)을 기반으로 테스트를 수행함
- 테스트 코드 작성에 도움이 되는 추가적인 정보도 제공함

### Jest

- 자체 expect 패키지로 Assertion을 수행함
- 콘솔 출력을 통해 테스트 내용, 소요시간, 케이스별 결과 등을 확인할 수 있음
- [공식문서](https://jestjs.io/)

## 8.2.3 리액트 컴포넌트 테스트 코드 작성하기

### 리액트 컴포넌트 테스트 절차

1. 컴포넌트 렌더링
2. 특정 액션 수행
3. 액션의 기대 결과와 실제 결과 비교

## 8.2.4 사용자 정의 훅 테스트하기

## 8.2.5 테스트를 작성하기에 앞서 고려해야 할 점

### 테스트 커버리지가 만능은 아니다

- 테스트 커버리지는 단순히 얼마나 많은 코드가 테스트되고 있는지를 나타내는 지표일 뿐
- TDD를 하더라도 프론트엔드는 사용자의 입력이 매우 자유로움 ⇒ 완벽한 커버리지가 불가능함
- **애플리케이션에서 가장 취약하거나 중요한 부분을 파악하는게 우선**

## 8.2.6 그 밖에 해볼 만한 여러가지 테스트

### 유닛테스트

- 각 코드나 컴포넌트가 독립된 환경에서 의도된 대로 작동하는지 검증

### 통합 테스트

- 유닛테스트 이후 여러 컴포넌트가 묶인 환경에서, 하나의 기능으로 잘 작동하는지 확인

### 엔드투엔드

- 실제 사용자처럼 동작하는 봇을 활용해 애플리케이션 전반의 기능을 테스트
- Cypress

## 8.2.7 정리

### 테스트의 목표

- 요구사항을 충족하는지 확인하는 것
- 처음부터 모두 테스트하려고 하지 말고, 핵심을 파악하고 우선순위에 맞게 테스트 해보