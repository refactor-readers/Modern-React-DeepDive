# 8.1 ESLint를 위한 정적 코드 분석

- 정적 코드 분석의 정의 : 코드 실행과 별개로 코드 자체만으로 코드 스멜(잠재적 버그 유발 코드)을 찾아내 사전에 수정하는 방법
- ESLint의 역할 : 자바스크립트 생태계에서 가장 많이 사용되는 정적 분석 도구로, 예기치 못한 버그를 방지하는 데 기여

## 8.1.1 ESLint 살펴보기

- ESLint의 동작 방식
  1. 자바스크립트 코드를 문자열로 읽습니다.
  2. **파서(Parser)**를 이용해 코드를 구조화합니다. (ESLint는 기본값으로 espree 사용)
  3. 구조화된 트리를 **AST(Abstract Syntax Tree)**라고 하며, 이를 규칙과 대조합니다.
  4. 규칙 위반 시 해당 코드를 알리거나(report) 수정(fix)합니다.
- AST(Abstract Syntax Tree) : 단순히 함수명이나 변수명만 파악하는 것이 아니라, 코드의 정확한 위치, 줄바꿈, 들여쓰기 등 세세한 정보를 JSON 형태로 구조화한 트리입니다.

## 8.1.2 eslint-plugin과 eslint-config

- eslint-plugin : 특정 도메인(예: 리액트, import 등)과 관련된 **규칙(rules)을 모아놓은 패키지**
  - 예: `eslint-plugin-react`는 JSX배열의 `key` 누락 여부 등을 확인
- eslint-config : 여러 `eslint-plugin`과 설정을 한데 묶어 **세트로 제공하는 패키지**
  - eslint-config-airbnb : 에어비앤비에서 관리하며 가장 대중적으로 사용되는 설정
  - @titicaca/triple-config-kit : 한국 커뮤니티(트리플)에서 유지보수하며 자체 규칙과 테스트 코드를 포함
  - eslint-config-next : Next.js 프로젝트 전용으로, 성능 향상 및 핵심 웹 지표 분석 기능을 포함

## 8.1.3 나만의 ESLint 규칙 만들기

    - 필요성 : 조직 내부의 특수한 규칙이나 일관된 코드 변화가 필요할 때 수동 확인의 실수를 방지하기 위해 직접 규칙을 생성할 수 있음
    - 예시 : 리액트 17부터 불필요해진 `import React` 구문을 제거하기 위한 규칙 등을 커스텀하여 적용할 수 있음

### ❓다들 ESLint rules 설정 어떻게 되어 있나요~?

#### 저의 eslint 설정 입니다.

> 그렇게 빡쎄게 해두지는 않았어요 하하 ! 좋은 설정 있으면 따라가도록 하겠습니다 :)

```mjs
import { fileURLToPath } from "node:url";
import { fixupConfigRules, fixupPluginRules } from "@eslint/compat";
import { FlatCompat } from "@eslint/eslintrc";
import js from "@eslint/js";
import tsParser from "@typescript-eslint/parser";
import eslintConfigPrettier from "eslint-config-prettier";
import _import from "eslint-plugin-import";
import eslintPluginNode from "eslint-plugin-node";
import reactRefresh from "eslint-plugin-react-refresh";
import simpleImportSort from "simple-import-sort"; // 오타 교정 권장
import unusedImportsPlugin from "eslint-plugin-unused-imports";
import globals from "globals";
import path from "path";

// 현재 파일 경로 및 디렉토리 설정 (ES 모듈 방식)
const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

// 과거 스타일(eslintrc)의 설정을 Flat Config 방식으로 변환하기 위한 도구
const compat = new FlatCompat({
  baseDirectory: __dirname,
  recommendedConfig: js.configs.recommended,
  allConfig: js.configs.all,
});

export default [
  // [1] 전역 무시 설정: 분석에서 제외할 폴더 및 파일
  {
    ignores: [
      "dist/**",
      "dist-electron/**",
      "release/**",
      ".github/pull_request_template.md",
    ],
  },

  // [2] 기본 추천 규칙 확장
  // fixupConfigRules: 과거 플러그인 호환성을 유지하면서 추천 규칙 적용
  ...fixupConfigRules(
    compat.extends(
      "eslint:recommended", // ESLint 기본 추천 규칙
      "plugin:@typescript-eslint/recommended", // 타입스크립트 추천 규칙
      "plugin:react-hooks/recommended" // 리액트 훅 규칙 (의존성 배열 체크 등)
    )
  ),

  // [3] 프로젝트 전역 설정
  {
    plugins: {
      "react-refresh": reactRefresh, // Fast Refresh를 위한 플러그인
      import: fixupPluginRules(_import), // 모듈 임포트/익스포트 관련 규칙
      node: eslintPluginNode, // Node.js 관련 규칙
      "simple-import-sort": simpleImportSort, // 임포트 구문 자동 정렬
      "unused-imports": unusedImportsPlugin, // 사용하지 않는 임포트 자동 제거/감지
    },

    languageOptions: {
      globals: {
        ...globals.browser, // 브라우저 환경 전역 변수(window, document 등) 허용
      },
      parser: tsParser, // 타입스크립트 파서 사용 [cite: 78]
    },

    rules: {
      // 리액트 컴포넌트만 export하도록 제한 (Fast Refresh 효율성)
      "react-refresh/only-export-components": [
        "warn",
        { allowConstantExport: true },
      ],

      "no-var": "warn", // var 대신 let/const 사용 권장
      "unused-imports/no-unused-imports": "error", // 사용하지 않는 임포트는 에러로 처리
      "no-unused-vars": ["warn", { argsIgnorePattern: "^_" }], // _로 시작하는 인자 외엔 미사용 변수 경고
      "@typescript-eslint/no-unused-vars": "off", // 기본 규칙과 충돌 방지를 위해 TS용은 끔
      "@typescript-eslint/no-require-imports": "off", // CommonJS require() 허용
      "@typescript-eslint/no-var-requires": "off",
      "@typescript-eslint/no-explicit-any": "warn", // any 타입 사용 시 경고

      "no-console": [
        "warn",
        { allow: ["warn", "error"] }, // console.warn, error 외엔 경고
      ],

      // Import 정렬: simple-import-sort 사용을 위해 기본 import/order는 끔
      "import/order": "off",
      "simple-import-sort/imports": "error",
      "simple-import-sort/exports": "error",
    },
  },

  // [4] Electron 관련 파일 특화 설정
  {
    files: ["electron/**/*.{js,ts}"],
    rules: {
      "no-console": "off", // 일렉트론 메인 프로세스에선 로그 확인을 위해 허용
      "node/no-unsupported-features/es-syntax": "off",
      "node/no-missing-import": "off",
      "node/no-unpublished-import": "off",
      "@typescript-eslint/no-require-imports": "off",
      "@typescript-eslint/no-var-requires": "off",
    },
    languageOptions: {
      globals: {
        ...globals.node, // Node.js 환경 전역 변수(process, __dirname 등) 허용
      },
      parser: tsParser,
      sourceType: "commonjs", // 일렉트론 메인은 보통 CommonJS 사용
      ecmaVersion: 2020,
    },
  },

  // [5] Prettier 충돌 방지: 마지막에 배치하여 Prettier와 겹치는 스타일 규칙 비활성화
  eslintConfigPrettier,
];
```

### 오늘도 돌아온 Q&A 시간~!

#### ❓Q.1 ESLint `tsParser`를 사용하여 코드를 분석하는 과정은 자바스크립트 엔진의 동작과 어떤 공통점이 있나요?

#### ❓Q.2 `plugin:react-hooks/recommended` 설정이 리액트의 '렌더링 일관성' 보장과 어떤 관계가 있나요?

#### ❓Q.3 리액트 17 이후 환경에서 `import React` 관련 규칙을 설정하거나 제거하는 것이 성능에 미치는 영향은 무엇인가요?

### ❓Q.4 `globals.browser`와 `globals.node`를 구분하여 설정하는 이유는 자바스크립트 실행 환경(Runtime)과 어떤 관련이 있나요?
