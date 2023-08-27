# useDeferredValue

- `useDeferredValue`는 UI의 일부 업데이트를 지연하는 `React` `hook`입니다.

```jsx
const deferredValue = useDeferredValue(value);
```

<br>

## 레퍼런스

### `useDeferredValue(value)`

- 컴포넌트의 최상단에서 `useDeferredValue`를 사용하여 지연된 값을 가져옵니다.

```jsx
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  // ...
}
```

<br>

### 매개변수

- `value`
  - 지연하려는 값입니다.
  - 모든 유형의 값을 가질 수 있습니다.

<br>

### 반환값

- 첫 렌더링 중에 반환되는 값은 처음 전달한 값과 동일합니다.
- 업데이트하는 동안 `React`는 이전 값으로 다시 렌더링을 시도한 다음(이전 값 반환) 백그라운드에서 새 값으로 리렌더링합니다.(업데이트된 값 반환)

<br>

### 주의사항

- `useDeferredValue`에 전달하는 값은 문자열, 숫자와 같은 원시값이거나 렌더링 외부에서 생성된 객체여야 합니다.
- 렌더링 중에 새 객체를 생성하고 즉시 `useDeferredValue`에 전달하면 불필요한 백그라운드 리렌더링이 발생합니다.

<br>

- `useDeferredValue`가 다른 값을 받으면 새 값으로 백그라운드에서 리렌더링되도록 예약합니다.
- 백그라운드 리렌더링은 멈출 수 있습니다.
- `value`에 대한 또 다른 변화가 있으면 `React`는 백그라운드 리렌더링을 처음부터 다시 시작합니다.
- 예를 들어 차트가 리렌더링되는 속도보다 빠르게 타이핑하는 경우 타이핑을 멈춘 후에 차트가 리렌더링됩니다.

<br>

- `useDeferredValue`는 `<Suspense>`와 통합됩니다.
- 백그라운드 업데이트(`value` 변경)로 인해 UI가 일시 중단되면 유저는 `fallback`을 보는것이 아닙니다.
- 데이터가 로드될 때까지 이전 값이 표시됩니다.

<br>

- `useDeferredValue` 자체적으로 추가적인 네트워크 요청을 막지는 않습니다.

<br>

- `useDeferredValue` 자체의 고정 지연 시간(예: `1000ms`)은 없습니다.
- `React`가 이전의 리렌더링을 완료하자마자 즉시 새로운 지연된 값을 사용하여 백그라운드 리렌더링 작업을 시작합니다.
- 타이핑과 같은 **이벤트로 인한 모든 업데이트**는 백그라운드 리렌더링을 중단하고 더 **높은 우선순위**를 부여받습니다.

<br>

- `useDeferredValue`로 인한 백그라운드 리렌더링은 화면에 적용될 때까지 `useEffect`를 실행하지 않습니다.
- 백그라운드 리렌더링이 중단되면 데이터가 로드되고 UI가 업데이트된 후에 `useEffect`가 실행됩니다.

<br>

## 사용법

### 새 콘텐츠가 로드되는 동안 이전 콘텐츠 표시하기

- 컴포넌트 최상단에서 `useDeferredValue`를 호출하여 UI의 일부 업데이트를 지연합니다.

```jsx
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  // ...
}
```

<br>

- 첫 렌더링의 **지연된 값**은 `useDeferredValue`의 **초기 값**입니다.
- 업데이트하는 동안 지연된 값은 최신 값보다 **뒤쳐져**있는 상태입니다.
- `React`는 지연된 값을 먼저 업데이트하지 않고 리렌더링한 다음, 백그라운드에서 새로 받은 값으로 리렌더링합니다.

<br>

### 참고

- 아래 예시는 `Suspense`를 사용 가능한 경우에 대해 다룹니다.

<br>

- [`Relay`](https://relay.dev/docs/guided-tour/rendering/loading-states/), [`Next.js`](https://nextjs.org/docs/app/building-your-application/rendering)와 같이 `Suspense`를 지원하는 프레임워크를 사용한 데이터 페치
- `lazy`를 사용한 컴포넌트 지연 로딩

<br>

- 검색 결과를 페치하는 동안 `SearchResults` 컴포넌트가 일시 중단됩니다.
- `"a"`를 입력하고 결과를 기다린 다음 `"ab"`로 편집해 보세요.
- `"a"`에 대한 결과가 로딩 `fallback`으로 대체됩니다.

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

- 일반적인 UI 패턴은 업데이트를 지연하고 새 화면이 준비될 때까지 이전 화면을 계속 표시하는 것입니다.
- `useDeferredValue`를 호출하여 쿼리의 지연된 버전을 전달합니다.

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

- 아래 예에서 `"a"`를 입력하고 결과가 로드될 때까지 기다린 다음 입력을 `"ab"`로 편집합니다.
- 이제 새 결과가 로드될 때까지 일시 중단 `fallback` 대신 이전 결과 목록이 표시됩니다.

<br>

## Deep Dive - 지연되는 값은 내부적으로 어떻게 작동하나요?

- 2단계로 나뉜다고 생각하면 됩니다.

<br>

1.  `React`는 새 `query`(`"ab"`)를 사용하지만 이전 `deferredQuery`(`"a"`)를 사용하여 리렌더링합니다.
    - 결과 목록에 전달하는 `deferredQuery` 값은 **지연**됩니다.
    - 즉 `query` 값보다 **뒤쳐져** 있습니다.
2.  백그라운드에서 `React`는 `query`와 `deferredQuery`를 모두 `"ab"`로 업데이트한 상태로 리렌더링을 시도합니다.
    - 리렌더링이 완료되면 `React`는 이를 화면에 표시합니다.
    - 중단되는 경우(`"ab"`에 대한 결과가 아직 로드되지 않은 경우) `React`는 렌더링 시도를 포기하고 데이터가 로드된 후 리렌더링을 다시 시도합니다.
    - 사용자는 데이터가 준비될 때까지 지연된 값(이전 값)을 계속 보게 됩니다.

<br>

- 지연된 "백그라운드" 렌더링은 중단할 수 있습니다.
- 예를 들어 `input` 태그에 타이핑하면 `React`는 해당 입력을 버리고 새 값으로 다시 시작합니다.
- `React`는 항상 최신 값을 사용합니다.

<br>

- 타이핑될 때마다 네트워크에 데이터를 요청합니다.
- 지연되는 것은 네트워크 요청 자체이 아니라 새로운 화면입니다.
- 계속 입력하더라도 각 **키 입력에 대한 응답은 캐시**되므로 백스페이스를 누르면 페치 요청을 다시 보내지 않습니다.

<br>

## 이전 콘텐츠를 표시하는 상태를 알립니다.

- 위 예시는 로딩 중이라는 표시가 없습니다.
- 결과를 로드하는 데 오래 걸리는 경우 혼란을 줄 수 있습니다.
- 결과 목록이 최신 쿼리와 일치하지 않는다는 것을 확실히 알리기 위해 시각적 표시를 추가합니다.

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

- 새 결과 목록이 로드될 때까지 이전 결과 목록이 약간 어두워집니다.
- `CSS` `transition`을 추가하여 점진적인 특성을 추가하는 방법도 존재합니다.

<br>

## UI 일부에 대한 리렌더링 지연하기

- 성능 최적화의 일환으로 `useDeferredValue`를 적용할 수 있습니다.
- UI의 일부가 리렌더링되는 속도가 느리고, 최적화할 쉬운 방법이 없으며, 나머지 UI를 차단하지 않는 경우에 유용합니다.
- 타이핑마다 리렌더링되는 텍스트 필드와 컴포넌트(예: 차트 또는 크기가 큰 배열)가 있다고 가정해 보겠습니다.

```jsx
function App() {
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <SlowList text={text} />
    </>
  );
}
```

<br>

- `prop`이 동일한 경우 리렌더링을 건너뛰도록 `SlowList`를 최적화합니다.
- `memo`로 래핑합니다.

```jsx
const SlowList = memo(function SlowList({ text }) {
  // ...
});
```

<br>

- 하지만 `SlowList` `prop`이 이전 렌더링 때와 **동일**한 경우에만 유용합니다.
- 문제는 `prop`이 다르고, 다른 시각적 결과를 렌더링할 때 느리다는 것입니다.

<br>

- 구체적으로 주요 성능 문제는 `input` 태그에 타이핑할 때마다 `SlowList`가 새로운 `prop`을 수신하고 전체 트리를 리렌더링하면 타이핑이 끊기는 느낌이 발생합니다.
- 이 경우 `useDeferredValue`를 사용하면 `input` 업데이트에 우선순위를 부여할 수 있습니다. (결과 목록 업데이트보다 `input` 업데이트가 더 빨리 업데이트되도록)

```jsx
function App() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <SlowList text={deferredText} />
    </>
  );
}
```

<br>

- `SlowList`의 리렌더링 속도가 빨라지지는 않습니다.
- 하지만 `input` 태그의 타이핑이 느려지지 않도록 `<SlowList/>`의 리렌더링 우선순위를 낮추는 것입니다.
- `<SlowList/>`는 타이핑보다 **지연**되었다가 **원상복구**됩니다.
- `React`는 가능한 빨리 결과 목록을 업데이트하며 유저의 입력을 차단하지는 않습니다.

<br>

### 주의

- 최적화를 위해서는 `SlowList`를 `memo`로 래핑해야합니다.
- `text`가 변경될 때마다 `React`가 상위 컴포넌트를 빠르게 리렌더링할 수 있어야 하기 때문입니다.
- 리렌더링하는 동안 `deferredText`는 이전 값을 가지므로 `SlowList`는 리렌더링을 건너뜁니다. (`prop`은 변경되지 않음)
- `memo`가 없다면 리렌더링해야 하므로 최적화에 의미가 없습니다.

<br>

## Deep Dive - 값의 지연은 디바운스 및 스로틀링과 어떻게 다른가요?

- 아래 2가지 최적화 기술이 있습니다.

<br>

1. **디바운싱**은 입력을 멈출 때까지(예: 1초 동안) 기다렸다가 업데이트하는 것입니다.
2. **스로틀링**은 목록을 가끔씩(예: 최대 1초에 한 번) 업데이트하는 것입니다.

<br>

- `React` 생태계에 최적화되어있고 디바이스에 맞게 조정되므로 렌더링 최적화에는 `useDeferredValue`가 더 적합합니다.

<br>

- 디바운싱이나 스로틀링과 달리 고정 지연 시간(`1000ms`)이 필요 없습니다.
- 디바이스가 빠른 경우(예: 고성능 노트북) 지연된 리렌더링은 즉시 발생하며 눈에 띄지 않습니다.
- 디바이스가 느린 경우 디바이스 속도에 비례하여 **지연**됩니다.

<br>

- 디바운싱이나 스로틀링과 달리 `useDeferredValue`로 지연된 리렌더링은 중단할 수 있습니다.
- `React`가 크기가 큰 결과(배열)를 리렌더링하는 도중에 다른 키 입력을 하면 `React`는 해당 리렌더링을 중단하고 키 입력을 처리한 후 백그라운드에서 리렌더링합니다.
- 반면 디바운싱과 스로틀링은 키 입력을 차단하는 동작 자체를 지연하므로 여전히 불안정합니다.

<br>

- 최적화 작업이 렌더링 중에 발생하지 않는 경우 디바운싱과 스로틀링이 유용합니다.
- 예를 들어 디바운싱과 스로틀링을 사용하면 네트워크 요청을 더 적게 처리할 수 있습니다.
- 또한 디바운싱과 스로틀링은 `useDeferredValue`를 함께 사용할 수 있습니다.
