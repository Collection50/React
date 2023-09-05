# useState

- `useState`는 컴포넌트에 상태 변수를 추가하는 `React` `hook`입니다.

```jsx
const [state, setState] = useState(initialState);
```

<br>

## 레퍼런스

### `useState(initialState)`

- 컴포넌트 최상단에서 `useState`를 호출하여 상태 변수를 선언합니다.

```js
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(28);
  const [name, setName] = useState('Taylor');
  const [todos, setTodos] = useState(() => createTodos());
  // ...
}
```

<br>

- 일반적으로 배열 디스트럭처링을 사용하여 `[something, setSomething]`과 같은 상태 변수의 이름을 지정합니다.

<br>

### 매개변수

- `initialState`
  - 초기 상태 값입니다.
  - 모든 유형의 값일 수 있지만 특별한 동작이 있습니다.
  - 초기값은 첫 렌더링 이후에는 무시됩니다.
  - 함수를 전달하면 **초기화 함수**로 취급됩니다.
  - 순수 함수여야 하고 인수가 없어야 하며 어떤 유형의 값도 반환할 수 있습니다.
  - `React`는 컴포넌트를 초기화할 때 초기화 함수를 호출하고 반환값을 초기 상태로 저장합니다.

<br>

### 반환값

- `useState`는 정확히 2개 값을 가진 배열을 반환합니다.

<br>

1. 현재 상태
   - 초기 렌더링 시에는 전달한 `initialState`와 동일합니다.
2. `set` 함수를 사용하면 상태를 업데이트하고 리렌더링합니다.

<br>

### 주의사항

- `useState` 는 `hook`이므로 **컴포넌트의 최상단**이나 커스텀 `hook`에서만 호출할 수 있습니다.
  - 반복문, 조건문 내부에서는 호출할 수 없습니다.
  - 필요하다면 새 컴포넌트를 추출하고 상태를 옮깁니다.
- `Strict Mode`에서는 **실수로 발생한 부수효과를 찾기 위해** `React`가 **초기화 함수를 두 번 호출**합니다.
  - 개발 모드에서만 이뤄지며 배포 환경에는 영향을 미치지 않습니다.
  - 초기화 함수가 순수하다면 영향을 미치지 않습니다.
  - 호출 결과 중 하나는 무시됩니다.

<br>

## `set` 함수(`setSomething(nextState)`)

- `useState`가 반환하는 `set` 함수를 사용하면 상태를 업데이트하고 리렌더링합니다.
- 상태를 직접 전달하거나 업데이트 함수를 전달할 수 있습니다.

```js
const [name, setName] = useState('Edward');

function handleClick() {
  setName('Taylor'); // 직접 전달
  setAge((a) => a + 1); // 업데이트 함수 전달
  // ...
}
```

<br>

### 매개변수

- `nextState`
  - 다음 상태 값입니다.
  - 모든 유형의 값이 될 수 있지만 특별한 동작이 있습니다.
  - 함수를 전달하면 **업데이트 함수**로 취급됩니다.
  - 순수 함수여야 하고 보류 중인 상태를 유일한 인수로 사용해야 하며 다음 상태를 반환해야 합니다.
  - `React`는 업데이트 함수를 큐에 넣고 컴포넌트를 리렌더링합니다.
  - 이후 렌더링에서 `React`는 큐의 모든 업데이트 함수를 이전 상태에 적용하여 상태를 연산합니다.

<br>

### 반환값

- `set` 함수에는 반환값이 없습니다.

<br>

### 주의사항

- `set` 함수는 **다음 렌더링에 대한 상태 변수만 업데이트**합니다.
  - `set` 함수를 호출한 후 상태 변수를 참조하면 호출 전의 **값을 계속 참조합니다.**
- `Object.is` 메서드에 따라 새로운 값이 현재 `state`와 동일하다면 컴포넌트와 하위 컴포넌트는 리렌더링되지 않습니다.
  - 최적화의 일부입니다.
  - `React`가 하위 컴포넌트를 건너뛰기 전에 컴포넌트를 호출해야 할 수도 있지만 코드에 영향을 미치지는 않습니다.
- `React`는 상태 업데이트를 **일괄적**으로 수행합니다.
  - 모든 **이벤트 핸들러가 실행되고 `set` 함수를 호출한 후**에 화면을 업데이트합니다.
  - 이렇게 하면 단일 이벤트 중에 여러 번 리렌더링되지 않습니다.
  - 드물지만 DOM에 접근하기 위해 화면을 일찍 업데이트하도록 해야 하는 경우엔 `flushSync`를 사용합니다.
- 렌더링 도중 `set` 함수를 호출하는 것은 **렌더링 중**인 컴포넌트 내에서만 가능합니다.
  - 렌더링 도중 `set` 함수를 호출하면 `React`는 모든걸 중단하고 즉시 새로운 상태로 리렌더링을 시도합니다.
  - 이 패턴은 거의 필요하지 않지만 **이전 렌더링의 정보를 저장**하는 데 사용할 수 있습니다.
- `Strict Mode`에서는 **실수로 발생한 부수효과를 찾기 위해** `React`가 **업데이트 함수를 두 번 호출**합니다.
  - 개발 모드에서만 이뤄지며 배포 환경에는 영향을 미치지 않습니다.
  - 초기화 함수가 순수하다면 영향을 미치지 않습니다.
  - 호출 중 하나의 결과는 무시됩니다.

<br>

## 사용법

### 컴포넌트에 상태 추가하기

- 컴포넌트 최상단에서 `useState`를 호출하여 1개 이상의 상태 변수를 선언합니다.

```jsx
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(42);
  const [name, setName] = useState('Taylor');
  // ...
}
```

<br>

- 일반적으로 배열 디스트럭처링을 사용하여 `[something, setSomething]`과 같은 상태 변수의 이름을 지정합니다.

<br>

- `useState`는 정확히 2개 항목이 있는 배열을 반환합니다.

<br>

1. `현재 상태 변수` (처음에 제공한 초기 상태)
2. 다른 값으로 변경할 수 있는 `set` 함수 (업데이트 함수)

<br>

- 상태를 업데이트하려면 `set` 함수를 호출합니다.

```jsx
function handleClick() {
  setName('Robin');
}
```

<br>

- `React`는 상태를 저장하고 새로운 값으로 컴포넌트를 다시 렌더링한 후 UI를 업데이트합니다.

<br>

### 주의

- `set` 함수를 호출해도 이미 실행 중인 코드의 현재 상태는 변경되지 않습니다.

```jsx
function handleClick() {
  setName('Robin');
  console.log(name); // "Taylor"!
}
```

<br>

- **다음 렌더링**부터 영향을 끼칩니다.

<br>

## 이전 상태를 기반으로 상태 업데이트

- `age`가 42라고 가정합니다.
- 아래 함수는 `setAge(age + 1)`를 세 번 호출합니다.

```jsx
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

<br>

- 그러나 결과적으로 `age`는 `45`가 아니라 `43`이 됩니다!
- `set` 함수를 호출해도 이미 실행 중인 코드에서 `age` 상태가 업데이트되지 않기 때문입니다.
- 따라서 `setAge(age + 1) === setAge(43)`입니다.

<br>

- 이를 해결하려면 `setAge`에 **업데이트 함수를 전달**합니다.

```jsx
function handleClick() {
  setAge((a) => a + 1); // setAge(42 => 43)
  setAge((a) => a + 1); // setAge(43 => 44)
  setAge((a) => a + 1); // setAge(44 => 45)
}
```

<br>

- 여기서 `a => a + 1`은 업데이트 함수입니다.
- **보류 중인 상태**를 가져와서 **다음 상태**를 계산합니다.

<br>

- `React`는 업데이트 함수를 **큐**에 넣습니다.
- 다음 렌더링 중에 동일한 순서로 호출합니다.

<br>

1. `a => a + 1`은 `42`를 받고 다음 상태로 `43`을 반환합니다.
2. `a => a + 1`은 `43`을 받고 다음 상태로 `44`를 반환합니다.
3. `a => a + 1`은 `44`를 받고 다음 상태로 `45`를 반환합니다.

<br>

- 큐에 업데이트 함수가 없으므로 `React`는 최종적으로 `45`를 현재 상태로 저장합니다.

<br>

- 일반적으로 이전 상태의 이름은 상태 이름의 첫 글자로 지정합니다.(예: `age`는 `a` 로 사용)
- `prevAge`와 같이 더 명확한 이름을 사용할 수 있습니다.

<br>

- 개발 모드에서 `React`는 업데이트 함수가 순수한지 확인하기 위해 2번 호출합니다.

<br>

## Deep Dive - 업데이트 함수를 사용하는 것이 항상 좋을까요?

- 이전 상태에 영향을 받는 경우 항상 `setAge(a => a + 1)`와 같은 코드를 작성하라고 권장합니다.
- 나쁠 것은 없지만 항상 필요한 것은 아닙니다.

<br>

- 대부분의 2가지 접근 방식 사이에는 차이가 없습니다.
- 클릭과 같은 유저 이벤트의 경우 `React`는 `age`가 클릭 전에 업데이트되도록 합니다.
- 즉 클릭 핸들러가 **이전의** `age`를 참조할 위험이 없습니다.

<br>

- 그러나 동일한 이벤트 내에서 여러 업데이트를 수행하는 경우 업데이트 함수가 유용합니다.
- 상태 변수 자체에 액세스하는 것이 어려운 경우에도 유용합니다.(리렌더링을 최적화할 때 이 문제가 발생할 수 있음)

<br>

- 이전 상태와 연관되어 다음 상태를 계산해야 할 때는 업데이트 함수를 작성합니다.
- 또다른 상태 변수의 이전 상태에서 계산된 경우 1개 객체로 결합하고 `reducer`를 사용할 수 있습니다.

<br>

## 객체 상태와 및 배열 상태 업데이트

- 객체와 배열을 상태에 넣을 수 있습니다.
- `React`에서 상태는 **읽기 전용**으로 간주되므로 **기존 객체를 변경하지 말고 교체해야 합니다.**
- 예를 들어 상태에 `form` 객체가 있는 경우 변경하지 안됩니다.

```js
// 🚩 객체를 변경하면 안됩니다.
form.firstName = 'Taylor';
```

<br>

- 새로운 객체를 생성하여 교체합니다.

```jsx
// ✅ 상태를 새로운 객체로 바꾸기
setForm({
  ...form,
  firstName: 'Taylor',
});
```

<br>

## 초기 상태 다시 생성하지 않기

- `React`는 초기 상태를 저장하고 이후 렌더링에서 이를 무시합니다.

```jsx
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  // ...
}
```

<br>

- `createInitialTodos()`의 결과는 초기 렌더링에만 사용되지만 매 렌더링마다 이 함수를 호출합니다.
- 큰 배열을 생성하거나 고비용 연산을 수행하는 경우 낭비입니다.

<br>

- 이 문제를 해결하려면 `useState`에 초기화 함수를 전달합니다.

```jsx
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  // ...
}
```

<br>

- 함수 호출 결과인 `createInitialTodos()`가 아니라 함수 자체를 전달합니다.
- 함수를 `useState`에 전달하면 `React는` 초기화 중에만 `createInitialTodos` 함수를 호출합니다.

<br>

- `React`는 순수한지 확인하기 위해 개발 모드에서 초기화 함수를 2번 호출합니다.

<br>

## `key`를 활용한 상태 재설정

- **배열을 렌더링**할 때 `key` 어트리뷰트를 자주 만납니다.
- 하지만 `key` 어트리뷰트는 다른 용도로도 사용됩니다.

<br>

- 컴포넌트에 **다른 키를 전달하여 컴포넌트의 상태를 재설정합니다.**
- 아래 예제는 초기화 버튼이 `version` 상태 변수를 변경하고 이를 `Form`에 `key`로 전달합니다.
- `key`가 변경되면 `React`는 `Form` 컴포넌트(및 하위 컴포넌트)를 처음부터 다시 생성하므로 상태가 초기화됩니다.

```jsx
import { useState } from 'react';

export default function App() {
  const [version, setVersion] = useState(0);

  function handleReset() {
    setVersion(version + 1);
  }

  return (
    <>
      <button onClick={handleReset}>Reset</button>
      <Form key={version} />
    </>
  );
}

function Form() {
  const [name, setName] = useState('Taylor');

  return (
    <>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <p>Hello, {name}.</p>
    </>
  );
}
```

<br>

## 이전 렌더링의 정보 저장

- 일반적으로 이벤트 핸들러에서 상태를 업데이트합니다.
- 드물게 렌더링되는 값에 따라 상태를 조정해야 하는 경우도 있습니다.
- `prop`이 변경될 때 상태를 변경하는 경우입니다.

<br>

- 대부분의 경우 이 기능은 필요하지 않습니다.
  - **필요한 값을 현재 `prop`이나 다른 상태에서 모두 계산할 수 있다면 중복되는 상태를 모두 제거합니다.**
  - 너무 자주 연산하는 것이 걱정된다면 `useMemo` `hook`을 사용합니다.
  - 컴포넌트 트리의 전체 상태를 초기화하려면 컴포넌트에 다른 `key`를 전달합니다.
  - 가능하다면 이벤트 핸들러에서 모든 상태를 업데이트합니다.

<br>

- 어느 것도 해당되지 않는 드문 경우 컴포넌트가 렌더링되는 동안 `set` 함수를 호출하여 이전 값을 기반으로 상태를 업데이트하는 데 사용하는 패턴이 있습니다.

<br>

- `CountLabel` 컴포넌트는 전달된 `count` `prop`를 표시합니다.

```jsx
export default function CountLabel({ count }) {
  return <h1>{count}</h1>;
}
```

<br>

- 카운터가 마지막 변경 이후 증감했는지 표시해봅시다.
- `count` `prop`는 이를 알려주지 않으므로 이전 값을 추적합니다.
- 이를 위해 `prevCount` 상태 변수를 추가합니다.
- `trend`를 추가하여 `count`의 증감 여부를 저장합니다.
- `prevCount`와 `count`가 다른 경우 `prevCount`와 `trend`를 모두 업데이트합니다.
- 이제 `count` `prop`과 마지막 렌더링 이후 `count`의 변경을 표시할 수 있습니다.

```jsx
// App.js
import { useState } from 'react';
import CountLabel from './CountLabel.js';

export default function App() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <CountLabel count={count} />
    </>
  );
}
```

```jsx
// CountLable.js
import { useState } from 'react';

export default function CountLabel({ count }) {
  const [prevCount, setPrevCount] = useState(count);
  const [trend, setTrend] = useState(null);
  if (prevCount !== count) {
    setPrevCount(count);
    setTrend(count > prevCount ? 'increasing' : 'decreasing');
  }
  return (
    <>
      <h1>{count}</h1>
      {trend && <p>The count is {trend}</p>}
    </>
  );
}
```

<br>

- 렌더링하는 동안 `set` 함수를 호출하는 경우 `prevCount !== count`와 같은 조건문 안에 있어야 하며 조건문 내부에 `setPrevCount(count)`와 같은 호출이 있어야 합니다.
- 그렇지 않으면 컴포넌트가 무한 리렌더링됩니다.
- 또한 현재 렌더링 중인 컴포넌트의 상태만 업데이트할 수 있습니다.
- 렌더링 중에 다른 컴포넌트의 `set` 함수를 호출하는 것은 오류입니다.
- 마지막으로 `set` 함수의 호출은 **부수효과 없이 상태를 업데이트**해야 합니다.

<br>

- 이 패턴은 이해하기 어려울 수 있으며 일반적으로 피하는 것이 가장 좋습니다.
- 하지만 `useEffect`에서 상태를 업데이트하는 것보다 낫습니다.
- 렌더링 도중 `set` 함수를 호출하면 `React`는 컴포넌트가 **`return`으로 종료된 직후와 하위 컴포넌트를 렌더링하기 전**에 리렌더링합니다.
- 이렇게 하면 **하위 컴포넌트를 2번 렌더링할 필요가 없습니다.**
- 나머지 컴포넌트는 계속 실행되고 결과는 버려집니다.
- 조건문이 모든 `hook` 호출 이후에 실행되면 `return;`을 추가하여 렌더링을 더 일찍 다시 시작할 수 있습니다.

<br>

## 트러블슈팅

### 상태를 업데이트했지만 로깅하면 이전 값이 표시됩니다.

- `set` 함수를 호출해도 실행 중인 코드의 상태는 변경되지 않습니다.

```jsx
function handleClick() {
  console.log(count); // 0

  setCount(count + 1); // 1로 다시 렌더링 요청하기
  console.log(count); // 여전히 0!

  setTimeout(() => {
    console.log(count); // 역시 0!
  }, 5000);
}
```

<br>

- 상태는 스냅샷처럼 동작합니다.
- 상태를 업데이트하면 렌더링을 요청하지만 **이미** 실행 중인 이벤트 핸들러의 `count` 변수에는 영향을 미치지 않습니다.

<br>

- 상태를 사용하는 경우 변수에 저장한 후 `set` 함수에 전달합니다.

```jsx
const nextCount = count + 1;
setCount(nextCount);

console.log(count); // 0
console.log(nextCount); // 1
```

<br>

### 상태를 업데이트했지만 렌더링된 값은 업데이트되지 않습니다.

- `Object.is` 메서드 의해 결정되므로 다음 상태가 이전 상태와 같으면 `React`는 업데이트를 무시합니다.
- 보통 객체나 배열의 상태를 직접 변경할 때 발생합니다.

```jsx
obj.x = 10; // 🚩 잘못: 기존 객체 변경
setObj(obj); // 🚩 아무것도 하지 않습니다.
```

<br>

- 기존 `obj`를 변경한 후 `setObj`를 전달했기 때문에 업데이트가 무시되었습니다.
- 객체와 배열을 **변경하는 대신 항상 상태의 객체와 배열을 교체**해야 합니다.

```jsx
// ✅  정답: 새 객체 만들기
setObj({
  ...obj,
  x: 10,
});
```

<br>

### 오류가 발생했습니다: "리렌더링 횟수가 너무 많습니다"

- 다음과 같은 오류가 발생할 수 있습니다 => `리렌더링 횟수가 너무 많습니다.`
- `React`는 **무한 루프**를 방지하기 위해 렌더링 횟수를 제한합니다.
- 일반적으로 렌더링 중에 상태를 설정하기 때문에 **컴포넌트 렌더링 => 상태 설정(렌더링 발생) => 렌더링 => 상태 설정(렌더링 발생)**의 사이클이 발생하는 것입니다.
- 이벤트 핸들러를 지정하는 과정에서 실수로 인해 발생합니다.

```jsx
// 🚩 오류: 렌더링 중에 핸들러를 호출합니다.
return <button onClick={handleClick()}>Click me</button>;

// ✅ 정답: 이벤트 핸들러를 전달합니다.
return <button onClick={handleClick}>Click me</button>;

// ✅ 정답: 인라인 함수를 전달합니다.
return <button onClick={(e) => handleClick(e)}>Click me</button>;
```

<br>

- 오류의 원인을 찾을 수 없는 경우 콘솔에서 오류 옆에 있는 화살표를 클릭하고 자바스크립트 스택을 살펴보고 오류의 원인이 되는 특정 `set` 함수를 찾습니다.

<br>

### 초기화 함수 or 업데이트 함수가 두 번 실행됩니다.

- `Strict Mode`에서 `React`는 일부 함수를 두 번 호출합니다.

```jsx
function TodoList() {
  // 이 컴포넌트 함수는 렌더링할 때마다 두 번 실행됩니다.
  const [todos, setTodos] = useState(() => {
    // 이 초기화 함수는 초기화 중에 두 번 실행됩니다.
    return createTodos();
  });

  function handleClick() {
    setTodos((prevTodos) => {
      // 이 업데이트 함수는 클릭할 때마다 두 번 실행됩니다.
      return [...prevTodos, createTodo()];
    });
  }
  // ...
}
```

<br>

- 이는 예상되는 현상이며 코드를 손상시키지 않습니다.
- 이러한 개발 모드 동작은 컴포넌트를 순수하게 유지하는 데 유용합니다.
- `React`는 하나의 결과를 사용하고 다른 결과는 무시합니다.
- 컴포넌트, 초기화 함수, 업데이트 함수가 순수하다면 로직에 영향을 주지 않습니다.
- 그러나 실수로 부수효과가 있는 경우 실수를 알아차리는 데 유용합니다.

<br>

- 예를 들어 아래 순수하지 않은 업데이트 함수는 상태의 배열을 변경합니다.

```jsx
setTodos((prevTodos) => {
  // 🚩 실수: 상태 변경
  prevTodos.push(createTodo());
});
```

<br>

- `React`는 업데이트 함수를 2번 호출하기 때문에 `todo`가 2번 추가되므로 오류입니다.
- 아래 예제는 배열을 **변경하는 대신 교체**합니다.

```jsx
setTodos((prevTodos) => {
  // ✅ 정답: 새 상태로 교체
  return [...prevTodos, createTodo()];
});
```

<br>

- 이제 업데이트 함수는 순수 하므로 2번 더 호출해도 차이가 없습니다.
- 그렇기 때문에 `React`가 2번 호출하면 실수를 찾는 데 유용합니다.
- **컴포넌트, 초기화 함수, 업데이트 함수만 순수해야 합니다.**
- 이벤트 핸들러는 순수할 필요가 없으므로 `React`는 이벤트 핸들러를 2번 호출하지 않습니다.

<br>

### 상태를 함수로 설정하는 대신 함수가 호출됩니다.

- 함수를 상태에 놓을 수는 없습니다.

```jsx
const [fn, setFn] = useState(someFunction);

function handleClick() {
  setFn(someOtherFunction);
}
```

<br>

- 함수를 전달하기 때문에 `React`는 `someFunction`이 **초기화 함수**고 `someOtherFunction`이 **업데이트 함수**라고 가정하고 호출하고 결과를 저장합니다.
- 함수를 저장하려면 함수 앞에 `() =>`를 넣어야 합니다.

```jsx
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  setFn(() => someOtherFunction);
}
```
