# React hooks

## useState

```js
import { useState } from 'react';

const [state, setState] = useState(initialState);
```

### useState를 사용하는 이유

만약 useState를 사용하지 않는다면?

```js
function Component() {
  let state = 'hello';

  const changeState = () => {
    state = 'changed';
  }

  return (
    <>
      <h1>{state}</h1>
      <button onClick={changeState}>change</button>
    <>
  )
}
```

위 코드가 작동하지 않는 이유

- 리엑트의 렌더링은 함수 컴포넌트의 return, 클래스형 컴포넌트의 render() 로 인해 발생한다.
- 실행 결과를 이전의 리엑트 트리와 비교해 리랜더링이 필요한 부분만 업데이트하여 이루어 진다.
- let을 이용한 상태값은, 리엑트에서 변화를 감지하지 못하므로, 리랜더링이 필요한 부분으로 인식하지 못한다.

> 실제 React useState의 구조는 useReducer로 구현되어 있다는 사실만 알려져있다.   
> useReducer + closure을 통해 리랜덩이 되더라도, 해당 환경의 변수를 기억할 수 있어, 변경 감지가 가능하다.
> 더 깊은 코드는 공개되어 있지 않다. (__SECCRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED)

### 게으른 초기화

```js
// 일반적인 초기화
const [count, setCount] = useState(getCount());

// 게으른 초기화
const [count, setCount] = useState(() => getCount());
```

- 초기값이 복잡하거나, 연산이 무거울 때 사용한다.
- 게으른 초기화 함수는 오로지 state가 처음 만들어질 때만 사용된다. (리렌더링 시, 무시)
- 실제로 게으른 초기화를 하지 않을 시, 해당 비용은 해당 컴포넌트의 리랜더링이 발생할때마다 비용이 발생할 것이며, 이는 state의 값에 영향이 하나도 없으므로, 쓸모없는 계산이 된다.
- 따라서 게으른 초기화는 이런 불필요한 비용을 절감시켜준다.

## useEffect

```js
function Component() {
  useEffect(() => {
    // do something
  }, [props, state]);
}
```
- 의존성 배열에 있는 인자들의 변화가 생기면, 첫번째 인자로 받은 콜백 함수가 실행된다.
- 이는 함수형 컴포넌트의 렌더링 방식과 연관이 있는데, 함수형 컴포넌트는 매번 함수를 실행하여 랜더링 된다.
- 이 때, 랜더링할 때마다 의존성에 있는 값을 보면서, 이 의존성에 있는 값이 이전과 다른게 있다면, 첫번째 인자인 콜백 함수가 실행되는 것이다.


### 클린업 함수

```js
useEffect(() => {
  const addMouseEvent = () => {
    // do something
  }

  window.addEventListener('click', addMouseEvent);

  // clean-up
  return () => {
    window.removeEventListener('click', addMouseEvent);
  }
})
```

- useEffect에 이벤트 함수 등을 추가하는 로직이 있다고 가정하자.
- 해당 로직으로 인해 컴포넌트가 리랜더링 될때마다 무한히 이벤트 함수가 등록된다.
- 이를 방지하기 위해 클린업 함수가 필요하다
- 클린업 함수는 콜백이 실행될때마다, 클린업 함수가 존재한다면, 먼저 그 클린업 함수를 실행한 뒤, 콜백을 실행한다.
- 또한 컴포넌트가 완전히 언마운트 될때도 실행된다.

### 주의할 점

- 비용이 큰 연산 혹은 무거운 연산을 포함시키지 않는다
- 외부 함수가 특정 useEffect 안에서만 쓰인다면, 내부에 포함시킨다

```js
function Component({ id }) {
  const doSomethingBig = async (id) => {
    // fetching ...
    // filtering or etc..
  };

  useEffect(() => {
    doSomethingBig(id);
  }, [id]);
}

function Component({ id }) {
  useEffect(() => {
    (async () => {
      // fetching ...
      // filtering or etc..
    })();
  }, [id]);
}
```

> 왜 useEffect 콜백 인수로 비동기 함수가 불가능한가?
>
> 비동기 함수의 응답이 오는 시간 도중, 리랜더링이 일어나서 값이 초기화 된다면, 유저가 초기화 된 값을 맞이해야 하지만, 비동기 함수의 응답이 늦게 온 뒤에 해당 응답으로 값이 덮어씌워질 수 있기 때문이다.
>
> 이런 상태를 경쟁 상태라고 한다.


## useMemo

```js
import { useMemo } from 'react';

const memoizedValue = useMemo(() => expensiveComputation(a, b), []);
```

- 콜백 함수에서는 어떤 값을 반환하는 생성 함수를 넣는다.
- 두번째 인자는 마찬가지로 의존성 배열을 넣는다.

> useMemo는 일반적으로 무거운 연산이나 렌더링에 의한 불필요한 계산을 방지하기 위해 쓰인다.  
> 컴포넌트도 useMemo로 랜더링 가능하지만, 그냥 React.memo 쓰는게 좋다.

## useCallback

- useMemo가 값을 기억했다면, useCallback은 인수로 넘겨받은 함수 자체를 기억한다.

```js
const ChildComponent = memo(({ name, value, onChange }) => {
  useEffect(() => {
    console.log('rendering!', name);
  })

  return (
    <>
      <h1>
        {name} {value ? '켜짐' : '꺼짐'}
      </h1>
      <button onClick={onChange}>Toggle</button>
    </ㅠ>
  )
});

function App() {
  const [status1, setStatus1] = useState(false);
  const [status2, setStatus2] = useState(false);

  const toggle = () => {
    setStatus1(!status1);
  }

  const toggle2 = () => {
    setStatus2(!status2);
  }

  return (
    <>
      <ChildComponent name="1" value={statu1} onChagne={toggle1} />
      <ChildComponent name="2" value={statu2} onChagne={toggle2} />
    </>
  )
}
```

## useRef

```js
const myRef = useRef(initialValue);
```

- 반환값인 객체 내부에 있는 current로 값에 접근 및 변경 가능

### vs useState
- 값이 변해도 랜더링에 영향을 주지 않음

### vs let
- let으로 선언한 변수는, 컴포넌트가 렌더링 되지 않더라도, 기본적으로 메모리에 존재하게 되어, 불필요한 메모리 점유가 생긴다.
- 컴포넌트가 리렌더링 될 경우 let은 initValue로 바뀌지만, ref값은 유지된다.

활용 예시
```js
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}
```

## useContext

### Context란?

- 많은 자식이 중첩된 React 구조의 경우 props 내려주기가 매우 번거롭다
- 이러한 현상을 극복하기 위해 만든 개념이 바로 Context다
- 명시적인 Props 전달 없이도, 선언한 하위 컴포넌트 모두에서 자유롭게 원하는 값을 사용할 수 있다.

```js
const Context = createContext();

function ParentComponent() {
  return (
    <Context.Provider value={{ hello: 'react' }}>
      <ChildComponent />
    </Context.Provider>
  )
}

function ChildComponent() {
  const value = useContext(Context);

  return <>{value ? value.hello : ''}<>;
}
```

## useReducer

```tsx
type State = {
  count: number;
};

type Action = { type: 'up' | 'down' | 'reset'; payload?: State };

function init(count: State): State {
  return count;
}

const initialState: State = { count: 0 };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'up':
      return { count: state.count + 1 };
    case 'down': 
      return { count: Math.max(0, state.count - 1) };
    case 'reset':
      return init(action.payload || { count: 0 });
    default: 
      throw new Error(`Unexpected action type ${action.type}`);
  }
}

export default function App() {
  const [state, dispatcher] = useReducer(reducer, initialState, nit);

  function handleUpButtonClick() {
    dispatcher({ type: 'up' });
  }

  function handleDownbuttonclick() {
    dispatcher({ type: 'down' });
  }

  function handleResetButtonClick() {
    dispatcher({ type: 'reset' });
  }

  return (
    <div>
      <h1>{state.count}</h1>
      <button onClick={handleUpButtonClick}>+</button>
      <button onClick={handleDownButtonClick}>-</button>
      <button onClick={handleResetButtonClick}>reset</button>
    </div>
  )
}
```

- 사전에 정의된 dispatcher type 으로만 업데이트 가능
- 상세 정의는 컴포넌트 밖에다 둔다.
- state를 사용하는 로직과, 이를 관리하는 로직의 분리할 수 있어 관리가 요이해진다.
- 3번째 인수인 게으른 초기화 함수는 굳이 사용하지 안하도 된다.

## useImperativeHandle

### forwardRef

- ref는 useRef에서 반환한 객체로, 리엑트 컴포넌트의 props인 ref에 넣어 HTMLElement에 접근하는 용도로 흔히 사용된다.
- 상위 컴포넌트에서 하위 컴포넌트로 ref를 전달할 때, 해당 훅을 사용한다.

```jsx
const ChildComponent = forwardRef((props, ref) => {
  return (
    // ...
  );
})

funciton ParentComponent() {
  const ref = useRef();

  return (
    <ChildComponent ref={ref} />
  )
}
```

### useImperativeHandle 이란?
 
 부모에게서 넘겨받은 ref를 원하는 대로 수정 가능한 훅

```jsx
const Input = forwardRef((props, ref) => {
  useImperativeHandle(
    ref,
    () => ({
      alert: () => alert(props.value);
    }),
    [props.value],
  );
})
```
- 자식 컴포넌트의 함수 호출을 부모 컴포넌트에서 가능하도록 해준다

## useLayoutEffect

함수의 시그니처는 useEffect와 동일하나, 모든 DOM의 변경 후에 동기적으로 발생한다

1. 리액트가 DOM을 업데이트
2. useLayoutEffect 실행
3. 브라우저에 번경 사항을 반영
4. useEffect 실행

해당 훅의 실행은 동기적으로 발생한다. 직, useLayoutEffect의 실행이 종료될 때 까지 react는 랜더링이 불가능하게 된다.

사용 예시)
- DOM은 계산됐지만, 반영되기 전 작업을 하려 할 때
- DOM기반 애니메이션, 스크롤 위치 제어

## useDebugValue

프로덕션에서 사용하는 훅은 아니다.

개발과정에서 디버깅하고 싶은 정보를 개발자 도구에서 확인 가능하게 해준다.

```jsx
function useDate() {
  const date = new Date();

  useDebugValue(date, date => `현재시간: ${date.toISOString()}`);
  return date;
}
```

## 훅의 규칙
- 최상위에서만 훅을 호출해야 한다. (반복문, 조건문, 중첩함수 내 사용 X)
  - 동일한 순서로 훅의 호출을 보장하기 위함
- 동일한 순서가 보장되지 않을 시, 에러를 발생시킨다

- 호출 가능한 곳은 리액트 함수 컴포넌트, 사용자 정의 훅 2가지 경우이다.

```jsx
function Form() {
  const [name, setName] = useState('Mary');

  if (name) {
    useEffect(() => {
      // do something
    });
  }

  const [surname, setSurname] = useState('Poppins');

  useEffect(() => {
    // do something
  });
}
```

## 사용자 정의 훅
```ts
function useFetch<T>(
  url: string,
  { method, body }: { method: stirng; body?: XMLHttpRequestBodyInit },
) {
  const [result, setResult] = useState<T | undefined>();
  const [isLoading, setIsLoading] = useState<boolean>(false);
  const [ok, setOk] = useState<boolean | undefined>();
  const [status, setStatus] = useState<number | undefined>();

  useEffect(() => {
    const aborController = new AbortController();

    (async () => {
      setIsLoading(true);

      const response = await fetch(
        // something
      );

      setOk(response.ok);
      setStatus(response.status);

      if (response.ok) {
        const apiResult = await response.json();
        setResult(apiResult);
      }

      setIsLoading(false);
    })();

    return () => {
      aborController.abort();
    }
  }, [url, method, body]);

  return { ok, result, isLoading, status }
}
```

- 무조건 훅 앞에 "use" 붙여야지 에러가 발생하지 않고, 리액트 훅의 감지에 영향을 준다

## 고차 컴포넌트

```jsx
const withLoginComponent(Component) {
  return function (props) {
    const { isLoginRequired, ...restProps } = props

    if (isLoginRequired) {
      return <>로그인이 필요합니다.</>
    }

    return <Component [...restProps] />;
  }
}

const Component = withLoginComponent((props: { value }) => {
  return <h3>{props.value}</h3>
});

```
- i18n의 withTranslation 함수

## Custom Hook vs 고차 컴포넌트

- 훅을 사용해야 하는 경우
  - useEffect, useState와 같이 리액트에서 제공하는 훅으로만 로직  가능한 경우
- 고차 컴포넌트를 사용해야 하는 경우
  - 렌더링 결과물에 영향을 미치는 공통 로직이 존재하는 경우