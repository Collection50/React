# useSyncExternalStore

- `useSyncExternalStore`는 외부 스토어를 구독하는 `React` `hook`입니다.

```jsx
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)
```

<br>

## 레퍼런스

### `useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`

- 컴포넌트 최상단에서 `useSyncExternalStore`를 호출하여 외부 데이터 스토어에서 값을 참조합니다.

```jsx
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(
    todosStore.subscribe,
    todosStore.getSnapshot
  );
  // ...
}
```

<br>

- 스토어에 있는 데이터의 스냅샷을 반환합니다.
- 2개 함수를 인수로 전달합니다.

1. `subscribe` 함수는 스토어를 구독하며 구독 취소 함수를 반환합니다.
2. `getSnapshot` 함수는 스토어에서 데이터의 스냅샷을 참조합니다.

<br>

### 매개변수

- `subscribe`
  - 단일 `callback` 함수를 받아 스토어에 구독하는 함수입니다.
  - 스토어가 변경되면 `callback`을 호출합니다.
  - 그러면 컴포넌트가 리렌더링됩니다.
  - `subscribe` 함수는 구독 취소 함수를 반환합니다.
- `getSnapshot`
  - 컴포넌트에 필요한 스토어 데이터의 스냅샷을 반환하는 함수입니다.
  - 스토어가 변경되지 않은 상태에서 `getSnapshot`을 반복적으로 호출하면 동일한 값을 반환합니다.
  - 스토어가 변경되고 반환된 값이 다른 경우(`Object.is` 메서드 사용) `React`는 컴포넌트를 리렌더링합니다.
- `getServerSnapshot` **(옵션)**
  - 스토어에 있는 데이터의 초기 스냅샷을 반환하는 함수입니다.
  - `SSR` 도중과 `SSR`된 콘텐츠를 `hydration`하는 중에만 사용됩니다.
  - 서버 스냅샷은 클라이언트와 서버와 동일해야 하며 일반적으로 직렬화(`serialize`)되어 클라이언트로 전달됩니다.
  - 이 인수를 생략하면 `SSR`에서 오류가 발생합니다.

<br>

### 반환값

- 렌더링 로직에 사용할 수 있는 스토어의 현재 스냅샷 데이터입니다.

<br>

### 주의사항

- `getSnapshot`이 반환하는 스토어 스냅샷은 변경 불가능합니다.
- 기본 스토어에 변경 가능한 데이터가 있는 경우, 데이터가 변경되면 변경 불가능한 새 스냅샷을 반환합니다.
- 데이터가 변경되지 않으면 캐시된 마지막 스냅샷을 반환합니다.

<br>

- 리렌더링하는 동안 다른 `subscribe` 함수가 전달되면 `React`는 새로 전달된 `subscribe` 함수를 사용하여 스토어를 다시 구독합니다.
- 컴포넌트 외부에서 `subscribe`을 선언하면 이를 방지할 수 있습니다.

<br>

## 사용법

### 외부 스토어 구독

- 대부분의 `React` 컴포넌트는 `prop`, 상태, 컨텍스트에서만 데이터를 참조합니다.
- 하지만 외부의 스토어에서 일부 데이터를 참조하는 경우가 존재합니다.

<br>

1. `React` 외부에 상태를 보관하는 서드파티 상태 관리 라이브러리
2. 변경 가능한 값, 변경 사항을 구독하는 브라우저 API

<br>

- 컴포넌트 최상단에서 `useSyncExternalStore`를 호출하여 외부 스토어에서 값을 참조합니다.

```jsx
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(
    todosStore.subscribe,
    todosStore.getSnapshot
  );
  // ...
}
```

<br>

- 스토어에 있는 데이터의 스냅샷을 반환합니다.
- 2개 함수를 인수로 전달합니다.

<br>

1. `subscribe` 함수는 스토어를 구독하며 구독 취소 함수를 반환합니다.
2. `getSnapshot` 함수는 스토어에서 데이터의 스냅샷을 참조합니다.

<br>

- `React`는 컴포넌트를 스토어에 구독한 상태로 유지합니다.

<br>

- 예를 들어 아래 `todosStore`는 `React` 외부에 데이터를 저장하는 외부 스토어입니다.
- `TodosApp` 컴포넌트는 `useSyncExternalStore` `hook`을 사용해 외부 스토어에 연결합니다.

```jsx
// App.js
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

export default function TodosApp() {
  const todos = useSyncExternalStore(
    todosStore.subscribe,
    todosStore.getSnapshot
  );
  return (
    <>
      <button onClick={() => todosStore.addTodo()}>Add todo</button>
      <hr />
      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}
```

```jsx
// todoStore.js
// 서드파티 스토어의 예시입니다.
// React와 통합해야 할 수도 있습니다.
// 앱이 React로 완전히 구축된 경우 React 상태를 사용하는 것이 좋습니다.

let nextId = 0;
let todos = [{ id: nextId++, text: 'Todo #1' }];
let listeners = [];

export const todosStore = {
  addTodo() {
    todos = [...todos, { id: nextId++, text: 'Todo #' + nextId }];
    emitChange();
  },
  subscribe(listener) {
    listeners = [...listeners, listener];
    return () => {
      listeners = listeners.filter((l) => l !== listener);
    };
  },
  getSnapshot() {
    return todos;
  },
};

function emitChange() {
  for (let listener of listeners) {
    listener();
  }
}
```

<br>

### 참고

- 가능하면 `React` 상태는 `useState` 및 `useReducer`와 함께 사용하는 것을 권장합니다.
- `useSyncExternalStore` API는 `NonReact` 코드와 통합할 때 유용합니다.

<br>

### 브라우저 API 구독

- `useSyncExternalStore`를 사용하는 다른 이유는 브라우저의 변경 값을 구독하는 경우입니다.
- 예를 들어 네트워크의 활성화 여부를 표시하는 경우입니다.
- 브라우저에는 `navigator.onLine`이라는 속성이 존재합니다.

<br>

- 이 값은 `React`가 모르는 사이에 변경될 수 있으므로 `useSyncExternalStore`로 참조합니다.

<br>

```jsx
import { useSyncExternalStore } from 'react';

function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}
```

<br>

- `getSnapshot` 함수를 구현하려면 브라우저 API의 현재 값을 참조합니다.

```jsx
function getSnapshot() {
  return navigator.onLine;
}
```

<br>

- 다음으로 `subscribe` 함수를 구현합니다.
- 예를 들어 `navigator.onLine`이 변경되면 브라우저의 `window` 개체에서 `online`, `offiline` 이벤트를 실행합니다.
- `callback` 인수를 이벤트에 구독하며 구독 취소 함수를 반환합니다.

```jsx
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

- `React`는 외부 `navigator.onLine` API에서 값을 참조하는 방법과 변경 사항을 구독하고 있습니다.
- 네트워크가 끊기면 컴포넌트가 리렌더링됩니다.

```jsx
import { useSyncExternalStore } from 'react';

export default function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function getSnapshot() {
  return navigator.onLine;
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

### 커스텀 hook으로 로직 추출하기

- 일반적으로 컴포넌트에서 직접 `useSyncExternalStore`를 작성하지 않습니다.
- 커스텀 `hook`에서 호출됩니다.
- 서로 다른 컴포넌트에서 동일한 외부 스토어를 사용할 수 있습니다.

<br>

- `useOnlineStatus` `hook`은 네트워크의 연결 여부를 저장합니다.

```jsx
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return isOnline;
}

function getSnapshot() {
  // ...
}

function subscribe(callback) {
  // ...
}
```

<br>

- 다른 컴포넌트에서 `useOnlineStatus`를 호출할 수 있습니다.

```jsx
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

<br>

### `SSR` 구현

- `SSR`을 사용하는 경우 `React` 컴포넌트는 브라우저 외부에서 실행되어 초기 HTML을 생성합니다.
- 이로 인해 외부 스토어에 연결할 때 몇 가지 문제가 발생합니다.

<br>

1. 브라우저 API는 서버에 존재하지 않기 때문에 작동하지 않습니다.
2. 서드파티 스토어에 연결하는 경우 서버와 클라이언트 간에 데이터가 일치해야 합니다.

<br>

- 이러한 문제를 해결하려면 `getServerSnapshot` 함수를 `useSyncExternalStore`의 3번째 인수로 전달합니다.

```jsx
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(
    subscribe,
    getSnapshot,
    getServerSnapshot
  );
  return isOnline;
}

function getSnapshot() {
  return navigator.onLine;
}

function getServerSnapshot() {
  return true; // 서버 생성 HTML에 항상 online 표시
}

function subscribe(callback) {
  // ...
}
```

<br>

- `getServerSnapshot` 함수는 `getSnapshot`과 유사하지만 2가지 상황에서만 실행됩니다.

<br>

1. HTML을 생성할 때 **서버**에서 실행됩니다.
2. `React`가 서버 HTML을 가져와서 인터랙티브하게 만들 때(`hydration`) **클라이언트**에서 실행됩니다.

<br>

- 앱이 상호작용되기 전에 사용될 초기 스냅샷 값을 제공합니다.
- `SSR`에 초기값이 없는 경우 이 인수를 생략하여 `CSR`을 실행합니다.

<br>

### 참고

- `getServerSnapshot`은 `CSR`에서 반환한 것과 동일한 정확한 데이터를 반환해야 합니다.
- `getServerSnapshot`이 서버에 이미 존재하는 스토어 데이터를 반환하는 경우 이 데이터를 클라이언트로 전송해야 합니다.
- 이를 수행하는 방법은 `SSR` 중에 `window.MY_STORE_DATA`와 같은 전역 변수를 설정하는 `<script>` 태그를 생성하고 클라이언트에서 `getServerSnapshot` 에서 해당 전역 변수를 읽어오는 것입니다.
- 외부 스토어에서 해당 방법에 대한 지침을 제공해야 합니다.

<br>

## 트러블슈팅

### 오류가 발생했습니다: "`getSnapshot`의 결과를 캐시해야 합니다."

- 예를 들어 `getSnapshot` 함수가 호출될 때마다 새 객체를 반환하는 경우입니다.

```jsx
function getSnapshot() {
  // 🔴 항상 다른 객체를 반환하면 안 됩니다.
  return {
    todos: myStore.todos,
  };
}
```

<br>

- `React`는 `getSnapshot` 반환값이 지난번과 다르면 컴포넌트를 리렌더링합니다.
- 항상 다른 값을 반환하면 무한 루프에 들어가서 오류가 발생합니다.

<br>

- 실제로 변경 사항이 있는 경우에만 `getSnapshot` 객체가 다른 객체를 반환해야 합니다.
- 스토어에 변경 불가능한 데이터가 포함된 경우 해당 데이터를 직접 반환합니다.

<br>

```jsx
function getSnapshot() {
  // ✅  불변 데이터를 반환합니다.
  return myStore.todos;
}
```

<br>

- 스토어 데이터가 변경 가능한 경우 `getSnapshot` 함수는 변경 불가능한 스냅샷 데이터를 반환해야 합니다.
- 새 객체를 생성해야 하지만 매번 호출할 때마다 새 객체를 생성하면 안 됩니다.
- 마지막 스냅샷을 저장하고 스토어의 데이터가 변경되지 않은 경우 이전의 스냅샷을 반환합니다.
- 데이터가 변경되었는지 확인하는 방법은 스토어에 따라 다릅니다.

<br>

### 리렌더링할 때마다 `subscribe` 함수가 호출됩니다.

- `subscribe` 함수는 컴포넌트 내부에 정의되므로 리렌더링될 때마다 달라집니다.

```jsx
function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);

  // 🚩 항상 새로 생성되므로 React는 리렌더링할 때마다 다시 구독합니다.
  function subscribe() {
    // ...
  }

  // ...
}
```

<br>

- 리렌더링 간 `React`가 스토어를 다시 구독합니다.
- 이로 인해 성능 문제가 발생하므로 `subscribe` 함수를 외부로 이동합니다.

```jsx
function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}

// ✅  항상 동일한 함수이므로 다시 구독할 필요가 없습니다.
function subscribe() {
  // ...
}
```
