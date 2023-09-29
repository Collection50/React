# lazy

- `lazy`를 사용하면 컴포넌트 코드가 처음 렌더링되기 전까지 데이터 로드를 지연시킬 수 있습니다.

```jsx
const SomeComponent = lazy(load);
```

<br>

## 레퍼런스

### `lazy(load)`

- 컴포넌트 외부에서 `lazy`를 호출하여 **지연**되는 `React` 컴포넌트를 선언합니다.

```jsx
import { lazy } from 'react';

const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
```

<br>

### 매개변수

- `load` 함수
  - `Promise` 또는 다른 `thenable`(`then` 메서드가 있는 `Promise`와 유사한 객체)을 반환하는 함수입니다.
  - 컴포넌트를 처음 렌더링하기 전까지 `load`를 호출하지 않습니다.
  - `React`가 처음 `load`를 호출한 후, 리졸브될 때까지 기다립니다.
  - 이후 리졸브된 값의 `.default`를 컴포넌트로 렌더링합니다.
  - 반환된 `Promise`와 `Promise`의 리졸브 값은 모두 캐시되므로 `React`는 `load`를 2번 이상 호출하지 않습니다.
  - `Promise`가 거부(`reject`)되면 `React`는 가장 가까운 에러 바운더리에 거부 이유를 `throw`합니다.

<br>

### 반환값

- `lazy`는 렌더링 가능한 `React` 컴포넌트를 반환합니다.
- `lazy` 컴포넌트의 코드가 로딩되는 동안 렌더링을 시도하면 **일시 중단**됩니다.
- 로딩 표시기를 표시하려면 `<Suspense>`를 사용합니다.

<br>

## `load` 함수

### 매개변수

- `load` 함수는 매개변수를 사용하지 않습니다.

<br>

### 반환값

- `Promise` 또는 다른 `thenable`(`then` 메서드가 있는 `Promise`와 유사한 객체)을 반환합니다.
- 결국 함수, `memo`, `forwardRef`와 같이 `.default` 프로퍼티가 존재하는 컴포넌트로 리졸브합니다.

<br>

## 사용법

### `Suspense`가 있는 `lazy` 로딩 컴포넌트

- 일반적으로 정적 `import`를 사용하여 컴포넌트를 `import`합니다.

```jsx
import MarkdownPreview from './MarkdownPreview.js';
```

<br>

- 컴포넌트가 처음 렌더링되기 전까지 로딩을 지연하려면 `import`를 아래와 같이 사용합니다.

```jsx
import { lazy } from 'react';

const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
```

<br>

- 동적 `import`를 사용하므로 번들러나 프레임워크의 도움이 필요할 수 있습니다.
- `import`하려는 `lazy` 컴포넌트는 `default export`로 설정되어야 합니다.

<br>

- 요청에 의해 컴포넌트가 로드되므로 로드되는 동안 표시할 내용도 지정합니다.
- `lazy` 컴포넌트나 상위 컴포넌트를 `<Suspense>` 경계로 래핑합니다.

```jsx
<Suspense fallback={<Loading />}>
  <h2>Preview</h2>
  <MarkdownPreview />
</Suspense>
```

<br>

- 아래 예제는 렌더링되기 전까지 `MarkdownPreview`에 대한 코드가 로드되지 않습니다.
- `MarkdownPreview`가 아직 로드되지 않은 경우 `Loading`이 표시됩니다.

```jsx
// App.js
import { useState, Suspense, lazy } from 'react';
import Loading from './Loading.js';

const MarkdownPreview = lazy(() =>
  delayForDemo(import('./MarkdownPreview.js'))
);

export default function MarkdownEditor() {
  const [showPreview, setShowPreview] = useState(false);
  const [markdown, setMarkdown] = useState('Hello, **world**!');
  return (
    <>
      <textarea
        value={markdown}
        onChange={(e) => setMarkdown(e.target.value)}
      />
      <label>
        <input
          type='checkbox'
          checked={showPreview}
          onChange={(e) => setShowPreview(e.target.checked)}
        />
        Show preview
      </label>
      <hr />
      {showPreview && (
        <Suspense fallback={<Loading />}>
          <h2>Preview</h2>
          <MarkdownPreview markdown={markdown} />
        </Suspense>
      )}
    </>
  );
}

// 로딩 상태를 볼 수 있도록 고정 지연 시간을 추가합니다
function delayForDemo(promise) {
  return new Promise((resolve) => {
    setTimeout(resolve, 2000);
  }).then(() => promise);
}
```

```jsx
// Loading.js
export default function Loading() {
  return (
    <p>
      <i>Loading...</i>
    </p>
  );
}
```

```jsx
// MarkdownPreview.js
import { Remarkable } from 'remarkable';

const md = new Remarkable();

export default function MarkdownPreview({ markdown }) {
  return (
    <div
      className='content'
      dangerouslySetInnerHTML={{ __html: md.render(markdown) }}
    />
  );
}
```

<br>

## 트러블슈팅

### `lazy` 컴포넌트의 상태가 예기치 않게 초기화됩니다.

- 다른 컴포넌트 **내부에** `lazy` 컴포넌트를 선언하지 않습니다.

```jsx
import { lazy } from 'react';

function Editor() {
  // 🔴 Bad: 리렌더링 시 모든 상태가 초기화됩니다.
  const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
  // ...
}
```

<br>

- 항상 **모듈**의 최상단에서 선언합니다.

```jsx
import { lazy } from 'react';

// ✅ Good: 컴포넌트 외부에 지연 컴포넌트를 선언합니다.
const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));

function Editor() {
  // ...
}
```
