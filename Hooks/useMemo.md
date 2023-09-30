# useMemo

- `useMemo`는 리렌더링 간 연산 결과를 캐시하는 `React` `hook`입니다.

```js
const cachedValue = useMemo(calculateValue, dependencies);
```

<br>

## 레퍼런스

### `useMemo(calculateValue, dependencies)`

- 컴포넌트 최상단에서 `useMemo`를 호출하여 리렌더링 간 연산을 캐시합니다.

```jsx
import { useMemo } from 'react';

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

<br>

### 매개변수

- `calculateValue`
  - 캐시하려는 값을 연산하는 **함수**입니다.
  - 순수 함수여야 하며 인수를 받지 않고 모든 타입의 값을 반환합니다.
  - `React`는 첫 렌더링에 함수를 호출합니다.
  - 이어지는 렌더링에서 `React`는 직전 렌더링의 `dependencies`가 변경되지 않은 경우 동일한 값을 다시 반환합니다.
  - `dependencies`가 변경되면 `calculateValue`를 다시 호출하고 결과를 반환한 후 캐시합니다.

<br>

- `dependencies`
  - `calculateValue` 코드 내에서 참조된 모든 반응형 값의 목록입니다.
  - 반응형 값에는 `prop`, state, 컴포넌트에서 선언된 모든 변수와 함수가 포함됩니다.
  - 린터 사용 시 모든 반응형 값이 종속성에 올바르게 지정되었는지 확인합니다.
  - 종속성 목록에는 일정한 수의 항목이 있어야 하며 `[dep1, dep2, dep3]`와 같이 인라인으로 작성해야 합니다.
  - `Object.is` 메서드를 사용하여 비교합니다.

<br>

### 반환값

- 첫 렌더링에서 `useMemo`는 인수 없이 `calculateValue`를 호출한 **결과**를 반환합니다.

<br>

- 이어지는 렌더링에서는 직전 렌더링에서 저장된 값을 반환하거나(종속성이 변경되지 않은 경우) `calculateValue`를 다시 호출하여 `calculateValue`가 반환한 결과를 반환합니다. (종속성이 변경된 경우)

<br>

### 주의사항

- `uesMemo`는 `hook`이므로 컴포넌트의 최상단 or 커스텀 `hook`에서만 호출할 수 있습니다.
  - 반복문, 조건문 내부에서 호출할 수 없습니다.
  - 필요하다면 새 컴포넌트를 추출하고 상태를 옮깁니다.

<br>

- `Strict Mode`에서는 **실수로 발생한 부수효과를 찾기 위해** `React`가 **함수를 두 번 호출**합니다.
  - 개발 모드에서만 이뤄지며 배포 환경에는 영향을 미치지 않습니다.
  - 함수가 순수하다면 영향을 미치지 않습니다.
  - 함수의 결과 중 1개는 무시됩니다.

<br>

- `React`는 **특별한 이유가 없는 한 캐시된 함수를 버리지 않습니다.**
  - 개발 모드에서 컴포넌트를 변경할 때 `React`는 캐시를 버립니다.
  - 개발 모드나 프로덕션 모드에서 컴포넌트가 첫 마운트 중에 일시 중단되면 `React`는 캐시를 버립니다.
  - 미래에 `React`는 캐시 버리기를 활용하는 더 많은 기능을 추가할 겁니다.
  - 예를 들어 `React`에 가상화된 목록에 대한 기본 지원이 추가되면 가상화된 테이블 뷰포트에서 스크롤되는 항목에 대한 캐시도 버리는 것이 합리적일 것입니다.
  - 성능 최적화를 위해 `uesMemo`를 사용하는 것이 좋습니다.
  - 그렇지 않다면 상태나 `ref`가 더 적합합니다.

<br>

### 참고

- 이와 같이 반환값을 캐싱하는 것을 **메모이제이션**라고도 하는데 이 `hook`을 `useMemo`라고 부르는 이유입니다.

<br>

## 사용법

## 고비용 연산 반복하지 않기

- 리렌더링 간 연산을 캐시하려면 컴포넌트 최상단에서 `useMemo`를 사용합니다.

```jsx
import { useMemo } from 'react';

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

<br>

- `useMemo`는 2가지 매개변수를 사용합니다.

<br>

1. `() =>`와 같이 인수 없이 연산하려는 값을 반환하는 **연산 함수**
2. 컴포넌트 내에서 연산에 사용되는 모든 값을 포함한 **종속성 목록**

<br>

- 첫 렌더링에서 `useMemo`의 반환값은 **연산(함수)** 을 호출한 결과입니다.

<br>

- 이어지는 렌더링에서 `React`는 종속성을 직전 렌더링과 비교합니다.
- 종속성이 변경되지 않은 경우 `useMemo`는 이전 값을 반환합니다.
- 종속성이 변경된 경우 `React`는 연산을 다시 실행하여 새 값을 반환합니다.

<br>

- `useMemo`는 종속성이 변경될 때까지 리렌더링 간 결과를 캐시합니다.

<br>

- 유용하게 사용되는 예시를 살펴보겠습니다.

<br>

- 기본적으로 `React`는 컴포넌트가 리렌더링할 때마다 컴포넌트 전체를 다시 실행합니다.
- 아래 `TodoList`가 상태를 업데이트하거나 상위 컴포넌트로부터 새로운 `prop`을 받으면 `filterTodos` 함수가 다시 실행됩니다.

```jsx
function TodoList({ todos, tab, theme }) {
  const visibleTodos = filterTodos(todos, tab);
  // ...
}
```

<br>

- 일반적으로 대부분의 연산은 매우 빠르기 때문에 문제되지 않습니다.
- 데이터가 변경되지 않았다면 고비용 연산을 다시 수행하지 않는 것이 좋습니다.
- `todos`과 `tab`이 직전 렌더링과 동일한 경우 `useMemo`로 감싸면 이전에 연산한 `visibleTodos`를 재사용할 수 있습니다.
- 이러한 유형의 캐싱을 **메모이제이션**이라고 합니다.

<br>

### 참고

- `useMemo`는 성능 최적화를 위해서만 사용합니다.
- `useMemo` 없이 코드가 작동하지 않는 경우 근본적인 문제를 찾아서 먼저 해결합니다.
- 그런 다음 `useMemo`를 추가하여 성능을 개선합니다.

<br>

## Deep Dive - 고비용 연산을 어떻게 구분하나요?

- 일반적으로 수천 개의 개체를 만들거나 반복하는 경우가 아니라면 고비용 연산이 아닙니다.
- 확신을 얻고 싶다면 로그를 추가하여 소요된 시간을 측정합니다.

```jsx
console.time('filter array');
const visibleTodos = filterTodos(todos, tab);
console.timeEnd('filter array');
```

<br>

- 측정하려는 상호작용을 수행합니다.(예: `input`에 타이핑)
- 그러면 `filter array: 0.15ms`와 같은 로그가 표시됩니다.
- 기록 시간이 큰 경우(예: `1ms` 이상) 해당 연산을 캐시하는 것이 좋습니다.
- `useMemo`로 래핑한 후 로깅 시간이 감소했는지 확인합니다.

```jsx
console.time('filter array');
const visibleTodos = useMemo(() => {
  return filterTodos(todos, tab);
  // todos와 tab이 변경되지 않은 경우 건너뛰기
}, [todos, tab]);
console.timeEnd('filter array');
```

<br>

- `useMemo`는 첫 렌더링을 빠르게 만들지 않습니다.
- 리렌더링 간 불필요한 작업을 건너뛰는 데 유용합니다.

<br>

- 개발용 컴퓨터가 유저 컴퓨터보다 빠를 수 있으므로 인위적으로 속도를 늦춰 성능을 테스트하는 것이 좋습니다.
- Chrome은 이를 위한 **CPU 스로틀링** 옵션을 제공합니다.

<br>

- 개발 모드에서 성능을 측정하는 것은 정확한 결과를 제공하지 않습니다.
  (`Strice Mode`는 각 컴포넌트를 2번 렌더링합니다.)
- 가장 정확한 결과를 얻으려면 배포하여 유저 기기에서 테스트합니다.

<br>

## Deep Dive - 모든 곳에 `useMemo`를 추가해야 하나요?

- 광범위한(페이지 또는 전체 섹션 교체 등) 상호 작용이 대부분이라면 필요하지 않습니다.
- 반면 드로잉 에디터나 도형 이동과 같이 세분화되어 있는 경우라면 메모이제이션이 매우 유용합니다.

<br>

- `useMemo`를 통한 최적화는 아래의 경우에만 유용합니다.

<br>

1. `useMemo`에 추가하는 연산이 눈에 띄게 느리고 종속성이 거의 변하지 않는 경우
2. `memo`로 래핑된 컴포넌트에 `prop`으로 전달하는 경우
   - 값이 변경되지 않는다면 렌더링을 건너뜁니다.
   - 메모이제이션을 사용하면 종속성이 다른 경우에만 컴포넌트를 리렌더링합니다.
3. 전달한 함수가 일부 `hook`의 종속성에 사용되는 경우
   - 다른 `useMemo`의 종속성에 있는 경우
   - 또는 `useEffect`의 종속성에 있는 경우

<br>

- 이외의 경우에는 `useMemo`를 사용하는 이점이 없습니다.
- 그렇다고 큰 결점이 되는 것도 아니기에 일부 팀에서는 개별 사례에 대해 생각하지 않고 가능한 한 많이 메모이제이션 하기도 합니다.
- 단점은 코드 가독성이 떨어집니다.
- 더하여 메모이제이션이 모두 효과적인 것은 아닙니다.
- **항상 새로운** 단일 값은 전체 컴포넌트에 대한 메모이제이션을 망가트립니다.

<br>

- **몇 가지 원칙을 따르면 많은 메모이제이션은 불필요합니다.**

<br>

1. 컴포넌트가 다른 컴포넌트를 래핑할 때 `JSX`를 하위 컴포넌트로 받아들이도록 합니다.
   - 래퍼 컴포넌트가 자체 상태를 업데이트하면 `React`는 하위 컴포넌트가 리렌더링할 필요가 없다는 것을 알 수 있습니다.
2. 지역 상태(변수)를 선호하고 필요 이상으로 **상태를 끌어올리지 마세요.**
   - `form` 태그나 `hover` 여부 같은 일시적인 상태를 트리의 상단이나 전역 상태 라이브러리에 유지하지 마세요.
3. **렌더링 로직을 순수**하게 유지하세요.
   - 컴포넌트를 리렌더링할 때 문제가 발생하거나 시각적으로 어색하다면 컴포넌트에 버그가 있는 것입니다!
   - 메모이제이션하는 대신 버그를 수정합니다.
4. **상태를 업데이트하는 불필요한 `useEffect`**는 피하세요.
   - 대부분의 성능 문제는 컴포넌트를 리렌더링하게 만드는 **`useEffect`** 에서 비롯된 연속적 업데이트 때문입니다.
5. `useEffect`에서 불필요한 종속성을 제거합니다.
   - 메모이제이션 대신 객체나 함수를 `useEffect` 내부 or 컴포넌트 외부로 이동하는 것이 더 간단할 때가 많습니다.

<br>

- 특정 상호작용이 여전히 느리다면 `React Dev Tools`를 사용합니다.
- 위 원칙은 컴포넌트를 더 쉽게 디버깅하고 이해할 수 있게 합니다.

<br>

## 컴포넌트 리렌더링 건너뛰기

- `useMemo`는 리렌더링 성능을 최적화하는 데 유용합니다.
- 아래 `TodoList` 컴포넌트가 하위 `List` 컴포넌트에 `visibleTodos` `prop`을 전달한다고 가정해 보겠습니다.

```jsx
export default function TodoList({ todos, tab, theme }) {
  // ...
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

<br>

- `theme` `prop`을 토글하면 잠시 멈추는 것을 보셨을 텐데 `JSX`에서 `<List />`을 제거하면 속도 향상을 느낄 수 있습니다.
- 이는 `List` 컴포넌트에 최적화할 부분이 있다는 것입니다.

<br>

- **컴포넌트가 리렌더링되면 React는 모든 자식들을 재귀적으로 리렌더링합니다.**
- `TodoList`가 다른 `theme`로 리렌더링될 때 `List` 컴포넌트도 리렌더링됩니다.
- 리렌더링에 많은 연산이 필요하지 않은 컴포넌트에는 괜찮습니다.
- 하지만 리렌더링이 오래 걸린다면 `List`을 `React.memo`로 감싸서 `prop`이 직전 렌더링과 동일한 경우 리렌더링을 건너뛸 수 있습니다.

```jsx
import { memo } from 'react';

const List = memo(function List({ items }) {
  // ...
});
```

<br>

- **이렇게 변경하면 `List`의 모든 `prop`이 직전 렌더링과 동일한 경우 리렌더링을 건너뜁니다.**
- 이때 함수 캐시이 중요해집니다!
- `useMemo`없이 `visibleTodos`을 정의했다고 가정해 봅시다.

```jsx
export default function TodoList({ todos, tab, theme }) {
  // 테마가 변경될 때마다 '다른' 배열이 됩니다.
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      {/* ... List의 prop은 동일하지 않기에 매번 리렌더링됩니다 */}
      <List items={visibleTodos} />
    </div>
  );
}
```

<br>

- **객체 리터럴(`{}`)이 항상 새 객체를 생성하는 것처럼 `filterTodos` 함수는 항상 다른 배열을 생성**합니다.
- 일반적으로는 문제가 되지 않지만 `List`의 `prop`은 동일하지 않기에 `memo`를 활용한 최적화가 이뤄지지 않습니다.
- 바로 이 지점에서 `useMemo`가 유용합니다.

```jsx
export default function TodoList({ todos, tab, theme }) {
  // 리렌더링 사이에 연산 결과를 캐시
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    // 의존성이 변하지 않는 한
    [todos, tab]
  );
  return (
    <div className={theme}>
      {/* ...List는 동일한 prop을 수신하고 리렌더링을 건너뛸 수 있습니다 */}
      <List items={visibleTodos} />
    </div>
  );
}
```

<br>

- **`visibleTodos` 연산을 `useMemo`로 래핑하면 리렌더링 간에 동일한 값을 유지합니다.**
- 특별한 이유가 없는 한 연산을 `useMemo`로 래핑할 필요는 없습니다.

<br>

## Deep Dive - 각각의 JSX 노드 메모화

- `List`를 `memo`로 래핑하는 대신 `<List />` 컴포넌트를 `useMemo`로 래핑합니다.

```jsx
export default function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  const children = useMemo(() => <List items={visibleTodos} />, [visibleTodos]);
  return <div className={theme}>{children}</div>;
}
```

<br>

- 동작은 동일합니다.
- `visibleTodos`가 변경되지 않은 경우 `List`는 리렌더링되지 않습니다.

<br>

- `<List items={visibleTodos} />`와 같은 `JSX` 노드는 `{ type: List, props: { items: visibleTodos } }`와 같은 객체입니다.
- 객체를 생성하는 것은 쉽지만 이전과 동일한지 여부는 알 수 없습니다.
- 그렇기 때문에 `React`는 `List` 컴포넌트를 렌더링합니다.

<br>

- 하지만 직전 렌더링과 동일한 `JSX`는 `React`가 리렌더링하지 않습니다.
- `JSX` 노드는 변하지 않기 때문입니다.
- `JSX` 노드 객체는 변경되지 않으므로 `React`는 리렌더링을 건너뛰어도 안전합니다.
- 하지만 실제로 동일한 객체여야 합니다.

<br>

- `JSX` 노드를 `useMemo`에 수동으로 래핑하는 것은 편리하지 않습니다.
- 조건부로 래핑할 수 없기 때문입니다.
- `JSX` 노드를 래핑하는 대신 `memo`로 컴포넌트를 래핑하는 이유입니다.

<br>

### 다른 `hook`의 종속성을 메모이제이션

- 컴포넌트 내부 객체에 의존하는 연산이 있습니다.

```jsx
function Dropdown({ allItems, text }) {
  const searchOptions = { matchMode: 'whole-word', text };

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // 🚩 주의: 컴포넌트에서 생성된 객체에 대한 종속성
  // ...
}
```

<br>

- 이렇게 객체에 의존하는 것은 메모이제이션의 의미가 없습니다.
- 컴포넌트가 리렌더링되면 컴포넌트 내부의 모든 코드가 다시 실행됩니다.
- 리렌더링될 때마다 `searchOptions` 객체를 생성하는 코드도 다시 실행됩니다.
- `searchOptions`는 `useMemo`의 종속성 목록에 속해있으므로 `React`는 매번 `searchItems`를 다시 연산합니다.

<br>

- 이를 해결하려면 `searchOptions` 객체를 종속성으로 전달하기 전에 `searchOptions` 자체를 메모이제이션합니다.

```jsx
function Dropdown({ allItems, text }) {
  const searchOptions = useMemo(() => {
    return { matchMode: 'whole-word', text };
  }, [text]); // ✅ text가 변경될 때만 변경

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // ✅ 모든 항목 또는 검색옵션이 변경될 때만 변경됨
  // ...
}
```

<br>

- 위의 예에서 `text`가 변경되지 않았다면 `searchOptions` 객체도 변경되지 않습니다.
- 더 나은 방법은 `searchOptions` 객체를 `useMemo` 함수 내부로 이동시키는 것입니다.

```jsx
function Dropdown({ allItems, text }) {
  const visibleItems = useMemo(() => {
    const searchOptions = { matchMode: 'whole-word', text };
    return searchItems(allItems, searchOptions);
  }, [allItems, text]); // ✅ 모든 항목 또는 text가 변경될 때만 변경됩니다.
  // ...
}
```

<br>

- `text`에 종속되는 코드입니다. (문자열이므로 '실수로' 달라질 수 없음)

<br>

### 함수 메모이제이션

- `Form` 컴포넌트가 `memo`로 감싸져 있다고 가정해봅시다.
- 함수를 `prop`으로 전달하려고 합니다.

```jsx
export default function ProductPage({ productId, referrer }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }

  return <Form onSubmit={handleSubmit} />;
}
```

<br>

- 객체 리터럴(`{}`)이 다른 객체를 생성하는 것처럼 `function() {}`과 같은 함수 선언, `() => {}`와 같은 표현식은 리렌더링마다 다른 함수를 생성합니다.
- 새로운 함수를 만드는 것 자체는 문제가 되지 않습니다.
- 그러나 `Form` 컴포넌트가 메모이제이션 된 경우 `props`가 변경되지 않는다면 리렌더링하고 싶지 않을 겁니다.
- 항상 다른 `prop`은 메모이제이션의 핵심을 무너뜨립니다.

<br>

- `useMemo`로 함수를 메모이제이션 하려면 연산 함수가 다른 함수를 반환해야 합니다.

```jsx
export default function Page({ productId, referrer }) {
  const handleSubmit = useMemo(() => {
    return (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    };
  }, [productId, referrer]);

  return <Form onSubmit={handleSubmit} />;
}
```

<br>

- 구려 보입니다!
- **함수를 메모이제이션하는 것은 충분히 흔한 일이며 `React`에는 `hook`이 내장되어 있습니다.**
- 중첩 함수를 추가로 작성할 필요가 없도록 **`useCallback`으로 래핑**합니다.

```jsx
export default function Page({ productId, referrer }) {
  const handleSubmit = useCallback(
    (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    },
    [productId, referrer]
  );

  return <Form onSubmit={handleSubmit} />;
}
```

<br>

- 두 예제는 완전히 동일합니다.
- `useCallback`은 중첩된 함수를 작성하지 않아도 되는 장점이 있습니다.

<br>

## 트러블슈팅

### 렌더링할 때마다 연산이 2번 실행됩니다.

- `Strict Mode`에서 `React`는 일부 함수를 2번 호출합니다.

```jsx
// 이 컴포넌트는 렌더링할 때마다 두 번 실행됩니다.
function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(() => {
    // 종속성 중 하나라도 변경되면 이 연산은 두 번 실행됩니다.
    return filterTodos(todos, tab);
  }, [todos, tab]);
  // ...
}
```

<br>

- 일반적인 현상이며 코드를 손상시키지 않습니다.

<br>

- 개발 모드의 이러한 동작은 **컴포넌트를 순수하게 유지**하는 데 유용합니다.
- `React`는 호출 중 하나의 결과를 사용하고 다른 호출의 결과는 무시합니다.
- 컴포넌트와 연산 함수가 순수하다면 로직에 영향을 미치지 않습니다.
- 하지만 부수효과가 있는 경우 실수를 알아채고 수정하는 데 유용합니다.

<br>

- 아래 연산 함수는 `prop`으로 받은 배열을 변경합니다.

```jsx
const visibleTodos = useMemo(() => {
  // 🚩 실수: prop 변경
  todos.push({ id: 'last', text: 'Go for a walk!' });
  const filtered = filterTodos(todos, tab);
  return filtered;
}, [todos, tab]);
```

<br>

- `React`는 함수를 2번 호출하므로 `todo`가 2번 추가되는 것을 알 수 있습니다.
- 연산이 기존 객체를 변경해서는 안되고 생성한 새 객체를 변경해야 합니다.
- `filterTodos` 함수가 항상 **다른 배열**을 반환하는 경우 배열을 변경할 수 있습니다.

```jsx
const visibleTodos = useMemo(() => {
  const filtered = filterTodos(todos, tab);
  // ✅ 정답: 연산 중에 생성한 객체를 변경합니다.
  filtered.push({ id: 'last', text: 'Go for a walk!' });
  return filtered;
}, [todos, tab]);
```

<br>

### `useMemo`가 정의되지 않은 객체를 반환합니다.

- 코드는 동작하지 않습니다.

```jsx
  // 🔴  () => {}를 사용하는 화살표 함수에서는 객체를 반환할 수 없습니다.
  const searchOptions = useMemo(() => {
    matchMode: 'whole-word',
    text: text
  }, [text]);
```

<br>

- 자바스크립트에서 `() => {`는 화살표 함수의 시작점이므로 중괄호`{`는 객체의 일부가 아닙니다.
- 이 때문에 실수가 발생합니다.
- `({`, `})`를 추가하여 이 문제를 해결합니다.

```jsx
// 작동하지만 누군가 오류를 범하기 쉽습니다.
const searchOptions = useMemo(
  () => ({
    matchMode: 'whole-word',
    text: text,
  }),
  [text]
);
```

<br>

- 위 방식은 여전히 혼란스럽고 괄호를 제거하면 오류를 범하기 쉽습니다.
- 이러한 실수를 방지하려면 `return`을 명시적으로 작성합니다.

```jsx
// ✅ 이것은 작동하며 명확합니다.
const searchOptions = useMemo(() => {
  return {
    matchMode: 'whole-word',
    text: text,
  };
}, [text]);
```

<br>

### 컴포넌트가 렌더링될 때마다 `useMemo`가 다시 실행됩니다.

- 2번째 인수로 종속성 배열을 지정했는지 확인하세요!
- 종속성 배열을 지정하지 않은 경우 `useMemo`는 매번 다시 실행됩니다.

```jsx
function TodoList({ todos, tab }) {
  // 🔴 매번 재연산: 종속선 배열 없음
  const visibleTodos = useMemo(() => filterTodos(todos, tab));
  // ...
}
```

<br>

- 종속성 배열을 2번째 인수로 전달하는 코드입니다.

```jsx
function TodoList({ todos, tab }) {
  // ✅ 불필요하게 재연산하지 않습니다.
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

<br>

- 그래도 해결되지 않는다면 종속성이 직전 렌더링과 다르다는 것입니다.
- 종속성을 콘솔에 로깅하여 디버깅합니다.

```jsx
const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
console.log([todos, tab]);
```

<br>

- 콘솔에서 서로 다른 리렌더링의 배열을 마우스 오른쪽 버튼으로 클릭하고 두 배열 모두에 대해 **전역 변수로 저장**을 선택합니다.
- 첫 번째 배열이 `temp1`로, 두 번째 배열이 `temp2`로 저장되었다고 가정하면 콘솔을 사용하여 두 종속성이 동일한지 확인할 수 있습니다.

```js
Object.is(temp1[0], temp2[0]); // 첫 번째 종속성이 배열 간에 동일한가요?
Object.is(temp1[1], temp2[1]); // 두 번째 의존성이 배열 간에 동일한가요?
Object.is(temp1[2], temp2[2]);
```

<br>

- 어떤 종속성이 메모이제이션을 방해하는지 찾은 후 종속성을 제거할 방법을 찾거나 함께 메모이제이션 하세요.

<br>

### 반복문에서 각 반복마다 `useMemo`를 호출하고 싶습니다.

- `Chart` 컴포넌트가 `memo`로 감싸져 있다고 가정해 봅시다.
- `ReportList` 컴포넌트가 리렌더링될 때 배열의 모든 `Chart`를 리렌더링하고 싶지 않을 겁니다.
- 하지만 `useMemo`를 반복문에서 호출할 수 없습니다.

```jsx
function ReportList({ items }) {
  return (
    <article>
      {items.map((item) => {
        // 🔴 반복문에서 useMemo를 호출할 수 없습니다.
        const data = useMemo(() => calculateReport(item), [item]);
        return (
          <figure key={item.id}>
            <Chart data={data} />
          </figure>
        );
      })}
    </article>
  );
}
```

<br>

- 컴포넌트를 추출하고 각 `item`을 메모이제이션합니다.

```jsx
function ReportList({ items }) {
  return (
    <article>
      {items.map((item) => (
        <Report key={item.id} item={item} />
      ))}
    </article>
  );
}

function Report({ item }) {
  // ✅ 최상단에서 useMemo를 호출합니다.
  const data = useMemo(() => calculateReport(item), [item]);
  return (
    <figure>
      <Chart data={data} />
    </figure>
  );
}
```

<br>

- 또는 `useMemo`를 제거하고 대신 `Report` 자체를 `memo`로 래핑할 수 있습니다.
- `item` `prop`이 변경되지 않으면 `Report`가 리렌더링을 건너뛰므로 `Chart`도 리렌더링을 건너뜁니다.
