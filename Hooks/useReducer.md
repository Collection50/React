# useReducer

- `useReducer`는 컴포넌트에 `reducer`를 추가하는 `React` `hook`입니다.

```jsx
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

<br>

## 레퍼런스

### `useReducer(reducer, initialArg, init?)`

- 컴포넌트의 최상단에서 `useReducer`를 사용하여 리듀서를 통해 상태를 관리합니다.

```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
}
```

<br>

### 매개변수

- `reducer`
  - 상태가 업데이트되는 방식을 지정하는 리듀서 함수입니다.
  - 순수 함수여야 하며 상태와 액션을 인수로 받고 다음 상태를 반환합니다.
  - 상태와 액션은 모든 유형의 값을 사용할 수 있습니다.
- `initialArg`
  - 초기 상태 값입니다.
  - 모든 유형의 값을 사용할 수 있습니다.
  - 초기 상태는 `init`에 따라 달라집니다.
- `init` (**옵션**)
  - 초기 상태를 반환하는 초기화 함수입니다.
  - 지정하지 않으면 초기 상태는 `initialArg`입니다.
  - 초기값을 지정하면 `init(initialArg)`의 결과입니다.

<br>

### 반환값

- `useReducer`는 정확히 2개의 값을 가진 배열을 반환합니다.

<br>

1. 현재 상태입니다.
   - 첫 렌더링에는 `init(initialArg)`로 설정됩니다. (초기값이 없는 경우 `initialArg`)
2. 상태를 업데이트하고 리렌더링을 트리거하는 `dispatch` 함수입니다.

<br>

### 주의사항

- `useReducer` 는 `hook`이므로 **컴포넌트의 최상단**이나 커스텀 `hook`에서만 호출할 수 있습니다.
  - 반복문, 조건문 내부에서는 호출할 수 없습니다.
  - 필요하다면 새 컴포넌트를 추출하고 상태를 옮깁니다.
- `Strict Mode`에서는 **실수로 발생한 부수효과를 찾기 위해** `React`가 **리듀서와 초기화 함수를 두 번 호출**합니다.
  - 개발 모드에서만 이뤄지며 배포 환경에는 영향을 미치지 않습니다.
  - 리듀서와 초기화 함수가 순수하다면 영향을 미치지 않습니다.
  - 호출 결과 중 하나는 무시됩니다.

<br>

## `dispatch` 함수

- `useReducer`가 반환하는 `dispatch`를 사용하여 상태를 업데이트하고 리렌더링합니다.
- `dispatch`는 유일한 인수로 **액션**을 전달합니다.

```jsx
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
  // ...
}
```

<br>

- `dispatch`에 `state`와 액션을 전달하여 상태를 설정합니다.

<br>

### 매개변수

- `action`
  - 사용자가 수행한 작업입니다.
  - 모든 유형의 값을 사용할 수 있습니다.
  - 일반적으로 액션은 일반적으로 `type` 속성이 있는 객체이며 다른 속성을 포함할 수 있습니다.

<br>

### 반환값

- `dispatch` 함수는 반환값이 없습니다.

<br>

### 주의사항

- `dispatch` 함수는 다음 렌더링에 대한 상태 변수만 업데이트합니다.
- `dispatch` 함수를 호출한 후 상태 변수를 참조하면 이전 값을 그대로 가져옵니다.

<br>

- 새 값이 `Object.is` 비교에 의해 결정된 현재 상태와 동일하다면 `React`는 컴포넌트와 하위 컴포넌트를 리렌더링하지 않습니다.
- 이것은 최적화입니다.

<br>

- `React`는 상태 업데이트를 **일괄적**으로 수행합니다.
  - 모든 **이벤트 핸들러가 실행되고 `set` 함수를 호출한 후**에 화면을 업데이트합니다.
  - 이렇게 하면 단일 이벤트 중에 여러 번 리렌더링되지 않습니다.
  - 드물지만 DOM에 접근하기 위해 화면을 일찍 업데이트하도록 해야 하는 경우엔 `flushSync`를 사용합니다.

<br>

## 사용법

### 컴포넌트에 리듀서 추가하기

- 컴포넌트 최상단에서 `useReducer`를 호출하고 리듀서를 통해 상태를 관리합니다.

```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
}
```

<br>

- `useReducer`는 정확히 2개의 값을 가진 배열을 반환합니다.

<br>

1. 상태 변수의 현재 값으로 초기 상태입니다.
2. 상호작용에 반응하여 상태를 변경할 수 있는 `dispatch` 함수입니다.

<br>

- 상태를 업데이트하려면 액션을 사용하여 `dispatch`를 호출합니다.

```jsx
function handleClick() {
  dispatch({ type: 'incremented_age' });
}
```

<br>

- `React`는 현재 상태와 액션을 **리듀서 함수**에 전달합니다.
- 리듀서는 다음 상태를 계산하고 반환합니다.
- `React`는 다음 상태를 저장하고 컴포넌트를 렌더링하고 UI를 업데이트합니다.

<br>

```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  if (action.type === 'incremented_age') {
    return {
      age: state.age + 1,
    };
  }
  throw Error('Unknown action.');
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  return (
    <>
      <button
        onClick={() => {
          dispatch({ type: 'incremented_age' });
        }}
      >
        Increment age
      </button>
      <p>Hello! You are {state.age}.</p>
    </>
  );
}
```

<br>

- `useReducer`는 `useState`와 매우 비슷하지만 상태 업데이트 로직을 컴포넌트 외부로 옮길 수 있습니다.

<br>

### 리듀서 함수 작성

- 리듀서 함수는 아래와 같이 선언됩니다.

```jsx
function reducer(state, action) {
  // ...
}
```

<br>

- 다음 상태를 계산하고 반환할 코드를 입력합니다.
- 일반적으로 `switch` 문으로 작성합니다.
- `switch`의 각 `case` 대해 상태를 계산하고 반환합니다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        name: state.name,
        age: state.age + 1,
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age,
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

<br>

- 액션은 어떤 형태든 가질 수 있습니다.
- 일반적으로 액션을 식별하는 `type` 프로퍼티가 있는 객체를 전달합니다.
- 리듀서가 상태를 계산하는 데 필요한 최소한의 정보가 포함됩니다.

<br>

```jsx
function Form() {
  const [state, dispatch] = useReducer(reducer, { name: 'Taylor', age: 42 });

  function handleButtonClick() {
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    dispatch({
      type: 'changed_name',
      nextName: e.target.value,
    });
  }
  // ...
}
```

<br>

- 액션 `type` 이름은 컴포넌트에 로컬로 지정됩니다.
- 각 액션은 하나의 상호작용을 설명합니다.
- 상태는 일반적으로 객체나 배열입니다.

<br>

### 주의

- 상태는 읽기 전용입니다.
- 객체나 배열을 직접 수정하지 않습니다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // 🚩 상태의 객체를 변경하지 마세요.
      state.age = state.age + 1;
      return state;
    }
  }
}
```

<br>

- 새 객체를 반환합니다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ✅ 새 객체를 반환합니다.
      return {
        ...state,
        age: state.age + 1,
      };
    }
  }
}
```

<br>

### 초기 상태 다시 생성하지 않기

- `React`는 초기 상태를 1번 저장하고 이어지는 렌더링에서 이를 무시합니다.

```jsx
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
  // ...
}
```

<br>

- `createInitialState(username)`의 결과는 첫 렌더링에만 사용되지만 이어지는 모든 렌더링에서 호출됩니다.
- 크기가 큰 배열을 생성하거나 고비용 연산을 수행하는 경우 낭비입니다.

<br>

- 이 문제를 해결하려면 `useReducer`의 3번째 인수로 **초기화** 함수를 전달합니다.

```jsx
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, username, createInitialState);
  // ...
}
```

<br>

- 함수 호출 결과가 아니라 함수 자체인 `createInitialState`를 전달합니다.
- 이렇게 하면 렌더링될 때마다 초기 상태가 다시 생성되지 않습니다.

<br>

- 위에서 `createInitialState`는 `username`을 받습니다.
- 초기 상태를 연산하는 데 아무런 정보가 필요하지 않은 경우 `useReducer`의 2번째 인수로 `null`을 전합니다.

<br>

## 트러블슈팅

### 액션을 `dispatch`했지만 이전 상태 값이 표시됩니다.

- `dispatch` 함수를 호출해도 실행 중인 코드의 상태는 변경되지 않습니다.

```jsx
function handleClick() {
  console.log(state.age); // 42

  dispatch({ type: 'incremented_age' }); // 43으로 다시 렌더링 요청
  console.log(state.age); // 여전히 42!

  setTimeout(() => {
    console.log(state.age); // 역시 42!
  }, 5000);
}
```

<br>

- 상태는 스냅샷처럼 동작하기 때문입니다.
- 상태를 업데이트하면 리렌더링을 요청하지만 실행 중인 이벤트 핸들러의 상태 변수에는 영향을 미치지 않습니다.

<br>

- 다음 상태 값을 예측해야 하는 경우 리듀서를 직접 호출하여 수동으로 계산합니다.

```jsx
const action = { type: 'incremented_age' };
dispatch(action);

const nextState = reducer(state, action);
console.log(state); // { age: 42 }
console.log(nextState); // { age: 43 }
```

<br>

### 액션을 `dispatch`했지만 리렌더링되지 않습니다.

- `Object.is` 메서드에 의해 다음 상태가 이전 상태와 같으면 `React`는 업데이트를 무시합니다.
- 보통 객체나 배열을 직접 변경할 때 발생합니다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // 🚩 오류: 기존 객체 변경
      state.age++;
      return state;
    }
    case 'changed_name': {
      // 🚩 오류: 기존 객체 변경
      state.name = action.nextName;
      return state;
    }
    // ...
  }
}
```

<br>

- 기존 객체를 변경했기 때문에 `React`가 업데이트를 무시했습니다.
- 항상 **객체나 배열을 업데이트합니다.**

<br>

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ✅ 정답: 새 객체 만들기
      return {
        ...state,
        age: state.age + 1,
      };
    }
    case 'changed_name': {
      // ✅ 정답: 새 객체 만들기
      return {
        ...state,
        name: action.nextName,
      };
    }
    // ...
  }
}
```

<br>

### `dispatch` 후 리듀서의 일부 데이터가 `undefined`입니다.

- `case` 분기가 기존 데이터를 복사하는지 확인합니다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        ...state, // 이 부분을 잊지 마세요!
        age: state.age + 1,
      };
    }
    // ...
  }
}
```

<br>

- 위의 `...state`가 없으면 `age` 값만 포함된 객체가 됩니다.

<br>

### `dispatch` 후 리듀서의 모든 데이터가 `undefined`입니다.

- 상태가 `undefined`인 경우 `case`에서 `return`이 없거나 액션 `type`이 `case` 분기와 일치하지 않는 경우입니다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ...
    }
    case 'edited_name': {
      // ...
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

<br>

- `TypeScript`와 같은 정적 `type` 분석을 사용하여 이러한 실수를 방지합니다.

<br>

### 오류가 발생했습니다: "리렌더링 횟수가 너무 많습니다"

- 다음과 같은 오류가 발생할 수 있습니다.
- `Too many re-renders. React limits the number of renders to prevent an infinite loop.`
- 일반적으로 렌더링 중에 무조건 액션을 `dispatch`하는 것을 의미하므로 컴포넌트 렌더링 => `dispatch`(렌더링 트리거) => 렌더링 => `dispatch`(렌더링 트리거) 등의 루프에 진입하게 됩니다.
- 이벤트 핸들러를 지정하는 과정에서 실수로 인해 발생하는 경우가 많습니다.

```jsx
// 🚩 오류: 렌더링 중에 핸들러를 호출합니다.
return <button onClick={handleClick()}>Click me</button>;

// ✅ 정답: 이벤트 핸들러를 전달합니다.
return <button onClick={handleClick}>Click me</button>;

// ✅ 정답: 인라인 함수를 전달합니다.
return <button onClick={(e) => handleClick(e)}>Click me</button>;
```

<br>

- 오류의 원인을 찾을 수 없는 경우 콘솔에서 오류 옆에 있는 화살표를 클릭하고 자바스크립트 스택을 살펴보고 원인이 되는 특정 디스패치 함수 호출을 찾습니다.

<br>

### 리듀서 또는 초기화 함수가 2번 실행됩니다.

- `Strict Mode`에서 `React`는 리듀서와 초기화 함수를 2번 호출합니다.
- 이 현상이 코드를 망가뜨리지 않습니다.

<br>

- 개발 모드 동작은 컴포넌트를 순수하게 유지하는 데 유용합니다.
- `React`는 1개 결과를 사용하고 다른 결과는 무시합니다.
- 컴포넌트, 초기화 함수, 리듀서 함수가 순수하다면 로직에 영향을 미치지 않습니다.
- 하지만 부수효과가 존재하는 경우 버그를 확인하는 데 유용합니다.

<br>

- 아래 리듀서 함수는 상태의 배열을 변경합니다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'added_todo': {
      // 🚩 실수: 상태 '변경'
      state.todos.push({ id: nextId++, text: action.text });
      return state;
    }
    // ...
  }
}
```

<br>

- `React`는 리듀서 함수를 2번 호출하기 때문에 `todo`가 2번 추가되었으므로 오류를 확인합니다.
- 배열을 변경하는 대신 교체하여 오류를 수정합니다.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'added_todo': {
      // ✅ 정답: 새 상태로 바꾸기
      return {
        ...state,
        todos: [...state.todos, { id: nextId++, text: action.text }],
      };
    }
    // ...
  }
}
```

<br>

- 리듀서 함수는 순수 함수이므로 1번 더 호출해도 동작에 차이가 없습니다.
- `React`가 2번 호출하면 실수를 찾는 데 유용합니다.
- **컴포넌트, 초기화 함수, 리듀서 함수만 순수해야 합니다.**
- 이벤트 핸들러는 순수할 필요가 없으므로 이벤트 핸들러를 2번 호출하지 않습니다.
