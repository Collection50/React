# startTransition

- `startTransition`을 사용하면 UI를 차단하지 않고 상태를 업데이트할 수 있습니다.

```jsx
startTransition(scope);
```

<br>

## 레퍼런스

### `startTransition(scope)`

- `startTransition` 함수를 사용하면 **상태 업데이트**를 트랜지션으로 표시할 수 있습니다.

```jsx
import { startTransition } from 'react';

function TabContainer() {
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
  - 이 함수는 **멈춰지지 않고** **원치 않는 로딩창을 표시하지 않습니다.** (`fallback` 표시 X)

<br>

### 반환값

- `startTransition`은 아무것도 반환하지 않습니다.

<br>

### 주의사항

- `startTransition`은 트랜지션의 보류 여부를 추적하는 방법을 제공하지 않습니다. (`pending` 없음)
- 트랜지션이 진행되는 동안 보류 여부를 표시하려면 [`useTransition`](https://github.com/Collection50/React/blob/master/Hooks/useTransition.md)을 사용합니다.

<br>

- 상태의 **`set` 함수를 사용할 수 있는 경우**에만 업데이트를 트랜지션으로 래핑할 수 있습니다.
- **`prop`이나 커스텀 `hook`에 대한 응답**으로 트랜지션을 시작하려면 [`useDeferredValue`](https://github.com/Collection50/React/blob/master/Hooks/useDeferredValue.md)를 사용합니다.

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

### 상태 업데이트를 트랜지션으로 표시하기

- 상태 업데이트를 `startTransition`으로 래핑하여 **트랜지션**으로 표시합니다.

```jsx
import { startTransition } from 'react';

function TabContainer() {
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

- 트랜지션을 사용하면 느린 디바이스에서도 UI 업데이트의 반응성을 유지할 수 있습니다.

<br>

- 트랜지션을 사용하면 리렌더링되는 동안에도 UI의 반응성을 유지할 수 있습니다.
- 예를 들어 사용자가 탭을 클릭했다가 마음이 바뀌어 다른 탭을 클릭하면 리렌더링이 완료될 때까지 기다릴 필요 없이 다른 탭을 클릭할 수 있습니다.

<br>

## 참고

- `startTransition`은 `useTransition`과 매우 유사하지만 트랜지션이 진행 중인지 여부를 추적하는 `isPending`을 제공하지 않는다는 점이 다릅니다.
- `useTransition`을 사용할 수 없을 때 `startTransition`을 사용합니다.
- 예를 들어 `startTransition`은 데이터 라이브러리와 같은 외부 컴포넌트에서 작동합니다.
