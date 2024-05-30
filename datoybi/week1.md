## Week1
# 01장 리액트 개발을 위해 꼭 알아야 할 자바스크립트


### 리엑트의 가장 큰 특징

- 리엑트의 `상태변화는 단방향`으로 그리고 `명시적`으로 이뤄진다. 단방향 바인딩은 항상 변화를 감지하고 업데이트하는 코드를 매번 작성해야 하며, 이에 따라 코드의 규모가 증가하는 등 단점이 있다, 데이터 흐름의 변화가 단순하기 때문에 코드를 읽기가 쉽고 버그를 야기할 가능성이 비교적 적다.

## 리엑트를 위한 JS

- 0bject.js
    - 동등비교(===)보다 조금 더 개발자가 기대하는 방식으로 동작하나 객체의 비교는 동등비교와 똑같다.
    - React에서는 객체간 얕은 비교(첫번째 깊이만 비교)만 수행한다.
        - 따라서 prop이 props.counter.counter인 경우 React.memo는 변경된 값이 없음에도 불구하고 메모이제이션된 컴포넌트를 반환하지 못한다.
- useEffect나 useCallback의 콜백 함수에 이름을 붙여 어떤 역할을 하는지 알 수 있게 하자
    
    ```jsx
    useEffect(function apiRequest() {
    	// ...
    }, [])
    ```
    
- 클로저
    - useState(prev ⇒ … )에서 사용되는데 prev값을 기억할 수 있는 이유
    - 클로저를 많이 사용할 경우 성능이 좋지 않다.
- 싱글 스레드
    - 자바스크립트는 싱글 스레드다.
    - 코드의 실행이 하나의 스레드에서 순차적으로 이루어진다는 것을 의미한다.
    - 태스크 큐 vs 마이크로 테스크 큐

## 타입스크립트

- 불가피하게 아직 타입을 단정할 수 없는 경우, any 대신 `unknown`을 사용하자
- 코드상으로 존재가 불가능한 타입을 나타낼 때 `never`를 사용하자
    
    ```jsx
    type what1 = string & number // string, number 둘다 만족시키는 타입? never
    type what2 = ('hello' | 'hi') & 'react' // never 왜재ㅣ1?!?!?!?!
    ```
    
    리엑트에서는 타입스크립트로 클래스형 컴포넌트를 선언할 때 poprs는 없지만 state가 존재하는 상황에서 이 빈 props, 정확히는 어떠한 props도 받아들이지 않는다는 뜻으로 사용이 가능하다.
    
    ```tsx
    type Props = Record<string, never>
    type State = {
    	counter : 0
    	}
    ```
    
- 타입 가드를 적극 활용하자 (instanceof, typeof, in)
- 제네릭
- 인덱스 시그니처
    
    ```bash
    [key: string]: string
    ```
    

## 느낀점

- 8개월만에 JS를 많이 잊은 것 같아서 JS deep dive를 한번 다시 시간내서 훑어봐야겠다.
- TypeScript의 공부 필요성을 느꼈다.
