## 2.1 JSX란?
JSX는 자바스크립트 내부에서 표현하기 까다로웠던 XML 스타일의 트리 구문을 작성하는 데 많은 도움을 주는 새로운 문법

### 2.1.1 JSX의 정의
* JSXElement
JSX를 구성하는 가장 기본 요소로, HTML 요소와 비슷한 역할을 함
  1. JSXOpeningElement: 일반적으로볼 수 있고 JSXClosingElement가 같은 단계에 선언돼 있어야 한다.
  ex) <JSXElement JSXAttributes(optional)>
  
  2. JSXClosingElement: JSXOpeningElement와 쌍으로 사용되고, JSXOpeningElement가 종료되었음을 알림
  ex) </ JSXElement >
  
  3. JSXSelfClosingElement: 요소가 시작되고 스스로 종료되는 형태, 내부에 자식을 포함할 수 없는 형태
  ex) < JSXElement JSXAttributes(optional) />
  
  4. JSXFragment: 아무런 요소가 없는 형태임
  ex) <> JSXChildren(optional) </>

* JSXElementName
JSXElement의 요소 이름으로 쓸 수 있는 것을 의미
  1. JSXIdentifier: JSX 내부에서 사용할 수 있는 식별자
  숫자로 시작하거나 $와 _외의 다른 특수문자로는 시작할 수 없음
  
  2. JSXNameSpacedName: JSXIdentifier: JSXIdentifier의 조합, :를 통해 서로 다른 식별자를 이어주는 것도 하나의 식별자로 취급된다.(: 로 묶을 수 있는 것은 한 개뿐임)
  ex)<foo:bar> </foo:bar>
  
  3. JSXMemberExpression: JSXIdentifier.JSXIdentifier의 조합으로 여러 개 이어서 하는 것도 가능하다.
  ex)<foo.bar.baz></foo.bar.baz>

* JSXAttributes
JSXElement에 부여할 수 있는 속성
  1. JSXSpreadAttributes: 자바스크립트의 전개 연산자와 동일한 역할
  {...AssignmentExpression}
  
  2. JSXAttribute: 속성을 나타내는 키와 값으로 짝을 이루어서 표현, 키는 JSXElementName, 값은 JSXAttributeValue로 불린다.

* JSXChildren
JSXElement의 자식 값을 나타냄

## 2.2 가상 DOM과 리액트 파이버
리액트의 가장 큰 특징 중 하나는 실제 DOM이 아닌 가상 DOM을 운영한다는 것이다. 그럼 가상 DOM은 어떻게 만들어졌고, 실제 DOM과는 어떤 차이가 있는가?

### 2.2.1 DOM과 브라우저 렌더링 과정
  1. 브라우저가 사용자가 요청한 주소를 방문해 HTML 파일을 다운로드
  2. 브라우저의 렌더링 엔진은 HTML을 파싱해 DOM 노드로 구성된 트리(DOM)을 만든다.
  3. 2번 과정에서 CSS 파일을 만나면 CSS 파일도 다운로드
  4. 브라우저 렌더링 엔진은 이 CSS도 파싱해 CSS 노드로 구성된 트리(CSSOM)을 만든다.
  5. 브라우저는 2번에서 만든 DOM 노드를 순회하는데, 모든 노드가 아닌 사용자 눈에 보이는 노드만 방문한다.(display: none 과 같은 화면에 보이지 않는 요소는 방문해 작업하지 않음) 이는 트리 분석 과정을 빠르게 하기 위해서다.
  6. 5번에서 제외된, 눈에 보이는 노드를 대상으로 해당 노드에 대한 CSSOM 정보를 찾고 CSS 스타일 정보를 노드에 적용한다.
  DOM 노드에 CSS를 적용하는 과정	
  	- 레이아웃: 각 노드가 브라우저 화면의 어느 좌표에 정확히 나타나야 하는지 계산하는 과정
      - 페인팅: 레이아웃 단게를 거친 노드에 색과 같은 유효한 모습을 그리는 과정 
      
    
### 2.2.2 가상 DOM의 탄생 배경
브라우저가 웹페이지를 렌더링하는 과정은 매우 복잡하고 많은 비용이 든다.
하지만 요즘 앱은 렌더링이 완료된 이후에도 사용자의 인터랙션으로 웹페이지가 변경되는 상황 또한 고려해야 한다.
*<span style="color: blue">가상 DOM은 실제 브라우저의 DOM이 아닌 리액트가 관리하는 가상의 DOM을 의미한다.
가상 DOM은 웹페이지가 표시해야 할 DOM을 메모리에 저장하고 실제 변경에 대한 준비가 완료됐을 때 실제 브라우저의 DOM에 반영하는데 이로 인해 렌더링을 최소화할 수 있고 브라우저와 개발자의 부담을 덜 수 있다.*
    
### 2.2.3 가상 DOM을 위한 아키텍처, 리액트 파이버
가상DOM과 렌더링 최적화를 가능하게 해주는 것이 바로 리액트 파이버(React Fiber)
  
* 리액트 파이버란?
  리액트에서 관리하는 평범한 자바스크립트 객체로, 가상 DOM과 실제 DOM을 비교해 둘 사이에 차이가 있으면 변경 관련 정보를 가지고 있는 파이버를 기준으로 화면에 요청하는 역할이며 모든 과정이 _비동기_로 일어난다.
  
  파이버의 역할
  1. 작업을 작은 단위로 분할하고 쪼갠 다음, 우선순위를 매긴다.
  2. 작업을 일시 중지하고 나중에 다시 시작할 수 있다.
  3. 이전에 했던 작업을 다시 재사용하거나 필요하지 않은 경우 폐기 가능
  
  파이버는 하나의 작업 단위로 구성돼 있고 작업 단위를 처리한 후 finishedWork()라는 작업으로 마무리한다. 그리고 이 작업을 커밋해 실제 브라우저 DOM에 적용한다.
  1. 렌더 단계에서 리액트는 사용자에게 노출되지 않는 모든 비동기 작업을 수행. 이 단계에서 파이버는 우선순위를 지정하거나 중지시키거나 버리는 등의 작업이 일어난다.
  2. 커밋 단계에서 DOM에 실제 변경 사항을 반영하기 위한 commitWork()가 실행되는데, 이 과정은 동기식으로 일어나고 중단될 수도 없다.
  
   *리액트 내부 코드에 작성돼 있는 파이버 객체
  
  ```
    function FiberNode(tag, pendingProps, key, mode) {
      this.tag = tag
      this.key = key
      this.elementType = null
      this.type = null
      this.stateNode = null

      // Fiber
      this.return = null
      this.child = null
      this.sibling = null
      this.index = 0
      this.ref = null
  
      this.refCleanup = null
      this.pendingProps = pendingProps
      this.memoizedProps = null
      this.updateQueue = null
      this.memoizedState = null
      this.dependencies = null
  
      this.mode = mode
      ...
    }
  ```  
 리액트 요소는 렌더링이 발생할 때마다 새로 생성이 되지만 파이버는 컴포넌트가 최초로 마운트되는 시점에 생성된 이후에는 가급적이면 재사용 된다.
  
  *리액트에 작성돼 있는 파이버를 생성하는 함수들
  ```
  var createFiber = function(tag, pendingProps, key, mode) {
    return new FiberNode(tag, pendingProps, key, mode)
  }

  function createFiberFromElement(element, mode, lanes){
    var owner = null
    {
      owner = element._owner
    }

    var type = element.type
    var key = element.key
    var pendingProps = element.props
    var fiber = createFiberFromTypeAndProps(
      type,
      key,
      pendingProps,
      owner,
      mode,
      lanes
    )

    {
      fiber._debugSource = element._source
      fiber._debugOwner = element._owner
    }

    return fiber
  }

  function createFiberFromFragment(elements, mode, lanes, key) {
    var fiber = createFiber(Fragment, elements, key, mode)
    fiber.lanes = lanes
    return fiber
  }
  ```
  
1. tag: 파이버는 하나의 element에 하나가 생성되는 1:1 관계를 가지고 있는데 매칭된 정보를 가지고 있는 것이 바로 tag
2. stateNode: 파이버 자체에 대한 참조 정보를 가지고 있으며, 이 참조를 바탕으로 리액트는 파이버와 관련된 상태에 접근
3. child, sibling, return: 파이버 간의 관계 개념을 나타내는 속성
4. index: 여러 형제들(sibling) 사이에서 자신의 위치가 몇 번째인지 표현
5. pendingProps: 여러 작업을 미처 처리하지 못한 props
6. memoizedProps: pendingProps를 기준으로 렌더링이 완료된 이후에 pendingProps를 memoizedProps로 저장해 관리
7. updateQueue: 상태 업데이트, 콜백 함수, DOM 업데이트 등 필요 작업을 담아두는 큐
8. memoizedState: 함수 컴포넌트의 훅 목록이 저장된다. 단순한 useState 뿐 아니라 모든 훅 리스트가 저장
9. alternate:리액트의 트리는 두 개인데, alternate는 반대편 트리 파이버이다.

### 리액트 파이버 트리
  파이버 트리는 리액트 내부에서 두 개가 존재한다.
  하나는 현재 모습의 파이버 트리이고, 나머지 하나는 작업 중인 상태를 나타내는 workInProgress 트리다.
  리액트 파이버 작업이 끝나면 리액트는 단순히 포인터만 변경해 workInProgress 트리를 현재 트리로 바꿔버린다.
  
  => 현재 UI 렌더링을 위해 존재하는 트리인 cureent를 기준으로 작업이 시작되고 업데이트 발생 시 리액트에서 새로 받은 데이터로 새로운 workInProgress 트리를 빌드하기 시작한다. workInProgress 트리 빌드 작업이 끝나면 다음 렌더링에 이 트리를 사용한다. UI에 최종 반영이 완료되면 현재 current가 workInProgress로 변경된다.
  
  * 파이버의 작업 순서
    1. 리액트느 beginWork() 함수를 실행해 파이버 작업을 수행하는데, 더 이상 자식이 없는 파이버를 만날 때까지 트리 형식으로 시작
    2. 1번에서 작업이 끝나면 그 다음 completeWork() 함수를 실행해 파이버 작업을 완료한다.
    3. 형제가 있다면 형제로 넘어간다.
    4. 2, 3번이 모두 끝나면 return으로 돌아가 자신의 작업이 완료됐음을 알림
  
  => 만약 setState 등으로 업데이트가 발생하면 위의 과정과 동일하게 동작한다.
  => 가급적 새로운 파이버를 생성하지 않는다는 의미가 이것이다.
  
## 2.3 클래스 컴포넌트와 함수 컴포넌트
  
### 2.3.1 클래스 컴포넌트
클래스 컴포넌트를 만들기 위해선 클래스 선언 후 만들고 싶은 컴포넌트를 extends 해야함 ( React.Component, React.PureComponent )
  
	클래스 컴포넌트 요소
  
  1. constructor(): 컴포넌트의 state를 초기화 할 수 있고, 여기에 선언돼 있는 super()는 컴포넌트를 만들면서 상속받은 상위 컴포넌트, React.Component의 생성자 함수를 먼저 호출해 상위 컴포넌트에 접근할 수 있게 도와준다.

  *ES2022에서 추가된 클래스 필드 때문에 constructor를 쓰지 않고 state를 초기화할 수 있다.(ES2022 환경을 제공하는 브라우저여야 함)
  
  2. props: 컴포넌트에 특정 속성을 전달하는 용도
  3. state: 클래스 컴포넌트 내부에서 관리하는 값, 이 값은 항상 객체여야만 한다.
  4. 메서드: 렌더링 함수 내부에서 사용되는 함수이고, DOM에서 발생하는 이벤트와 함께 사용된다.
  
* 클래스 컴포넌트의 생명주기 메서드

  1. render()
  리액트 클래스 컴포넌트의 유일한 필수값으로 항상 쓰이고, 항상 순수해야 하며 부수 효과가 없어야 한다. 따라서 render 함수 내부에서 state를 직접 업데이트 할 수 없다.
  2. componentDidMount()
  클래스 컴포넌트가 마운트되고 난 후 다음 생명주기 메서드
  3. componentDidUpdate()
  컴포넌트 업데이트가 일어난 이후 바로 실행, state나 props의 변화에 따라 DOM을 업데이트 하는 등에 쓰임
  4. componentWillUnmount()
  컴포넌트가 언마운트되거나 더 이상 사용되지 않기 직전에 호출
  메모리 누수나 불필요한  작동을 막기 위한 클린업 함수 호출의 최적 위치
  5. shouldComponentUpdate()
  state나 props의 변경으로 컴포넌트 리렌더링을 막고 싶을 때 사용하는 메서드
  return이 true일 경우 업데이트 한다.
  
  6. static getDerivedStateFromProps()
  사라진 componentWillReceiveProps를 대체하는 메서드로 render()를 호출하기 직전에 호출된다.
  여기서 반환하는 객체는 해당 객체의 내용이 모두 state로 들어감
  
  7. getSnapShotBeforeUpdate()
  componentWillUpdate() 대체 메서드로 DOM이 업데이트되기 직전에 호출된다.
  
  8. getDerivedStateFromError(), componentDidCatch()
  에러 상황에서 실행되는 메서드
  getDerivedStateFromError는 렌더 단계에서 실행되고 에러 메시지 전달에 주로 사용하지만,
  componentDidCatch 메서드는 커밋 단계에서 실행되고 에러 정보를 로깅하는 등의 용도로 사용할 수 있다.
  
* 클래스 컴포넌트의 한계
  1. 데이터 흐름 추적의 어려움:  state의 흐름을 추적하기가 매우 어렵다. 서로 다른 여러 메서드에서 state의 업데이트가 일어날 수 있고 코드를 읽는 과정에서 숙련된 개발자라도 state가 어떤 식의 흐름으로 변경되서 렌더링이 일어나는지 판단하기 어렵다.
  2. 애플리케이션 내부 로직 재사용이 어렵다
  3. 기능이 많아질수록 컴포넌트 크기가 커진다: 데이터 흐름이 복잡해져 생명주기 메서드 사용이 잦아드는 경우 컴포넌트 크기가 기하급수적으로 커짐
  4. 클래스는 함수에 비해 상대적으로 어렵다
  5. 코드 크기를 최적화하기 어렵다: 클래스 컴포넌트는 최종 결과물인 번들 크기를 줄이는 데도 어려움이 있다.
  ex) 사용하지 않는 함수(handleClick)가 있을 때 빌드 시 이름이 최소화 되지 않고, 번들에 그대로 포함이 된다.
  6. 핫 리로딩을 하는데 상대적으로 불리하다: 코드에 변경 사항이 있을 때 해당 변경된 코드만 업데이트 하는 것을 핫 리로딩이라 하는데, 클래스 컴포넌트는 instance 내부에서 state를 관리하는데 render를 수정하게 되고 핫 리로딩이 일어나면 instance를 새로 만들 수 밖에 없기 때문에 state 값이 초기화 될 수 밖에 없다.
  
  	*핫 리로딩 예제코드
  ```
  function FunctionalComponent() {
    const [count, setCount] = useState(0)

    function handleClick() {
      setCount((prev) => prev +1)
    }

    return (
      <>
        <button onClick={handleClick}>{count} + </button>
      </>
    )
  }

  class ClassComponent extends PureComponent<{}, {count:number}> {
    constructor(props:{}) {
      super(props)
      this.state ={
        count: 0
      }
    }

    handleClick = () => {
      this.setState((prev) => ({count: prev.count + 1}))
    }

    render() {
      return <button onClick={this.handleClick}>{this.state.count} + </button>
    }
  }

  export default function App() {
    return (
      <>
        <FunctionalComponent />
        <ClassComponent />
      </>
    )
  }
  ```
  
 > 함수, 클래스 컴포넌트 코드 수정
  
  ![](https://velog.velcdn.com/images/kdh3543/post/137d29cd-865c-4b62-88d5-8c15bb92b05c/image.png)
  
  ![](https://velog.velcdn.com/images/kdh3543/post/d16f8ee1-4259-4dee-a421-db59c8e4adad/image.png)

 > 코드 수정 후 화면
 
 함수 컴포넌트

 ![](https://velog.velcdn.com/images/kdh3543/post/a5ce1826-c78b-47bb-8696-2ea17921e226/image.png)

 클래스 컴포넌트
 ![](https://velog.velcdn.com/images/kdh3543/post/a0cf16e2-4774-4271-be0c-707965c7e68e/image.png)

### 2.3.2 함수 컴포넌트
클래스 컴포넌트에 비해 코드가 간결해지고, this 바인딩을 조심할 필요가 없으며, state는 각각의 원시값으로 관리되어 훨씬 사용하기가 편해졌다.

## 2.4 렌더링은 어떻게 일어나는가?
### 2.4.1 리액트의 렌더링이란?
리액트 애플리케이션 트리 안에 있는 모든 컴포넌트들이 자신들이 가지고 있는 props와 state의 값을 기반으로 어떻게 UI를 구성하고 이를 바탕으로 어떤 DOM 결과를 브라우저에 제공할 것인지 계산하는 일련의 과정

### 2.4.2 리액트의 렌더링이 일어나는 이유
1. 최초 렌더링
애플리케이션 진입시 브라우저에 정보를 제공하기 위해 최초 렌더링을 수행
2. 리렌더링
애플리케이션 진입 후 최초 렌더링 이후 발생하는 모든 렌더링을 의미한다.
리렌더링 발생 경우
	- 클래스 컴포넌트의 setState가 실행되는 경우
    - 클래스 컴포넌트의 forceUpdate가 실행되는 경우
    - 함수 컴포넌트의 useState()의 두 번째 배열 요소인 setter가 실행되는 경우
    - 함수 컴포넌트의 useReducer()의 두 번째 배열 요소인 dispatcher가 실행되는 경우
    - 컴포넌트의 key props가 변경되는 경우
    - props가 변경되는 경우
    - 부모 컴포넌트가 렌더링될 경우
    
### 2.4.3 리액트의 렌더링 프로세스
렌더링 프로세스가 시작되면 리액트는 컴포넌트 루트에서부터 차례대로 업데이트가 필요하다고 지정돼 있는 모든 컴포넌트를 찾고, 클래스 컴포넌트의 경우 render()함수, 함수 컴포넌트의 경우 FunctionComponent() 그 자체를 호출한 뒤 결과물을 저장한다.

리액트의 렌더링은 렌더 단계와 커밋 단계로 분리되어 실행된다.

### 2.4.4 렌더와 커밋
렌더 단계(Render Phase)는 컴포넌트를 렌더링하고 변경 사항을 계산하는 모든 작업을 말한다.
즉, 렌더링 프로세스에서 컴포넌트를 실행해 이 결과와 이전 가상 DOM을 비교하는 과정을 거쳐 변경이 필요한 컴포넌트를 체크하는 단계다.

커밋 단계(Commit Phase)는 렌더 단계의 변경 사항을 실제 DOM에 적용해 사용자에게 보여주는 단계이다.

=> 리액트의 렌더링이 일어난다고 해서 무조건 DOM 업데이트가 일어나는 것은 아니다.
커밋 단계까지 거쳐야 DOM 업데이트가 일어난다.

## 2.5 컴포넌트와 함수의 무거운 연산을 기억해 두는 메모이제이션
### 2.5.1 주장1: 섣부른 최적화는 독이다. 꼭 필요한 곳에만 메모이제이션을 추가하자
꼭 필요한 곳을 신중히 골라 메모이제이션해야 한다는 입장이다.
메모이제이션도 비용이 드는 작업이므로 항상 신중해야 한다는 주장이다.

>**리액트 공식 문서**
useMemo는  성능 최적화를 위해 사용할 수 있지만 의미상으로 그것이 보장된다고 생각하지 마세요. 가까운 미래에 리액트에서는 이전에 메모이제이션된 값들의 일부를 '잊어버리고' 다음 렌더링 시에 그것들을 재계산하는 방향을 택할지도 모르겠습니다. 
예를 들면, 오프스크린 컴포넌트의 메모리를 해제하는 등이 있을 수 있습니다. useMemo를 사용하지 않고도 작동할 수 있도록 코드를 작성하고 그것을 추가해 성능을 최적화하세요.

리액트 공식 문서에 나와있는 것처럼 리액트에서 메모이제이션을 활요한 최적화는 신중을 기해야 한다.
일단 애플리케이션을 어느 정도 만든 이후에 개발자 도구나 useEffect를 사용해 실제로 어떻게 렌더링이 일어나고 있는지 확인한 후 최적화하는 것이 옳다.

### 2.5.2 주장2: 렌더링 과정의 비용은 비싸다. 모조리 메모이제이션해 버리자
두 번째 주장에서 공통으로 깔고 가는 전제는 컴포넌트에서는 메모이제이션을 하는 것은 성능에 도움이 된다.

1. memo를 컴포넌트의 사용에 따라 잘 살펴보고 일부에만 적용
가장 이상적인 상황이지만, 애플리케이션 규모가 커지고, 컴포넌트 복잡성이 증가하는 상황에서 최적화나 성능 향상에 쏟을 시간이 많지 않다라는 단점이 있다.
2. memo를 일단 그냥 다 적용하는 방법
memo가 하지 않았을 때 발생할 수 있는 문제
	- 렌더링을 함으로써 발생하는 비용
    - 컴포넌트 내부의 복잡한 로직 재실행
    - 위 두가지 모두 모든 자식 컴포넌트에서 반복해서 일어남
    - 리액트가 구 트리와 신규 트리를 비교
    memo를 하지 않았을 때 치러야 할 잠재적인 비용 위험이 더 크기 때문에 memo를 적용하는게 더 이점이 될 수 있다.
