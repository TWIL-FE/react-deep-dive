## 리액트 18 버전

- 새로 추가된 훅
  - `useId`
    - 컴포넌트별로 유니크한 값을 생성하는 훅.
    - SSR에서 하이드레이션을 할 때, 랜덤 값의 경우 에러가 발생한다.
    - 이 훅을 사용하면, 모든 컴포넌트와 그 인스턴스까지도 포함해서 유니크한 값을 가질 수 있다. 하이드레이션 문제도 안 난다.
  - `useTransition`
    - 상태 업데이트를 동기에서 비동기로 변경해 UI 변경을 동기적으로 진행.
    - 동시성을 다루는 훅 중 하나로, 느린 렌더링이 앱 전체에 미치는 영향을 경감시킨다.
  - `useDeferredValue`
    - `useTransition`과 같은 역할을 하며, 첫 번째 렌더링이 완료된 이후에 `useDeferredValue`는 지연된 렌더링을 수행한다.
    - 렌더링 우선순위가 낮다고 판단되는 수정할 수 없는 상태(`props` 등)에 적용하도록 한다.
  - `useSyncExternalStore`
    - 동시성을 지원하면서 동시성 이슈인 tearing 현상이 발생, 이전 상태와 현재 상태가 꼬이면서 다른 형태의 컴포넌트가 나타날 수 있음.
    - 지연 렌더링 중 외부 스토어의 상태가 바뀔 수 있어서 다른 상태에 대한 결과가 하나의 컴포넌트에 나타나는 tearing 현상이 발생.
    - 이 훅의 외부 스토어 데이터 변경은 리렌더링을 발생시킬 수 있다.
  - `useInsertionEffect`
    - CSS-in-js 라이브러리 훅으로, 브라우저가 레이아웃을 계산하기 전에, DOM이 실제로 변경되기 전에 동기적으로 실행된다.
    - 실행 순서는 `useInsertionEffect` > DOM 변경 > `useLayoutEffect` > DOM 렌더링 이다.
- react-dom/client
  - `createRoot`
  - `hydrateRoot`
- react-dom/server
  - `renderToPipeableStream`
    - 리액트 컴포넌트를 node.js 서버에서 HTML로 렌더링하는 메서드.
    - 스트림을 지원하고, HTML을 점진적으로 렌더링하고 클라이언트는 script를 삽입할 수 있다.
    - 렌더링 순서나 오래 걸리는 렌더링에 영향을 받지 않는다.
- 자동 배치
  - 상태를 한꺼번에 리렌더링해서 성능을 향상시키는 방법.
  - 기존의 React 이벤트 핸들러 뿐만 아니라, 비동기 함수와 다른 이벤트들 전부 자동 배치를 지원한다.
  - 동기적으로 처리할거면 `flushSync` 쓰기
- 엄격모드
  - 개발자 모드에서만 작동하며, 컴포넌트 형태다.
  - deprecated된 메서드와 API에 대한 경고를 준다.
  - 모든 컴포넌트가 순수 함수인지 확인하기 위해, 훅과 함수 바디 등을 이중으로 호출한다.
- `Suspense` 기능 강화
  - 컴포넌트를 동적으로 가져오고, `React.lazy`와 쌍을 이룬다.
  - 지연 컴포넌트를 로딩하기 전에는 `fallback`을 보여주고, 지연 로딩이 완료되면 컴포넌트를 보여준다.
  - 상대적으로 중요하지 않은 무거운 컴포넌트에 주로 사용한다.

## 리액트 19 버전

- Actions, `useActionState` 등 `form` 관련 API와 메서드, 훅 추가.
- React Server Components, Server Actions 추가.
