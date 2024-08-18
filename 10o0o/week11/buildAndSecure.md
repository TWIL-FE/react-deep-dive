# 모던 리액트 개발 도구로 개발 및 배포 환경 구축

## tsconfig.json

-$schema 키값을 제공하면, 해당 schemaStore에서 제공하는 값을 미리 알려줄수 있다.

```json
{
  "$schema": "https://json.schemastore.org/tsconfig.json"
}
```

### 옵션 종류

- compilerOptions: 타입스크립트를 컴파일할 때 사용하는 옵션
  - target: 타입스크립트가 변환을 목표로 하는 언어의 버전을 의미. es5로 설정돼 있으면, es6의 화살표 함수는 일반 환수로 변환됨. (단 폴리필 미지원, babel과 같은 도구 써야함)
  - lib: 만약 es5에서 지원하지 않지만, 프로젝트에서 es5 지원을 목표로 하고 있고, Promise와 Map 같은 객체들도 폴리필을 붙여서 지원할 환경을 준비했다면, lib에 가장 최신 버전을 의미하는 esnext를 추가하면 target은 es5라 할지라도 신규 기능에 대한 API 정보를 확인할 수 있게 되어 에러가 발생하지 않는다.
  - allowJs: TS가 JS파일 또한 컴파일할지를 결저앟ㄴ다. (.js, .ts 혼재했을 때 사용하는 옵션)
  - skipLibCheck: 라이브러리에서 제공하는 d.ts에 대한 검사 여부를 결정한다. (일반적으로는 꺼놓는다)
  - strict: 해당 옵션을 키면, 모든 자바스크립트 파일에 use strict를 추가함과, strictNullChecks, strictBindCallApply, strictFunctionTypes, strictPropertyInitializtion, noImplicitAny, ... 등을 추가한다.

- forceConsistentCasingInFileNames: 파일 이름의 대소문자를 구분하도록 강제한다.
- noEmit: 컴파일을 하지 않고, 타입 체크만 한다. 주로 SWC를 사용하는 환경에서 쓰인다
- esModuleInterop: CommonJS 방식으로 보낸 모듈을 ES 모듈 방식의 import로 가져올 수 있게 해준다.
- module: 모듈 시스템을 설정한다. 대표적으로 commonjs, esnext
- moduleResolution: 모듈을 해석하는 방식 설정 
  - node: node_modules 기준
  - classic: tsconfigjson이 있는 디렉터리 기준 모듈 해석

- resolveJsonModule: JSON파일을 import할수있게 해준다 이 옵션을 키면 allowJS옵션도 켜진다.
- isolateModules: 파일에 import나 export가 없다면, 단순 스크립트 파일로 인식해 이러한 파일이 생성되지 않도록 막는다.
- jsx: .tsx파일 내부에 이쓴ㄴ jsx를 어떻게 컴파일할지 설정한다.
  - react: 기본값, React.createElement로 변환
  - react-jsx: react/jsx-runtime을 사용해 변환. 이 방식을 사용하면, react를 import하지 않아도 된다.
  ```jsx
  import { jsx as _jsx } from 'react/jsx-runtime';
  export const Hello = () => _jsx('div', { children: 'Hello' });
  ```

  - react-jsxdev: react-jsx와 동일하지만, 디버깅 정보가 추가
  - preserve: 변환하지 않고 유지
  - react-native: 마찬가지로 변환하지 않고 유지

## next.config.js

- reactStrictMode
- poweredByHeader: 일반적으로 보안 취약점으로 취급되는 X-Powered-By 헤더를 제거한다
- eslint.ignoreDuringBuilds: 빌드 시에 eslint 오류 무시

# 웹사이트 보안을 위한 리액트와 이슈

## XSS(Cross-Site Scriptiing, XSS)
웹사이트 개발자가 아닌 제3자가 웹사이트에 악성 스크립트를 삽입해 실행할 수 있는 ㅜ치약점

### 리액트 내에서 발생 가능성

### dangerouslySetInnerHTML prop
해당 props에 사용자가 직접 컨트롤 하여, 영구적으로 스크립트를 저장하도록 유도 가능하다면, XSS 스크립트 공격이 언제든지 가능하다.

따라서 해당 props에 들어가는 값을 항상 검증된 값이여야 한다.

```jsx
const html = `<span><svg onload=alert(origin)/></span>`;

function App() {
  return <div dangerouslySetInnerHTML={{ __html: html }}/>;
}
```

### useRef 사용한 직접 삽입
위와 동일

## XSS 문제를 피하는 방법

### 라이브러리 사용

다음과 같은 유명한 라이브러리가 있다.

- DOMpurity
- sanitize-html
- js-xss

### HTTP 보안 헤더 설정

#### Strict-Transport-Security

모든 사이트가 HTTPS를 통해 접근해야 하며, 만약 HTTP로 접근하는 경우, 모든 시도가 HTTPS로 변경되게 한다.

```
Strict-Transport-Security: max-age=<expire-time>; includeSubDomains
```

#### X-Frame-Options
페이지를 frame, iframe, embed, object 내부에서 렌더링을 허용할지를 나타낼 수 있다.

예를들어, 네이버와 비슷한 도메인을 가진 공격자가, 네이버를 iframe으로 렌더링 시킬 수도 있다.
따라서 네이버에서는 이를 방지하고자 해당 옵션을 사용할 수 있다.

#### Permissions-Policy
웹사이트에서 사용할 수 있는 기능과 불가능한 기능을 명시적으로 선언

```
# 모든 geolocation 기능을 막는다.
Permissions-Policy: geolocation=()

# geolocation을 페이지 자신과 몇가지 페이지에 대해서만 허용한다.
Permissions-Policy: geolocation=(self "https://a.yceffort.kr" "https://b.yceffort.kr")

# 카메라 모든곳에서 허용한다
Permissions-Policy: camera=*;
```

#### X-Content-Type-Options

>MIME이란?
>
> MIME는 Multipurpose Internet Mail Extension의 약자로, Content-type의 값으로 사용된다.

MIME에서 Content-Type 옵션이 origin에서 임의의 조작으로 바뀌는걸 방지해준다.
예를들어, 공격자가 악성 스크립트 파일을 jpg의 MIME로 변경해 숨기고, 서버에서 해당 스크립트를 실행하게끔 할 수 있다.

#### Referrer-Policy
사용자가 어디서 와서 방문중인지 인식 가능하다.

#### Content-Security-Policy

- -src: font-src, img-src, script-src 등 다양한 src를 제어할 수 있는 지시문

```
Content-Security-Policy: font-src <source>;
Content-Security-Policy: font-src <source> <source>;
```

위와같이 선언하면, 폰트를 가져올 수 있는 src 제한이 가능하다.