## 3.1 리액트의 모든 훅 파헤치기
### 3.1.1 useState
함수 컴포넌트 내부에서 상태를 정의하고, 이 상태를 관리할 수 있게 해주는 훅

> 기본 사용법
```
import { useState } from 'react'
const [state, setState] = useState(initialState)
```

useState의 첫번째 인수인 state는 초깃값을 넘겨준다. 아무런 값을 넘겨주지 않으면 초깃값은 undefined다.
두 번째 인수인 setState 함수를 사용해 state 값을 변경할 수 있다.

*useState 사용하지 않았을 때
```
function Component() {
	const [, triggerRender] = useState()
    
    let state = 'hello'
    
    function handleButtonClick() {
    	state = 'hi'
        triggerRender()
    }
    
    return(
    	<>
        	<h1>{state}</h1>
            <button onClick={handleButtonClick}>hi</button>
        </>
    )
}
```
state가 업데이트되도 렌더링이 발생할 때 마다 함수가 새로 실행되고 state는 hello로 초기화되므로 state를 변경해도 바뀌지 않는다.

*가상으로 useState를 구현한 코드
```
const MyReact = (function () {
  const global = {};
  let index = 0;

  function useState(initialState) {
    if (!global.states) {
      // 애플리케이션 전체 states 배열을 초기화
      global.states = [];
    }

	// states 정보 조회하여 현재 상태값이 있는지 확인 후,
    // 없다면 초깃값으로 설정
    const currentState = global.states[index] || initialState;
    // states 값을 위에서 조회한 현재 값으로 업데이트

    global.states[index] = currentState;

	// 즉시 실행 함수로 setter
    const setState = (function () {
      // 현재 index를 클로저로 가둬놔서 이후에도 동일한 indx에 접근
      let currentIndex = index;

      return function (value) {
        global.states[currentIndex] = value;
      };
    })();

	// useState 사용할 때마다 index를 하나씩 추가한다.
    // index는 setState에서 사용
    // 하나의 state마다 index가 할당돼 있어 그 index가 배열의 값을 가리키고 필요할 때마다 그 값을 가져온다.
    index = index + 1

    return [currentState, setState]
  }

  function Component(){
    const [value, setValue] = useState(0)

    return ...
  }
})()
```
실제 리액트 코드에서는 useReducer를 이용해 구현돼있다.
함수 컴포넌트 환경에서 state 값을 유지하고 사용하기 위해서 클로저를 활용하는데 클로저 내부에 useState와 관련된 정보를 저장해두고, 필요할 때마다 꺼내놓는 형식

> 실제 리액트 내부 훅
리액트 깃허브 저장소를 보면 <span style="color:red;"> __SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED</span>라는 문구가 있다.
리액트에서 일반 사용자 접근을 차단하고, 프로덕션 코드에서 사용하지 못하게 하기 위해 변수명을 설정
실제로 리액트 팀에서도 접근을 권장하지 않음

* 게으른 초기화
useState의 인수로 특정한 값을 넘기는 함수를 인수로 넣어주는 것을 게으른 초기화(lazy initialization)라고 한다.
무거운 연산, localStorage나 sessionStorage에 대한 접근 등 실행 비용이 많이 드는 경우 게으른 초기화를 사용하는 것이 좋다.

### 3.1.2 useEffect
애플리케이션 내 컴포넌트의 여러 값들을 활용해 동기적으로 부수 효과를 만드는 메커니즘

useEffect의 형태
```
function Component(){
	useEffect(() => {
    	// do something
    },[props,state])
}
 ```
첫 번째 인수로는 실행할 부수 효과가 포함된 함수를, 두 번째 인수로는 의존성 배열을 전달
useEffect는 렌더링할 때마다 의존성에 있는 값을 보면서 이 의존성의 값이 이전과 다른 게 하나라도 있다면 부수 효과를 실행하는 평범한 함수

* 클린업 함수의 목적
useEffect에 이벤트를 추가했을 때 클린업 함수가 없다면 특정 이벤트의 핸들러가 무한히 추가될 수 있다.
클린업 함수가 있다면 이 상황을 방지할 수 있다.
클린업 함수는 생명주기 언마운트와는 차이가 있다. 언마운트는 컴포넌트가 DOM에서 사라지지만, 클린업 함수는 컴포넌트가 리렌더링 됐을 때 이전 상태를 청소해 주는 개념

* 의존성 배열
의존성 배열은 보통 빈 배열을 두거나, 아무런 값도 넘기지 않거나, 원하는 값을 넣어줄 수 있다.
아무런 값을 넣지 않는다면 렌더링할 때마다 실행된다.

* useEffect 구현
가상의 useEffect 구현 코드
```
const MyReact = (function () {
  const global = {};
  let index = 0;

  function useEffect(callback, dependencies) {
    const hooks = global.hooks

    let previousDependencies = hooks[index]

    // 이전 값이 있다면 얕은 비교로 비교해 변경이 일어났는지 확인
    // 없다면 최초 실행으로 변경이 일어난 것으로 간주해 실행 유도
    let isDependenciesChanged = previousDependencies 
      ? dependencies.some((value, idx) => !Object.is(value,previousDependencies[idx]))
      : true
    
    // 변경이 일어났다면 첫 번째 인수인 콜백 함수를 실행
    if(isDependenciesChanged) {
      callback()

      // 다음 훅이 일어날 때를 대비해 index 추가
      index++

      // 현재 의존성을 훅에 다시 저장
      hooks[index] = dependencies
    }

  }
  return { useEffect }

})()
```

* useEffect 사용시 주의점
1. 주석은 최대한 자제하라
2. useEffect 첫 번째 인수에 함수명을 부여
useEffect의 코드가 복잡하고 많아질수록 무슨 일을 하는지 파악하기 어려워지기 때문에 적절한 이름을 사용하면 왜 만들어졌는지 파악하기 쉽다.
ex)
```
useEffect(function logActiveUser(){
	logging(user.id)
},
[user.id]
)
```
3. 거대한 useEffect를 만들지 마라
가능한 useEffect는 간결하게 만들고 만약 큰 useEffect를 만들어야 하면 적은 의존성 배열을 사용하는 여러 개의 useEffect로 분리하는 게 좋다.
4. 불필요한 외부 함수를 만들지 마라.
불필요한 코드가 많아지고 가독성이 떨어질 수 있다.

### 3.1.3 useMemo
비용이 큰 연산에 대한 결과를 저장(메모이제이션)해 두고, 이 저장된 값을 반환하는 훅

첫 번째 인수로는 어떠한 값을 반환하는 생성 함수를, 두 번째 인수로는 해당 함수가 의존하는 값을 전달

useMemo는 렌더링 발생 시 의존성 배열의 값이 변경되지 않았으면 함수를 재실행하지 않고 이전에 기억해 둔 값을 반환하고, 변경됐다면 첫 번째 인수의 함수를 실행한 후 그 값을 반환하고 기억해둔다.

### 3.1.4 useCallback
useMemo가 값을 기억했다면, useCallback은 인수로 넘겨받은 콜백 자체를 기억한다.
useCallback은 특정 함수를 새로 만들지 않고 다시 재사용한다.

첫 번째 인수로 함수를, 두 번째 인수로 의존성 배열을 넣는다.

useMemo와 차이는 메모이제이션을 하는 대상이 변수냐 함수냐일 뿐이고 동일한 역할을 한다.

### 3.1.5 useRef
useRef는 useState와 동일하게 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장하는 공통점이 있지만 큰 차이점이 두 가지 있다.
1. useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경 가능
2. useRef는 그 값이 변해도 렌더링을 발생시키지 않는다.

useRef의 가장 일반적인 사용 예는 바로 DOM에 접근하고 싶을 때다.

개발자가 원하는 시점의 값을 렌더링에 영향을 미치지 않고 보관해 두고 싶다면 useRef를 사용하는 것이 좋다.

### 3.1.6 useContext
useContext는 상위 컴포넌트에서 만들어진 Context를 함수 컴포넌트에서 사용할 수 있도록 만들어진 훅이다.

### 3.1.7 useReducer
useReducer는 useState의 심화 버전으로 볼 수 있다.
useState는 단순한 값을 관리할 때 사용하지만 useReducer는 state 하나가 가져야 할 값이 복잡하고 이를 수정하는 경우의 수가 많아질 때 사용하는 것이 좋다.

2개에서 3개의 인수를 필요로 한다.
1. reducer: useReducer의 기본 action을 정의하는 함수
2. initialState: useReducer의 초깃값
3. init: 필수값은 아니며 초깃값을 지연해서 생성시키고 싶을 때 사용하는 함수이다.

길이가 2인 배열을 반환한다.
1. state: 현재 useReducer가 가지고 있는 값
2. dispatcher: state를 업데이트하는 함수

### 그외
useDebugValue, useImperativeHandle
useLayoutEffect: useEffect보다 먼저 실행되며 DOM은 계산됐지만 화면에 반영되기 전에 하고 싶은 작업이 있을 때 사용하며 사용자 경험을 더 자연스럽게 해준다.

