# 9장. 모던 리액트 개발 도구로 개발 및 배포 환경 구축하기

## 9.1 Next.js로 리액트 개발 환경 구축하기

1. package.json 작성하기
2. tsconfig.json 작성하기
3. next.config.js 작성하기
4. ESLint와 Prettier 설정하기
5. 스타일 설정하기
6. 애플리케이션 코드 작성

## 9.2 깃허브 100% 활용하기

### 깃허브 Dependabot으로 보안 취약점 해결하기

#### package.json의 dependencies 이해하기

버전은 주,부,수로 구성되어 있다
- 기존 버전과 호환되지 않게 API가 바뀌면 "주버전"을 올리고,
- 기존 버전과 호환되면서 새로운 기능을 추가할 때는 "부 버전"을 올리고,
- 기존 버전과 호환되면서 버그를 수정한 것이라면 "수 버전"을 올린다.

#### 의존성

- dependencies: npm install을 실행하면 설치되는 의존성이다
- devDependencies: 개발 단계에서 필요한 패키지들을 여기에 선언한다
- peerDependencies: 서비스보다 라이브러리와 패키지에서 자주 쓰이는 단위다.

## 9.3 리액트 애플리케이션 배포하기

- Netlify
- Vercel
- Digitalocean

## 9.4 리액트 애플리케이션 도커라이즈하기

pass