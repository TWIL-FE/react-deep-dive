Next.js 버전 13은 Next.js 릴지스 역사에서 가장 큰 변화가 있는 릴리스라 해도 과언이 아니다.

# 11.1 app 디렉터리 등장
이전까지 Next.js의 아쉬운 점 중 하나는 레이아웃의 존재
13버전 이전까지 모든 페이지는 각각의 물리적으로 구별되 파일로 독립돼 있었고 페이지 공통으로 무언가를 넣을 수 있는 곳은 _document와 _app이 유일했지만 이 파일들은 서로 다른 목적을 지니고 있었다.

_document
서버에서만 작동, 페이지에서 쓰이는 html, body 태그를 수정하거나, SSR시 일부 CSS-in-JS를 지원하기 위한 코드를 삽입하는 제한적인 용도로 사용됨
_app
페이지를 초기화하기 위한 용도로 사용
1.페이지 변경 시 유지하고 싶은 레이아웃
2.페이지 변경 시 상태 유지
3.componentDidCatch를 활용한 에러 핸들링
4.페이지간 추가 데이터 삽입
5.global CSS 주입
이러한 레이아웃의 한계를 극복하기 위해 나온 것이 Next.js의 app 레이아웃

## 11.1.1 라우팅
/pages로 정의한 라우팅 방식이 /app 디렉터리로 이동했고 파일명으로 라우팅하는 것이 불가능해졌다.

* 라우팅 정의방법
  * Next.js 12이하: /pages/a/b.tsx 또는 /pages/a/b/index.tsx는 모두 동일한 주소로 변환된다. 파일명이 index라면 이 내용은 무시된다.
  * Next.js 13: /app/a/b는 /a/b로 변환되며, 파일명은 무시된다. 폴더명까지만 주소로 변환된다.

* layout.js
페이지의 기본적인 레이아웃을 구성하는 요소

루트에는 단 하나의 layout을 만들어 둘 수 있음
ex) app/layout.tsx, app/blog/layout.tsx

layout은 주소별 공통 UI를 포함할 수 있을 뿐 아니라 웹페이지 시작시 필요한 공통 코드를 삽입할 수도 있다.
_app, _document처럼 모든 애플리케이션에 영향을 미치지 않고 오직 자신과 자식 라우팅에만 미치게 됨 =>더욱 유연한 레이아웃 구성이 가능해짐

document.jsx에서만 처리할 수 있던 부자연스러움이 사라짐

* 주의할 점

layout은 예약어이기 때문에 layout.js|jsx|ts|tsx로만 사용해야 된다.
layout은 children을 props로 받아서 렌더링해야 함
layout 내부에는 반드시 export default로 내보내는 컴포넌트가 있어야 함
layout 내부에서도 API 요청과 같은 비동기 처리가 가능
page.js, error.js, not-found.js, loading.js

route.js
app 디렉터리가 출시되면서 /pages/api에 대한 app 디렉터리 내부의 지원이 추가됨. /app/api를 기준으로 디렉터리 라우팅을 지원한다. 대신 디렉터리가 라우팅 주소를 담당하고 파일명은 route.js로 통일됐다.

# 11.2 리액트 서버 컴포넌트
## 11.2.1 기존 리액트 컴포넌트와 서버 사이드 렌더링 한계
자바스크립트 번들 크기가 0인 컴포넌트를 만들 수 없다.
게시판 등 사용자가 작성한 HTML에 위험한 태그를 제거하기 위해 타사 라이브러리를 이용해야만 했음
백엔드 리소스에 대한 직접적인 접근이 불가능한다.
백엔드 데이터 접근하기 위해서 REST API와 같은 방법을 사용해야 했음
자동 코드 분할이 불가능하다.
일반적인 리액트에선 lazy를 사용해 구현했음
연쇄적으로 발생하는 클라이언트와 서버의 요청을 대응하기 어렵다.
추상화에 드는 비용이 증가한다.
서버 사이드 렌더링은 정적 콘텐츠를 빠르게 제공하고, 서버에 있는 데이터에 손쉽게 제공할 수 있지만 사용자의 인터랙션에 따른 다양한 사용자 경험을 제공하기 어려움
클라이언트 사이드 렌더링은 사용자의 인터랙션에 따른 다양한 사용자 경험을 제공할 수 있지만 서버에 비해 느리고 데이터를 가져오는 것도 어렵다.
=> 두 구조의 장점을 모두 취하려고 하는 것이 리액트 서버 컴포넌트

## 11.2.2 서버 컴포넌트란?
서버 컴포넌트란 하나의 언어, 하나의 프레임워크 하나의 API와 개념을 사용하면서 서버와 클라이언트 모두에서 컴포넌트를 렌더링할 수 있는 기법
=> 일부 컴포넌트는 클라이언트에서, 일부 컴포넌트는 서버에서 렌더링되는 것이다.
=> 클라이언트 컴포넌트는 서버 컴포넌트를 import 할 수 없다.

```
'use client'
export default function ClientComponent({children}) {
  return(
    <div>
      <h1>클라이언트</h1>
    </div>
  )
}

export default function ServerComponent() {
  return <span>서버</span>
}

import ClientComponent from './ClientComponent'
import ServerComponent from './ServerComponent'
export default function ParentServerComponent() {
  return (
    <ClientComponent>
      <ServerComponent />
    </ClientComponent>
  )
}
```

* 서버 컴포넌트
  * 요청이 오면 서버에서 딱 한번 실행될 뿐 상태를 가질 수 없다.(useState 등의 훅 사용x)
  * 렌더링 생명주기 사용할 수 없다.(useEffect등 사용x)
  * 사용자 정의 훅 또한 사용할 수 없다.
  * DOM API를 쓰거나 window, document에 접근할 수 없다.
  * 서버에만 있는 데이터를 async/await로 접근할 수 있다.
  * 다른 서버 컴포넌트 혹은 div, span, p 혹은 클라이언트 컴포넌트를 렌더링할 수 있다.

* 클라이언트 컴포넌트
  * 서버 컴포넌트 혹은 서버 전용 훅, 유틸리티를 불러올 수 없다.
  * 서버 컴포넌트가 클라이언트 컴포넌트를 렌더링 할 때 자식으로 서버 컴포넌트를 갖는 구조는 가능하다.

* 공용 컴포넌트
  * 서버와 클라이언트 모두 사용할 수 있다. 서버 컴포넌트와 클라이언트 컴포넌트의 모든 제약을 받는 컴포넌트가 된다.

*리액트는 모든 것을 다 공용 컴포넌트라 판단하지만 클라이언트 컴포넌트라는 것을 명시적으로 선언하기 위해 첫 줄에 'use client'를 작성해줘야 한다.

## 11.2.3 서버 사이드 렌더링과 서버 컴포넌트 차이
서버 사이드 렌더링은 전체 페이지를 HTML로 렌더링 하는 과정을 서버에서 수행한 후 클라이언트로 보내준다. 이후 클라이언트에서 하이드레이션 과정을 거쳐 서버의 결과물을 확인하고 이벤트를 붙이는 작업을 수행한다.
정적인 HTML을 빠르게 내려주는게 목적

=> 초기 HTML을 로딩한 이후 결국 클라이언트에서 자바스크립트 코드 다운, 파싱, 실행시 비용이 발생한다.
=> 서버 컴포넌트 활용으로 클라이언트 및 서버 컴포넌트를 빠르게 보여줄 수 있고, 클라이언트에서 내려받는 자바스크립트의 양도 줄어든다.
=> SSR과 서버 컴포넌트는 상호보완의 개념

## 11.2.4 서버 컴포넌트는 어떻게 작동하는가?
서버가 렌더링 요청을 받는다.
서버는 받은 요청에 따라 컴포넌트를 JSON으로 직렬화 한다. 브라우저는 결과물을 받아 역직렬화(or 파싱)한 다음 렌더링을 수행한다.
* 직렬화: 객체를 문자열로 변환하는 것
브라우저가 리액트 컴포넌트 트리를 구성한다.
서버 컴포넌트 작동 방식의 특별한 점
서버에서 클라이언트로 정보를 보낼 때 스트리밍 형태로 보냄으로써 브라우저에서는 되도록 빨리 사용자에게 결과물을 보여줄 수 있다.
컴포넌트들이 하나의 번들러 작업에 포함되어 있지 않고 각 컴포넌트별로 별개로 번들링이 되어 있어 필요에 따라 컴포넌트를 지연해서 받거나 따로 받을 수 있음
서버 사이드 렌더링과 다르게 HTML이 아닌 JSON으로 보내기 때문에 리액트 컴포넌트 트리 구성을 최대한 빠르게 할 수 있도록 도와준다.
# 11.3 Next.js에서의 리액트 서버 컴포넌트
루트 컴포넌트는 무조건 서버 컴포넌트가 되기 때문에 page.js, layout.js는 반드시 서버 컴포넌트여야 한다.
Next.js에서 서버 컴포넌트를 도입하면서 달라진 부분이 있다.

## 11.3.1 새로운 fetch 도입과 getServerSideProps, getStaticProps, getInitialProps 삭제
getServerSideProps, getStaticProps, getInitialProps 대신 모든 데이터 요청은 웹에서 제공하는 표준 API인 fetch를 기반으로 한다.
=> fetch 요청에 대한 내용을 서버에서는 렌더링이 한 번 끝날때까지 캐싱하며, 클라이언트에서는 요청이 없는 이상 해당 데이터를 최대한 캐싱해 중복된 요청을 방지함

## 11.3.2 정적 렌더링과 동적 렌더링
기존 Next.js에서는 getStaticProps를 정적 페이지를 만들어 제공할 수 있는 기능이 있었음
Next.ja 13에서는 정적 라우팅은 기본 빌드 타임에 미리 렌더링해두고, 동적 라우팅은 서버에 요청이 올 때마다 컴포넌트를 렌더링 하도록 변경했다.

fetch 옵션에 따른 작동 방식
fetch(URL, { cache: 'force-cache' }): 기본값으로 getStaticProps와 유사하게 불러온 데이터에 캐싱해 해당 데이터로만 관리
fetch(URL, { cache: 'no-store' }): getServerSideProps와 유사하게 캐싱하지 않고 매번 새로운 데이터를 불러옴
fetch(URL, { next: { revalidate: 10 } }): getStaticProps에 revalidate를 추가한 것과 동일하며, 정해진 유효시간 동안 캐싱하고, 유효시간이 지나면 캐시를 파기함
## 11.3.3 캐시와 mutating, 그리고 revalidating
Next.js는 fetch의 기본 작동을 재정의해 데이터 유효 시간을 정해두고 다시 데이터를 불러와 렌더링하는 것이 가능하다.

캐시와 갱신 과정
최초로 해당 라우트 요청이 오면 미리 정적으로 캐시해 둔 데이터를 보여줌
캐시된 초기 요청은 revalidate에 선언된 값만큼 유지
해당 시간이 지나도 일단은 캐시된 데이터를 보여줌
시간이 경과했으므로 백그라운드에서 다시 데이터를 불러옴
4번 작업이 성공적으로 끝나면 캐시된 데이터를 갱신하고, 아니면 과거 데이터를 보여줌
*캐시를 전체 무효화 하고 싶다면 router.refresh()를 사용하면 됨
## 11.3.4 스트리밍을 활용한 점진적 페이지 불러오기
페이지를 한 번에 불러오는 것이 아닌 HTML을 작은 단위로 쪼개 완성되는 대로 클라이언테 점진적으로 보냄으로써 사용자 경험이 개선될 수 있다.

경로에 loading.tsx 배치
loading 파일 배치시 자동으로 Suspense가 배치된다.
Suspense 배치
직접 리액트의 Suspense를 배치
# 11.4 웹팩의 대항마, 터보팩 등장(beta)
Next.js 13에서 웹팩을 대신해 터보팩이 출시됐는데 터보팩은 웹팩 대비 최대 700배, vite 대비 10배가 빠르다고 함

# 11.5 서버 액션(alpha)
API를 굳이 생성하지 않더라도 함수 수준에서 서버에 직접 접근해 데이터 요청 등을 수행할 수 있는 기능
서버 컴포넌트와 다르게, 특정 함수 실행 그 자체만을 서버에서 수행할 수 있다는 장점이 있음

next.config.js에서 실험 기능을 활성화 해야함
```
const nextConfig = {
	experimental: {
    	serverActions: true
    }
}
module.exports = nextConfig
서버 액션을 사용하기 위해선 함수 내부 또는 파일 상단에 'use server' 지시자를 선언해야 함, 함수는 반드시 async여야 함

## 11.5.1 form의 action
export default async function Page(){
  async function handleSubmit() {
    'use server'

    const res = await fetch('https://jsonplaceholder.typicode.com/1',{
      method: 'post',
      body: JSON.stringify({
        title: 'foo',
        body: 'bar',
        userId: 1
      }),
      headers: {
        'Content-type': 'application/json; charset=UTF-8'
      }
    })

    const result = await res.json()

  }

  return(
    <form action={handleSubmit}>
      <button type='submit'>요청</button>
    </form>
  )
}
```
form.action에 handleSubmit이라는 서버 액션을 만들어 props로 넘겨줄 수 있음
이 함수가 수행되는 것은 서버가 된다.

## 11.5.2 input의 submit과 image의 formAction
form.action처럼 서버액션을 추가할 수 있다.

## 11.5.3 startTransition과의 연동
useTransition에서 제공하는 startTransition에서도 서버 액션을 활용할 수 있다.

useTransition이 반환하는 배열의 두 번째 요소인 startTransition을 사용해 서버 액션을 구현할 수 있는데 page 단위인 loading.jsx를 사용하지 않아도 되고 컴포넌트 단위의 로딩 처리가 가능해진다.

## 11.5.4 서버 액션 사용 시 주의할 점
서버 액션은 클라이언트 컴포넌트 내에서 정의될 수 없다.
'use client' 가 선언되어 있는 컴포넌트 내에서 사용할 경우 에러가 발생한다.
서버 액션을 import 하는 것 뿐만 아니라, props 형태로 서버 액션을 클라이언트 컴포넌트에 넘기는 것도 가능하다.
서버 컴포넌트가 클라이언트 컴포넌트를 불러올 수 있는 것과 동일한 원리
