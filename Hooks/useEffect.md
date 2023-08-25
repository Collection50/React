# useEffect

- `useEffect`는 컴포넌트를 외부 시스템과 동기화하는 `React` `hook`입니다.

```jsx
useEffect(setup, dependencies?);
```

## 레퍼런스

### `useEffect(setup, dependencies?)`

- 컴포넌트의 최상단에서 `useEffect`를 호출해 사용합니다.

```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```

### 매개변수

- `setup`
  - `useEffect`의 로직이 포함된 함수입니다.
  - 선택적으로 초기화 함수를 반환합니다.
  - 컴포넌트가 DOM에 **추가될 때** 실행됩니다.
  - 종속성 변경으로 인해 리렌더링될 때마다 초기화 함수(존재하는 경우)를 실행한 후 변경된 종속성을 활용하여 `setup` 함수를 실행합니다.
  - 컴포넌트가 DOM에서 제거된 후 초기화 함수를 실행합니다.
- `dependencies`(**optional**)
  - `setup` 코드에서 참조하는 모든 반응형 값의 목록입니다.
  - 반응형 값에는 `prop`, state, 컴포넌트에서 선언된 모든 변수와 함수가 포함됩니다.
  - 린터 사용 시 모든 반응형 값이 종속성에 올바르게 지정되었는지 확인합니다.
  - 종속성 목록에는 일정한 수의 항목이 있어야 하며 `[dep1, dep2, dep3]`와 같이 인라인으로 작성해야 합니다.
  - `Object.is` 메서드를 사용하여 비교합니다.
  - 종속성 배열이 없다면 컴포넌트를 다시 렌더링할 때마다 `useEffect`가 다시 실행됩니다.

<br>

### 반환값

- `useEffect`는 `undefined`를 반환합니다.

<br>

### 주의사항

- `useEffect`는 `hook`이므로 **컴포넌트 최상단** 혹은 커스텀 `hook`에서만 호출합니다.
  - 반복문 or 조건문 내부에서는 호출할 수 없습니다.
  - 필요한 경우 새 컴포넌트를 추출하고 `useEffect`를 내부로 옮깁니다.
- **외부 시스템과 동기화하는 것이 아니라면** `useEffect`가 필요하지 않을 겁니다.
- `Strict Mode`를 사용하면 초기 `setup` 전에 **`setup` + 초기화 사이클을 한 번 더 실행**합니다. (개발 모드에서만)
  - 초기화 로직이 `setup` 로직을 "미러링"하고 `setup`이 수행 중인 모든 작업을 중지하거나 취소하는지 확인하기 위한 스트레스 테스트입니다.
  - 문제가 발생하면 초기화 함수를 구현합니다.
- 종속성이 컴포넌트 내부에 정의된 객체 or 함수인 경우 **`useEffect`가 필요 이상으로 자주 다시 실행될 위험**이 있습니다.
  - 문제를 해결하려면 불필요한 객체 및 함수 종속성을 제거합니다.
  - 상태 업데이트와 비반응형 로직을 `useEffect` 외부로 추출합니다.
- 클릭과 같은 상호작용으로 인해 `useEffect`가 발생한 것이 아니라면 **`useEffect` 실행 전에 브라우저가 업데이트된 화면을 먼저 `paint`합니다.**
  - `useEffect`가 시각적인 작업(예: 툴팁 위치 지정)을 하고 있는데 지연이 눈에 띄는 경우(깜빡임 등) `useEffect`를 `useLayoutEffect`로 변경합니다.
- 클릭과 같은 상호작용으로 인해 `useEffect`가 실행되더라도 브라우저는 `useEffect` 내부의 상태 업데이트를 처리하기 전에 화면을 다시 `paint`할 수 있습니다.
  - 일반적으로 이는 사용자가 원하는 것입니다.
  - 그러나 브라우저가 화면을 다시 `paint` 못하도록 차단하는 경우 `useEffect`를 `useLayoutEffect`로 변경합니다.
- **`useEffect`는 `CSR`에서만 실행됩니다.**
  - `SSR`에서는 실행되지 않습니다.

<br>

## 사용법

## 외부 시스템에 연결

- 일부 컴포넌트는 페이지에 표시되는 동안 네트워크, 일부 브라우저 API, 타사 라이브러리에 연결된 상태를 유지해야 합니다.
- 이러한 시스템은 `React`에 의해 제어되지 않으므로 외부 시스템입니다.

<br>

- 컴포넌트를 외부 시스템에 연결하려면 `useEffect`를 사용합니다.

<br>

```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```

- `useEffect`에 아래 2개 인수를 전달합니다.

1. 시스템 연결 준비 로직이 포함된 **`setup`** 함수입니다.
   - 해당 시스템과의 연결을 끊는 로직이 포함된 **초기화** 함수를 반환해야 합니다.
2. `setup` 함수에서 사용되는 모든 값을 포함한 종속성 배열입니다.

<br>

- **React는 필요할 때마다 `setup` 및 초기화 함수를 호출합니다.**

<br>

1.  컴포넌트가 마운트될 때 `setup` 코드가 실행됩니다.
2.  컴포넌트의 종속성이 변경될 때마다(렌더링될 때마다) 실행됩니다.
    - 초기화 코드가 이전 값과 상태로 실행됩니다. (종속성 배열이 바뀌기 이전 데이터)
    - 그런 다음 `setup` 코드가 새 값과 상태로 실행됩니다.
3.  컴포넌트가 언마운트된 후 초기화 코드가 마지막으로 한 번 실행됩니다.

<br>

- 위의 코드 예시를 이어 설명하겠습니다.
- `ChatRoom` 컴포넌트가 마운트되면 초기 `serverUrl` 및 `roomId`로 채팅방에 연결됩니다.
- `serverUrl` or `roomId`가 변경되면(다른 채팅방을 선택하는 경우) `useEffect`는 이전 채팅방과의 연결을 끊고 다음 채팅방에 연결합니다.
- `ChatRoom` 컴포넌트가 언마운트되면 `useEffect`가 마지막으로 연결을 끊습니다.

<br>

- **유용한 디버깅을 위해 개발 모드에서 React는 `setup` 함수 실행 전에 `setup`, 초기화 함수를 1번 더 실행합니다.**
- `useEffect`의 로직이 올바르게 구현되었는지 확인하는 스트레스 테스트입니다.
- 눈에 띄는 문제가 발생한다면 초기화 함수에 로직이 누락된 것입니다.
- 초기화 함수는 `setup` 함수가 수행하던 작업을 중지하거나 실행을 취소합니다.
- 사용자 관점에서 배포 환경의 `setup`과 개발 환경의 `setup` → 초기화 → `setup` 단계를 시각적으로 구분할 수 없어야 합니다.

<br>

- **모든 `useEffect`를 독립적인 프로세스로 작성하고 한 번에 하나의 `setup`/초기화 사이클을 작성합니다.**
- 컴포넌트의 마운트, 언마운트, 업데이트 상태는 중요하지 않습니다.
- 초기화 로직이 `setup` 로직을 올바르게 '미러링'하면 `useEffect`는 필요한 만큼 자주 `setup` 및 초기화를 실행하여 탄력성을 갖습니다.

<br>

### 참고

- `useEffect`를 사용하면 컴포넌트를 외부 시스템(예: 채팅 서비스)과 동기화할 수 있습니다.
- **외부 시스템**이란 `React`로 제어되지 않는 모든 코드입니다.

<br>

- `setInterval()` 및 `clearInterval()`로 관리되는 타이머
- `window.addEventListener()` 및 `window.removeEventListener()`를 사용하는 이벤트 구독
- `animation.start()` 및 `animation.reset()`과 같은 API가 있는 타사 애니메이션 라이브러리.
- **외부 시스템에 연결하지 않는 경우 `useEffect`가 필요하지 않을겁니다.**

<br>

## 커스텀 `hook`에 `useEffect` 래핑하기

- `useEffect`는 [Escpae Hatch](https://github.com/Collection50/React/blob/master/Learn/04.%20Escape%20Hatches/01.%20Referencing%20Values%20with%20Refs.md)입니다.
- **외부 시스템과 동기화 할 때** 혹은 빌트인 함수가 유용하지 않을 때 사용합니다.
- 자주 `useEffect`를 작성해야 한다면 커스텀 `hook`을 추출해야 한다는 신호입니다.

<br>

- 아래 `useChatRoom` 커스텀 `hook`은 선언적인 API 뒤에 `useEffect`의 로직을 **숨깁니다.**

```jsx
function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

<br>

- 모든 컴포넌트에서 사용할 수 있습니다.

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
  });
  // ...
}
```

<br>

- `React` 생태계에는 훌륭한 커스텀 `hook`이 많습니다.

<br>

## 비반응형 위젯 제어하기

- 외부 시스템을 컴포넌트의 `prop`이나 상태와 동기화하는 경우가 존재합니다.

<br>

- 서드파티 지도 위젯이나 비디오 플레이어 컴포넌트가 있다면 `useEffect`를 사용해 그 상태를 컴포넌트의 현재 상태와 일치시키는 메서드를 호출할 수 있습니다.
- 아래 `useEffect`는 `map-widget.js`에 정의된 `MapWidget` 클래스의 인스턴스를 생성합니다.
- `Map` 컴포넌트의 `zoomLevel` `prop`를 변경하면 `useEffect`는 `setZoom()`을 호출하여 동기화 상태를 유지합니다.

```jsx
// App.js
import { useState } from 'react';
import Map from './Map.js';

export default function App() {
  const [zoomLevel, setZoomLevel] = useState(0);
  return (
    <>
      Zoom level: {zoomLevel}x
      <button onClick={() => setZoomLevel(zoomLevel + 1)}>+</button>
      <button onClick={() => setZoomLevel(zoomLevel - 1)}>-</button>
      <hr />
      <Map zoomLevel={zoomLevel} />
    </>
  );
}
```

```jsx
// Map.js
import { useRef, useEffect } from 'react';
import { MapWidget } from './map-widget.js';

export default function Map({ zoomLevel }) {
  const containerRef = useRef(null);
  const mapRef = useRef(null);

  useEffect(() => {
    if (mapRef.current === null) {
      mapRef.current = new MapWidget(containerRef.current);
    }

    const map = mapRef.current;
    map.setZoom(zoomLevel);
  }, [zoomLevel]);

  return <div style={{ width: 200, height: 200 }} ref={containerRef} />;
}
```

```jsx
// map-widget.js
import 'leaflet/dist/leaflet.css';
import * as L from 'leaflet';

export class MapWidget {
  constructor(domNode) {
    this.map = L.map(domNode, {
      zoomControl: false,
      doubleClickZoom: false,
      boxZoom: false,
      keyboard: false,
      scrollWheelZoom: false,
      zoomAnimation: false,
      touchZoom: false,
      zoomSnap: 0.1,
    });
    L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '© OpenStreetMap',
    }).addTo(this.map);
    this.map.setView([0, 0], 0);
  }
  setZoom(level) {
    this.map.setZoom(level);
  }
}
```

<br>

- 위 예제는 `MapWidget` 클래스가 자신에게 전달된 DOM 노드만 관리하기 때문에 초기화 함수가 필요하지 않습니다.
- `Map` 컴포넌트가 렌더 트리에서 제거된 후 DOM 노드와 `MapWidget` 클래스 인스턴스는 브라우저 자바스크립트 엔진의 가비지 콜렉터에 의해 자동으로 삭제됩니다.

<br>

## `useEffect`로 데이터 페치하기

- `useEffect`를 사용하여 컴포넌트의 데이터를 가져올 수 있습니다.
- 프레임워크를 사용하는 경우 프레임워크의 데이터 불러오기 메커니즘을 사용하는 것이 훨씬 효율적입니다. [(Next.js Fetch)](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating)

<br>

- `useEffect`에서 수동으로 데이터를 가져오려면 다음과 같은 코드가 필요합니다:

```jsx
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then((result) => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    };
  }, [person]);
  // ...
}
```

<br>

- `igonre` 변수는 `false`로 초기화되고 초기화 단계에서 `true`로 설정됩니다.
- 네트워크 요청 순서와 다른 순서로 응답하게 되는 **'경쟁 상태'가 발생하지 않습니다.**

```jsx
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then((result) => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    };
  }, [person]);

  return (
    <>
      <select
        value={person}
        onChange={(e) => {
          setPerson(e.target.value);
        }}
      >
        <option value='Alice'>Alice</option>
        <option value='Bob'>Bob</option>
        <option value='Taylor'>Taylor</option>
      </select>
      <hr />
      <p>
        <i>{bio ?? 'Loading...'}</i>
      </p>
    </>
  );
}
```

<br>

- `async/await`를 사용할 수 있지만 여전히 초기화 함수를 제공해야 합니다.

```jsx
import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    async function startFetching() {
      setBio(null);
      const result = await fetchBio(person);
      if (!ignore) {
        setBio(result);
      }
    }

    let ignore = false;
    startFetching();
    return () => {
      ignore = true;
    };
  }, [person]);

  return (
    <>
      <select
        value={person}
        onChange={(e) => {
          setPerson(e.target.value);
        }}
      >
        <option value='Alice'>Alice</option>
        <option value='Bob'>Bob</option>
        <option value='Taylor'>Taylor</option>
      </select>
      <hr />
      <p>
        <i>{bio ?? 'Loading...'}</i>
      </p>
    </>
  );
}
```

<br>

- `useEffect`에서 직접 데이터를 가져오는 작업을 반복적으로 작성하면 나중에 캐싱 및 서버 렌더링과 같은 최적화를 추가하기가 어려워집니다.
- 직접 만들거나 커뮤니티에서 유지 관리하는 커스텀 `hook`을 사용합니다.

<br>

## `useEffect`에서 데이터 페치를 대체할 수 있는 좋은 대안은 무엇인가요?

- 데이터 페치를 위해 `useEffect` 내에서 `fetch` 메서드를 호출하는 것은 널리 사용되는 방법입니다.
- 하지만 매우 수동적인 접근 방식이며 큰 단점이 있습니다.

<br>

- **`useEffect`는 서버에서 실행되지 않습니다.**
  - 서버에서 렌더링되는 `HTML`에는 데이터가 없습니다.
  - 클라이언트 컴퓨터는 모든 `JavaScript`를 다운로드하고 앱을 렌더링한 이후에 데이터를 로드합니다.
  - 매우 비효율적입니다.
- **`useEffect`에서 데이터를 페치하면 "네트워크 폭포" 현상이 발생합니다.**
  - 상위 컴포넌트 렌더링 => 데이터 페치 => 하위 컴포넌트 렌더링 => 하위 컴포넌트 데이터 페치와 같은 과정입니다.
  - 데이터를 병렬로 페치하는 것보다 훨씬 느립니다.
- **`useEffect`에서 페치하는 것은 데이터를 미리 로드하거나 캐시하지 않습니다.**
  - 컴포넌트가 언마운트 이후 리마운트하는 경우 데이터를 다시 페치합니다.
- **개발자 경험(DX) 성능이 하락합니다.**
  - 경쟁 상태과 같은 버그가 발생하지 않는 방식으로 페치하려면 상용구 코드가 상당히 많이 필요합니다.

<br>

- 위 단점들은 `React`에만 국한된 것이 아닙니다.
- 모든 라이브러리에서 데이터를 가져올 때 적용됩니다.
- 아래 방식을 권장합니다.

<br>

1. **프레임워크를 사용하는 경우 프레임워크에 내장된 데이터 페치를 사용하세요.**
   - 최신 `React` 프레임워크에는 효율적이고 단점을 보완한 통합 데이터 불러오기 메커니즘이 있습니다.
2. **클라이언트 측 캐시를 사용하거나 빌드합니다.**
   - 인기 있는 오픈 소스 솔루션으로는 [React Query](https://tanstack.com/query/v3/), [useSWR](https://swr.vercel.app/ko), [React Router 6.4+](https://beta.reactrouter.com/en/main/start/overview) 등이 있습니다.
   - 자체 솔루션을 구축하는 경우 내부적으로 `useEffect`를 사용하되 요청 중복 제거, 응답 캐싱, 네트워크 폭포 방지(데이터를 미리 로드하거나 데이터 요구 사항을 경로에 올려서)를 위한 로직을 추가합니다.

<br>

- 2가지 방법 모두 적합하지 않은 경우 `useEffect`에서 데이터를 페치할 수 있습니다.

<br>

## 반응형 값을 종속성 목록에 추가

- **`useEffect`의 종속성은 "선택"할 수 없습니다.**
- `useEffect` 코드에서 사용되는 모든 반응형 값은 종속성으로 선언해야 합니다.
- `useEffect`의 종속성 배열은 주변 코드에 의해 결정됩니다.

```jsx
// roomId는 반응형 값입니다.
function ChatRoom({ roomId }) {
  // serverUrl 또한 반응형 값입니다.
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    // useEffect는 반응형 값을 참조합니다.
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
    // ✅ 따라서 종속성으로 지정해야 합니다.
  }, [serverUrl, roomId]);
  // ...
}
```

<br>

- `serverUrl` or `roomId`가 변경되면 `useEffect`가 새 값을 사용하여 채팅에 다시 연결됩니다.

<br>

- **반응형 값에는 `prop`와 컴포넌트 내부에서 직접 선언된 모든 변수 및 함수가 포함됩니다.**
- `roomId`와 `serverUrl`은 반응형 값이기 때문에 종속성에서 제거할 수 없습니다.
- 값을 생략하려고 시도하면 린터는 오류를 반환합니다.

<br>

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
    // 🔴 useEffect에 누락된 종속성: 'roomId', 'serverUrl'
  }, []);
  // ...
}
```

<br>

- **종속성을 제거하려면 컴포넌트가 종속성이 필요가 없다는 것을 린터에 "증명"해야 합니다.**
- `serverUrl`을 컴포넌트 밖으로 이동하여 반응형이 아니며 리렌더링 시 변경되지 않음을 증명합니다.

```jsx
// 상수이므로 더이상 변경되지 않습니다. (반응형 값이 아닙니다.)
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
    // ✅ 모든 종속성 선언 완료
  }, [roomId]);
  // ...
}
```

<br>

- `serverUrl`은 반응형 값이 아니므로(그리고 리렌더링할 때 변경되지 않으므로) 종속성에 선언될 필요가 없습니다.
- `useEffect` 코드가 반응형 값을 사용하지 않는다면 종속성 목록은 비어 있어야 합니다`([])`.

```jsx
// 상수이므로 더이상 변경되지 않습니다. (반응형 값이 아닙니다.)
const serverUrl = 'https://localhost:1234';
const roomId = 'music';

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
    // ✅ 모든 종속성 선언 완료
  }, []);
  // ...
}
```

<br>

- **빈 종속성 배열 있는 `useEffect`는 컴포넌트의 `prop`이나 상태가 변경되어도 다시 실행되지 않습니다.**

<br>

## 주의점

- 기존 `useEffect`에 린터를 억제하는 코드가 있을 수 있습니다.

```jsx
useEffect(() => {
  // ...
  // 🔴 다음과 같이 린터를 억제하지 마세요:
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

<br>

- 종속성 배열이 코드와 일치하지 않으면 버그가 발생할 위험이 높습니다.
- 린터를 억제하면 `useEffect`가 종속하는 값에 대해 "거짓말"을 하는 것입니다.

<br>

## `useEffect`의 이전 상태를 기반으로 상태 업데이트하기

- `useEffect`의 이전 상태를 기반으로 상태를 업데이트하려는 경우 문제가 발생할 수 있습니다.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      // 매초마다 카운터를 증가시키고 싶습니다.
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(intervalId);
    // 🚩 하지만 종속성으로 'count'를 지정하면 항상 Interval이 초기화됩니다.
  }, [count]);
  // ...
}
```

<br>

- `count`는 반응형 값이므로 종속성 목록에 지정해야 합니다.
- 하지만 `count`가 변경될 때마다 `useEffect`를 초기화하고 다시 설정해야 합니다.
- 이상적이지 않습니다.

<br>

- `c => c + 1` 상태 업데이터를 `setCount`에 전달해볼까요?

```jsx
import { useState, useEffect } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      // ✅ 상태를 전달합니다.
      setCount((c) => c + 1);
      updater;
    }, 1000);
    return () => clearInterval(intervalId);
    // ✅  count는 종속성에 없습니다.
  }, []);

  return <h1>{count}</h1>;
}
```

<br>

- 이제 `count + 1` 대신 `c => c + 1`을 전달하므로 `useEffect`는 더 이상 `count`에 의존할 필요가 없습니다.
- `count`가 변경될 때마다 `Interval`을 다시 초기화하고 `setup`할 필요가 없습니다.

<br>

## 불필요한 오브젝트 종속성 제거하기

- `useEffect`가 렌더링 중에 생성된 객체 or 함수에 종속된 경우 너무 자주 실행될 수 있습니다.
- 아래 `useEffect`는 렌더링될 때마다 `options` 객체가 달라지므로 렌더링될 때마다 다시 연결됩니다.

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // 🚩 리렌더링할 때마다 새로 생성됩니다.
  const options = {
    serverUrl: serverUrl,
    roomId: roomId,
  };

  useEffect(() => {
    // Effect 내부에서 사용됩니다.
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
    // 🚩 결과적으로 이러한 종속성은 리렌더링에서 항상 달라집니다.
  }, [options]);
  // ...
}
```

<br>

- 렌더링 중에 생성된 객체, 함수를 종속성에 추가하지 마세요.
- 대신 `useEffect` 내에서 오브젝트를 생성하세요.

```jsx
// App.js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId,
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={(e) => setMessage(e.target.value)} />
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

```jsx
// chat.js
export function createConnection({ serverUrl, roomId }) {
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

- `useEffect` 내부에 `options` 객체를 만들었으므로 `useEffect`는 `roomId`에만 의존합니다.

<br>

- `input` 태그에 타이핑해도 채팅이 다시 연결되지 않습니다.
- 매번 다시 생성되었던 객체와 달리 `roomId`와 같은 문자열은 다른 값으로 설정하지 않는 한 변경되지 않습니다.

<br>

## `useEffect`에서 최신 `prop` 및 상태 참조하기

- 아래는 아직 `React`에 추가되지 않은 실험적 API이므로 아직 사용할 수 없는 부분에 대한 설명입니다.

<br>

- `useEffect`에서 반응형 값을 참조할 떄는 종속성에 추가해야 합니다.
- `useEffect`가 변경에 "반응"하도록 할 수 있습니다.

<br>

- **때로는 '반응'하지 않고 `useEffect`에서 최신 `prop`과 상태를 참조하는 경우가 있습니다.**
- 페이지 방문 시마다 장바구니에 있는 상품의 수를 기록한다고 가정해 보겠습니다.

```jsx
function Page({ url, shoppingCart }) {
  useEffect(() => {
    logVisit(url, shoppingCart.length);
    // ✅ 모든 종속성 선언 완료
  }, [url, shoppingCart]);
  // ...
}
```

<br>

- **`url`이 변경될 때마다 새 페이지 방문을 기록하지만 `shoppingCart`만 변경되는 경우는 기록하지 않으려면 어떻게 해야 하나요?**
- 반응형 규칙을 위반하지 않고 `shoppingCart`를 종속성에서 제외할 수 없습니다.
- 그러나 `useEffect` 내부에서 호출되더라도 변경에 '반응'하지 않는 방법이 존재합니다.
- `useEffectEvent` `hook`를 사용하여 [`Effect` 이벤트를 선언](https://github.com/Collection50/React/blob/master/Learn/04.%20Escape%20Hatches/06.%20Separating%20Events%20from%20Effects.md)하고 `shoppingCart`를 읽는 코드를 내부로 이동시킵니다.

```jsx
function Page({ url, shoppingCart }) {
  const onVisit = useEffectEvent((visitedUrl) => {
    logVisit(visitedUrl, shoppingCart.length);
  });

  useEffect(() => {
    onVisit(url);
    // ✅ 모든 종속성 선언 완료
  }, [url]);
  // ...
}
```

<br>

- **`Effect` 이벤트는 반응형이 아니므로 `useEffect`의 종속성에 포함되선 안 됩니다.**
- 비반응형 코드(`prop` 및 상태의 최신 값을 참조하는 코드)를 내부에 위치할 수 있습니다.
- `onVisit` 내부에서 `shoppingCart`를 읽으면 쇼핑 카트가 `useEffect`를 다시 실행하지 않도록 할 수 있습니다.

<br>

## 서버와 클라이언트에 서로 다른 콘텐츠 표시하기

- `SSR`을 사용하는 경우 컴포넌트는 2개 환경에서 렌더링됩니다.
- 서버에서는 초기 HTML을 렌더링합니다.
- 클라이언트에서는 렌더링 코드를 실행하여 이벤트 핸들러를 해당 HTML에 첨부할 수 있도록 합니다.
- 위와 같은 `hydration`이 작동하려면 클라이언트와 서버에서 초기 렌더링 출력이 동일해야 합니다.

<br>

- 흔하진 않지만 클라이언트에 다른 콘텐츠를 표시해야 하는 경우가 있을 수 있습니다.
- 예를 들어 `localStorage`에서 일부 데이터를 읽는 경우 서버에서는 이를 수행할 수 없습니다.

```jsx
function MyComponent() {
  const [didMount, setDidMount] = useState(false);

  useEffect(() => {
    setDidMount(true);
  }, []);

  if (didMount) {
    // ... 클라이언트 전용 JSX 반환 ...
  } else {
    // ... 서버 전용 JSX 반환 ...
  }
}
```

<br>

- 컴포넌트가 로드되는 동안 사용자는 초기 렌더링을 볼 수 있습니다.
- 그런 다음 `hydration`되면 `useEffect`가 실행되고 `didMount`를 `true`로 설정하여 리렌더링됩니다.
- 다시 클라이언트 렌더링으로 전환됩니다.
- `useEffect`는 서버에서 실행되지 않으므로 초기 서버 렌더링 중에 `didMount`는 `false`입니다.

<br>

- 이 패턴을 많이 사용하면 안 됩니다.
- 네트워크 속도가 느린 사용자는 초기 콘텐츠를 꽤 오랜 시간(몇 초)동안 보게 되므로 컴포넌트의 모양을 갑작스럽게 변경하면 안 되기 떄문입니다.
- 대부분의 경우 `CSS`를 사용하여 조건부로 렌더링하는 방법으로 해결합니다.

<br>

## 트러블슈팅

### 컴포넌트가 마운트될 때 `useEffect`가 두 번 실행됩니다.

- `Strict Mode`가 켜져 있으면 개발 모드에서 `React`는 실제 `setup` 전에 `setup` 및 초기화를 한 번 더 실행합니다.

<br>

- `useEffect`의 로직이 올바르게 구현되었는지 확인하는 스트레스 테스트입니다.
- 눈에 띄는 문제가 발생하면 초기화 함수에 일부 로직이 누락된 것입니다.
- 초기화 함수는 `setup` 함수가 수행하던 작업을 중지하거나 취소합니다.
- 사용자 관점에서 배포 환경의 `setup`과 개발 환경의 `setup` → 초기화 → `setup` 단계를 시각적으로 구분할 수 없어야 합니다.

<br>

### `useEffect`가 리렌더링할 때마다 실행됩니다.

- 종속성 배열을 지정했는지 확인합니다.

```jsx
useEffect(() => {
  // ...
}); // 🚩 의존성 배열 없음: 렌더링할 때마다 재실행!
```

<br>

- 종속성 배열을 지정했는데도 `useEffect`가 여전히 다시 실행되는 경우 리렌더링마다 종속성이 달라지기 때문입니다.
- 종속성을 콘솔에 수동으로 로깅하여 이 문제를 디버그할 수 있습니다.

```jsx
useEffect(() => {
  // ..
}, [serverUrl, roomId]);

console.log([serverUrl, roomId]);
```

<br>

- 콘솔에서 서로 다른 리렌더링의 배열을 마우스 오른쪽 버튼으로 클릭하고 두 배열 모두에 대해 "전역 변수로 저장"을 선택합니다.
- 첫 번째 배열이 `temp1`로, 두 번째 배열이 `temp2`로 저장되었다고 가정하면 콘솔을 사용하여 두 종속성이 동일한지 확인할 수 있습니다.

```js
Object.is(temp1[0], temp2[0]); // 첫 번째 종속성이 배열 간에 동일한가요?
Object.is(temp1[1], temp2[1]); // 두 번째 의존성이 배열 간에 동일한가요?
Object.is(temp1[2], temp2[2]);
```

<br>

- 리렌더링할 때마다 종속성이 달라지는 경우 다음 방법 중 하나로 해결할 수 있습니다.
  - `useEffect`의 이전 상태를 기반으로 상태 업데이트
  - 불필요한 객체, 함수의 종속성 제거하기
  - `useEffect`에서 최신 `props` 및 상태 참조하기

<br>

- 최후의 수단으로(위 메서드가 도움이 되지 않는 경우), `useMemo` 또는 `useCallback`(함수의 경우)으로 생성을 래핑합니다.

<br>

### `useEffect`가 무한으로 계속 재실행됩니다.

- `useEffect`가 무한으로 실행되는 경우 아래 2가지가 모두 충족되어야 합니다.
  1. `useEffect`가 어떤 상태를 업데이트하고 있습니다.
  2. 상태는 리렌더링으로 이어지며 이로 인해 `useEffect`의 종속성이 변경됩니다.

<br>

- 문제 해결을 시작하기 전에 `useEffect`가 외부 시스템(예: DOM, 네트워크, 타사 위젯 등)에 연결되고 있는지 확인합니다.
- `useEffect`에 업데이트 함수가 필요한 이유는 무엇인가요?
  - 외부 시스템과 동기화되나요?
  - 애플리케이션의 데이터 흐름을 관리하나요?

<br>

- 외부 시스템이 없는 경우 `useEffect`를 완전히 제거하면 로직이 간소화되는지 확인합니다.

<br>

- 외부 시스템과 동기화하는 경우 `useEffect`가 상태를 업데이트하는 이유와 조건에 대해 생각해 보세요.
- 컴포넌트에 시각적으로 영향을 미치는 변경 사항이 있나요?
- 렌더링에 사용되지 않는 일부 데이터를 참조해야 하는 경우 리렌더링을 트리거하지 않는 `ref`가 더 적합할 수 있습니다.
- `useEffect`가 필요 이상으로 상태를 업데이트하지 않는지(그리고 리렌더링을 트리거하지 않는지) 확인하세요.

<br>

- 마지막으로 `useEffect`가 적절한 시점에 상태를 업데이트하고 있지만 여전히 무한 반복되는 경우 상태 업데이트로 인해 `useEffect`의 종속성 중 하나가 변경되기 때문입니다.
- 종속성 변경을 디버깅합니다.

<br>

### 컴포넌트가 언마운트 되지 않았는데 초기화 로직이 실행됩니다.

- 초기화 함수는 언마운트뿐만 아니라 변경된 종속성으로 **리렌더링하기 전**에 실행됩니다.
- 또한 개발 모드에서 컴포넌트 마운트 직후에 `setup` + 초기화 함수를 1번 더 실행합니다.

<br>

- 상응하는 `setup` 로직이 없는 초기화 로직이 있다면 코드에서 악취가 나는 것입니다.

```jsx
useEffect(() => {
  // 🔴 지양하세요: 해당 setup 로직이 없는 초기화 로직
  return () => {
    doSomething();
  };
}, []);
```

<br>

- 초기화 로직은 `setup` 로직과 **데칼코마니**되어야 하며 `setup` 로직이 수행한 모든 작업을 중지하거나 취소해야 합니다.

```jsx
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => {
    connection.disconnect();
  };
}, [serverUrl, roomId]);
```

<br>

### `useEffect`가 시각적인 작업을 수행하는데 실행되기 전에 깜박임이 나타납니다.

- `useEffect`로 인해 브라우저가 `paint`하는 것을 방지해야 하는 경우 `useEffect`를 `useLayoutEffect`로 바꾸세요.
- 대부분의 `useEffect`에는 필요하지 않습니다.
- 브라우저 `paint` 전에 `useEffect`를 실행하는 것이 중요한 경우(예: 사용자가 보기 전에 툴팁을 측정하고 위치를 지정하는 경우)에만 이 옵션이 필요합니다.
