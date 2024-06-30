# Next.js 13

## Root Directory

- root 디렉토리인 app에서 css, 레이아웃, 메타데이터, favicon 등을 설정할 수 있다.
- 폴더나 파일 이름 앞에 언더스코어를 붙이면 라우팅에서 제외된다.

보호되는 파일 명

- `page.js` => 신규 페이지 생성
- `layout.js` => 형제 및 중첩 페이지를 감싸는 신규 레이아웃 생성
- `not-found.js` => ‘Not Found’ 오류에 대한 폴백 페이지
- `error.js` => 기타 오류에 대한 폴백 페이지
- `loading.js` => 데이터를 가져오는 동안 표시되는 폴백 페이지
- `route.js` => API 경로 생성

## Layout

- layout은 여러 경로 간에 공유되는 UI이고, 레이아웃의 상태를 유지하고 다시 렌더링되지 않는다. 페이지를 wrapping 하고, children을 props로 받는다.
- app 폴더에 root layout이 반드시 필요하며, root layout은 웹사이트의 뼈대를 세우는 역할을 한다.
- root layout에서는 head 태그를 작성하는 대신, metadata 객체 내보내기로 적용한다. title, meta 등을 수정할 수 있다.
- root layout은 최상위 트리에서 적용되며 중첩 레이아웃은 하위에 wrapping된다.

## Error handling

- error.js는 에러가 발생했을 때, next.js가 보여주는 컴포넌트다. 반드시 클라이언트 컴포넌트에 있어야 한다.
- 파일이 위치한 트리 레벨에서 중첩된 자식들까지 전부 다 적용된다. 라우팅 별로 서로 다른 에러 UI를 렌더링할 수 있다.
- React Error Boundary를 만들어 같은 레벨에 위치한 컴포넌트를 래핑한다. 전체 페이지 리로드 없이 에러 부분만 복구를 시도한다.
- 리랜더링을 시도하는 `reset()` 함수를 제공한다.
- 같은 레벨의 `layout`에서 발생한 에러는 상위 레벨의 `layout`에서 처리된다. 이유는 `<Layout><Error>{children}</Error></Layout>`의 구조로 페이지가 렌더링되기 때문이다.

작동 방식

- 자식 컴포넌트를 래핑하면, error.js가 React Error Boundary를 생성한다.
- 에러가 발생하면, error.js의 컴포넌트가 fallback 컴포넌트로 사용되어 렌더링된다.
- 다시 시도 가능한 에러는 `reset()` 함수를 통해 사용자가 복구를 시도하도록 유도한다.

## Route Handlers

- 웹 API `req`, `res`를 이용해서 커스텀 핸들러를 만들도록 해준다. app 디렉토리 안에 route 파일로 만든다.
- page 파일과 route 파일을 같은 디렉토리 레벨에 두면 충돌이 발생한다.
- 캐싱, 쿠키, 헤더, 리다이렉트 처리를 할 수 있다.
