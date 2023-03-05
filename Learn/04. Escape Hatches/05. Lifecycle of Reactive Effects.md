# Lifecycle of Reactive Effects

- `Effect`는 컴포넌트와 생명 주기가 다르니다.
- 컴포넌트는 마운트, 언마운트, 업데이트할 수 있습니다.
- `Effect`는 2가지 일만 수행할 수 있습니다.
  1. 동기화 시작
  2. 동기화 중지
- 변하는 `props`와 상태의 변화에 따라 `Effect`가 달라지는 경우, 이 사이클은 여러번 발생할 수 있습니다.
- `React`는 `Effect`의 종속성을 올바르게 지정했는지 확인하는 린터 규칙을 제공합니다.
- 이렇게 하면 `Effect`가 최신 상태의 `props` 및 상태와 동기화됩니다.

## 배우게 될 것

- `Effect`의 생명 주기와 컴포넌트의 생명 주기의 차이점
- 각각의 개별적 `Effect`를 고립적으로 생각하는 방법
- `Effect`가 다시 동기화되는 시점과 이유
- `Effect`의 종속성이 결정되는 방법
- 값이 반응적으로 변한다는 것의 의미
- 빈 종속성 배열이 의미하는 것
- `React`가 린터를 사용하여 종속성이 올바른지 확인하는 방법
- 린터를 사용하지 않을 때 수행되는 것

## `Effect`의 생명 주기

- 모든 `React` 컴포넌트는 같은 생명 주기를 갖고 있습니다.
- 컴포넌트가 화면에 추가될 때 **마운트**됩니다.
- 컴포넌트는 새 `props`나 상태를 전달받을 때 **업데이트**됩니다.
  - 이는 일반적으로 인터랙션에 대한 응답으로 발생합니다.
- 컴포넌트는 화면에서 사라질 때 **언마운트**됩니다.
  <br><br>
- 컴포넌트에 대해 생각하는 좋은 방법이지만, `Effect`에 대해서는 효과적이지 않습니다.
- 대신, 각 `Effect`를 컴포넌트의 생명 주기와 독립적으로 생각해볼까요?
- `Effect`는 상태와 `props`를 **외부 시스템과 동기화** 하는 방법을 설명합니다.
- 코드가 변경되면 이 동기화가 자주 수행되어야 합니다.
  <br><br>
- 이 점을 설명하기 위해 컴포넌트를 채팅 서버에 연결하는 아래 `Effect`를 살펴봅시다.

```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

<br>

- `Effect`의 바디는 **동기화 시작** 방법을 지정합니다.

```js
// ...
const connection = createConnection(serverUrl, roomId);
connection.connect();
return () => {
  connection.disconnect();
};
// ...
```

<br>

- `Effect`가 반환한는 초기화 함수는 **동기화 중지** 방법을 지정합니다.

```js
// ...
const connection = createConnection(serverUrl, roomId);
connection.connect();
return () => {
  connection.disconnect();
};
// ...
```

<br>

- 직관적으로 `React`는 컴포넌트가 마운트될 때 **동기화를 시작**하고, 컴포넌트가 언마운트될 때 **동기화를 중지**한다고 생각할 수 있습니다.
- 하지만 이게 전부는 아닙니다.
- 때떄로 컴포넌트가 마운트된 상태에서 여러 번 **동기화를 시작하고 중지**해야 합니다.
  <br><br>
- 왜 필요하고, 언제 발생하는지, 어떻게 이 동작을 통제할 수 있는지 알아봅시다.

## 주의

- 일부 `Effect`는 초기화 함수를 반환하지 않습니다.
- 반환하지 않으면 `React`는 아무 동작도 수행하지 않는 빈 초기화 함수를 반환한 것처럼 동작합니다.

## 동기화가 2번 이상 발생해야 하는 이유

- `ChatRoom` 컴포넌트는 사용자가 드롭 다운에서 선택한 `roomId` `prop`을 받는다고 가정해봅시다.
- 처음에 사용자가 `general` 채팅방을 `roomId`로 설정한다고 해보겠습니다.
- 앱에 `genral` 채팅방이 표시됩니다.

```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId /* 'general' */ }) {
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

<br>

- UI가 화면에 표시된 이후, `React`는 **동기화를 시작**하기 위해 `Effect`를 실행합니다.
- 이는 `general` 채팅방에 연결됩니다.

```js
function ChatRoom({ roomId /* 'general' */ }) {
  useEffect(() => {
    // 'general' 채팅방에 연결
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      // 'general' 채팅방에서 연결 해제
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

<br>

- 사용자는 드롭다운 메뉴에서 다른 채팅방(`travel`)을 선택했습니다.
- 먼저 `React`가 UI를 업데이트합니다.

```js
function ChatRoom({ roomId /* 'travel' */ }) {
  // ...
  return <h1>Welcome to the {roomId} room!</h1>;
}
```

<br>

- 다음에 어떤 일이 발생할까요?
- 유저는 UI에서 `travel`이 선택된 채팅방을 확인합니다.
- 하지만 마지막으로 실행된 `Effect`는 여전히 `general` 채팅방에 연결됩니다.
- `roomId` `prop`이 변경되었으므로, 그 당시에 `Effect`와 연결된 채팅방은 UI와 일치하지 않습니다.
  <br><br>
- 이 부분에서 `React`는 2가지 작업을 수행합니다.

1. 이전 `roomId`와 동기화 중지 (`general`)
2. 새 `roomId`와 동기화 시작 (`travel`)

<br>

- 운이 좋게도 이미 `React`는 무엇을 해야 할지 알고 있습니다.
- `Effect`의 바디는 동기화 시작 방법을 지정했고, 초기화 함수는 동기화 중지 방법을 지정합니다.
- `React`가 지금 해야 할 일은 올바른 순서로 올바른 `props`와 상태를 호출하는 것입니다.
- 동작 원리를 자세히 살펴봅시다.

## React가 Effect를 다시 동기화하는 방법

- `ChatRoom` 컴포넌트는 해당 `roomId` `prop`에 대한 새 값을 받았습니다.
- `'general'`에서 `'travel'`로 변경되었습니다.
- 다른 채팅방에 연결하려면 `Effect`를 다시 동기화해야 합니다.
  <br><br>
- **동기화를 중지**하려면 `React`는 `'general'` 채팅방에 연결한 후 `Effect`가 반환한 초기화 함수를 호출합니다.
- `roomId`가 `'general'`이었기 때문에 초기화 함수는 `'general'` 채팅방과 연결을 끊습니다.

```js
function ChatRoom({ roomId /* 'general' */ }) {
  useEffect(() => {
    // 'general' 채팅방에 연결
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      // 'general' 채팅방과 연결 해제
      connection.disconnect();
    };
  });
}
```

<br>

- 이후 `React`는 렌더링 도중에 제공했던 `Effect`를 실행합니다.
- 이 시점에서 `roomId`가 `'travel'`이므로 `'travel'` 채팅방에 동기화하기 시작합니다.

```js
function ChatRoom({ roomId /* 'travel' */ }) {
  useEffect(() => {
    // 'travel' 채팅방에 연결
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // ...
  });
}
```

<br>

- 이제 사용자가 UI에서 선택한 것과 동일한 채팅방에 연결됩니다.
  <br><br>
- 컴포넌트가 다른 `roomId`로 리렌더링될 때마다 `Effect`는 다시 동기화됩니다.
- 예를 들어 `roomId`가 `'travel'`에서 `'music'`으로 바뀌었다고 가정합시다.
- `React`는 초기화 함수를 호출하여 `Effect` 동기화를 중지합니다. (`'travel'` 동기화 중지)
- 그런 다음 새로운 `roomId` `prop`을 활용하여 다시 동기화를 시작합니다. (`'music'` 동기화 시작)
  <br><br>
- 마지막으로 유저가 다른 화면으로 이동할 때 `ChatRoom` 컴포넌트는 언마운트됩니다.
- 연결을 유지할 필요가 없기 때문입니다.
- `React`는 마지막으로 `Effect` 동기화를 중지하고 `'music'` 채팅방의 연결을 끊습니다.

## Effect의 관점에서 생각하기

- `ChatRoom` 컴포넌트 관점에서 일어난 일들을 요약해보겠습니다.

1. `roomId`가 `'general'`로 설정되며 `ChatRoom` 컴포넌트가 마운트됩니다.
2. `roomId`가 `'travel'`로 설정되며 `ChatRoom` 컴포넌트가 업데이트됩니다.
3. `roomId`가 `'music'`로 설정되며 `ChatRoom` 컴포넌트가 업데이트됩니다.
4. `ChatRoom` 컴포넌트는 언마운트됩니다.

<br>

- 컴포넌트의 생명 주기에 각 요소에서 `Effect`는 다른 일을 수행했습니다.

1. `Effect`는 `'general'` 채팅방과 연결됩니다.
2. `Effect`는 `'general'` 채팅방과 연결 해제되고, `'travel'` 채팅방과 연결되었습니다.
3. `Effect`는 `'travel'` 채팅방과 연결 해제되고, `'music'` 채팅방과 연결되었습니다.
4. `Effect`는 `'travel'` 채팅방과 연결 해제되었습니다.

<br>

- 이제 `Effect` 자체의 관점에서 무슨 일이 일어났는지 살펴보겠습니다.

```js
useEffect(() => {
  // Effect는 roomId가 특정하는 채팅방과 연결됩니다.
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => {
    // ... 연결이 끊기기 전까지
    connection.disconnect();
  };
}, [roomId]);
```

<br>

- 이 코드 구조는 시간 순서에 따라 어떤 일이 일어났는지 확인할 수 있습니다.

1. `Effect`는 `'general'` 채팅방에 연결되었습니다. (연결이 끊기기 전까지)
2. `Effect`는 `'travel'` 채팅방에 연결되었습니다. (연결이 끊기기 전까지)
3. `Effect`는 `'music'` 채팅방에 연결되었습니다. (연결이 끊기기 전까지)

<br>

- 이전에는 컴포넌트 관점에서 생각했습니다.
- 컴포넌트 관점에서 바라볼 때 `Effect`를 **렌더링 이후**나 **마운트 해제 전**과 같이 특정 시간에 발생하는 **콜백** 혹은 **생명 주기 이벤트**로 생각하기 쉽습니다.
- 이러한 사고방식은 혼란스러움을 가중시키므로 피하는 것이 좋습니다.
  <br><br>
- **항상 한 번에 하나의 시작/정지 사이클에 집중해야 합니다.**
- **컴포넌트가 마운트되든, 업데이트되든, 언마운트되든 상관 없습니다.**
- **동기화를 시작하는 방법과 중지하는 방법만 알려주면 됩니다.**
- **이를 잘 알려주면 `Effect`가 필요한 횟수만큼 시작 및 중지하는데 효율적으로 동작할 것입니다.**
  <br><br>
- 이는 `JSX`를 생성하는 렌더링 로직을 작성할 때 컴포넌트가 마운트 혹은 업데이트 여부를 생각하지 않아도 됩니다.
- 화면에 무엇이 있어야 하는지 설명하면, `React`는 **나머지를 알아냅니다.**

## Effect가 다시 동기화될 수 있음을 확인하는 방법

- `ChatRoom` 컴포넌트를 마운트하려면 `Open chat` 버튼을 클릭합니다.

```js
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
        Choose the chat room:{` `}
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
export function createConnection(serverUrl, roomId) {
  // 서버에 실제로 연결할 수 있는 코드
  return {
    connect() {
      console.log(
        '✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...'
      );
    },

    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    },
  };
}
```

<br>

- 컴포넌트가 처음으로 마운트될때 아래 로그를 확인하게 될 겁니다.

1. ✅ Connecting to "general" room at `https://localhost:1234...` (개발환경 only)
2. ❌ Disconnected from "general" room at `https://localhost:1234...` (개발환경 only)
3. ✅ Connecting to "general" room at `https://localhost:1234...`

<br>

- 첫 2개 로그는 개발 환경에서만 확인할 수 있습니다.
- 개발환경에서 `React`는 각 컴포넌트를 1번씩 리마운트합니다.
- 다른 말로 `React`는 개발 환경에서 즉시 강제로 리마운트하도록 강제하여 `Effect` 재동기화할 수 있는지 확인합니다.
- 잠금 장치가 작동하는지 확인하기 위해 문을 열고 닫을 수 있는 방법을 알려줄 것입니다.
- `React`는 개발 과정에서 한 번 더 `Effect`를 시작하고 중지하여 초기화 함수를 잘 구현했는지 확인합니다.
  <br><br>
- 실제로 `Effect`가 다시 동기화되는 주요한 이유는 `Effect`가 사용하는 데이터가 변경되었기 때문입니다.
- 위에서 선택한 채팅방을 변경해봅시다.
- `roomId`가 변경될 때 `Effect`가 다시 동기화되는 방법을 확인하세요.
  <br><br>
- 그러나 재동기화가 필요한 특이한 경우도 많이 존재합니다.
- 예를 들어 채팅방이 열려있는 동안 `serverUrl`을 변경해봅시다.
- 코드 변경에 따라 `Effect`가 다시 동기화되는 방법을 확인해보세요.
- `React`는 재동기화의 이점을 활용하는 기능을 추가할 수도 있습니다.

## Effect의 재동기화가 필요하다는 것을 어떻게 알까?

- `roomId`가 변경된 후 `Effect`를 다시 동기화해야 한다는 것을 어떻게 확인할 수 있었을까요?
- `Effect`의 코드가 종속성 리스트에 포함되어, `roomId`에 종속된다고 **말했기** 때문입니다.

```js
// roomId는 변경될 것입니다.
function ChatRoom({ roomId }) {
  useEffect(() => {
    // 이 Effect는 roomId를 참조합니다.
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // React에게 이 Effect는 roomId에 종속된다고 말합니다.
  // ...
}
```

<br>

- 동작 원리는 아래와 같습니다.

1. `roomId`는 `prop`이기에 시간이 지남에 따라 변할 수 있습니다.
2. `Effect`가 `roomId`를 참조한다는 것을 알고 있습니다.
3. 이것이 `roomId`를 `Effect`의 종속성으로 지정한 이유입니다.

<br>

- 컴포넌트가 리렌더링될 때마다, `React`는 전달된 종속성 배열을 확인ㅇ하빈다.
- 종속성 배열의 값이 이전 렌더링의 동일한 위치에 있는 값과 다른 경우, `Effect`를 다시 동기화합니다.
- 예를 들어 첫 렌더링에서 `['general']`을 전달하고, 다음 렌더링에서 `['travel']`을 전달한 경우, `React`는 `'general'`과 `'travel'`을 비교합니다.
- 이는 `Object.is`에 의해 비교되었을 때 다른 값이므로, `React`는 `Effect`를 다시 동기화합니다.
- 반면에 컴포넌트가 리렌더링되었지만 `roomId`가 변경되지 않은 경우, `Effect`는 동일한 채팅방에 연결된 상태로 유지됩니다.

## 각 효과는 개별적인 동기화 프로세스를 표현합니다.

- 아래 로직은 이미 작성된 `Effect`와 동시에 실행되어야 하므로, 관련 없는 로직을 `Effect`에 추가하면 안 됩니다.
- 에를 들어 유저가 채팅방을 방문할 때 분석 이벤트를 전송해보겠습니다.
- 이미 `roomId`에 따라 `Effect`가 달라지므로, 분석을 호출하는 함수를 바로 추가하고 싶을 수도 있습니다.

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

<br>

- 그러나 `Effect`에 연결을 다시 설정해야 하는 종속성을 추가해본다면요?
- 이 `Effect`가 다시 동기화되는 경우, 의도하지 않은 채팅방에 대한 `logVisit(roomId)`도 호출됩니다.
- 방문 기록과 연결은 별개의 **프로세스**입니다.
- 이것이 2개의 `Effect`로 작성되어야 하는 이유입니다.

```js
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId);
  }, [roomId]);

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    // ...
  }, [roomId]);
  // ...
}
```

<br>

- **코드의 각 `Effect`는 독립적인 동기화 프로세스를 표현해야 합니다.**
  <br><br>
- 위 예제에서 `Effect` 1개를 삭제하는 것은 다른 `Effect` 로직에 영향을 미치지 않습니다.
- 이는 각 `Effect`가 서로 다른 것을 동기화한다는 좋은 의미이기에 별개로 구분하는 것입니다.
- 반면 응집력 있는 로직의 일부분을 `Effect`로 분리하면 코드가 더 **깔끔해 보이지만** 유지하기는 어려워집니다.
- 코드가 더 깔끔해보이는 것보다 프로세스가 같은지, 분리되어있는지 생각하는 것이 더 중요합니다.

## 반응형 값에 **반응**하는 효과

- `Effect`는 2개 변수(`serverUrl`, `roomId`)를 참조하지만 `roomId`만 종속성으로 지정했습니다.

```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

- 왜 `severUrl`은 종속성이 될 필요가 없을까요?
  <br><br>
- `serverUrl`이 리렌더링으로 인해 변경되지 않기 때문입니다.
- 컴포넌트가 몇 번이고 리렌더링되어도 `props`와 상태는 변하지 않습니다.
- `serverUrl`은 변하지 않기에 종속성으로 지정할 필요가 없습니다.
- 종속성은 변하는 무언가에만 적용되는 것입니다.
  <br><br>
- 반면 `roomId`는 리렌더링에 의해 변합니다.
- `props`, 상태, 컴포넌트에 선언된 다른 변수 등은 렌더링 도중에 계산되고 `React` 데이터 흐름에 참여하기 때문에 **반응적입니다.**
  <br><br>
- `serverUrl`이 상태 변수라면 반응적입니다.
- 반응적 값들은 종속성에 포함되어야 합니다.

```js
// props는 변할 수 있습니다.
function ChatRoom({ roomId }) {
  // 상태는 변할 수 있습니다.
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    // // Effect는 상태와 props를 참조합니다.
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]);
  // 이 Effect는 상태와 props에 의존합니다.
  // ...
}
```

<br>

- `serverUrl`을 종속성으로 포함하면 `Effect`가 변경된 후 다시 동기화는 것을 보장할 수 있습니다.
  <br><br>
- 선택한 대화방을 변경하거나 서버 URL을 편집해보세요.

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={(e) => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
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
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

<br>

- `roomId`, `serverUrl`과 같은 반응형 값을 변경할 때마다, `Effect`는 채팅 서버에 다시 연결됩니다.

## 빈 의존성을 가진 Effect의 의미는 무엇일까요?

- `roomId`, `serverUrl`을 컴포넌트 외부로 이동하면 어떻게 될까요?

```js
const serverUrl = 'https://localhost:1234';
const roomId = 'general';

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ✅ 모든 종속성은 위에서 선언되었습니다.
  }, []);
  // ...
}
```

<br>

- `Effect`의 코드는 반응형 값을 사용하지 못하므로 종속성은 비어있습니다.
  <br><br>
- 컴포넌트 관점에서 생각해보자면, 종속성 배열이 비어있다는 것은(`[]`), `Effect`가 마운트될 때만 채팅방에 연결되고, 언마운트될 때 연결이 끊어진다는 것을 의미합니다.

```js
// App.js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = 'general';

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom />}
    </>
  );
}
```

```js
// chat.js
export function createConnection(serverUrl, roomId) {
  // 서버에 실제로 연결할 수 있는 코드
  return {
    connect() {
      console.log(
        '✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...'
      );
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    },
  };
}
```

<br>

- 하지만 `Effect`의 관점에서 보면 마운트나 언마운트를 생각할 필요가 없습니다.
- 중요한 것은 동기화의 시작과 정지를 위해 `Effect`가 수행하는 작업을 지정했다는 것입니다.
- 이제 반응적인 의존성을 갖고 있지 않습니다.
- 그러나 `roomId`, `serverUrl`을 변경하고 싶어도 `Effect` 코드는 변하지 않습니다.
- 종속성에만 추가하면 됩니다.

## 컴포넌트 바디에 선언된 모든 변수는 반응형입니다.

- `props`와 상태만 반응형 값이 아닙니다.
- 이 값을 사용하여 계산한 값도 반응형입니다.
- `props`와 상태가 변한다면 컴포넌트는 리렌더링 하고, 리렌더링에 의해 계산된 값들 또한 변경됩니다.
- 따라서 `Effect`에 사용되는 컴포넌트 바디의 모든 변수도 종속성 목록에 있어야 합니다.
  <br><br>
- 유저가 드롭다운에서 채팅 서버를 선택할 수 있지만, 설정에서 기본 서버를 선택할 수 있다고 가정해봅시다.
- 컨텍스트에 이미 설정 상태를 추가하였기에, 컨텍스트에서 `settings`를 읽을 수 있다고 가정합니다.
- 이제 `props`으로 선택된 서버와, 기본 서버를 기준으로 `serverUrl`을 계산합니다.

```js
// roomId는 반응형입니다.
function ChatRoom({ roomId, selectedServerUrl }) {
  // settings는 반응형입니다.
  const settings = useContext(SettingsContext);
  // serverUrl는 반응형입니다.
  const serverUrl = selectedServerUrl ?? settings.defaultServerUrl;
  useEffect(() => {
    // Effect는 props와 상태를 참조합니다.
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // 상태나 props이 변경된다면 재동기화가 필요합니다.
  }, [roomId, serverUrl]);
  // ...
}
```

<br>

- 이 예에서 `serverUrl`은 `prop`이나 상태 변수가 아닙니다.
- 렌더링 중에 계산하는 정규 변수입니다.
- 하지만 렌더링 중에 계산되기 때문에 리렌더링에 의해 변경될 수 있습니다.
- `serverUrl`이 반응형인 이유입니다.
  <br><br>
- **컴포넌트 내부의 모든 값은 반응형입니다.**
- **모든 반응형 값은 리렌더링 시 변경될 수 있으므로, 반응형 값을 `Effect`의 종속성으로 포함해야 합니다.**
  <br><br>
- 즉 `Effect`는 컴포넌트 바디의 모든 값에 **반응**합니다.

## Deep Dive - 전역 변수 or 변경 가능한 변수가 종속 상태일 수 있을까?

- 전역 변수와 변경 가능한 변수는 반응형이지 않습니다.
  <br><br>
- **`location.pathname`과 같이 변경 가능한 변수는 종속 상태일 수 없다.**
- 변경 가능하기에 `React` 렌더링 데이터 흐름 외부에서 언제나 변경될 수 있다.
- 전역 변수, 변경 가능한 변수를 변경해도 컴포넌트는 다시 렌더링되지 않습니다.
- 심지어 종속성을 지정한다고 해도, `React`는 `Effect`가 변경될 때 재동기화를 해야 하는지 **알 수 없습니다.**
- 또한 렌더링 중에 변경 가능한 데이터를 참조하면, 렌더링의 순수성을 해치기 때문에 `React`의 규칙이 깨집니다.
- 대신 `SyncExternalStore`를 사용하여 외부의 변경 가능한 값을 읽고 구독해야 합니다.
  <br><br>
- `ref.current`와 같은 변수값이나 그 값에서 참조한 것도 종속 상태일 수 없습니다.
- `useRef`가 반환한 `ref` 객체는 종속될 수 있지만, `current` 프로퍼티는 의도적으로 변경 가능합니다.
- 이는 **리렌더링을 트리거하지 않고 무언가를 트래킹**할 수 있습니다.
- 하지만 변경을 해도 리렌더링이 트리거되지 않으므로 반응형 값이 아니며, `current`가 변경될 때 `Effect`를 재실행할 수 없습니다.
  <br><br>
- 아래 내용에서 알 수 있듯, 린터는 이러한 문제를 자동으로 확인합니다.

## React는 모든 반응형 값을 종속성으로 지정했는지 확인합니다.

- 린터가 `React`에 대해 구성된 경우, `Effect` 코드에서 사용하는 모든 반응형 값이 종속성 배열에 지정되었는지 확인합니다.
- 예를 들어 `roomId`, `serverUrl`이 모두 반응형 값이므로 아래 예제는 린터 오류입니다.

```js
// App.js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  // roomId은 반응형 값입니다.
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl은 반응적 값입니다.

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // <-- 이 부분이 문제입니다.

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={(e) => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
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
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

```js
// chat.js
export function createConnection(serverUrl, roomId) {
  // 서버에 실제로 연결할 수 있는 코드
  return {
    connect() {
      console.log(
        '✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...'
      );
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    },
  };
}
```

<br>

- `React` 오류라고 보이겠지만, `React`는 코드의 버그를 가리키고 있습니다.
- `roomId`와 `serverUrl` 모두 변경될 수 있지만, 변경될 때 `Effect`를 다시 동기화하지 않고 있습니다.
- 따라서 유저가 UI에서 다른 값을 선택하더라도, 초기 `roomId`, `serverUrl`에 연결된 상태로 유지됩니다.
  <br><br>
- 버그를 수정하기 위해 린터의 제안에 따라 `roomId`, `serverUrl`을 `Effect`의 종속성에 지정해야 합니다.

```js
function ChatRoom({ roomId, serverUrl }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // serverUrl는 반응적 값입니다.
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]); // ✅ 모든 종속성이 지정되었습니다.
  // ...
}
```

<br>

- 린터 오류가 없어졌는지 확인하고, 채팅방이 제때 연결되는지 확인합니다.

## 주의

- `React`는 어떤 값이 컴포넌트 내부에 선언되어도 변경되지 않는다는 것을 알고 있습니다.
- 예를 들어 `useState`에서 반환된 `set` 함수와 `useRef`에서 반환된 `ref`는 안정적입니다.
- 따라서 리렌더링할 때 변경되지 않도록 보장됩니다.
- 안정적인 값들은 반응형이 아니기에, 종속성 배열에서 빠뜨려도 린터가 오류를 발생하지 않습니다.
- 더하여 안정적인 값을 종속성 배열에 추가하는 것도 아무런 오류가 발생하지 않습니다.

## 다시 동기화되지 않을 때 수행할 작업

- 이전 예제에서는 `roomId`, `serverUrl`을 종속성으로 지정하여 린터 오류를 수정했습니다.
  <br><br>
- 그러나 이러한 값이 반응형 값이 아니라는 것을 린터에 **증명**할 수 있습니다.
- 즉, 리렌더링의 결과로 변경되는 것이 **아니라는** 것입니다.
- 예를 들어 `roomId`, `severUrl`가 항상 동일한 값을 갖는 경우 컴포넌트 외부로 이동할 수 있습니다.
- 더이상 종속성 배열에 있을 이유가 없습니다.

```js
// serverUrl은 반응형 값이 아닙니다.
const serverUrl = 'https://localhost:1234';
// roomId은 반응형 값이 아닙니다.
const roomId = 'general';

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ 모든 종속성이 지정되었습니다.
  // ...
}
```

<br>

- 또한 `roomId`와 `serverUrl`을 `Effect` 내부로 옮길수도 있습니다.
- 렌더링 도중 계산되지 않으므로 반응형 값이 아니기 때문입니다.

```js
function ChatRoom() {
  useEffect(() => {
    // serverUrl은 반응형 값이 아닙니다.
    const serverUrl = 'https://localhost:1234';
    // roomId은 반응형 값이 아닙니다.
    const roomId = 'general';
    const connection = createConnection(serverUrl, roomId);

    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []); // ✅ 모든 종속성이 지정되었습니다.
  // ...
}
```

<br>

- `Effect`는 반응형 코드 블록입니다.
- 내부에서 참조한 값이 변경되면 다시 동기화됩니다.
- 인터랙션 당 1번 실행되는 이벤트 핸들러와 달리, `Effect`는 동기화가 필요할 때마다 실행됩니다.
  <br><br>
- 종속성을 **선택**할 수 없습니다.
- 종속성에는 `Effect`에서 참조한 모든 반응형 값이 포함되어야 합니다.
- 린터가 이를 강제합니다.
- 때로는 무한 루프 문제가 발생하고 `Effect`가 너무 자주 동기화될 수 있습니다.
- 억지로 해결하려고 해선 안 됩니다.
- 아래 방법을 시도하세요.
  <br><br>

1. **`Effect`가 독립적인 동기화 프로세스를 나타내는지 확인합니다.**
   - `Effect`가 동기화되지 않으면 **필요하지 않을** 수 있습니다.
   - 여러 개의 독립적인 값을 동기화하는 경우 **분할합니다.**
2. `props`나 상태의 최신 값과 **무관**하게 `Effect`를 다시 동기화하려면, `Effect`를 **반응적인**부분과 **반응적이지 않은**부분으로 나눌 수 있습니다.
   - (반응적 부분: `Effect`에서 유지할 부분)
   - (비반응적 부분: 이벤트 핸들러를 추출하는 부분)
3. **객체 및 함수를 종속성에 사용하지 않습니다.**
   - 렌덜이하는 동안 객체와 함수를 만든 다음, `Effect`를 참조하는 경우 각 렌더링마다 결과가 달라집니다.
   - 이렇게 하면 `Effect`는 매번 동기화됩니다.

## 함정

- 린터는 훌륭한 친구이지만, 만능은 아닙니다.
- 린터는 종속성이 잘못된 경우에만 올바르게 동작합니다.
- 린터는 오류를 해결하는 최선책을 알지 못합니다.
- 린터는 종속성을 제안하지만, 종속성을 추가했을 때 루프가 발생한다고 해서 린터를 무시해야 하는 것은 아닙니다.
- 즉, `Effect` 내외부 코드를 변경하여 해당 값이 반응적이지 않고, 종속성이 필요 없도록 해야 합니다.
  <br><br>
- 기존 코드베이스가 있는 경우, 다음과 같이 린터를 억제하는 효과가 있을 수 있습니다.

```js
useEffect(() => {
  // ...
  // 🔴 아래와 같이 린터를 제한하는 것을 피해야 합니다.
}, []);
```

## 요약

- 컴포넌트는 마운트, 업데이트, 언마운트할 수 있습니다.
- 각 `Effect`는 컴포넌트와 구분되는 생명 주기가 존재합니다.
- 각 `Effect`는 **시작/중지**할 수 있는 별도의 동기화 프로세스를 설명합니다.
- `Effect`를 쓰고 참조할 때는 컴포넌트의 관점보다, 각 `Effect`의 관점에서 생각해야 합니다.
- 컴포넌트 바디에서 선언된 값은 **반응형**입니다.
- 반응형 값은 변하기 때문에 `Effect`를 다시 동기화해야 합니다.
- 린터는 `Effect` 내부에서 사용되는 모든 반응형 값이 종속성에 지정되었는지 확인합니다.
- 린터에 의해 확인되는 모든 에러는 올바른 오류입니다.
  - 규칙을 어기지 않는 코드를 고치는 방법은 항상 존재합니다.
