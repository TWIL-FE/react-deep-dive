## 5.1 상태 관리는 왜 필요한가?
웹 애플리케이션에서의 상태 분류
1. UI
2. URL: https://www.airbnb.co.kr/rooms/4132?adults=2 => adults, roomId 값
3. 폼: 로딩(loading) 중인지, 제출(submit)됐는지 등
4. 서버에서 가져온 값: API 요청 등

### 5.1.1 리액트 상태 관리의 역사
* Flux 패턴의 등장
기존 리액트에서 상태가 어떻게 변했는지 추적하기 위해 양방향 데이터 바인딩을 사용했다.
뷰(HTML)가 모델(자바스크립트)을 변경할 수 있고, 반대의 경우 모델도 뷰를 변경할 수 있다.
=> 코드의 양이 많아지고 변경 시나리오가 복잡해질수록 관리가 어려워짐
이를 해결하기 위해 단방향 데이터 흐름으로 변경(Flux 패턴)

 Flux패턴의 용어
 1. 액션(action): 어떠한 작업을 처리할 액션과 그 액션 발생 시 함께 포함시킬 데이터를 의미. 액션 타입과 데이터를 가각각 정의해 디스패처로 보냄
 2. 디스패처(dispatcher): 액션을 스토어에 보내는 역할(콜백 함수 형태)
 3. 스토어(store): 실제 상태에 따른 값과 상태를 변경할 수 있는 메서드를 가지고 있다. 액션의 타입에 따라 어떻게 변경할지 정의돼 있음
 4. 뷰(View): 리액트의 컴포넌트에 해당하는 부분으로, 스토어에 만들어진 데이터를 가져와 화면을 렌더링하는 역할
 ```
 type StoreState = {
 	count: number
 }
 
 type Action = { type: 'add', payload: number }
 
 function reducer(prevState: StoreState, action: Action) {
 	const { type: ActionType } = action
    if(ActionType === 'add') {
    	return {
        	count: prevState.count + action.payload,
        }
    }
    
    throw new Error(`Unexpected Action [${ActionType}]`)
 }
 
 export default fuction App() {
 	const [state, dispatcher] = useReducer(reducer, { count: 0})
    
    function handleClick() {
    	dispatcher({ type: 'add', payload: 1 })
    }
    
    return (
    	<div>
        	<h1>{state.count}</h1>
            <button onClick={handleClick}>+</button>
        </div>
    )
 }
 ```
 
 * 시장 지배자 리덕스 등장
 리덕스는 대표적인 Flux 패턴을 사용하는 상태관리 라이브러리이고 Elm 아키텍처를 도입했다.
 리덕스는 하나의 상태 객체를 스토어에 저장해 두고, 이 객체를 업데이트하는 작업을 디스패치해 업데이트를 수행한다.
 
 하나의 글로벌 상태 객체를 통해 하위 컴포넌트에 상태를 전파
 
 장점: prop 내려주기 문제를 해결, 스토어가 필요한 컴포넌트는 connect만 쓰면 스토어에 바로 접근 가능
 
 단점: 하나의 상태를 바꾸려고 해도 타입 선언, creator, dispatcher, selector 등 해야 할 일이 많음
 
 * Context API와 useContext
 준비해야 하는 보일러플레이트가 많은 리덕스를 사용하기에도 부담스럽고, 깊은 컴포넌트까지 prop으로 계속 전달해주기에도 번거로움
 => 16.3 버전에서 전역 상태를 하위 컴포넌트에 주입할 수 있는 Context API 출시
 => prop로 번거롭게 넘겨주지 않아도 사용이 가능하지만, 상태 관리가 필요없는 컴포넌트도 렌더링이 되기 때문에 사용 시 주의가 필요하다.
 
 * 훅의 탄생, React Query와 SWR
 두 라이브러리 모두 외부에서 데이터를 불러오는 fetch를 관리하는 데 특화된 라이브러리지만, API 호출에 대한 상태를 관리하기 때문에 HTTP 요청에 특화된 상태 관리 라이브러리이다.
 
 	* SWR 사용 코드
    ```
    import React from 'react'
    import useSWR from 'swr'
    
    const fetcher = (url) => fetch(url).then((res) => res.json())
    
    export default function App(){
    	const { data, error } = useSWR(
        	`https://api.github.com/repos/vercel/swr',
            fetcher,
        )
        
        if(error) return 'An error has occured'
        if(!data) return 'Loading...'
        
        return (
        	<div>
            	<p>{JSON.stringify(data)}</p>
            </div>
        )
    }
    ```
    useSWR의 첫 번째 인수로 조회할 API 주소를, 두 번째 인수로 fetch를 넘겨준다.
    첫 번째 인수인 API 주소는 키로도 사용되며, 이후에 다른 곳에서 동일한 키로 호출시 useSWR이 관리하고 있는 캐시의 값을 활용
    
* Recoil, Zustand, Jotai, Vlatio에 이르기까지
```
//Recoil
const counter = atom({ key:'count', default: 0 })
const todoList = useRecoilValue(counter)

//Jotai
const countAtom = atom(0)
const [count, setCount] = useAtom(countAtom)

//Zustand
const useCounterStore = create((set) => ({
  count: 0,
  increase: () => set((state) => ({ count: state.count + 1 })),
}))
    
const count = useCounterState((state) => state.count)

//Valtio
const state = proxy({ count: 0 })
const snap = useSnapshot(state)
state.count++
```
16.8 이상의 버전을 요구하고 있고 작은 크기의 상태를 효율적으로 관리하기 좋은 라이브러리들이다.

## 5.2 리액트 훅으로 시작되는 상태 관리
### 5.2.1 가장 기본적인 방법: useState와 useReducer
*useState로 만든 기본적인 훅
```
function useCounter(init: number = 0) {
	const[counter, setCounter] = useState(init)
    
    function inc() {
    	setCounter((prev) => prev + 1)
    }
    
    return {counter, inc}
}

function Counter1() {
	const {counter, inc} = useCounter()
    
    return (
    	<>
        	<h3>Counter: {counter}</h3>
            <button onClick={inc}>+</button>
        </>
    )
}
```
useCounter라는 훅이 없었다면 해당 함수를 각각의 컴포넌트에서 구현해야 함
훅 내부에서 관리해야 하는 상태가 복잡하거나 상태를 변경할 수 있는 시나리오가 다양해진다면 훅으로 코드를 격리해 제공할 수 있다는 장점이 더욱 크게 드러남
useReducer 또한 지역 상태를 관리할 수 있는 훅

### 상태 관리 라이브러리 Recoil, Jotai, Zustand
1. Recoil과 Jotai는 Context와 Provider, 훅을 기반으로 가능한 작은 상태를 효율적으로 관리하는 데 초점을 맞추고 있음
2. Zustand는 리덕스와 비슷하게 하나의 큰 스토어를 기반으로 상태를 관리하는 라이브러리
=>큰 스토어는 Context가 아니라 스토어가 가지는 클로저를 기반으로 생성되며, 상태 변경시 이 상태를 구독하고 있는 컴포넌트에 전파해 리렌더링하는 방식

* 페이스북이 만든 상태 관리 라이브러리 Recoil
아직 실험적으로 개발되고 운영되는 라이브러리, 실제 프로덕션에 사용하기에는 안정성, 성능, 사용성 등을 보장할 수 없고 마이너 버전이 변경돼도 호한성이 깨지는 위험성을 안고 있다.

  * RecoilRoot
  Recoil을 사용하기 위해선 RecoilRoot를 애플리케이션 최상단에 선언해야 한다.
  특징
  1. Recoil의 상태값은 RecoilRoot로 생성된 Context의 스토어에 저장된다.
   2. 스토어의 상태값에 접근할 수 있는 함수들이 있으며, 이 함수를 활용해 상태값에 접근 또는 변경할 수 있다.
   3. 값의 변경이 발생하면 이를 참조하는 하위 컴포넌트에 모두 알림   
   
   * atom
   Recoil의 최소 상태 단위
   atom은 key 값을 필수로 가지며, 다른 atom과 구별하는 식별자가 된다.
   
   * useRecoilValue
   atom의 값을 읽어오는 훅
   
   * useRecoilState
   atom의 값을 가져오고, 또 이 값을 변경할 수 있는 훅이다.
   useState와 매우 유사한 구조
  
  
  
  *특징
  페이스북 팀에서 주도적으로 개발하고 있기 때문에 어떤 라이브러리보다 잘 지원될 것으로 기대
  리덕스와 달리 redux-saga나 redux-thunk 등 미들웨어를 사용하지 않더라도 비동기 작업을 수월하게 처리할 수 있다.
  
* Recoil보다 더 유연한 Jotai
Jotai는 Recoil의 atom 모델에 영감을 받아 만들어진 상태 관리 라이브러리다.
상향식(bottom-up) 접근법을 취하고 있는데, 리덕스와 달리 작은 단위의 상태를 위로 전파할 수 있는 구조
=> 리액트 Context의 문제점인 불필요한 리렌더링을 해결하고자 설계돼 있다.

  * atom
  최소 상태 단위를 의미하며, Recoil과 다르게 atom 하나만으로도 상태를 만들 수도, 이에 파생된 상태도 만들 수 있다.
  Recoil과 다르게 별도의 key를 넘겨주지 않아도 된다.
  
  * useAtomValue
  atom에 따로 상태를 저장하지 않고, atom의 값이 변경되면 useAtomValue로 값을 사용하는 쪽에서 최신 값의 atom을 사용해 렌더링 할 수 있다.
  
  * useAtom
  useState와 동일한 형태의 배열을 반환한다. 첫 번째로는 atom의 현재 값을 나타내는 useAtomValue 훅의 결과를 반환하며, 두 번째로 useSetAtom 훅을 반환하는데, atom을 수정할 수 있는 기능을 제공

  * 특징
  Recoil과 유사하지만 Recoil의 단점을 극복하기 위한 노력이 보인다.
  atom 개념을 도입하면서도 API가 간결(키 불필요, selector 불필요)
  
* 작고 빠르며 확장에도 유연한 Zustand
리덕스에 영감을 받아 만들어진 라이브러리여서 중앙 집중형으로 활용해 스토어 내부에서 상태를 관리함

	*Zustand Code
    ```
    import { create } from 'zustand'
    
    const useCounterStore = create((set) => ({
    	count: 1,
        inc: () => set((state) => ({ count: state.count + 1})),
        dec: () => set((state) => ({ count: state.count - 1}))
    }))
    
    function Counter() {
    	const { count, inc, dec } = useCounterStore()
        
        return (
        	<div class='counter'>
            	<span>{count}</span>
                <button onClick={inc}>up</button>
                <button onClick={dec}>down</button>
            </div>
        )
    }
    ```
create를 사용해 스토어를 만들고, 반환 값으로 이 스토어를 컴포넌트 내부에서 사용할 수 있는 훅을 받음, getter와 setter 모두 접근이 가능하다.

  * 특징
  Zustand는 많은 코드 없이 빠르게 스토어를 만들고 사용할 수 있다는 큰 장점이 있다.
  API가 복잡하지 않고 사용이 간단해 쉽게 접근할 수 있는 상태 관리 라이브러리이다.
  타입스크립트 기반으로 작성돼 있기 때문에 자연스럽게 타입스크립트를 사용할 수 있음
  리덕스와 마찬가지로 미들웨어를 지원
  create의 두 번째 인수로 원하는 미들웨어를 추가하면 된다.
  => 스토어 데이터를 영구 보존할 수 있는 persist, 복잡한 객체를 관리하기 쉽게 도와주는 immer 등 이 미들웨어를 사용하면 상태를 sessionStorage에 추가로 저장하는 등의 기본적인 상태 관리 작동 외에 추가적인 작업을 정의할 수 있음

