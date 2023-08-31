# useDebugValue

- `useDebugValue`는 `React DevTools`에서 커스텀 `hook`에 레이블을 추가하는 `React` `hook`입니다.

```jsx
useDebugValue(value, format?)
```

<br>

## 레퍼런스

### `useDebugValue(value, format?)`

- 커스텀 `hook`의 최상단에서 `useDebugValue`를 호출하여 디버그 값을 표시합니다.

```jsx
import { useDebugValue } from 'react';

function useOnlineStatus() {
  // ...
  useDebugValue(isOnline ? 'Online' : 'Offline');
  // ...
}
```

<br>

### 매개변수

- `value`
  - `React DevTools`에 표시할 값입니다.
  - 어떠한 유형의 값도 사용할 수 있습니다.
- `foramt`(**옵션**)
  - `React DevTools`는 `format(value)`를 호출하고 반환값(포맷된 값)을 표시합니다.
  - `format` 함수를 지정하지 않으면 기존의 값이 표시됩니다.

<br>

### 반환값

- `useDebugValue`는 아무것도 반환하지 않습니다.

<br>

## 사용법

### 커스텀 `hook`에 레이블 추가하기

- 커스텀 `hhok`의 최상단에서 `useDebugValue`를 호출하여 `React DevTools` **디버그 값**의 가독성을 향상합니다.

```jsx
import { useDebugValue } from 'react';

function useOnlineStatus() {
  // ...
  useDebugValue(isOnline ? 'Online' : 'Offline');
  // ...
}
```

<br>

- `useOnlineStatus`를 호출하는 컴포넌트에 `OnlineStatus`와 같은 레이블이 지정됩니다
- **`Online`** 레이블이 표시됩니다.

<img width="648" alt="image" src="https://github.com/Leets-Official/Leets-FE/assets/86355699/be1f9ca4-0abf-4823-a842-3a4d5ecee96f">

<br>

- `useDebugValue` 호출이 없으면 기본 데이터(예제에서는 `true`)만 표시됩니다.

<br>

```jsx
// App.js
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

export default function App() {
  return <StatusBar />;
}
```

```jsx
// useOnlineStatus.js
import { useSyncExternalStore, useDebugValue } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(
    subscribe,
    () => navigator.onLine,
    () => true
  );
  useDebugValue(isOnline ? 'Online' : 'Offline');
  return isOnline;
}

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

<br>

### 참고

- 모든 커스텀 `hook`에 디버그 값을 추가하지 않습니다.
- 이 기능은 공유 라이브러리의 일부이며 복잡한 구조를 가진 커스텀 `hook`에 유용합니다.

<br>

### 디버그 값의 포맷팅 연기하기

- 2번째 인수에 포맷팅 함수를 전달할 수도 있습니다.

```jsx
useDebugValue(date, (date) => date.toDateString());
```

<br>

- 포맷팅 함수는 **디버그 값**을 매개변수로 받고 **포맷팅된 값**을 반환합니다.
- 컴포넌트가 검사되면 `React DevTools`가 이 함수를 호출하고 결과를 표시합니다.

<br>

- 컴포넌트를 실제로 검사하지 않는 한 고비용 포맷팅 로직을 실행하지 않습니다.
- 예를 들어 `date`가 날짜 값인 경우 렌더링할 때마다 `toDateString()`을 호출하지 않아도 됩니다.
