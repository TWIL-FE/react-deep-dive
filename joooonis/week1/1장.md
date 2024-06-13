### typeof

`typeof` 연산자는 피연산자의 자료형을 나타내는 **문자열**을 반환합니다.

문제 1

```jsx
console.log(typeof null)
1. 'null'
2. null
3. 'object'
4. object
```

정답

```jsx
    console.log(typeof null)
    1. 'null'
    2. null
    3. 'object' ✅
    4. object
```

문제 2

```jsx
console.log(typeof {});
console.log(typeof []);
```

정답

```jsx
console.log(typeof {}); // "object"
console.log(typeof []); // "object" (배열도 객체로 간주됨)
```

TypeScript에서는 타입 컨텍스트에서 변수나 프로퍼티의 타입을 추론하는데 사용할 수 있습니다.

예를들어 아래처럼 `React.ComponentProps<T>` generic, T 자리에 component의 타입을 반환해서 넣어줄 수 있습니다.

```tsx
import HeaderBase from "@/components/Header";

interface HeaderProps extends React.ComponentProps<typeof HeaderBase> {
  isShowIntroAnimation?: boolean;
}
```

### React에서 동등비교가 일어나는 경우

- props, state 변화를 감지하여 re-rendering
- useEffect의 dependency 배열
- React.memo, useMemo, useCallback 등에서의 meoization
- React Context

### Object.is

ES6에서 추가된 비교 문법, 동등비교(===) 연산자의 문제점들을 해결하기 위해 만들어짐. 두 개의 인수를 받으며 이 인수 두 개가 동일한지 확인하고 반환하는 매서드입니다.

1. == vs [Object.is](http://Object.is)

- == 는 자동 형변환이 일어나지만 Object.is는 그렇지 않다

1. === vs [Object.is](http://Object.is)

- [Object.is](http://Object.is) 가 좀 더 개발자가 기대하는 방식으로 동작합니다.
- 객체간의 비교는 두 연산자 동일하게 참조를 기반으로 동작합니다.

```jsx
-0 === +0;
Object.is(+0, -0);

Number.NaN === NaN;
Object.is(Number.NaN, NaN);

NaN === 0 / 0;
Object.is(NaN, 0 / 0);
```

정답

```jsx
-0 === +0; // false
Object.is(+0, -0); // true

Number.NaN === NaN; // false
Object.is(Number.NaN, NaN); // true

NaN === 0 / 0; // false
Object.is(NaN, 0 / 0); // true
```

React 에서 값을 비교하는 함수인 [objectIs](https://github.com/facebook/react/blob/main/packages/shared/objectIs.js), Object.is가 제공되지 않는 환경을 고려해서 Polyfill인 is 함수를 함께 제공하고 있습니다.

```jsx
/**
 * Copyright (c) Meta Platforms, Inc. and affiliates.
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
 *
 * @flow
 */

/**
 * inlined Object.is polyfill to avoid requiring consumers ship their own
 * https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is
 */
function is(x: any, y: any) {
  return (
    (x === y && (x !== 0 || 1 / x === 1 / y)) || (x !== x && y !== y) // eslint-disable-line no-self-compare
  );
}

const objectIs: (x: any, y: any) => boolean =
  // $FlowFixMe[method-unbinding]
  typeof Object.is === "function" ? Object.is : is;

export default objectIs;
```

objectIs 기반으로 React에서 동등비교를 위해서는 [shallowEqual](https://github.com/facebook/react/blob/main/packages/shared/shallowEqual.js) 함수를 사용합니다. 객체 간 얉은 비교(1-depth)까지 수행합니다.

```jsx
/**
 * Copyright (c) Meta Platforms, Inc. and affiliates.
 *
 * This source code is licensed under the MIT license found in the
 * LICENSE file in the root directory of this source tree.
 *
 * @flow
 */

import is from "./objectIs";
import hasOwnProperty from "./hasOwnProperty";

/**
 * Performs equality by iterating through keys on an object and returning false
 * when any key has values which are not strictly equal between the arguments.
 * Returns true when the values of all keys are strictly equal.
 */
function shallowEqual(objA: mixed, objB: mixed): boolean {
  if (is(objA, objB)) {
    // 여기서 같은 참조라면 true
    return true;
  }

  if (
    typeof objA !== "object" ||
    objA === null ||
    typeof objB !== "object" ||
    objB === null
  ) {
    return false;
  }

  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);

  if (keysA.length !== keysB.length) {
    // 객체의 길이가 다르면 false
    return false;
  }

  // Test for A's keys different from B.
  for (let i = 0; i < keysA.length; i++) {
    // 객체의 값들을 순회하면서 objectIs 함수 실행
    const currentKey = keysA[i];
    if (
      !hasOwnProperty.call(objB, currentKey) ||
      // $FlowFixMe[incompatible-use] lost refinement of `objB`
      !is(objA[currentKey], objB[currentKey])
    ) {
      return false;
    }
  }

  return true;
}

export default shallowEqual;
```

### 일급객체

일급객체란 다른 객체들에 일반적으로 적용 가능한 연산을 모두 지원하는 객체를 의미합니다. JavaScript에서 함수는 일급객체입니다. 함수는 다른 함수의 매개변수가 될 수 도 있고, 반환값이 될 수도 있으며, 다른 변수에 할당도 가능합니다.

### 함수 선언문 vs 함수 표현식

함수 선언문(Function Declaration)은 `function` 키워드로 시작하며, 함수의 이름과 본문을 포함합니다. 함수 선언문은 코드가 실행되기 전에 미리 선언되므로, 함수가 정의되기 전에 호출할 수 있습니다. 이를 *호이스팅(hoisting)*이라고 합니다.

```jsx
sayHello(); // "Hello!" , 호이스팅이 일어남

function sayHello() {
  console.log("Hello!");
}
```

함수 표현식(Function Expression)은 변수에 익명 함수를 할당하거나, 이름을 가진 함수를 할당하는 방식으로 정의됩니다. 함수 표현식은 코드가 실행되는 시점에 평가되므로, 정의된 이후에만 호출할 수 있습니다.

```jsx
sayHi(); // Uncaught TypeError: sayHi is not a function, *sayHi는 undefined로 초기화 되었습니다.

var sayHi = function () {
  console.log("Hello!");
};
```

자 그렇다면 아래 코드는 어떻게 동작할까?

```jsx
sayHi2();

const sayHi2 = function () {
  console.log("Hello!");
};
```

정답
const 키워드로 변수 선언시 호이스팅이 일어나도 초기화 이전에는 TDZ 이기때문에 접근이 불가능하여 참조에러가 발생합니다.

```jsx
sayHi2(); // Uncaught ReferenceError: Cannot access 'sayHi2' before initialization
```

함수 표현식이므로 정의된 이후에만 호출할 수 있다는 점은 동일하지만 변수를 선언하는 키워드에 따라서 그 원인은 다릅니다.

- `var`: 함수 스코프, 선언은 호이스팅되며 초기화 전에는 `undefined`가 할당됨.
- `let`: 블록 스코프, 선언은 호이스팅되지만 초기화 전에는 접근 불가 (TDZ).
- `const`: 블록 스코프, 선언은 호이스팅되지만 초기화 전에는 접근 불가 (TDZ), 선언과 동시에 초기화 필요.

### 클로저

클로저란 함수와 함수가 선언된 어휘적 환경(Lexical Scope)의 조합 입니다.

클로저는 함수가 **실행되는 시점**이 아니라 **선언된 시점** 을 토대로 동작합니다.

클로저는 함수가 선언될 때의 환경을 기억하고, 그 환경에 접근할 수 있게 합니다. 예를 들어, 함수 내부에 선언된 변수에 접근하는 또 다른 함수를 반환하는 함수가 클로저의 대표적인 예입니다.

클로저 예시

```jsx
function createCounter() {
  let count = 0; // 클로저가 기억할 렉시컬 환경(변수)

  return {
    increment: function () {
      count += 1;
      return count;
    },
    decrement: function () {
      count -= 1;
      return count;
    },
    getCount: function () {
      return count;
    },
  };
}

const counter = createCounter();

console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.decrement()); // 1
console.log(counter.getCount()); // 1
```

리액트에서의 클로저

```jsx
function Component() {
  const [state, setState] = useState();
}
```

외부함수(useState)가 반환한 내부 함수(setState)는 외부함수의 호출이 끝났음에도 자신이 선언된 외부함수가 선언된 환경(state가 저장돼 있는 어딘가)를 기억하기 때문에 계속해서 state 값을 사용할 수 있다.

클로저를 통해 state를 외부(전역 스코프)로부터 분리하여 사용자가 직접 수정하는 경우를 방지할 수 있다.(state 값은 setState로만 변경가능하다.)

단 클로저는 공짜가 아니다. 외부함수를 기억하고 이를 내부함수에서 가져다 쓰는 메커니즘은 당연하게도 메모리를 사용하는 것이고 성능에 영향을 미친다.
