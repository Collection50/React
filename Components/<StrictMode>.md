# `<StrictMode>`

- `<StrictMode>`를 사용하면 컴포넌트에서 발생하는 버그를 조기에 발견할 수 있습니다.

```jsx
<StrictMode>
  <App />
</StrictMode>
```

<br>

## 레퍼런스

#### `<StrictMode>`

- `StrictMode`를 사용하면 컴포넌트 트리의 개발 모드 동작 및 경고를 활성화합니다.

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

<br>

- `Strict Mode`는 다음과 같은 개발 전용 동작이 활성화됩니다.

<br>

1. 불완전한 렌더링으로 인한 버그를 찾기 위해 **리렌더링합니다.**
2. `Effects`의 초기화 함수가 누락되어 발생한 버그를 찾기 위해 `Effects`를 **1번 더 실행합니다.**
3. **사용하지 않는 API의 사용 여부를 확인합니다.**

<br>

### `Props`

- `StrictMode`는 `props`를 받지 않습니다.

<br>

### 주의사항

- `<StrictMode>`로 래핑된 트리 내부에서 `StrictMode`를 해제할 수 있는 방법은 없습니다.
- 따라서 `<StrictMode>` 하위의 모든 컴포넌트가 검증됩니다.

<br>

## 사용법

### 전체 앱에 대해 `Strict Mode` 사용

- `StrictMode`를 사용하면 전체 컴포넌트 트리에 대해 개발 모드 전용 검사를 수행합니다.
- 개발 초기에 컴포넌트의 버그를 발견할 수 있습니다.

<br>

- 전체 앱에 `StrictMode`를 사용하려면 루트 컴포넌트를 `<StrictMode>`로 래핑합니다.

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

<br>

- 새로 만든 앱의 경우 `StrictMode`로 래핑하는 것이 좋습니다.
- `createRoot`를 호출하는 프레임워크를 사용하는 경우 설명서를 참조하여 `StrictMode`를 활성화합니다.

<br>

- `StrictMode`는 배포 환경에서 안정적으로 재현하기 까다로운 버그를 찾는 데 유용합니다.
- `StrictMode`를 사용하면 버그를 사전에 방지할 수 있습니다.

<br>

### 일부에서만 `StrictMode` 적용하기

- 모든 트리에 `StrictMode`를 활성화할 수도 있습니다.

```jsx
import { StrictMode } from 'react';

function App() {
  return (
    <>
      <Header />
      <StrictMode>
        <main>
          <Sidebar />
          <Content />
        </main>
      </StrictMode>
      <Footer />
    </>
  );
}
```

<br>

- 위 예제는 `Header`, `Footer` 컴포넌트에 대해 `StrictMode`가 실행되지 않습니다.
- 그러나 `SideBar`, `Content`의 모든하위 컴포넌트에는 `StrictMode`가 적용됩니다.

<br>

### 2번의 렌더링을 통해 발견한 버그 수정

- **`React`는 모든 컴포넌트가 순수 함수라고 가정합니다.**
- 여러분이 작성하는 `React` 컴포넌트는 동일한 입력(`prop`, 상태, 컨텍스트)이 주어지면 항상 동일한 JSX를 반환합니다.

<br>

- 규칙을 위반하는 컴포넌트는 예측할 수 없으며 버그를 유발합니다.
- 오류를 찾기 위해 `StrictMode`는 개발 과정에서 순수 함수를 2번 호출합니다.
- 순수 함수의 종류는 아래와 같습니다.

<br>

1. 컴포넌트 (최상위 로직만 포함하므로 이벤트 핸들러 내부의 코드는 포함되지 않음)
2. `useState`, `set` 함수, `useMemo` 또는 `useReducer`에 전달하는 함수 등
3. `contructor`, `render`, `shouldComponentUpdate`와 같은 일부 클래스 메서드

<br>

- 순수 함수는 매번 동일한 결과를 생성하므로 2번 실행해도 동작이 변경되지 않습니다.
- 그러나 순수하지 않은 경우(예: 매개변수를 변경하는 경우) 2번 실행하면 오류가 발생합니다.
- `StrictMode`는 버그를 조기에 발견하고 수정하는 데 유용합니다.

<br>

- 아래는 `StrictMode`에서 2번의 렌더링이 버그를 조기에 발견하는 예시입니다.

<br>

- `StoryTray` 컴포넌트는 `stories` 배열을 가져와 마지막에 **Create Story** 항목을 하나 추가합니다.

```jsx
// index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

```jsx
// App.js
import { useState } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  { id: 0, label: "Ankit's Story" },
  { id: 1, label: "Taylor's Story" },
];

export default function App() {
  let [stories, setStories] = useState(initialStories);
  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <StoryTray stories={stories} />
    </div>
  );
}
```

```jsx
// StoryTray.js
export default function StoryTray({ stories }) {
  const items = stories;
  items.push({ id: 'create', label: 'Create Story' });
  return (
    <ul>
      {items.map((story) => (
        <li key={story.id}>{story.label}</li>
      ))}
    </ul>
  );
}
```

<br>

- 위 코드에 실수가 있지만 첫 렌더링은 올바르게 나타나기 때문에 놓치기 쉽습니다.

<br>

- `StoryTray` 컴포넌트가 여러 번 리렌더링하는 경우 실수가 더 눈에 띄게 됩니다.
- 예를 들어 `StoryTray`를 `hover`할 때마다 다른 배경색으로 리렌더링 해 보겠습니다.

```jsx
// index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

```jsx
// App.js
import { useState } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  { id: 0, label: "Ankit's Story" },
  { id: 1, label: "Taylor's Story" },
];

export default function App() {
  let [stories, setStories] = useState(initialStories);
  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <StoryTray stories={stories} />
    </div>
  );
}
```

```jsx
// StoryTray.js
import { useState } from 'react';

export default function StoryTray({ stories }) {
  const [isHover, setIsHover] = useState(false);
  const items = stories;
  items.push({ id: 'create', label: 'Create Story' });
  return (
    <ul
      onPointerEnter={() => setIsHover(true)}
      onPointerLeave={() => setIsHover(false)}
      style={{
        backgroundColor: isHover ? '#ddd' : '#fff',
      }}
    >
      {items.map((story) => (
        <li key={story.id}>{story.label}</li>
      ))}
    </ul>
  );
}
```

<br>

- `StoryTray` 컴포넌트 `hover`할 때마다 **Create Story**가 목록에 추가됩니다.
- 이 코드의 의도는 마지막에 1번 추가하는 것이었습니다.
- `StoryTray`는 `prop`에서 스토리 배열을 직접 수정합니다.
- `StoryTray`는 렌더링마다 **Create Story**를 `push`합니다.
- `StoryTray`는 순수 함수가 아니므로 여러 번 실행하면 다른 결과가 생성됩니다.

<br>

- 이 문제를 해결하려면 배열의 복사본을 만든 수정하면 됩니다.

```jsx
export default function StoryTray({ stories }) {
  const items = stories.slice(); // 배열 복사
  // ✅ 굿: 새 배열로 push
  items.push({ id: 'create', label: 'Create Story' });
}
```

<br>

- 이렇게 하면 `StoryTray` 함수가 순수해집니다.
- 배열의 새 복사본만 수정하고 외부 객체나 변수에 영향을 주지 않습니다.
- 버그는 해결되었지만 컴포넌트를 더 자주 리렌더링합니다.

<br>

- **원래 예제에서는 버그가 분명하지 않았습니다.**
- **이제 원래의(버그가 있는) 코드를 `<StrictMode>`로 래핑해 보겠습니다.**

<br>

```jsx
// index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```jsx
// App.js
import { useState } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  { id: 0, label: "Ankit's Story" },
  { id: 1, label: "Taylor's Story" },
];

export default function App() {
  let [stories, setStories] = useState(initialStories);
  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <StoryTray stories={stories} />
    </div>
  );
}
```

```jsx
// StoryTray.js
export default function StoryTray({ stories }) {
  const items = stories;
  items.push({ id: 'create', label: 'Create Story' });
  return (
    <ul>
      {items.map((story) => (
        <li key={story.id}>{story.label}</li>
      ))}
    </ul>
  );
}
```

<br>

- **`StrictMode`에서는 렌더링 함수를 2번 호출하므로 실수를 바로 확인할 수 있습니다.**
- 따라서 초기에 이러한 실수를 발견합니다.
- 컴포넌트를 `StrictMode`에서 렌더링하면 `hover`와 같이 향후 배포 환경에서 발생할 수 있는 버그도 수정할 수 있습니다.

```jsx
// index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```jsx
// App.js
import { useState } from 'react';
import StoryTray from './StoryTray.js';

let initialStories = [
  { id: 0, label: "Ankit's Story" },
  { id: 1, label: "Taylor's Story" },
];

export default function App() {
  let [stories, setStories] = useState(initialStories);
  return (
    <div
      style={{
        width: '100%',
        height: '100%',
        textAlign: 'center',
      }}
    >
      <StoryTray stories={stories} />
    </div>
  );
}
```

```jsx
// StoryTray.js
import { useState } from 'react';

export default function StoryTray({ stories }) {
  const [isHover, setIsHover] = useState(false);
  const items = stories.slice(); // Clone the array
  items.push({ id: 'create', label: 'Create Story' });
  return (
    <ul
      onPointerEnter={() => setIsHover(true)}
      onPointerLeave={() => setIsHover(false)}
      style={{
        backgroundColor: isHover ? '#ddd' : '#fff',
      }}
    >
      {items.map((story) => (
        <li key={story.id}>{story.label}</li>
      ))}
    </ul>
  );
}
```

<br>

- `StrictMode`가 없으면 리렌더링 전까지는 버그를 놓치기 쉽습니다.
- `StrictMode`를 사용하면 버그가 바로 나타납니다.

<br>

- `React DevTools`를 설치한 경우 2번째 렌더링 호출 중 `console.log` 호출이 약간 흐리게 표시됩니다.
- 이를 제어하는 설정(기본적으로 **off**)도 제공합니다.

<br>

### `Effect` 재실행에 의해 발견된 버그 수정하기

- `StrictMode`는 버그를 찾는 데 유용합니다.

<br>

- 모든 `Effects`에는 `setup` 함수나 초기화 함수가 있을 수 있습니다.
- 일반적으로 `React`는 컴포넌트가 마운트될 때 `setup`을 호출하고 언마운트 해제될 때 초기화 함수를 호출합니다.
- 종속성이 변경된 경우 `setup` 함수와 초기화 함수를 다시 호출합니다.

<br>

- **`StrictMode`가 켜져 있으면 `Effects`에 대해 `setup` + 초기화 사이클을 1번 더 실행합니다.**
- 수동으로 확인하기 어려운 미묘한 버그를 발견하는 데 유용합니다.

<br>

- `StrictMode`에서 `Effects`를 1번 더 실행하여 버그를 조기에 발견하는 예시입니다.

<br>

- 컴포넌트를 채팅에 연결하는 예시를 살펴보겠습니다.

```jsx
// index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

```jsx
// App.js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'general';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
  }, []);
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

```jsx
// chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // 서버에 실제로 연결할 수 있는 코드
  return {
    connect() {
      console.log(
        '✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...'
      );
      connections++;
      console.log('Active connections: ' + connections);
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
      connections--;
      console.log('Active connections: ' + connections);
    },
  };
}
```

<br>

- 위 코드에 문제가 있지만 명확하지 않을 수 있습니다.

<br>

- 문제를 더 명확하게 파악하기 위해 기능을 구현해 보겠습니다.
- 아래 예제는 `roomId`가 하드코딩되어 있지 않습니다.
- 사용자가 드롭다운에서 연결하려는 `roomId`를 선택할 수 있습니다.
- **채팅 열기**를 클릭한 다음 다른 채팅방을 하나씩 선택합니다.
- 활성화된 연결 수를 콘솔에서 확인합니다.

```jsx
// index.js
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

```jsx
// App.js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select value={roomId} onChange={(e) => setRoomId(e.target.value)}>
          <option value='general'>general</option>
          <option value='travel'>travel</option>
          <option value='music'>music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```jsx
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // 서버에 실제로 연결할 수 있는 코드
  return {
    connect() {
      console.log(
        '✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...'
      );
      connections++;
      console.log('Active connections: ' + connections);
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
      connections--;
      console.log('Active connections: ' + connections);
    },
  };
}
```

<br>

- 활성화된 연결 수가 증가되는 것을 확인합니다.
- 실제 앱에서는 성능 및 네트워크 문제가 발생할 수 있습니다.
- 문제는 **`Effects`에 초기화 함수가 없다는 것입니다.**

```jsx
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => connection.disconnect();
}, [roomId]);
```

<br>

- 이제 `Effects`가 자체적으로 **초기화**하고 이전의 연결을 파괴하므로 오류가 해결되었습니다.
- 더 많은 기능을 추가하기 전까지는 문제가 보이지 않는다는 점에 조심해야 합니다.

<br>

- **이전 예제에서는 버그가 명확하지 않았습니다.**
- **이제 <StrictMode>로 래핑해 보겠습니다.**

```jsx
// index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```jsx
// App.js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'general';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
  }, []);
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

```jsx
// chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // 서버에 실제로 연결할 수 있는 코드
  return {
    connect() {
      console.log(
        '✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...'
      );
      connections++;
      console.log('Active connections: ' + connections);
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
      connections--;
      console.log('Active connections: ' + connections);
    },
  };
}
```

<br>

- **`StrictMode`를 사용하면 문제가 있음을 즉시 알 수 있습니다.**
- `StrictMode`는 모든 `Effects`에 대해 `setup` + 초기화 주기를 1번 더 실행합니다.
- 위의 `Effect`에는 초기화 로직이 없으므로 추가 연결을 생성하지만 파괴하지는 않습니다.
- 이것은 초기화 함수가 누락되었다는 증거입니다.

```jsx
// index.js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './styles.css';

import App from './App';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

```jsx
// App.js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select value={roomId} onChange={(e) => setRoomId(e.target.value)}>
          <option value='general'>general</option>
          <option value='travel'>travel</option>
          <option value='music'>music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

```js
// chat.js
let connections = 0;

export function createConnection(serverUrl, roomId) {
  // 서버에 실제로 연결할 수 있는 코드
  return {
    connect() {
      console.log(
        '✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...'
      );
      connections++;
      console.log('Active connections: ' + connections);
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
      connections--;
      console.log('Active connections: ' + connections);
    },
  };
}
```

<br>

- 활성된 연결 수가 더 이상 증가하지 않는 것을 콘솔에서 확인할 수 있습니다.

<br>

- `StrictMode`가 없으면 `Effect`에 초기화가 필요하다는 사실을 놓치기 쉽습니다.
- 개발 단계에서 `Effect`에 대하여 `setup` → 초기화 → `seupt`을 실행하여 `StrictMode`에서 누락된 초기화 로직을 더 확실히 알 수 있습니다.

<br>

### `StrictMode`에서 활성화된 중단 경고 고치기

- `React`는 `<StrictMode>` 트리 하위의 컴포넌트가 아래 API 중 하나를 사용하는 경우 경고를 표시합니다.

<br>

- `findDOMNode`.
- **`UNSAFE_componentWillMount`** 와 같은 `UNSAFE_` 클래스 생명주기 메서드.
- 레거시 컨텍스트(`childContextTypes`, `contextTypes` 및 `getChildContext`).
- 레거시 문자열 참조(`this.refs`).

<br>

- 이러한 API는 주로 레거시 버전인 클래스 컴포넌트에서 사용되므로 최신 앱에는 거의 사용되지 않습니다.
