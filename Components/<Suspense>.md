# `<Suspense>`

- `<Suspense>`를 사용하면 하위 컴포넌트의 로딩이 완료될 때까지 `fallback`을 표시할 수 있습니다.

```jsx
<Suspense fallback={<Loading />}>
  <SomeComponent />
</Suspense>
```

## 레퍼런스

### `<Suspense>`

### `Props`

- `children` - 렌더링하려는 실제 UI입니다.
  - 렌더링하는 동안 하위 컴포넌트가 일시 중단되면 `<Suspense>` 경계가 `fallback`으로 전환됩니다.
- `fallback`
  - 로딩이 완료되지 않은 경우 렌더링하는 대체 UI입니다.
  - 유효한 `React` 노드는 무엇이든 사용할 수 있지만 일반적으로 `fallback`은 로딩 스피너나 스켈레톤을 사용하빈다.
  - `<Suspense>`는 하위 컴포넌트가 일시 중단되면 자동으로 `fallback`으로 전환되고 데이터가 준비되면 하위 컴포넌트로 전환됩니다.
  - 렌더링 중에 `fallback`이 중단되면 가까운 상위 `<Suspense>` 경계를 활성화합니다.

<br>

### 주의사항

- `React`는 처음 마운트하기 전에 일시 중단된 컴포넌트의 상태를 보존하지 않습니다.
  - 컴포넌트가 로드되면 `React`는 중단된 트리의 컴포넌트를 처음부터 렌더링합니다.
- `<Suspense>`가 콘텐츠를 표시하고 있다가 일시 중단된 경우 원인이 `startTransition` 또는 `useDeferredValue`로 인한 것이 아니라면 `fallback`이 다시 표시됩니다.
- `React`가 다시 일시 중단되어 이미 표시된 콘텐츠를 숨겨야 하는 경우 `useLayoutEffect`를 초기화합니다.
  - 콘텐츠가 다시 표시될 준비가 되면 `React`는 `useLayoutEffect`를 다시 실행합니다.
  - 콘텐츠가 숨겨져 있는 동안 `useLayoutEffect`가 실행되지 않도록 합니다.
- `React`에는 `<Suspense>`와 통합된 **스트리밍 `SSR`**와 **선택적 `hydration`** 등의 내부 최적화가 이뤄져있습니다.
- [아키텍처 개요](https://github.com/reactwg/react-18/discussions/37)를 읽고 자세히 알아보세요.

<br>

## 사용법

### 콘텐츠가 로드되는 동안 `fallback` 표시

- 애플리케이션의 모든 부분을 `<Suspense>` 경계로 래핑할 수 있습니다.

```jsx
<Suspense fallback={<Loading />}>
  <Albums />
</Suspense>
```

<br>

- `React`는 하위 컴포넌트의 코드와 데이터가 로드될 때까지 **`fallback`**을 표시합니다.

<br>

- 아래 예시는 앨범 목록을 가져오는 동안 `Album` 컴포넌트가 일시 중단됩니다.
- 준비가 될 때까지 위에서 가장 가까운 `<Suspense>` 경계로 전환하여 `Loading` 컴포넌트를 렌더링합니다.
- 데이터가 로드되면 `React`는 `Loading` `fallback`을 숨기고 `Album` 컴포넌트를 렌더링합니다.

```jsx
import { Suspense } from 'react';
import Albums from './Albums.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <Albums artistId={artist.id} />
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>🌀 Loading...</h2>;
}
```

<br>

### 참고

- `<Suspense>`를 지원하는된 데이터만 `<Suspense>` 컴포넌트를 사용할 수 있습니다.
- `<Suspense>`를 지원하는 것은 아래와 같습니다.

<br>

- `Relay`, `Next.js` 같이 `<Suspense>`를 지원하는 프레임워크를 사용한 데이터 페치
- `lazy`를 사용하여 지연 로딩하는 컴포넌트

<br>

- `<Suspense>`는 `useEffect` 또는 이벤트 핸들러 내부에서 데이터를 가져올 때는 **감지하지 않습니다.**

<br>

- `Album` 컴포넌트에서 데이터의 페치 방법은 프레임워크마다 다릅니다.
- `<Suspense>` 지원 프레임워크를 사용하는 경우 데이터 페치 문서에서 자세한 내용을 확인합니다.

<br>

- `opinionated` 프레임워크 없이 `<Suspense>`를 사용한 데이터 페치는 아직 사용할 수 없습니다.
- `<Suspense>`를 지원하는 데이터를 구현하기 위한 요구 사항은 불안정하고 문서화되지 않았습니다.
- 데이터를 `<Suspense>`와 통합하기 위한 공식 API는 이후의 `React` 버전에서 출시될 예정입니다.

<br>

### 콘텐츠를 한 번에 공개

- 기본적으로 `<Suspense>`의 전체 트리는 1게 단위로 취급됩니다.
- 예를 들어 컴포넌트 중 **1개만** 일시 중단되더라도 **모든** 컴포넌트가 함께 로딩 표시기로 대체됩니다.

```jsx
<Suspense fallback={<Loading />}>
  <Biography />
  <Panel>
    <Albums />
  </Panel>
</Suspense>
```

<br>

- 모든 항목이 렌더링 준비가 되면 모두 함께 표시됩니다.

<br>

- 아래 예제는 `Biography`와 `Album` 모두 데이터를 페치합니다.
- 그러나 1개 `<Suspense>` 경계에 그룹화되어 있기 때문에 컴포넌트는 항상 함께 **렌더링**됩니다.

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
      <Suspense fallback={<Loading />}>
        <Biography artistId={artist.id} />
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>🌀 Loading...</h2>;
}
```

```jsx
// Panel.js
export default function Panel({ children }) {
  return <section className='panel'>{children}</section>;
}
```

<br>

- 데이터를 페치하는 컴포넌트가 `<Suspense>` 경계의 바로 아래의 하위 컴포넌트일 필요는 없습니다.
- 예를 들어 `Biography`와 `Album`을 새 `Details` 컴포넌트로 이동할 수 있습니다.
- 동작은 변경되지 않습니다.
- `Biography`와 `Album`은 가장 가까운 상위 `<Suspense>` 경계를 공유하므로 함께 렌더링됩니다.

```jsx
<Suspense fallback={<Loading />}>
  <Details artistId={artist.id} />
</Suspense>;

function Details({ artistId }) {
  return (
    <>
      <Biography artistId={artistId} />
      <Panel>
        <Albums artistId={artistId} />
      </Panel>
    </>
  );
}
```

<br>

### 중첩된 콘텐츠가 로드될 때 표시하기

- 컴포넌트가 일시 중단되면 가장 가까운 상위 `<Suspense>`가 `fallback`을 표시합니다.
- 여러 `<Suspense>` 컴포넌트를 중첩하여 로딩 시퀀스를 만들 수 있습니다.
- `<Suspense>`의 `fallback`은 하위 컴포넌트를 사용할 수 있게 되면 렌더링됩니다.
- 예를 들어 `Album` 목록에 고유한 `fallback`을 지정할 수 있습니다.

```jsx
<Suspense fallback={<BigSpinner />}>
  <Biography />
  <Suspense fallback={<AlbumsGlimmer />}>
    <Panel>
      <Albums />
    </Panel>
  </Suspense>
</Suspense>
```

<br>

- `Biography`를 표시할 때 `Album`이 로드될 때까지 **기다릴** 필요가 없습니다.

<br>

- 순서는 다음과 같습니다.

<br>

1. `Biography`가 로드되지 않은 경우 콘텐츠 대신 `BigSpinner`가 표시됩니다.
2. `Biography`가 로드되면 `BigSpinner`가 콘텐츠로 렌더링됩니다.
3. `Album`이 로드되지 않은 경우 `AlbumsGlimmer`가 표시됩니다.
4. 마지막으로 `Album`이 로드되면 `Album`이 렌더링됩니다.

<br>

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
      <Suspense fallback={<BigSpinner />}>
        <Biography artistId={artist.id} />
        <Suspense fallback={<AlbumsGlimmer />}>
          <Panel>
            <Albums artistId={artist.id} />
          </Panel>
        </Suspense>
      </Suspense>
    </>
  );
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
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

```jsx
export default function Panel({ children }) {
  return <section className='panel'>{children}</section>;
}
```

<br>

- `<Suspense>`를 사용하면 UI의 어떤 부분이 항상 동시에 **렌더링**되어야 하는지, 어떤 부분이 점진적으로 콘텐츠를 표시해야 하는지 조정할 수 있습니다.
- 나머지 동작에 영향을 주지 않고 트리의 어느 위치에서나 `<Suspense>` 경계를 추가, 이동, 삭제할 수 있습니다.

<br>

- 모든 컴포넌트에 `<Suspense>`를 설정하지 마세요.
- `<Suspense>`는 유저가 경험하는 로딩 시퀀스보다 더 세분화되어서는 안 됩니다.
- 디자이너와 함께 작업하는 경우 로딩 상태를 어디에 배치해야 하는지 디자이너에게 물어보세요.
- 디자인 와이어프레임에 포함되었을 가능성이 높습니다.

<br>

### 새 콘텐츠가 로드되는 동안 이전 콘텐츠 표시하기

- 아래 예시는 검색 결과를 가져오는 동안 `SearchResults` 컴포넌트가 일시 중단됩니다.
- `"a"`를 입력하고 결과를 기다린 다음 `"ab"`를 입력합니다.
- `"a"`에 대한 결과는 로딩 `fallback`으로 대체됩니다.

```jsx
import { Suspense, useState } from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
  const [query, setQuery] = useState('');
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={(e) => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={query} />
      </Suspense>
    </>
  );
}
```

<br>

- 일반적인 UI 대체 패턴은 목록 업데이트를 **지연**하고 새 결과가 준비될 때까지 이전 결과를 계속 표시하는 것입니다.
- `useDeferredValue` `hook`을 사용하면 `query`의 **지연**된 버전을 전달합니다.

```jsx
export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={(e) => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <SearchResults query={deferredQuery} />
      </Suspense>
    </>
  );
}
```

<br>

- `query`가 즉시 업데이트되므로 `input`에 새 값이 표시됩니다.
- 그러나 `deferredQuery`는 데이터가 로드될 때까지 이전 값을 유지하므로 `SearchResults`는 잠시 동안 이전 결과를 표시합니다.

<br>

- 유저에게 더 명확하게 알리기 위해 이전 결과 목록이 표시될 때 시각적 효과를 추가할 수 있습니다.

```jsx
<div
  style={{
    opacity: query !== deferredQuery ? 0.5 : 1,
  }}
>
  <SearchResults query={deferredQuery} />
</div>
```

<br>

- 아래에서 `"a"`를 입력하고 결과가 로드될 때까지 기다린 다음 `"ab"`를 입력합니다.
- 일시 중단 `fallback` 대신 이전 결과 목록이 표시됩니다.

```jsx
import { Suspense, useState, useDeferredValue } from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  const isStale = query !== deferredQuery;
  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={(e) => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
        <div style={{ opacity: isStale ? 0.5 : 1 }}>
          <SearchResults query={deferredQuery} />
        </div>
      </Suspense>
    </>
  );
}
```

<br>

### 참고

- 지연된 값과 트랜지션을 모두 사용하면 `<Suspense>` `fallback`을 표시하지 않을 수 있습니다.
- 트랜지션은 **전체** 업데이트를 급하지 않은 것으로 간주하므로 일반적으로 프레임워크, 라우터 라이브러리의 네비게이션에서 사용됩니다.
- 반면에 지연된 값은 UI의 **일부**를 긴급하지 않은 것으로 표시하고 다른 UI보다 **지연**시키려는 데 유용합니다.

<br>

### 이미 보여지는 콘텐츠가 숨겨지지 않도록 방지

- 컴포넌트가 일시 중단되면 가장 가까운 상위 `<Suspense>` 경계가 `fallback`을 표시하도록 전환됩니다.
- 이미 콘텐츠가 보여지고 있는 경우 사용자 경험은 하락합니다.

```jsx
// App.js
import { Suspense, useState } from 'react';
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

  function navigate(url) {
    setPage(url);
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
  return <Layout>{content}</Layout>;
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}
```

```jsx
// Layout.js
export default function Layout({ children }) {
  return (
    <div className='layout'>
      <section className='header'>Music Browser</section>
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

- 버튼을 누르자 `Router` 컴포넌트가 `IndexPage` 대신 `ArtistPage`를 렌더링합니다.
- `ArtistPage`의 하위 컴포넌트가 일시 중단되었기 때문에 가장 가까운 `<Suspense>`가 `fallback`을 표시했기 때문입니다.
- 가장 가까운 `<Suspense>`는 `root` 근처에 있었기 때문에 전체 레이아웃이 `BigSpinner`로 변경되었습니다.

<br>

- 이를 방지하려면 `startTransition`을 사용하여 상태 업데이트를 트랜지션으로 표시할 수 있습니다.

```jsx
function Router() {
  const [page, setPage] = useState('/');

  function navigate(url) {
    startTransition(() => {
      setPage(url);
    });
  }
  // ...
}
```

<br>

- 이는 트랜지션이 급하지 않으며 이미 보여지는 콘텐츠를 숨기는 대신 이전 페이지를 렌더링하도록 합니다.
- 이제 버튼을 클릭하면 `Biography`가 로드될 때까지 **대기** 상태가 됩니다.

```jsx
// App.js
import { Suspense, startTransition, useState } from 'react';
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
  return <Layout>{content}</Layout>;
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}
```

```jsx
// Layout.js
export default function Layout({ children }) {
  return (
    <div className='layout'>
      <section className='header'>Music Browser</section>
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

- 트랜지션은 모든 콘텐츠가 로드될 때까지 기다리지 않습니다.
- 이미 보여지는 콘텐츠가 숨겨지지 않도록 합니다.
- 예를 들어 `Layout`이 이미 보여지고 있다면, 로딩 스피너 뒤에 숨기는 것은 좋지 않습니다.
- 그러나 `Album` 주위의 중첩된 `<Suspense>`는 새로운 것이므로 트랜지션이 기다리지 않습니다.

<br>

### 참고

- `<Suspense>`를 지원하는 라우터는 기본적으로 네비게이션 업데이트를 트랜지션으로 래핑합니다.

<br>

### 트랜지션이 발생하고 있음을 나타냅니다.

- 위 예시는 버튼을 클릭해도 네비게이션이 진행 중이라는 시각적 표시가 없습니다.
- 안내문을 추가하려면 `startTransition`을 `useTransition`으로 변경합니다.
- 아래 예시는 트랜지션이 진행되는 동안 헤더 스타일을 변경하는 데 사용됩니다.

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

### 네비게이션에서 `<Suspense>` 재설정하기

- 트랜지션하는 동안 `React`는 이미 보여지는 콘텐츠를 숨기지 않습니다.
- 하지만 다른 매개변수가 있는 라우트로 이동하는 경우 `React`에게 다른 콘텐츠라고 표시해야 하는 경우가 존재합니다.
- 이를 `key`로 표현합니다.

```jsx
<ProfilePage key={queryParams.id} />
```

<br>

- 프로필 페이지에서 다른 페이지로 이동 중인데 무언가 일시 중단되었다고 가정해 보겠습니다.
- 해당 업데이트가 트랜지션으로 래핑되어 있으면 이미 보여지는 콘텐츠에 대한 `fallback`이 트리거되지 않습니다.

<br>

- 하지만 2개의 서로 다른 프로필로 이동한다고 가정해 보겠습니다.
- 이 경우 `fallback`을 표시하는 것이 좋습니다.
- 예를 들어 한 사용자의 타임라인은 다른 사용자의 타임라인과 다른 콘텐츠입니다.
- `key`를 지정하면 `React`가 각 프로필을 서로 다른 컴포넌트로 취급하고 네비게이션 중에 `<Suspense>`를 재설정할 수 있습니다.
- `<Suspense>`를 통합하는 라우터는 이 작업을 자동으로 수행합니다.

<br>

### 서버 오류 및 클라이언트 전용 콘텐츠에 대한 `fallback` 제공

- [스트리밍 `SSR` API](https://react.dev/reference/react-dom/server) 중 하나(또는 프레임워크)를 사용하는 경우 `React`는 서버에서 발생하는 오류를 처리하기 위해 `<Suspense>`를 사용합니다.
- 컴포넌트가 서버에서 에러를 발생시키더라도 `React`는 `SSR`을 중단하지 않습니다.
- 가장 가까운 `<Suspense>` 컴포넌트를 찾아서 생성된 서버 HTML에 `fallback`을 포함시킵니다.
- 유저는 처음에 스피너를 보게 됩니다.

<br>

- 클라이언트에서 `React`는 동일한 컴포넌트의 리렌더링을 시도합니다.
- 클라이언트에서도 에러가 발생하면 `React`는 에러를 던지고 가장 가까운 **에러 경계**를 표시합니다.
- 클라이언트에서 에러가 발생하지 않는다면 콘텐츠가 성공적으로 렌더링되었으므로 `React`는 에러를 표시하지 않습니다.

<br>

- 이를 활용하여 일부 컴포넌트에 `SSR`을 사용하지 않도록 선택할 수 있습니다.
- 이렇게 하려면 서버 환경에서 오류를 발생시킨 다음 해당 컴포넌트를 `<Suspense>` 경계로 래핑하여 `fallback`으로 대체합니다.

<br>

```jsx
<Suspense fallback={<Loading />}>
  <Chat />
</Suspense>;

function Chat() {
  if (typeof window === 'undefined') {
    throw Error('Chat should only render on the client.');
  }
  // ...
}
```

<br>

- 서버 HTML에 로딩 화면이 포함됩니다.
- 로딩 화면은 다시 클라이언트의 `Chat` 컴포넌트로 변경됩니다.

<br>

## 트러블슈팅

### 업데이트 중에 UI가 `fallback`으로 대체되는 것을 방지하려면 어떻게 해야 하나요?

- UI를 `fallback`으로 대체하면 사용자 환경이 불안정해집니다.
- 위 상황은 업데이트로 인해 컴포넌트가 일시 중단었고, 가장 가까운 `<Suspense>` 경계에 이미 콘텐츠가 표시되고 있을 때 발생합니다.

<br>

- 이를 방지하려면 **`startTransition`을 사용하여 업데이트를 급하지 않은 것으로 표시**합니다.
- 트랜지션이 진행되는 동안 `React`는 `fallback`이 나타나지 않도록 데이터가 로드될 때까지 기다립니다.

```jsx
function handleNextPageClick() {
  // 업데이트가 일시 중단되면 이미 보여지는 콘텐츠를 숨기지 않습니다.
  startTransition(() => {
    setCurrentPage(currentPage + 1);
  });
}
```

<br>

- 이렇게 하면 기존 콘텐츠가 숨겨지지 않습니다.
- 새로 렌더링된 `<Suspense>` 경계는 즉시 `fallback`을 표시하여 UI를 가리지 않고 콘텐츠가 로드된후 보여줍니다.

<br>

- **`React`는 긴급하지 않은 업데이트 중에만 `fallback`을 방지합니다.**
- 긴급한 업데이트라면 렌더링을 지연시키지 않습니다.
- `startTransition` 또는 `useDeferredValue`와 같은 API를 사용합니다.

<br>

- `<Suspense>`와 통합된 라우터를 사용하는 경우 라우터는 업데이트를 `startTransition`에 자동으로 래핑합니다.
