# useLayoutEffect

## 주의

- `useLayoutEffect`는 성능을 저하시킬 수 있습니다.
- 가급적이면 `useEffect`를 사용합니다.

<br>

- `useLayoutEffect`는 브라우저가 화면을 `paint` 하기 전에 실행되는 `useEffect`입니다.

```jsx
useLayoutEffect(setup, dependencies?)
```

<br>

## 레퍼런스

### `useLayoutEffect(setup, dependencies?)`

- 브라우저가 화면을 `paint` 하기 전에 레이아웃을 측정하려면 `useLayoutEffect`를 사용합니다.

```js
import { useState, useRef, useLayoutEffect } from 'react';

function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
  }, []);
  // ...
}
```

<br>

### 매개변수

- `setup`
  - `useLayoutEffect`의 로직이 포함된 함수입니다.
  - 선택적으로 초기화 함수를 반환합니다.
  - 컴포넌트가 **DOM에 추가되기 전**에 `setup` 함수를 실행합니다.
  - 종속성 변경으로 인해 리렌더링될 때마다 초기화 함수(존재하는 경우)를 실행한 후 변경된 종속성을 활용하여 `setup` 함수를 실행합니다.
  - 컴포넌트가 **DOM에서 제거되기 전**에 초기화 함수를 실행합니다.
- `dependencies`
  - `setup` 코드에서 참조하는 모든 반응형 값의 목록입니다.
  - 반응형 값에는 `prop`, state, 컴포넌트 내부에서 선언된 모든 변수와 함수가 포함됩니다.
  - 린터 사용 시 모든 반응형 값이 종속성에 올바르게 지정되었는지 확인합니다.
  - 종속성 목록에는 일정한 수의 항목이 있어야 하며 `[dep1, dep2, dep3]`와 같이 인라인으로 작성해야 합니다.
  - `Object.is` 메서드를 사용하여 비교합니다.
  - 종속성 배열이 없다면 컴포넌트를 다시 렌더링할 때마다 `useLayoutEffect`가 다시 실행됩니다.

<br>

### 반환값

- `useLayoutEffect`는 `undefined`를 반환합니다.

<br>

### 주의사항

- `useLayoutEffect`는 `hook`이므로 **컴포넌트 최상단** 혹은 커스텀 `hook`에서만 호출합니다.
  - 반복문 or 조건문 내부에서는 호출할 수 없습니다.
  - 필요한 경우 새 컴포넌트를 추출하고 `useLayoutEffect`를 내부로 옮깁니다.
- `Strict Mode`를 사용하면 초기 `setup` 전에 **`setup` + 초기화 사이클을 한 번 더 실행**합니다. (개발 모드에서만)
  - 초기화 로직이 `setup` 로직을 "미러링"하고 `setup`이 수행 중인 모든 작업을 중지하거나 취소하는지 확인하기 위한 스트레스 테스트입니다.
  - 문제가 발생하면 초기화 함수를 구현합니다.
- 종속성이 컴포넌트 내부에 정의된 객체 or 함수인 경우 **`useLayoutEffect`가 필요 이상으로 자주 다시 실행될 위험**이 있습니다.
  - 문제를 해결하려면 불필요한 객체 및 함수 종속성을 제거합니다.
  - 상태 업데이트와 비반응형 로직을 `useLayoutEffect` 외부로 추출합니다.
- **`useEffect`는 `CSR`에서만 실행됩니다.**
  - `SSR`에서는 실행되지 않습니다.
- `useLayoutEffect` 내부의 코드와 예약된 모든 상태 업데이트는 브라우저가 화면을 `paint`하는 것을 막습니다.

<br>

- 과도하게 사용하면 성능 저하가 발생하므로 가급적 필요한 경우에만 사용합니다.

<br>

## 사용법

### 브라우저가 화면을 다시 `paint`하기 전에 레이아웃 측정하기

- 대부분의 컴포넌트는 레이아웃을 알 필요가 없습니다.
- 일부 `JSX`만 반환하기 때문입니다.
- 이후 브라우저는 레이아웃(위치 및 크기)을 계산하고 화면을 다시 `paint`합니다.

<br>

- `hover` 시 툴팁이 표시되는 상황을 가정헤보겠습니다.
- 공간이 충분하면 툴팁이 요소 위에 표시되어야 하지만 공간이 충분하지 않으면 아래에 표시되어야 합니다.
- 툴팁을 올바른 위치에 렌더링하려면 툴팁의 높이(상단에 맞는지 여부)를 알아야 합니다.

<br>

- 이렇게 하려면 2번의 전달을 통해 렌더링해야 합니다.

1. 툴팁을 원하는 위치에 렌더링합니다(위치가 잘못된 경우에도)
2. 높이를 측정하고 툴팁을 배치할 위치를 결정합니다.
3. 올바른 위치에 툴팁을 리렌더링합니다.

<br>

- **모든 작업은 브라우저가 화면을 `paint`하기 전에 이루어져야 합니다.**
- 사용자에게 툴팁이 움직이는 것을 보여주고 싶지 않기 때문입니다.
- 브라우저가 화면을 다시 `paint`하기 전에 `useLayoutEffect`를 호출하여 레이아웃 측정을 수행합니다.

```js
function Tooltip() {
  const ref = useRef(null);
  // 아직 실제 높이는 알 수 없습니다.
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    // 이제 실제 높이를 알았으니 리렌더링합니다.
    setTooltipHeight(height);
  }, []);

  // ...아래 렌더링 로직에 tooltipHeight 사용...
}
```

<br>

- 단계별 작동 방식은 다음과 같습니다.

<br>

1. `Tooltip`의 첫 렌더링에서 `tooltipHeight = 0`으로 렌더링됩니다. (따라서 툴팁의 위치가 잘못될 수 있습니다)
2. `React`는 `Tooltip`을 DOM에 배치하고 `useLayoutEffect`에서 코드를 실행합니다.
3. `useLayoutEffect`는 툴팁 콘텐츠의 **높이를 측정**하고 즉시 리렌더링합니다.
4. `Tooltip`은 실제 `tooltipHeight`로 리렌더링됩니다. (따라서 툴팁의 위치가 올바르게 지정됩니다)
5. `React`가 DOM을 업데이트하면 툴팁이 최종적으로 표시됩니다.

<br>

- `Tooltip` 컴포넌트는 2번의 전달을 통해 렌더링해야 최종 결과를 확인할 수 있습니다.
- `useEffect` 대신 `useLayoutEffect`가 필요한 이유입니다.

<br>

## 트러블슈팅

### 오류가 발생합니다: "`useLayoutEffect`가 서버에서 아무것도 수행하지 않습니다."

- `useLayoutEffect`의 목적은 **컴포넌트 렌더링에 레이아웃 정보를 사용**하는 것입니다.

<br>

1. 초기 콘텐츠를 렌더링합니다.
2. 브라우저가 화면을 다시 `paint`하기 전에 레이아웃을 측정합니다.
3. 측정한 레이아웃 정보를 사용하여 최종 콘텐츠를 렌더링합니다.

<br>

- `SSR`은 초기 렌더링을 위해 서버의 HTML로 렌더링됩니다.
- 자바스크립트 코드가 로드되기 전에 초기 HTML을 표시할 수 있습니다.

<br>

- 문제는 서버에 레이아웃 정보가 없다는 것입니다.

<br>

- 위 예시에서 `Tooltip` 컴포넌트의 `useLayoutEffect`은 콘텐츠 높이에 따라 콘텐츠 위 또는 아래에 올바르게 배치할 수 있도록 합니다.
- 서버 HTML을 활용해 툴팁을 렌더링할 수 없습니다.
- 서버에는 아직 레이아웃이 없기 때문입니다!
- 따라서 `SSR`에서는 자바스크립트가 로드되고 실행된 후 클라이언트에서 툴팁의 위치가 **순간이동**됩니다.

<br>

- 레이아웃 정보에 의존하는 컴포넌트는 `SSR`을 사용할 필요가 없습니다.
- 렌더링 중에 툴팁을 표시하는 것은 의미가 없습니다.
- 클라이언트 상호작용에 의해 트리거되기 떄문입니다.

<br>

- 하지만 문제가 발생하는 경우 아래 옵션이 있습니다.

1. `useLayoutEffect`를 `useEffect`로 바꿉니다.
   - `paint`를 막지 않고 초기 렌더링 결과를 표시합니다. (`useEffect`가 실행되기 전에 원래 HTML이 표시되므로)
2. **`CSR`을 사용합니다.**
   - `SSR` 중에 가장 가까운 `<Suspense>` 영역까지 콘텐츠를 로딩 폴백(예: 스피너 또는 글리머)으로 대체합니다.
3. `hydration` 후에만 `useLayoutEffect`를 사용하여 컴포넌트를 렌더링할 수 있습니다.
   - `false`로 초기화된 `isMounted` 상태를 유지하고 `useLayoutEffect` 내에서 `true`로 재설정합니다.
   - 그러면 렌더링 컴포넌트를 `return isMounted ? <RealContent /> : <FallbackContent />`과 조건부로 반환합니다.
   - `hydration`되는 동안 `useLayoutEffect`를 호출해서는 안 되는 `FallbackContent`를 보게 됩니다.
   - 그러면 `React`는 클라이언트에서만 실행되고 `useLayoutEffect` 호출을 포함할 수 있는 `RealContent`로 대체합니다.
4. 레이아웃 측정이 아닌 다른 이유로 `useLayoutEffect`를 사용하는 경우 `SSR`을 지원하는 `useSyncExternalStore`를 사용하는 것이 좋습니다.
