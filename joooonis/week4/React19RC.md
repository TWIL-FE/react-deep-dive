### Actions

async transtion을 사용하는 함수를 Actions 이라고 부릅니다. Actions은 폼 제출과 같은 비동기 작업을 다룹니다. Actions는 서버로부터의 응답을 기다른 **Pending state**를 제공하고 서버로부터의 응답을 기다리지 않고 우선적으로 UI를 바꿔주는 **Optimistic updates**를 useOptimistic hook을 통해 제공하며 요청이 실패했을때 Error Boundary안에서 Error UI를 표시해주는 **Error handling** 을 지원합니다.

위의 Actions를 다루기 위한 새로운 hook으로 React 19에서 useActionState, useOptimistic이 추가됩니다.

React DOM에서는 `<form>`, `<input>` 그리고 `<button>` element에 action, formAction props가 추가와 함게 useFormStatus hook이 추가되었습니다.

```jsx
'react' : useActionState
'react-dom' : useFormStatus, <form>
```

### use

use 훅은 렌더링 중(기존의 hook을 생각하면 됩니다)에도 Promise와 같은 리소스를 읽을 수 있도록 고안되었습니다.

use가 Promise를 읽는 동안(resolve 이전) React는 Suspend 되어 가장 가까이에 있는 Suspense boundary를 보여줍니다.

use hook을 사용하면 if 문 안에서 Context를 읽을 수 있습니다. 예를 들어 early return문 이후에 use hook을 사용해서 Context를 읽어 올 수 있습니다.

```jsx
import { use } from "react";
import ThemeContext from "./ThemeContext";

function Heading({ children }) {
  if (children == null) {
    return null;
  }

  // This would not work with useContext
  // because of the early return.
  const theme = use(ThemeContext);
  return <h1 style={{ color: theme.color }}>{children}</h1>;
}
```

### Server Component

서버 컴포넌트는 번들링 전에 클라이언트 앱이나 SSR 서버와는 분리된 환경에서 미리 렌더링되는 새로운 유형의 컴포넌트입니다.

이 별도의 환경이 바로 React 서버 컴포넌트에서의 “서버”입니다. 서버 컴포넌트는 빌드 시간에 CI 서버에서 한 번 실행되거나, 각 요청마다 웹 서버를 통해 실행될 수 있습니다.

- 서버 컴포넌트는 빌드 타임에 한 번만 실행되어 정적 HTML을 생성할 수 있습니다. 이는 CI 서버에서 수행되며, 정적 콘텐츠를 클라이언트에 제공하는 방식입니다.
- 또한, 각 클라이언트 요청마다 서버에서 컴포넌트를 렌더링하여 동적으로 HTML을 생성할 수도 있습니다. 이는 웹 서버를 통해 이루어지며, 동적 콘텐츠를 제공하는 데 유용합니다.

아직 bundler나 framework에서 안정적인 지원이 가능하도록 작업중에 있습니다.

https://ko.react.dev/reference/rsc/server-components#noun-labs-1201738-(2)

서버 컴포넌트는 `"use server"`로 표시된다는 오해가 있지만, 서버 컴포넌트에 대한 지시어은 없습니다. `"use server"` 지시어은 서버 액션에 사용됩니다.

서버 컴포넌트는 async/await를 사용하는 새로운 방법을 소개합니다. 비동기 컴포넌트에서 `await`를 사용할 때 React는 중단하고, Promise가 해결될 때까지 렌더링을 기다린 후 다시 렌더링을 재개합니다. 이는 서버/클라이언트 경계를 넘어 서스펜스 스트리밍 지원과 함께 작동합니다.

심지어 서버에서 Promise를 생성하고 클라이언트에서 이를 기다릴 수 있습니다.

비동기 컴포넌트는 [클라이언트에서 지원되지 않으므로](https://ko.react.dev/reference/rsc/server-components#why-cant-i-use-async-components-on-the-client) Promise를 `use`로 기다립니다.

### 기능향상

ref as a prop

이전에는 부모컴포넌트로부터 ref를 전달받기 위해서는 자식 컴포넌트에서 forwardRef를 사용해야 했지만19 버전 부터는 ref를 직접 props로 전달할 수 있습니다.

`<Context>` as a provder

`<Context>`를 `<Context.Provider>` 대신 바로 provider로 사용할 수 있습니다.

Cleanup function for refs

ref prop의 콜백에 unmount시에 동작하는 cleanup function을 넣을 수 있습니다.

```tsx
<input
  ref={(ref) => {
    // ref created

    // NEW: return a cleanup function to reset
    // the ref when element is removed from DOM.
    return () => {
      // ref cleanup
    };
  }}
/>
```

Support for Document Metadata

19버전에서는 컴포넌트 단위로 `<title>`, `<meta>`, `<link>` 태그를 컴포넌트 마다 삽입하여 문서의 metadata를 작성할 수 있습니다. 더이상 react-helmet 같은 라이브러리를 사용하지 않아도 됩니다. client-only app, streaming SSR, Server Component 그 어느 방식도 지원합니다.
