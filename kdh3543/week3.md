10 리액트 17과 18의 변경 사항 \*리액트를 사용하고 있는 대부분의 사이트들은 16버전을 사용

10.1 리액트 17버전
리액트 17 버전과 16버전의 차이는 거의 없고 새롭게 추가된 기능이 없음
호환성이 깨지는 변경사항을 최소화했다는게 가장 큰 특징
10.1.1 리액트의 점진적인 업그레이드
16에서 17버전으로 업그레이드 시 크게 어려움이 없음

앞으로의 점진적인 버전 업데이트를 위한 리액트 언어의 빌드업(업데이트를 위한 업데이트)
10.1.2 이벤트 위임 방식의 변경
16버전까지는 이벤트가 document에서 위임이 됐지만, 17부터는 최상단 트리인 루트에서 이벤트 위임이 됨

이벤트 위임을 통해 불필요한 이벤트 리스너의 추가 및 제거를 방지하고, 이를 통해 애플리케이션의 성능을 유지
최상단으로 수정함으로써 각 이벤트는 해당 리액트 컴포넌트 트리 수준으로 격리되므로 이벤트 버블링으로 인한 혼선을 방지할 수 있음
단점

이벤트 위임을 잘못 사용하면 이벤트가 상위 요소로 전파되는 과정에서 원치 않는 이벤트가 발생할 수 있음
=> 그로 인한성능 저하 유발 가능성
=> 이벤트 핸들러 내에서 이벤트 타겟을 정확히 식별
10.1.3 import React from 'react'가 필요가 없음
import React 구문을 없애므로 인해 번들링 크기를 줄이고, 컴포넌트 작성을 더욱 간결하게 해줌

10.2 리액트 18 버전 살펴보기
리액트 18의 가장 큰 변경점은 동시성 지원, 다양한 훅 추가

10.2.1 새로 추가된 훅
useId
컴포넌트별로 유니크한 값을 생성하는 새로운 훅
=>하나의 컴포넌트가 여러 군데 재사용되거나, 서버 사이드 렌더링 환경에서 하이드레이션이 발생할 때 서버와 클라이언트에서 동일한 값을 가져야 될 때 등의 경우에 유니크한 값을 생성시켜줌

useTransition
UI 변경을 가로막지 않고 상태를 업데이트 할 수 있는 리액트 훅
=> 무거운 렌더링 작업을 뒤로 미룸으로써, 사용자 경험을 최적화하기 좋음
=> 로딩 화면 및 진행 중인 렌더링을 버리고 새로운 상태값으로 렌더링 하는 등의 동시성을 지원

useDeferredValue
리액트 컴포넌트 트리에서 리렌더링이 급하지 않은 부분을 지연할 수 있게 도와주는 훅
=> useTransition과 비슷한 역할을 하지만 useTransition은 state 값을 업데이트 하는 함수를 사용하고, useDeferredValue는 state값 자체를 사용

10.2.2 react-dom/client \*리액트 18로 버전 업그레이드 시 반드시 index.ts(js)를 변경해야 함

createRoot
기존의 render 메서드를 리액트 18로 버전 업그레이드시 createRoot와 render를 함께 사용해야 함

hydrateRoot
서버 사이드 렌더링 애플리케이션에서 하이드레이션을 하기 위한 메서드
자체적으로 서버 사이드 렌더링 구현시 수정 필요

10.2.3 react-dom/server
renderToPipeableStream
리액트 컴포넌트를 HTML로 렌더링하는 메서드
서버에서는 Suspense를 사용해 빠르게 렌더링이 필요한 부분을 우선으로 하고, 값비싼 연산으로 구성된 부분은 이후에 렌더링되게 함
=> hydrateRoot를 호출하면 서버에서는 HTML을 렌더링하고, 클라이언트의 리액트에서는 이벤트만 추가함으로써 첫 번째 로딩을 빠르게 수행할 수 있음
=> 기존 renderToNodeStream은 렌더링 순서대로 해야 되기 때문에 중간에 렌더링이 오래걸리는 작업이 있다면 나머지 렌더링이 안되는 문제가 있음
10.2.4 자동배치(Automatic Batching)
리액트가 여러 상태 업데이트를 하나의 리렌더링으로 묶어 성능을 향상

```
import { Profiler, useCallback, useEffect, useState } from 'react';

function App() {
  const sleep = (ms) => {
    return new Promise((res) => setTimeout(res, ms));
  };

  const [count, setCount] = useState(0);
  const [flag, setFlag] = useState(false);

  const callback = useCallback((id, ph, ac, ba, st, co) => {
    console.group(ph);
    console.table({ id, ph, co });
    console.groupEnd();
  }, []);

  useEffect(() => {
    console.log('rendered');
  });

  function handleClick() {
    sleep(3000).then(() => {
      setCount((c) => c + 1);
      setFlag((f) => !f);
    });
  }
  return (
    <div className="App">
      <Profiler onRender={callback}>
        <header className="App-header">
          <button onClick={handleClick}>click</button>
          <h1 style={{ color: flag ? 'blue' : 'black' }}>{count}</h1>
        </header>
      </Profiler>
    </div>
  );
}

export default App;
```

react 17

![image](https://github.com/TWDL-FE/react-deep-dive/assets/91539013/136cb269-1d41-4b18-a646-cfeccc987ce1)

react 18

![image](https://github.com/TWDL-FE/react-deep-dive/assets/91539013/54c671e1-3f9b-4af1-887b-b3badf277870)

=> 리액트 17에서도 비동기 이벤트에서는 자동 배치가 이뤄지지 않아 일관성이 없었지만 리액트 18 버전붙터는 루트 컴포넌트를 createRoot를 사용해서 만들면 모든 업데이트가 배치 작업으로 최적화할 수 있게 됐음

10.2.5 더욱 엄격해진 엄격 모드 \*리액트 엄격 모드
리액트 엄격 모드는 개발자 모드에서만 동작하고, 리액트 어플리케이션에서 발생할 수 있는 잠재적 버그를 찾는 데 도움이 되는 컴포넌트

더 이상 안전하지 않은 특정 생명주기를 사용하는 컴포넌트 경고
componentWillMount, componentWillReceiveProps, componentWillUpdate는 더 이상 사용할 수 없어 경고 로그가 뜬다.
문자열 ref 사용 금지
문자열로 값을 주는 것은 여러 컴포넌트에 걸쳐 사용될 수 있으므로 충돌 여지가 있다.
리액트 공식 문서에서는 문자열 ref 대신 콜백 ref를 사용하는 것을 권장

```
// 문자열 ref
class UnsafeClass extends Component {
	componentDidMount() {
    	console.log(this.refs.myInput)
    }

    render(){
    	return(
        	<div>
            	<input ref="myInput" type="text" />
            </div>
        )
    }
}

// 콜백 ref
const callbackRef = useCallback((node) => {
	// 후속 작업, node에는 div 엘리먼트의 값이 들어오게 된다.
},[])

<div ref={callbackRef} />

```

findDOMNode 사용에 대한 경고
React에서 주어진 클래스 인스턴스를 바탕으로 트리를 탐색해 DOM 노드를 찾을 수 있는 findDOMNode를 지원했지만
findDOMNode 사용시 부모가 특정 자식만 별도로 렌더링하는게 가능해져 리액트가 추구하는 트리 추상화 구조를 무너뜨림
findDOMNode는 항상 첫 번째 자식을 반환하는데, Fragment 사용 시 어색해진다는 문제점이 있음
항상 변하지 않는 단일 노드를 반환하는 정적인 컴포넌트에만 동작하는 반쪽짜리 메서드임
구 ContextAPI 사용 시 경고
예상치 못한 side-effects 검사
리액트 엄격 모드는 다음 내용을 의도적으로 이중 호출시킨다.
클래스 컴포넌트의 constructor, render, shouldComponentUpdate, getDerivedStateFromProps
클래스 컴포넌트의 setState 첫 번째 인수
함수 컴포넌트 body
useState, useMemo, useRender에 전달되는 함수
=> 함수형 프로그래밍의 원칙에 따라 모든 컴포넌트는 항상 순수하다고 가정하는데, 항상 순수한 결과물을 내는지 개발자에게 확인시켜 주기 위해 로그가 두 번 실행됨

리액트 18에서 추가된 엄격 모드
향후 리액트에서 컴포넌트가 마운트가 해제된 상태에서도 컴포넌트 내부의 값을 유지할 수 있는 기능을 제공할 예정인데, 이를 위해 컴포넌트가 최초 마운트될 때 자동으로 모든 컴포넌트를 마운트 해제하고 두번째 마운트에서 이전 상태를 복원하게 된다. 이 기능은 오직 개발 모드에서만 적용됨

10.2.6 Suspense 기능 강화
Suspense는 컴포넌트를 동적으로 가져올 수 있게 도와주는 기능인데, 첫 번째 인수는 fallback props로 지연시켜 불러온 컴포넌트를 불러오지 못했을 때 보여주는 fallback이다.
children으로는 React.lazy로 선언한 지연 컴포넌트를 받는다.
=> 지연 컴포넌트를 로딩하기 전에 fallback을 보여주고, 로딩 완료 시 fallback 대신 지연 컴포넌트를 보여주게 된다.

기존 Suspense 문제점
Suspense 자식으로 비동기 데이터에 의존하지 않는 컴포넌트는 Suspense 작업 진행 중임에도 불구하고, effect가 실행되고 있었다.
Suspense는 서버에서 사용할 수 없었음
기존 서버 사이드 렌더링 구조에서 Suspense 활용시 useMount와 같은 훅을 구현해서 클라이언트에서만 작동하도록 처리해야 했음
리액트 18에서 Suspense
마운트되기 직전에 실행된 effect 수정으로 컴포넌트가 화면에 노출될 때 effect가 실행됨
기존에는 컴포넌트 스스로가 Suspense에 의해 현재 보여지고 있는지, 숨겨져 있는지 알 방법이 없었지만 Suspense에 의해 노출이 된다면 useLayoutEffect의 effect가, 가려진다면 useLayoutEffect의 cleanUp이 실행됨
서버에서 Suspense 사용이 가능
Suspense 내 스로틀링 추가로 중첩된 Suspense의 fallback이 있다면 자동으로 스로틀되어 최대한 자연스럽게 보여지게 됨

- 리액트 19
  React Compiler
  리액트 19의 가장 큰 업데이트 부분은 리액트 컴파일러
  기존에 불필요한 렌더링을 줄이기 위해 메모이제이션을 위해 useCallback, useMemo, memo등을 사용해야 했지만 남발시 오히려 성능이 저하되는 단점이 있기도 했음
  리액트 19는 리액트 컴파일러가 자동으로 리액트 코드를 최적화해 메모이제이션을 위한 훅을 사용할 필요가 없음
  Actions
  form에 대한 처리를 쉽게 할 수 있게 했음
  사용자가 Form Data 제출시 API 요청 처리를 한 다음 응답을 처리하는데 이전 버전에서는 대기 상태, 오류, optimistic updates(낙관적 업데이트), 연속 요청 등을 수동으로 처리했음 \*기존 Actions

```
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, setIsPending] = useState(false);

  const handleSubmit = async () => {
    setIsPending(true);
    const error = await updateName(name);
    setIsPending(false);
    if (error) {
      setError(error);
      return;
    }
    redirect("/path");
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

리액트 19에서는 이런 처리를 위한 훅이 제공됨

useTransition
useTransition을 사용해 pending State(대기 상태)를 처리 할 수 있음

```
function UpdateName({}) {
  const [name, setName] = useState("");
  const [error, setError] = useState(null);
  const [isPending, startTransition] = useTransition();

  const handleSubmit = async () => {
    startTransition(async () => {
      const error = await updateName(name);
      if (error) {
        setError(error);
        return;
      }
      redirect("/path");
    })
  };

  return (
    <div>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button onClick={handleSubmit} disabled={isPending}>
        Update
      </button>
      {error && <p>{error}</p>}
    </div>
  );
}
```

useActionState
액션의 일반적인 경우를 더 쉽게 처리할 수 있게 함
Action을 하는 function을 인자로 받고, 래핑된 Action을 반환한다.
래핑된 Action호출 시 useActionState는 Action의 마지막 결과를 데이터로 반환하고, 대기중인 상태라면 대기중으로 반환한다.

```
const [error, submitAction, isPending] = useActionState(async (previousState, newName) => {
  const error = await updateName(newName);
  if (error) {
    // You can return any result of the action.
    // Here, we return only the error.
    return error;
  }

  // handle success
});
```

useOptimistic
데이터 변형 수행 시 비동기 요청이 진행되는 동안 최종 상태를 낙관적으로 표시하는데 리액트 19에서는 이를 더 쉽게 하기 위해 useOptimistic 훅을 새로 추가

```
function ChangeName({currentName, onUpdateName}) {
  const [optimisticName, setOptimisticName] = useOptimistic(currentName);

  const submitAction = async formData => {
    const newName = formData.get("name");
    setOptimisticName(newName);
    const updatedName = await updateName(newName);
    onUpdateName(updatedName);
  };

  return (
    <form action={submitAction}>
      <p>Your name is: {optimisticName}</p>
      <p>
        <label>Change Name:</label>
        <input
          type="text"
          name="name"
          disabled={currentName !== optimisticName}
        />
      </p>
    </form>
  );
}
```

updateName 요청이 진행되는 동안 optimisticName을 렌더링하고, 업데이트 완료 혹은 오류 발생 시 리액트는 자동으로 currentName값으로 다시 전환한다.

use
use hook 사용으로 promise와 context를 쉽게 처리할 수 있다.
기존에 서버 데이터를 fetching하기 위해 useEffect를 사용
// useEffect 사용

```
function Person() {
  const [person, setPerson] = useState(null);

  useEffect(() => {
    fetchPerson().then((data) => setPerson(data));
  }, []);

  if (!person) return <h1>불러오는 중...</h1>;

  return <h1>{person.name}</h1>;
}

// use 사용
function Person() {
  const person = use(fetchPerson());

  return <h1>{person.name}</h1>;
}

function App() {
  return (
    <Suspense fallback={<h1>Loading...</h1>}>
      <Person />
    </Suspense>
  );
}
```

use는 데이터를 가져오는 동안 promise를 반환하는 함수에 사용될 수 있다.
Suspense를 사용해 가져오는 동안 UI를 표시해주고, promise가 resolve 될 때 데이터를 보여줄 수 있다.

context에 접근하기 위해 useContext()를 사용해 접근하려는 context를 생성한 후 접근할 수 있었고, 접근을 위해서는 해당 Context Provider로 wrapping이 되어있어야 함
// useContext 사용

```
const UserContext = createContext();

function User() {
  const user = useContext(UserContext);
  return <h1>안녕 {user.name}!</h1>;
}

function App() {
  return (
    <UserContext.Provider value={{ name: "Yongveloper" }}>
      <User />
    </UserContext.Provider>
  );
}

// use 사용
const UserContext = createContext();

function User() {
  const user = use(UserContext);
  return <h1>안녕 {user.name}!</h1>;
}

function App() {
  return (
    <UserContext value={{ name: "Yongveloper" }}>
      <User />
    </UserContext>
  );
}
```

useContext 대신 use로 사용할 수 있고 Context.Provider 대신 Context를 프로바이더로 렌더링할 수 있음
