# useTransition

- `useTransition`은 UI를 차단하지 않고 상태를 업데이트하는 `React` `hook`입니다.

```jsx
const [isPending, startTransition] = useTransition();
```

<br>

## 레퍼런스

### `useTransition()`

- 컴포넌트 최상단에서 `useTransition`을 호출하여 일부 상태 업데이트를 트랜지션으로 표시합니다.

```jsx
import { useTransition } from 'react';

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  // ...
}
```

<br>

### 매개변수

- `useTransition`은 매개 변수를 받지 않습니다.

<br>

### 반환값

- `useTransition`은 정확히 2개의 항목이 있는 배열을 반환합니다.

<br>

1. 보류 중인 트랜지션이 있는지 여부를 알려주는 `isPending`
2. 상태 업데이트를 트랜지션으로 표시할 수 있는 **`startTransition` 함수**

<br>

## `startTransition` 함수

- `useTransition`에서 반환하는 `startTransition` 함수를 사용하면 상태 업데이트를 트랜지션으로 표시할 수 있습니다.

```jsx
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

<br>

### 매개변수

- `scope`
  - 1개 이상의 **`set` 함수**를 호출하여 일부 상태를 업데이트하는 함수
  - `React`는 매개변수 없이 즉시 `scope`를 호출하고 `scope` 호출 중에 동기적으로 예약된 모든 상태 업데이트를 트랜지션으로 표현합니다.
  - 이 함수는 **멈춰지지 않고** **원치 않는 로딩창을 표시하지 않습니다.**

<br>

### 반환값

- `startTransition`은 아무것도 반환하지 않습니다.

<br>

### 주의사항

- `useTransition`은 `hook`이므로 컴포넌트나 커스텀 `hook` 내부에서만 호출할 수 있습니다.
- 다른 곳(예: 데이터 라이브러리)에서 트랜지션을 시작하는 경우 독립형 `startTransition`을 호출합니다.

<br>

- 상태의 **`set` 함수를 사용할 수 있는 경우**에만 업데이트를 트랜지션으로 래핑할 수 있습니다.
- **`prop`이나 커스텀 `hook`**에 대한 응답으로 트랜지션을 시작하려면 `useDeferredValue`를 사용합니다.

<br>

- `startTransition`에 전달하는 함수는 동기로 동작합니다.
- `React`는 함수를 즉시 실행하여 실행하는 동안 발생하는 모든 상태 업데이트를 트랜지션으로 표시합니다.
- 나중에 더 많은 상태 업데이트를 수행하려고 하면(예: 타임아웃) 트랜지션으로 표시되지 않습니다.

<br>

- 트랜지션으로 표시된 상태 업데이트는 다른 상태 업데이트에 의해 **중단**됩니다.
- 예를 들어 트랜지션 내에서 차트 컴포넌트를 업데이트한 다음, 차트가 다시 렌더링되는 도중에 타이핑을 시작하면 `React`는 타이핑 업데이트를 먼저 처리한 후 차트 컴포넌트에서 렌더링 작업을 다시 시작합니다.

<br>

- 트랜지션 업데이트는 텍스트 입력을 제어하는 데 사용할 수 없습니다.

<br>

- 진행 중인 트랜지션이 여러 개 있는 경우 `React`는 현재 트랜지션을 함께 **일괄 처리**합니다.
- 향후 버전에서 제거될 가능성이 높습니다.

<br>

## 사용법

### 상태 업데이트를 멈추지 않고 트랜지션으로 표시하기

- 컴포넌트 최상단에서 `useTransition`을 호출하여 상태 업데이트를 멈추지 않고 트랜지션으로 표현합니다.

```jsx
import { useState, useTransition } from 'react';

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  // ...
}
```

<br>

- `useTransition`은 정확히 2개 항목이 있는 배열을 반환합니다.

<br>

1. 보류 중인 트랜지션이 있는지 여부를 알려주는 `isPending`
2. 상태 업데이트를 트랜지션으로 표시할 수 있는 `startTransition` 함수

<br>

- 다음과 같이 상태 업데이트를 트랜지션으로 표시할 수 있습니다.

```jsx
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

<br>

- 트랜지션을 사용하면 느린 디바이스에서도 UI의 반응성을 유지할 수 있습니다.

<br>

- 트랜지션을 사용하면 리렌더링하는 동안에도 UI의 반응성을 유지할 수 있습니다.
- 예를 들어 사용자가 탭을 클릭했다가 마음이 바뀌어 다른 탭을 클릭하면 첫 번째 리렌더링이 완료될 때까지 기다릴 필요 없이 다른 탭을 클릭할 수 있습니다.

<br>

### 트랜지션에서 상위 컴포넌트 업데이트

- `useTransition` 호출에서 상위 컴포넌트의 상태를 업데이트할 수 있습니다.
- 예를 들어 `TabButton` 컴포넌트는 `onClick` 로직을 트랜지션으로 래핑합니다.

```jsx
export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>;
  }
  return (
    <button
      onClick={() => {
        startTransition(() => {
          onClick();
        });
      }}
    >
      {children}
    </button>
  );
}
```

<br>

- 상위 컴포넌트가 `onClick` 이벤트 핸들러 내에서 **상태를 업데이트**하기 때문에 해당 상태 업데이트는 트랜지션으로 표시됩니다.
- 앞의 예시에서처럼 **탭**을 클릭한 다음 바로 **다른 탭**을 클릭할 수 있습니다.
- 선택한 탭을 업데이트하는 것은 트랜지션으로 표시되므로 유저의 상호 작용을 차단하지 않습니다.

```jsx
import { useTransition } from 'react';

export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>;
  }
  return (
    <button
      onClick={() => {
        startTransition(() => {
          onClick();
        });
      }}
    >
      {children}
    </button>
  );
}
```

<br>

### 트랜지션 중 대기 중인 시각적 상태 표시하기

- `useTransition`이 반환하는 `isPending` 불리언 값을 사용하여 트랜지션이 진행 중임을 표시할 수 있습니다.
- 예를 들어 탭 버튼은 **대기 중** 시각적 상태를 표현합니다.

```jsx
function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  // ...
  if (isPending) {
    return <b className='pending'>{children}</b>;
  }
  // ...
}
```

<br>

- 탭 버튼 자체가 바로 업데이트되므로 클릭에 대한 반응이 더 빨라졌습니다.

```jsx
// Tabbutton.js
import { useTransition } from 'react';

export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>;
  }
  if (isPending) {
    return <b className='pending'>{children}</b>;
  }
  return (
    <button
      onClick={() => {
        startTransition(() => {
          onClick();
        });
      }}
    >
      {children}
    </button>
  );
}
```

<br>

### 원치 않는 로딩창 방지

- 아래 예제에서 `PostsTab` 컴포넌트는 **`Suspense`를 지원**하는 데이터를 가져옵니다.
- **Posts** 탭을 클릭하면 `PostsTab` 컴포넌트가 일시 **중단**되어 가장 가까운 로딩 `fallback`이 표시됩니다.

```jsx
// App.js
import { Suspense, useState } from 'react';
import TabButton from './TabButton.js';
import AboutTab from './AboutTab.js';
import PostsTab from './PostsTab.js';
import ContactTab from './ContactTab.js';

export default function TabContainer() {
  const [tab, setTab] = useState('about');
  return (
    <Suspense fallback={<h1>🌀 Loading...</h1>}>
      <TabButton isActive={tab === 'about'} onClick={() => setTab('about')}>
        About
      </TabButton>
      <TabButton isActive={tab === 'posts'} onClick={() => setTab('posts')}>
        Posts
      </TabButton>
      <TabButton isActive={tab === 'contact'} onClick={() => setTab('contact')}>
        Contact
      </TabButton>
      <hr />
      {tab === 'about' && <AboutTab />}
      {tab === 'posts' && <PostsTab />}
      {tab === 'contact' && <ContactTab />}
    </Suspense>
  );
}
```

```jsx
export default function TabButton({ children, isActive, onClick }) {
  if (isActive) {
    return <b>{children}</b>;
  }
  return (
    <button
      onClick={() => {
        onClick();
      }}
    >
      {children}
    </button>
  );
}
```

<br>

- 로딩 창을 표시하기 위해 전체 탭 컨테이너이 사라지면 어색할겁니다.
- `TabButton`에 `useTransition`을 추가하여 탭 버튼에 대기 상태를 표시합니다.

<br>

- **`Post`**를 클릭하면 더 이상 전체 탭 컨테이너가 스피너로 대체되지 않습니다.

```jsx
// App.js
import { Suspense, useState } from 'react';
import TabButton from './TabButton.js';
import AboutTab from './AboutTab.js';
import PostsTab from './PostsTab.js';
import ContactTab from './ContactTab.js';

export default function TabContainer() {
  const [tab, setTab] = useState('about');
  return (
    <Suspense fallback={<h1>🌀 Loading...</h1>}>
      <TabButton isActive={tab === 'about'} onClick={() => setTab('about')}>
        About
      </TabButton>
      <TabButton isActive={tab === 'posts'} onClick={() => setTab('posts')}>
        Posts
      </TabButton>
      <TabButton isActive={tab === 'contact'} onClick={() => setTab('contact')}>
        Contact
      </TabButton>
      <hr />
      {tab === 'about' && <AboutTab />}
      {tab === 'posts' && <PostsTab />}
      {tab === 'contact' && <ContactTab />}
    </Suspense>
  );
}
```

```jsx
// TabButton.js
import { useTransition } from 'react';

export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>;
  }
  if (isPending) {
    return <b className='pending'>{children}</b>;
  }
  return (
    <button
      onClick={() => {
        startTransition(() => {
          onClick();
        });
      }}
    >
      {children}
    </button>
  );
}
```

<br>

### 참고

- 트랜지션은 이미 표시된 콘텐츠(예: 탭 컨테이너)를 숨기지 않을 만큼만 **대기**합니다.
- 게시물 `Post`에 **중첩된 `<Suspense>` 바운더리**가 있는 경우 트랜지션은 이를 **대기**하지 않습니다.

<br>

### `Suspense` 지원 라우터 구축하기

- `React` 프레임워크나 라우터를 사용하는 경우 페이지 이동을 트랜지션으로 표시하는 것이 좋습니다.

```jsx
function Router() {
  const [page, setPage] = useState('/');
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }
  // ...
}
```

<br>

- 아래 2가지 이유를 들어 트랜지션 사용을 권장합니다.

<br>

1. 트랜지션은 중단할 수 있으므로 유저는 리렌더링이 완료될 때까지 기다리지 않고 바로 클릭할 수 있습니다.
2. 트랜지션은 원치 않는 로딩 창을 방지하여 부자연스러운 이동을 방지합니다.

<br>

- 다음은 트랜지션을 사용하는 아주 간단한 라우터 예제입니다.

```jsx
// App.js
import { Suspense, useState, useTransition } from 'react';
import IndexPage from './IndexPage.js';
import ArtistPage from './ArtistPage.js';
import Layout from './Layout.js';

export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState('/');
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }

  let content;
  if (page === '/') {
    content = <IndexPage navigate={navigate} />;
  } else if (page === '/the-beatles') {
    content = (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  }
  return <Layout isPending={isPending}>{content}</Layout>;
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}
```

```jsx
// Layout.js
export default function Layout({ children, isPending }) {
  return (
    <div className='layout'>
      <section
        className='header'
        style={{
          opacity: isPending ? 0.7 : 1,
        }}
      >
        Music Browser
      </section>
      <main>{children}</main>
    </div>
  );
}
```

```jsx
// IndexPage.js
export default function IndexPage({ navigate }) {
  return (
    <button onClick={() => navigate('/the-beatles')}>
      Open The Beatles artist page
    </button>
  );
}
```

```jsx
// ArtistPage.js
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Biography artistId={artist.id} />
      <Suspense fallback={<AlbumsGlimmer />}>
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function AlbumsGlimmer() {
  return (
    <div className='glimmer-panel'>
      <div className='glimmer-line' />
      <div className='glimmer-line' />
      <div className='glimmer-line' />
    </div>
  );
}
```

<br>

### 참고

- `Suspense` 지원 라우터는 기본적으로 페이지 이동을 트랜지션으로 래핑할 것으로 예상됩니다.

<br>

## 트러블슈팅

### 트랜지션에서 `input` 태그의 업데이트가 작동하지 않습니다.

- 입력을 제어하는 상태 변수에는 트랜지션을 사용할 수 없습니다.

```jsx
const [text, setText] = useState('');
// ...
function handleChange(e) {
  // ❌ 입력 상태에는 트랜지션을 사용할 수 없습니다.
  startTransition(() => {
    setText(e.target.value);
  });
}
// ...
return <input value={text} onChange={handleChange} />;
```

<br>

- `onChange` 이벤트에 대한 응답으로 `input`을 업데이트하는 것은 동기적으로 이루어져야 하기 때문입니다.
- `input`에 대한 응답으로 트랜지션을 실행하려면 2가지 옵션이 있습니다.

<br>

1. 항상 동기적으로 업데이트되는 `input` 상태와 트랜지션 시 업데이트할 상태 변수를 별도로 선언합니다.
   - 동기 상태를 사용하여 `input`을 제어하고 나머지 렌더링 로직에 `input`보다 **지연**되는 트랜지션 상태 변수를 전달합니다.
2. 상태 변수가 하나 있고 실제 값보다 **지연**되는 `useDeferredValue`를 추가할 수 있습니다.
   - 그러면 새 값을 자동으로 캐치하도록 리렌더링됩니다.

<br>

### `React`가 상태 업데이트를 트랜지션으로 처리하지 않습니다.

- 상태 업데이트(`set` 함수)를 트랜지션으로 래핑할 때는 `startTransition` 호출 **중에** 발생하도록 하세요.

```jsx
startTransition(() => {
  // ✅  호출 "중" 상태 설정
  setPage('/about');
});
```

<br>

- `startTransition`에 전달하는 함수는 동기적이어야 합니다.

<br>

- 아래와 같이 업데이트를 트랜지션으로 표시할 수 없습니다.

```jsx
startTransition(() => {
  // ❌ startTransition 호출 "후" 상태 설정 (비동기)
  setTimeout(() => {
    setPage('/about');
  }, 1000);
});
```

<br>

- 아래처럼 사용해야 합니다.

```jsx
setTimeout(() => {
  startTransition(() => {
    // ✅ startTransition 호출 "중" 상태를 설정합니다.
    setPage('/about');
  });
}, 1000);
```

<br>

- 마찬가지로 업데이트를 아래와 같이 표시할 수 없습니다.

```jsx
startTransition(async () => {
  await someAsyncFunction();
  // ❌ startTransition 호출 "후" 상태 설정
  setPage('/about');
});
```

<br>

- 아래와 같이 사용합니다.

```jsx
await someAsyncFunction();
startTransition(() => {
  // ✅ startTransition 호출 "중" 상태를 설정합니다.
  setPage('/about');
});
```

<br>

### 컴포넌트 외부에서 `useTransition`을 호출하고 싶습니다.

- `hook`이기 때문에 컴포넌트 외부에서 `useTransition`을 호출할 수 없습니다.
- 독립적인 `startTransition` 메서드를 사용합니다.
- 같은 방식으로 작동하지만 `isPending`을 제공하지 않습니다.

<br>

### `startTransition`에 전달한 함수가 즉시 실행됩니다.

- 아래 코드를 실행하면 `1, 2, 3`이 출력됩니다.

```jsx
console.log(1);
startTransition(() => {
  console.log(2);
  setPage('/about');
});
console.log(3);
```

<br>

- **`1, 2, 3`을 출력할 것으로 예상됩니다.**
- `startTransition`에 전달한 함수는 지연되지 않습니다.
- `setTimeout`과 달리 나중에 콜백을 실행하지 않습니다.
- `React`는 함수를 즉시 실행하지만 **실행되는 동안** 예약된 모든 상태 업데이트는 트랜지션으로 표시됩니다.
- 이렇게 작동한다고 상상하면 됩니다.

```jsx
// React가 작동하는 방식을 단순화한 버전
let isInsideTransition = false;

function startTransition(scope) {
  isInsideTransition = true;
  scope();
  isInsideTransition = false;
}

function setState() {
  if (isInsideTransition) {
    // ... 트랜지션 상태 업데이트 예약 ...
  } else {
    // ... 긴급한 상태 업데이트 예약 ...
  }
}
```
