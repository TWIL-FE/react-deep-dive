## 8장 좋은 리액트 코드 작성을 위한 환경 구축하기

### 8.1 ESLint를 활용한 정적 코드 분석

정적 코드 분석 - 코드의 실행과 별개로 코드 그 자체만으로 코드 스멜(잠재적으로 버그를 야기할 수 있는 코드)을 찾아내서 문제의 소재가 있는 코드를 사전에 수정하는 것
ESLint - 자바스크립트에서 가장 많이 사용되는 정적 코드 분석 도구

### 8.1.1 ESLint 살펴보기

ESLint의 코드 분석 원리

1. 자바스크립트 코드를 문자열로 읽음
2. 자바스크립트 코드를 분석할 수 있는 파서(parser)로 코드를 구조화함
3. 2번에서 구조화한 트리를 AST(Abastract Syntax Tree)라 하는데, 이 트리를 기준으로 각종 규칙과 대조함
4. 규칙과 대조시 위반한 코드를 알리거나 수정한다.
   \*ESLint는 자바스크립트 분석시 기본값으로 espree를 사용, 타입스크립트의 경우 @typescript-eslint/typescript-estree로 코드 분석
   plugins
   어떤 코드가 잘못됐으며, 어떻게 수정하는지 정하는 ESLint 규칙의 모음

### 8.1.2 eslint-plugin과 eslint-config

- eslint-plugin
  eslint 규칙을 모아놓은 패키지
  ex)

  1. eslint-plugin-import
     import 와 관련된 다양한 규칙을 제공
  2. exlint-plugin-react
     react/jsx-key는 jsx 배열에 키를 선언하지 않았다고 경고
     => ESLint는 정적 코드 분석이라 key가 유니크 값인지는 모르지만 존재 여부는 확인 가능

- eslint-config
  eslint-plugin을 묶어서 한 세트로 제공하는 패키지
  여러 프로젝트에 걸쳐 동일하게 사용할 수 있는 ESLint 관련 설정을 제공하는 패키지

- 자주 사용하는 eslint-config

1. eslint-config-airbnb
2. @titicaca/triple-config-kit
3. eslint-config-next(Next.js 프레임워크 config)
   => next는 정적 코드 분석 뿐 아니라 JSX 구문 및 \_app, \_document에 작성돼있는 HTML 코드 또한 정적 분석 대상으로 분류해 제공

**주의할 점**

Prettier와의 충돌
Prettier는 자바스크립트, HTML, CSS, 마크다운, JSON 등 다양한 언어에도 적용 가능하지만 ESLint는 자바스크립트에만 작동
=> 충돌 위험성이 있음

\*해결방법

1. Prettier에서 제공하는 규칙을 어기지 않도록, ESLint에서 규칙을 끄는 방법
   자바스크립트, 타입스크립트는 모두 ESLint 적용 / 그 외의 파일은 모두 Prettier 적용 단, 자바스크립트에 추가적으로 필요한 Prettier 규칙은 모두
2. eslint-plugin-prettier를 사용
   규칙에 대한 예외 처리, react-hooks/no-exhaustive-deps

- 일부 코드에서 특정 규칙을 임시로 제외시키고 싶다면 eslint-disable-주석을 사용하면 됨
  => 생각치 못한 버그를 야기시킬 수 있으므로 꼭 점검한 뒤 사용

### 8.2.1 React Testing Library

\*테스트란 개발자가 만든 프로그램이 코딩을 한 의도대로 작동하는지 확인하는 일련의 작업

React Testing Library란 DOM Testing Library를 기반으로 만들어진 테스팅 라이브러리로, 리액트를 기반으로 한 테스트를 수행하기 위해 만들어짐

- DOM Testing Library
  jsdom을 기반으로 하는 테스팅 라이브러리
  jsdom이란 순수하게 자바스크립트로 작성된 라이브러리로 Node.js와 같은 환경에서 HTML과 DOM을 사용할 수 있게 하는 라이브러리

- 리액트 테스팅 라이브러리 이점

1. 실제로 컴포넌트를 렌더링하지 않고도 원하는대로 렌더링되고 있는지 확인가능
2. 테스트에 소요되는 시간을 효과적으로 단축
3. 컴포넌트뿐만 아니라 Provider, 훅 등 리액트를 구성하는 다양한 요소들 테스트 가능

### 8.2.2 자바스크립트 테스트의 기초

기본적인 테스트 코드 작성 방식

1. 테스트할 함수나 모듈을 선정
2. 함수나 모듈이 반환하길 기대하는 값 작성
3. 함수나 모듈의 실제 반환 값 작성
4. 3번의 기대에 따라 2번의 결과가 일치하는지 확인
5. 기대하는 결과를 반환한다면 성공, 다르면 에러

\*Node.js에는 assert라는 모듈을 제공하며 테스트 결과를 확인할 수 있음
=> 테스트 결과를 확인할 수 있게 도와주는 라이브러리를 어설션(assertion) 라이브러리라고 함

- 테스팅 프레임워크
  assertion을 기반으로 테스트를 수행하는 프레임워크
  종류

  - Jest, Mocha, Karma, Jasmine
  - 리액트에서는 Jest를 많이 사용

- Jest 특징
  1. test, expect등의 메서드를 import나 require 같은 구문 없이 사용가능
     => Jest를 비롯한 테스팅 프레임워크는 실행 시 전역 스코프에 기본적으로 넣어주는 값들이 있음

### 8.2.3 리액트 컴포넌트 테스트 코드 작성

- 정적 컴포넌트
  정적 컴포넌트, 별도의 상태가 존재하지 않아 항상 같은 결과를 반환하는 컴포넌트를 테스트하는 방법으로,테스트를 원하는 컴포넌트를 렌더링한 다음, 테스트를 원하는 요소를 찾아 원하는 테스트를 수행하면 된다.

1. beforeEach: 각 테스트(it)를 수행하기 전 실행하는 함수다. 여기서는 StaticComponent를 렌더링한다.
2. describe: 비슷한 속성을 가진 테스트를 하나의 그룹으로 묶는 역할. 꼭 필요한 메스드는 아니지만 테스트가 많아지고 관리가 어려워지면 describe로 묶어서 관리할 수 있다.
3. it: test와 동일하며, test의 축약어이다. it을 사용하는 이유는 테스트 코드를 좀 더 읽기 쉽게 하기 위해서다. describe ... it(something)과 같은 형태로 작성하면 테스트 코드가 문어체같이 표현되어 읽기 쉬워진다.
4. testId: testId는 리액트 테스팅 라이브러리의 예약어로, 웹에서 사용하는 querySelector([data-testid = "${yourId}"])와 동일한 역할을 한다.

\*데이터셋
HTML의 특정 요소와 관련된 임의 정보를 추가할 수 있는 HTML 속성,
HTML의 특정 요소에 data-로 시작하는 속성은 무엇이든 사용할 수 있다.
ex)data-testid => getByTestId

- 동적 컴포넌트

1. setup 함수
   setup 함수는 내부에서 컴포넌트를 렌더링하고, 또 테스트에 필요한 input, button을 반환한다.
2. userEvent.type
   userEvent.type은 사용자가 타이핑하는 것을 흉내내는 메서드다. 이 메서드를 사용하면 사용자가 키보드로 타이핑하는 것과 동일한 작동을 만들 수 있다. userEvent는 @testing-library/react에서 제공하는 fireEvent와 차이가 있다. 기본적으로 userEvent는 fireEvent의 여러 이벤트를 순차적으로 진행해 좀 더 자세하게 사용자의 작동을 흉내 낸다. userEvent.click 수행 시 다음과 같은 fireEvent가 실행된다.

- fireEvent.mouseOver
- fireEvent.mouseMove
- fireEvent.mouseDown
- fireEvent.mouseUp
- fireEvent.click

3. jest.spyOn
   Jest가 제공하는 spyOn은 어떠한 특정 메서드를 오염시키지 않고 실행과 관련된 정보만 얻고 싶을 때 사용한다. 여기서는 window 객체의 메서드 alert를 구현하지 않고 해당 메서드가 실행됐는지만 관찰한다는 의미이다.
4. mockImplementation
   해당 메서드에 대한 모킹 구현을 도와준다. Node.js에는 window.alert가 존재하지 않으므로 해당 메서드를 모의 함수로 구현해아 하는데, 이것이 바로 mockImplementation의 역할이다.
   ==> 여기서는 Node.js 에 존재하지 않는 window.alert를 테스트하기 위해 jest.spyOn을 사용해 alert를 관잘하게 하고, mockImplementation을 통해 alert가 실행됐는지 등의 정보를 확인할 수 있도록 처리한 것이다.

- 비동기 이벤트가 발생하는 컴포넌트
  fetch와 같은 비동기 컴포넌트를 테스트하기 위해서는 MSW(Mock Service Worker)를 사용한다.
  MSW는 Node.js나 브라우저에서 모두 사용할 수 있는 모킹 라이브러리로, 실제 네트워크 요청을 가로채는 방식으로 모킹을 구현한다.
  Node.js나 브라우저에서는 fetch 요청을 하는 것과 동일하게 네트워크 요청을 수행하고, 이 요청을 중간에 MSW가 감지하고 미리 준비한 모킹 데이터를 제공한다.

- 사용자 정의 훅 테스트
  react-hooks-testing-library를 이용하여 테스트

- 테스트 작성할 때 고려할 점
  테스트 커버리지는 얼마나 많은 코드가 테스트되고 있는지를 나타내는 지표일 뿐, 테스트가 잘되고 있는지를 나타내는 것은 아니다.
  테스트 코드를 작성하기 전에 애플리케이션에서 가장 취약하거나 중요한 부분을 파악해야함
