# useRef

- `useRef`는 렌더링에 필요하지 않은 값을 참조하는 `React` `hook`입니다.

```jsx
const ref = useRef(initialValue);
```

<br>

## 레퍼런스

### `useRef(initialValue)`

- 컴포넌트의 최상위 레벨에서 `useRef`를 호출하여 `ref`를 선언합니다.

```jsx
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  // ...
}
```

<br>

### 매개변수

- `initialValue`
  - `ref` 객체의 단일 프로퍼티인 `current`의 초기 값입니다.
  - 모든 값을 사용할 수 있습니다.
  - 초기 렌더링 이후에는 무시됩니다.

<br>

### 반환값

- `useRef`는 `current`라는 단일 프로퍼티를 가진 객체를 반환합니다.

<br>

- `current`
  - `useRef` 호출 시 전달한 초기값으로 설정됩니다.
  - 나중에 다른 값으로 설정할 수 있습니다.
  - `ref` 객체를 `JSX` 노드에 어트리뷰트로 전달하면 `React` 노드가 `current`에 할당됩니다.

<br>

### 주의사항

- `ref.current`는 변경될 수 있습니다.
  - 상태와 달리 변경 가능합니다.
  - 하지만 렌더링에 사용되는 객체(상태)를 담고 있다면 변경할 수 없습니다.
- `ref.current`를 변경해도 `React`는 컴포넌트는 리렌더링되지 않습니다.
  - `ref`는 일반 `JS` 객체이기 때문에 `React`는 변경 여부를 알지 못합니다.
- 초기화를 제외하고는 렌더링 도중 `ref.current`를 참조하거나 할당하지 않습니다.
- `Strict Mode`에서는 **실수로 발생한 부수효과를 찾기 위해** `React`가 **`useRef`를 두 번 호출**합니다.
  - 개발 모드에서만 이뤄지며 배포 환경에는 영향을 미치지 않습니다.
  - `ref` 객체는 2번 생성되지만 1개는 버려집니다.
  - 컴포넌트가 순수하다면 영향을 미치지 않습니다.

<br>

## 사용법

### `ref`로 값 참조하기

- 컴포넌트 최상단에서 `useRef`를 호출하여 `ref`를 선언합니다.

```jsx
import { useRef } from 'react';

function Stopwatch() {
  const intervalRef = useRef(0);
  // ...
}
```

<br>

- `useRef`는 처음에 제공한 **초기 값**으로 설정된 **단일** 프로퍼티 `current`가 있는 **`ref` 객체**를 반환합니다.

<br>

- 다음 렌더링에서 `useRef`는 해당 객체를 반환합니다.
- `current` 프로퍼티를 변경하여 정보를 저장하고 참조합니다.
- 상태와 유사하지만 중요한 차이점이 있습니다.

<br>

- `ref`를 변경해도 리렌더링 되지 않습니다.
- `ref`는 컴포넌트의 시각적 출력에 영향을 주지 않는 정보 저장에 유용합니다.
- 예를 들어 `interval ID`를 저장했다가 나중에 참조해야 하는 경우 `ref`를 사용합니다.
- `ref` 내부의 값을 업데이트하려면 `current` 프로퍼티를 **수동**으로 변경합니다.

```js
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}
```

<br>

- 나중에 `ref`의 `interval ID`를 참조하여 해당 `interval`을 초기화할 수 있습니다.

```js
function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```

<br>

- `ref`를 사용하면 아래 내용을 보장할 수 있습니다.
  - 렌더링할 때마다 재설정되는 일반 변수와 달리 리렌더링 간 **정보를 저장**합니다.
  - 변경해도 리렌더링되지 않습니다. (리렌더링을 트리거하는 상태와의 가장 큰 차이점)
  - 공유되는 외부 변수와 달리 컴포넌트에 **로컬로 저장**됩니다.

<br>

- `ref`를 변경해도 리렌더링되지 않으므로 시각적 정보 저장에는 `ref`가 적합하지 않습니다.
- 대신 상태를 사용합니다.

<br>

### 주의

- 렌더링 중에는 `ref.current`를 참조하거나 할당하지 않습니다.

<br>

- 컴포넌트는 순수 함수처럼 동작해야 합니다.
  - 입력(`props`, state, 컨텍스트)이 동일하다면 동일한 `JSX`를 반환해야 합니다.
  - 다른 순서나 다른 인수를 사용하여 호출해도 결과에 영향을 미치지 않아야 합니다.

<br>

- **렌더링 도중** `ref`를 참조하거나 할당하면 이러한 규칙이 깨집니다.

```jsx
function MyComponent() {
  // ...
  // 🚩 렌더링 중에 ref에 할당하지 않습니다.
  myRef.current = 123;
  // ...
  // 🚩 렌더링 중에 ref를 참조하지 않습니다.
  return <h1>{myOtherRef.current}</h1>;
}
```

<br>

- 이벤트 핸들러나 `useEffect`에서 `ref`를 참조하거나 할당합니다.

```jsx
function MyComponent() {
  // ...
  useEffect(() => {
    // ✅ `useEffect`에서 ref를 참조하거나 할당합니다.
    myRef.current = 123;
  });
  // ...
  function handleClick() {
    // ✅  이벤트 핸들러에서 ref를 참조하거나 할당합니다.
    doSomething(myOtherRef.current);
  }
  // ...
}
```

<br>

- 렌더링 **도중** 무언가를 참조하거나 할당하는 경우 **상태를 사용합니다.**
- 규칙을 어겨도 컴포넌트는 여전히 작동할 수 있지만 `React`에 추가되는 새로운 기능은 이러한 규칙을 준수합니다.

<br>

### ref로 DOM 조작하기

- `ref`를 사용해 DOM을 조작하는 것이 일반적입니다.
- `React`가 기본적으로 제공하는 기능입니다.

<br>

- 먼저 초기값이 `null`인 `ref` 객체를 선언합니다.

```jsx
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  // ...
}
```

<br>

- `ref` 객체를 조작하려는 DOM 노드의 JSX에 전달합니다

```jsx
// ...
return <input ref={inputRef} />;
// ref.current = input 노드
```

<br>

- DOM 노드를 렌더링하면 `React`는 **`ref` 객체의 `current` 프로퍼티에 해당 DOM 노드를 할당합니다.**
- 이제 `<input>`의 DOM 노드에 접근하여 `focus()`와 같은 메서드를 호출할 수 있습니다

```jsx
function handleClick() {
  inputRef.current.focus();
}
```

<br>

- `React`는 노드가 **화면에서 제거되면 `current` 프로퍼티를 다시 `null`로 설정**합니다.

<br>

### `ref` 콘텐츠의 재생성 방지

- `React`는 초기 `ref` 값을 1번 저장하고 다음 렌더링부터는 사용하지 않습니다.

```jsx
function Video() {
  const playerRef = useRef(new VideoPlayer());
  // ...
}
```

<br>

- `new VideoPlayer()`의 결과는 첫 렌더링에 사용되지만 이어지는 렌더링에서 매번 이 함수를 호출합니다.
- 고비용 객체를 생성하는 경우 낭비입니다.

<br>

- 이 문제를 해결하려면 다음과 같이 참조를 초기화합니다.

```jsx
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
}
```

<br>

- 일반적으로 렌더링 중에 `ref.current`를 참조하거나 할당하는 것은 허용되지 않습니다.
- 하지만 결과가 항상 동일하고 초기화 중에만 조건이 실행되므로 사용 가능합니다.

<br>

## Deep Dive - `useRef`를 나중에 초기화할 때 `null` 체크를 우회하는 방법

- 타입 체크 때문에 매번 `null`을 검사하고 싶지 않다면 아래 패턴을 사용합니다.

```jsx
function Video() {
  const playerRef = useRef(null);

  function getPlayer() {
    if (playerRef.current !== null) {
      return playerRef.current;
    }
    const player = new VideoPlayer();
    playerRef.current = player;
    return player;
  }
  // ...
}
```

<br>

- 여기서 `playerRef` 자체는 `null`이 가능합니다.
- 하지만 타입 검사에서 `getPlayer()`가 `null`을 반환하지 않는다는 것을 확실히 해야합니다.
- 그런 다음 이벤트 핸들러에서 `getPlayer()`를 사용합니다.

<br>

## 트러블슈팅

### 커스텀 컴포넌트에 대한 `ref`를 얻을 수 없습니다.

```jsx
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

<br>

- 예시와 같이 커스텀 컴포넌트에 `ref`를 전달하는 경우 아래 오류를 마주칠 수 있습니다.

```
Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
```

<br>

- 기본적으로 커스텀 컴포넌트는 DOM 노드에 대한 `ref`를 노출하지 않습니다.
- 문제를 해결하려면 `ref`를 가져오고자 하는 컴포넌트를 `forwardRef`로 래핑합니다.

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(({ value, onChange }, ref) => {
  return <input value={value} onChange={onChange} ref={ref} />;
});

export default MyInput;
```

<br>

- 이제 상위 컴포넌트가 참조할 수 있습니다.
