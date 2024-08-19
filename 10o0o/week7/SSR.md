# SSR

## SPA
렌더링과 라우팅에 필요한 대부분의 기능을 서버가 아닌 브라우저의 자바스크립트에 의존하는 방식

## SSR
최초에 사용자에게 보여줄 페이지를 서버에서 렌더링해 빠르게 사용자에게 화면을 제공하는 방식

#### 장점
- 최초 페이지 진입이 비교적 빠르다
- 검색 엔진과 SNS 공유 등 메타데이터 제공이 쉽다
- 누적 레이아웃 이동이 적다
- 사용자의 디바이스 성능에 비교적 자유롭다
- 보안에 좀 더 안전하다

#### 단점
- 소스코드를 작성할 때 항상 서버를 고려해야 한다. (window 객체 등 브라우저 환경에서만 쓸 수 있는 것들을 못쓴다)
- 적절한 서버의 구축이 필요하다
- 서비스 지연에 따른 문제가 발생할 수 있다

## SSR을 위한 React API

### renderToString
React 컴포넌트를 렌더링해 HTML 문자열로 반환하는 함수

### renderToStaticMarkup
renderToString과 똑같은 기능을 한다. 다만, data-reactroot와 같은 리액트에서만 사용하는 추가적인 DOM 속성을 만들지 않는다.

> data-reactroot이란? 
>
> data-reactroot은 리액트 컴포넌트의 루트 엘리먼트가 무엇인지 식별하는 역할을 한다. 이후에 자바스크립트를 실행하기 위한 hydrate 함수에서 루트를 식별하는 기준점이 된다.

### renderToNodeStream
renderToString과 동일하나 2가지 차이점

- 브라우저에서 사용이 완전히 불가능하다 (nodejs 환경에 의존)
- 결과물의 타입은 ReadableStream이다. (utf8로 인코딩된 바이트 스트림으로, nodejs 와 같은 서버 환경에서만 만드는게 가능하다.)
  - 브라우저 환경에서 사용은 가능하나 만들 수 없다

### renderToStaticNodeStream
renderToStaticMarkup와 비슷하게, 리액트 속성이 제공되지 않는다. 
hydrate 할 필요 없는 순수 html 결과물이 필요할 때 사용하는 메서드다.

### hydrate

- renderToString
- renderToNodeStream

두 함수로 생선된 HTML 컨텐츠에 자바스크립트 핸들러나 이벤트를 붙히는 역할을 한다.

#### vs Render()
이미 렌더링된 HTML이 있다는 가정하에 작업이 수행된다.

만약 서버에서 넘겨받은 html과 DOM 트리를 비교했을 때, 차이가 생긴다면, 경고 문구가 출력되면서, CSR로 전환해서 랜더링 한다.

> 만약 Date.now()와 같이 어쩔수 없이 차이가 생기는 부분이 있다면, 다음과 같이 처리가능하다
>
> <div suppressHydrationWarning>{new Date().getTime()}<div>
>
> 그러나, 그냥 useEffect로 처리하는 것이 더 권장된다.

## 서버 사이드 렌더링 예제

```jsx
import { hydrate } from 'react-dom';

import App from './components/App';
import { fetchTodo } from './fetch';

async function main() {
  const fetchedResult = await fetchTodo();
  const app = <App todos={result} />;
  const rootEl = document.getElementById('root');

  hydrate(app, el);
}

main()
```

fetchTodo는 서버에서 렌더링하는 결과물과 싱크를 맞추기 위한 fetch


## Next.js

- 디렉토리 기반 라우팅을 지원한다.