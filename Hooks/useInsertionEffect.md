# useInsertionEffect

### 주의

- `useInsertionEffect`는 CSS-in-JS 라이브러리 사용자를 위한 `hook`입니다.
- CSS-in-JS 라이브러리를 사용하지만 스타일을 삽입할 위치가 필요하지 않다면 `useEffect` 또는 `useLayoutEffect`를 사용합니다.

<br>

- `useInsertionEffect`를 사용하면 `useLayoutEffect`가 실행되기 전에 요소를 DOM에 삽입할 수 있습니다.

<br>

```jsx
useInsertionEffect(setup, dependencies?)
```

<br>

## 레퍼런스

### `useInsertionEffect(setup, dependencies?)`

- `useLayoutEffect`가 실행되기 전에 스타일을 삽입하려면 `useInsertionEffect`를 사용합니다.

```jsx
import { useInsertionEffect } from 'react';

// CSS-in-JS 라이브러리 내부
function useCSS(rule) {
  useInsertionEffect(() => {
    // ... 여기에 <style> 태그 삽입 ...
  });
  return rule;
}
```

<br>

### 매개변수

- `setup`
  - `useInsertionEffect`의 로직이 포함된 함수입니다.
  - `setup` 함수는 선택적으로 초기화 함수를 반환합니다.
  - 컴포넌트가 DOM에 추가되었지만 `useInsertionEffect`가 실행되기 전에 `React`는 `setup` 함수를 실행합니다.
  - 리렌렌더링될 때마다 이전 값으로 초기화 함수를 실행한 다음 새 값으로 `setup` 함수를 실행합니다.
  - 컴포넌트가 DOM에서 제거되면 `React`는 초기화 함수를 실행합니다.
- `dependencies`(옵션)
- `setup` 함수에서 참조된 모든 반응형 값의 목록입니다.
  - 반응형 값에는 `prop`, state, 컴포넌트 내부에서 선언된 모든 변수와 함수가 포함됩니다.
  - 린터 사용 시 모든 반응형 값이 종속성에 올바르게 지정되었는지 확인합니다.
  - 종속성 목록에는 일정한 수의 항목이 있어야 하며 `[dep1, dep2, dep3]`와 같이 인라인으로 작성해야 합니다.
  - `Object.is` 메서드를 사용하여 비교합니다.
  - 종속성 배열이 없다면 컴포넌트를 다시 렌더링할 때마다 `useInsertiocEffect`가 다시 실행됩니다.

<br>

### 반환값

- `useInsertiocEffect`는 `undefined`를 반환합니다.

<br>

### 주의사항

- `useInsertiocEffect`는 `CSR`에서만 실행됩니다.
- `SSR`에서는 실행되지 않습니다.
- `useInsertionEffect` 내부에서는 상태를 업데이트할 수 없습니다.
- `useInsertionEffect`가 실행될 때까지 `ref`는 DOM 노드에 할당되지 않습니다.
- `useInsertionEffect`는 DOM이 업데이트되기 전이나 후에 실행될 수 있습니다.
- 특정 시점에 업데이트되는 DOM에 의존해서는 안 됩니다.
- 초기화 함수를 실행한 다음 `setup`을 실행하는 다른 유형의 `Effect` 함수와 달리 `useInsertionEffect`는 한 번에 하나의 컴포넌트에 대해 초기화 함수와 `setup`을 모두 실행합니다.
- 초기화 함수와 `setup` 기능이 **인터리빙**됩니다.

<br>

## 사용법

### CSS-in-JS 라이브러리에서 동적 스타일 삽입하기

- 예전에는 CSS를 사용해 컴포넌트의 스타일을 지정했습니다.

```jsx
// JS 파일에 추가합니다.
<button className="success" />

// CSS 파일
.success { color: green; }
```

<br>

- 일부 사용자는 CSS 대신 자바스크립트 코드에서 직접 스타일을 작성하는 것을 선호합니다.
- 이 경우 일반적으로 CSS-in-JS 라이브러리를 사용합니다.
- CSS-in-JS에는 3가지 일반적인 접근 방식이 있습니다.

<br>

1. 컴파일러를 사용하여 CSS 파일을 정적으로 추출하기
2. 인라인 스타일(`<div style={{ opacity: 1 }}>`)
3. `<style>` 태그의 런타임 삽입

<br>

- CSS-in-JS를 사용하는 경우 1, 2번 접근 방식을 조합하여 사용하는 것이 좋습니다.
- **런타임 `<style>` 태그 삽입은 아래 2가지 이유로 권장하지 않습니다.**

<br>

1. 런타임 인젝션은 브라우저가 스타일을 훨씬 더 자주 다시 계산하도록 합니다.
2. `React` 생명 주기에서 런타임 인젝션이 잘못된 시점에 발생하면 속도가 매우 느려질 수 있습니다.

<br>

- 1번째 문제는 해결할 수 없지만 `useInsertionEffect`를 사용하면 2번째 문제를 해결할 수 있습니다.

<br>

- `useLayoutEffect`가 실행되기 전에 스타일을 삽입하려면 `useInsertionEffect`를 사용합니다.

```jsx
// CSS-in-JS 라이브러리 내부
let isInserted = new Set();
function useCSS(rule) {
  useInsertionEffect(() => {
    // 앞서 설명했듯이 <style> 태그의 런타임 삽입은 권장하지 않습니다.
    // 하지만 꼭 삽입해야 한다면 useInsertionEffect에서 하는 것이 중요합니다.
    if (!isInserted.has(rule)) {
      isInserted.add(rule);
      document.head.appendChild(getStyleForRule(rule));
    }
  });
  return rule;
}

function Button() {
  const className = useCSS('...');
  return <div className={className} />;
}
```

<br>

- `useEffect`와 마찬가지로 `useInsertionEffect`는 서버에서 실행되지 않습니다.
- 서버에서 어떤 CSS 규칙이 사용되었는지 수집해야 하는 경우 렌더링 중에 이뤄집니다.

```jsx
let collectedRulesSet = new Set();

function useCSS(rule) {
  if (typeof window === 'undefined') {
    collectedRulesSet.add(rule);
  }
  useInsertionEffect(() => {
    // ...
  });
  return rule;
}
```

<br>

## Deep Dive - 렌더링 중에 스타일을 주입하거나 `useLayoutEffect`하는 것보다 어떻게 더 나은가요?

- 렌더링 중에 스타일을 삽입하고 `React`가 차단하지 않는 업데이트를 처리하는 경우 브라우저는 컴포넌트 트리를 렌더링하는 동안 매 프레임마다 스타일을 다시 계산하므로 속도가 매우 느려질 수 있습니다.

<br>

- `useInsertionEffect`는 `useLayoutEffect` 또는 `useEffect`에서 스타일을 삽입하는 것보다 낫습니다.
  - 컴포넌트에서 다른 `Effect` 함수가 실행될 때까지 `<style>` 태그가 삽입되었는지 확인하기 때문입니다.
- 그렇지 않으면 이전 스타일로 인해 일반 `Effects` 함수의 레이아웃 계산이 잘못될 수 있습니다.
