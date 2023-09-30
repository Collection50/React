# useCallback

- `useCallback`은 리렌더링 간 함수의 정의를 캐시하는 `hook`입니다.

```js
const cachedFn = useCallback(fn, dependencies);
```

## 레퍼런스

- 컴포넌트의 최상단에서 `useCallback`을 호출하여 리렌더링 간 함수 정의를 캐시합니다.

```jsx
import { useCallback } from 'react';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback(
    (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    },
    [productId, referrer]
  );
}
```

### 매개변수

- `fn`
  - 캐시하려는 함수입니다.
  - 어떤 값도 매개변수로 사용할 수 있고 어떤 값도 반환할 수 있습니다.
  - `dependencies`가 변하지 않는 한 동일한 **함수**를 제공합니다.
  - `dependencies`가 변한다면 렌더링 중에 전달한 함수를 제공하고 다시 캐시합니다.
  - **함수를 호출하는 것이 아닙니다.**
  - 사용자가 호출 시점을 결정할 수 있습니다.

<br>

- `dependencies`
  - `fn`에서 참조하는 모든 반응형 값의 목록입니다.
  - 반응형 값에는 `prop`, 상태, 컴포넌트에서 선언된 모든 변수와 함수가 포함됩니다.
  - 린터를 사용하면 모든 반응형 값이 종속성에 올바르게 지정되었는지 체크합니다.
  - 종속성 배열에는 일정한 수의 항목이 있어야 하며 `[dep1, dep2, dep3]`와 같이 인라인으로 작성해야 합니다.
  - `React`는 `Object.is` 비교 메서드를 사용합니다.

<br>

### 반환값

- 초기 렌더링에서 `useCallback`은 전달한 `fn`을 반환합니다.
- 이후 렌더링에선 2가지로 나뉩니다.
- 종속성이 변경되지 않은 경우 이전에 캐시된 `fn`을 반환합니다.
- 종속성이 변경된 경우 직전 렌더링에서 전달한 `fn`을 반환합니다. (변경된 `fn`)

<br>

### 주의사항

- `useCallback`은 `hook`이므로 컴포넌트의 최상단 or 커스텀 `hook`에서만 호출할 수 있습니다.
  - 반복문, 조건문 내부에서 호출할 수 없습니다.
  - 필요하다면 새 컴포넌트를 추출하고 상태를 옮깁니다.

<br>

- `React`는 **특별한 이유가 없는 한 캐시된 함수를 버리지 않습니다.**
  - 개발 모드에서 컴포넌트를 변경할 때 `React`는 캐시를 버립니다.
  - 개발 모드나 프로덕션 모드에서 컴포넌트가 초기 마운트 중에 일시 중단되면 `React`는 캐시를 버립니다.
  - 미래에 `React`는 캐시 버리기를 활용하는 더 많은 기능을 추가할 겁니다.
  - 예를 들어 `React`에 가상화된 목록에 대한 기본 지원이 추가되면 가상화된 테이블 뷰포트에서 스크롤되는 항목에 대한 캐시도 버리는 것이 합리적일 것입니다.
  - 성능 최적화를 위해 `useCallback`을 사용하는 것이 좋습니다.
  - 그렇지 않다면 상태나 `ref`가 더 적합합니다.

<br>

## 사용법

## 컴포넌트 리렌더링 건너뛰기

- 렌더링 최적화를 수행할 때 하위 컴포넌트에 전달하는 함수를 캐시해야 할 때가 있습니다.
- 먼저 문법을 살펴본 이후 유용한 예시에 대해 알아보겠습니다.

<br>

- 리렌더링 간 함수를 캐시하려면 `useCallback` `hook`을 아래와 같이 매핑합니다.

```jsx
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback(
    (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    },
    [productId, referrer]
  );
  // ...
}
```

<br>

- `useCallback`에 두 가지를 전달해야 합니다.

1. 리렌더링 사이에 캐시하려는 함수 정의.
2. 함수 내부에 사용되는 모든 종속성 목록.

<br>

- 초기 렌더링에서 `useCallback`는 전달된 함수를 그대로 반환합니다.

<br>

- 이후 렌더링부터는 이전 렌더링과 종속성을 비교합니다.
- 종속성이 변경되지 않았다면(`Object.is` 사용) `useCallback`은 이전과 동일한 함수를 반환합니다.
- 종속성이 변경 되었다면 `useCallback`은 직전 렌더링에서 전달한 함수를 반환합니다. (변경된 함수)

<br>

- 정리하자면 `useCallback`은 종속성이 변경될 때까지 함수를 캐시합니다.

<br>

- 유용하게 사용되는 경우를 살펴보겠습니다.
- `ProductPage`에서 `ShippingForm` 컴포넌트로 `handleSubmit` 함수를 전달한다고 가정해 보겠습니다:

```js
function ProductPage({ productId, referrer, theme }) {
  // ...
  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

<br>

- `theme` `prop`을 토글하면 잠시 멈추는 것을 보셨을 텐데 `JSX`에서 `<ShippingForm />`을 제거하면 속도 향상을 느낄 수 있습니다.
- 이는 `ShippingForm` 컴포넌트에 최적화할 부분이 있다는 것입니다.

<br>

- **컴포넌트가 리렌더링되면 React는 모든 자식들을 재귀적으로 리렌더링합니다.**
- `ProductPage`가 다른 `theme`로 리렌더링될 때 `ShippingForm` 컴포넌트도 리렌더링됩니다.
- 리렌더링에 많은 계산이 필요하지 않은 컴포넌트에는 괜찮습니다.
- 하지만 리렌더링이 오래 걸린다면 `ShippingForm`을 `React.memo`로 감싸서 `prop`이 직전 렌더링과 동일한 경우 리렌더링을 스킵하도록 할 수 있습니다.

```jsx
import { memo } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

<br>

- **이렇게 변경하면 `ShippingForm`의 모든 `prop`이 직전 렌더링과 동일한 경우 리렌더링을 건너뜁니다.**
- 이때 함수 캐시이 중요해집니다!
- `useCallback`없이 `handleSubmit`을 정의했다고 가정해 봅시다.

<br>

```jsx
function ProductPage({ productId, referrer, theme }) {
  // 테마가 변경될 때마다 '다른' 함수가 제공됩니다.
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }

  return (
    <div className={theme}>
      {/* 따라서 ShippingForm의 prop은 동일하지 않기에 매번 리렌더링됩니다 */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

<br>

- **`JavaScript`에서 `function() {}` or `() => {}`는 객체 리터럴(`{}`)이 항상 새 객체를 생성하는 것처럼** 항상 다른 함수를 생성합니다.
- 일반적으로는 문제가 되지 않지만 `ShippingForm` `prop`은 매번 달라지기에 `memo`를 활용한 최적화가 이뤄지지 않습니다.
- 바로 이 지점에서 `useCallback` 유용합니다.

```jsx
function ProductPage({ productId, referrer, theme }) {
  // 리렌더링 간 함수를 캐시합니다.
  const handleSubmit = useCallback(
    (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    },
    // 종속성 배열이 변경되지 않는 한 함수를 캐시합니다.
    [productId, referrer]
  );

  return (
    <div className={theme}>
      {/* ShippingForm은 동일한 prop을 전달받으므로 리렌더링을 건너뛸 수 있습니다 */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

<br>

- `handleSubmit`을 `useCallback`으로 래핑하면 종속성 배열이 변경될 때까지 동일한 함수를 전달합니다.
- 특별한 이유가 없다면 `useCallback`으로 래핑할 필요는 없습니다.

<br>

### 참고

- `useCallback`은 성능 최적화를 위해서만 사용합니다.
- `useCallback`없이 코드가 작동하지 않는 경우 근본적인 문제를 찾아야 합니다.

<br>

## Deep Dive - `useCallback`은 `useMemo`와 어떤 관계가 있나요?

- `useMemo`와 `useCallback`이 함께 사용되는 경우가 많습니다.
- 둘 다 하위 컴포넌트를 최적화하려고 할 때 유용합니다.
- 이를 통해 **메모이제이션(캐시)** 을 수행합니다.

```jsx
import { useMemo, useCallback } from 'react';

function ProductPage({ productId, referrer }) {
  const product = useData('/product/' + productId);

  const requirements = useMemo(() => {
    // 함수를 '호출'하고 '결과'를 캐시합니다.
    return computeRequirements(product);
  }, [product]);

  const handleSubmit = useCallback(
    (orderDetails) => {
      // 함수 '자체'를 캐시합니다.
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    },
    [productId, referrer]
  );

  return (
    <div className={theme}>
      <ShippingForm requirements={requirements} onSubmit={handleSubmit} />
    </div>
  );
}
```

<br>

- 두 함수의 차이점은 캐시 **대상**에 있습니다.

- **`useMemo`는 함수 호출 '결과'를 캐시합니다.**
  - `useMemo`는 `product`가 변경되지 않는 한 `computeRequirements(product)`의 반환값을 캐시합니다.
  - `ShippingForm`을 리렌더링하지 않고 `requirements` 객체를 전달합니다.

<br>

- **`useCallback`은 함수 자체를 캐시합니다.**
  - `useMemo`와 달리 함수를 호출하지 않습니다.
  - 대신 함수를 캐시에 저장하여 `projudctId`나 `referrer`가 변경되지 않는 한 `handleSubmit` 자체가 변경되지 않도록 합니다.
  - `ShippingForm`의 불필요한 리렌더링을 방지하며 `handleSubmit` 함수를 전달합니다.
  - 사용자가 `form`을 제출할 때까지 코드가 실행되지 않습니다.

<br>

- `useMemo`에 익숙하다면 `useCallback`을 이렇게 생각하면 쉽습니다.

```jsx
// React 내부 동작의 간소화된 표현
function useCallback(fn, dependencies) {
  return useMemo(() => fn, dependencies);
}
```

<br>

## Deep Dive - 모든 곳에 `useCallback`을 추가해야 하나요?

- 광범위한(페이지 또는 전체 섹션 교체 등) 상호 작용이 대부분이라면 필요하지 않습니다.
- 반면 드로잉 에디터나 도형 이동과 같이 세분화되어 있는 경우라면 메모이제이션이 매우 유용합니다.

<br>

- `useCallback`으로 함수를 캐시하는 것은 아래의 경우에만 유용합니다.

1.  `memo`로 래핑된 컴포넌트에 `prop`으로 전달할 때
    - 값이 변경되지 않았다면 렌더링을 건너뛰고 싶을 것입니다.
    - 메모이제이션을 사용하면 종속성 배열이 변경된 경우에만 컴포넌트를 리렌더링합니다.
2.  함수가 다른 `hook`의 종속성 배열의 요소로 사용될 때
    - 예를 들어 `useCallback`으로 감싸진 다른 함수가 종속되거나 `useEffect`에 종속되는 경우입니다.

<br>

- 이외의 경우에는 `useCallback`을 사용하는 이점이 없습니다.
- 그렇다고 큰 결점이 되는 것도 아니기에 일부 팀에서는 개별 사례에 대해 생각하지 않고 가능한 한 많이 메모이제이션 하기도 합니다.
- 단점은 코드 가독성이 떨어집니다.
- 더하여 메모이제이션이 모두 효과적인 것은 아닙니다.
- **항상 새로운** 단일 값은 전체 컴포넌트에 대한 메모이제이션을 망가트립니다.

<br>

- `useCallback`은 함수 **생성**을 막지 않는다는 점에 주의해야 합니다.
- 여러분은 항상 함수를 생성하지만(그것은 괜찮습니다!), `React`는 이를 무시하고 변경된 것이 없다면 캐시된 함수를 반환합니다.

<br>

- **몇 가지 원칙을 따르면 많은 메모이제이션은 불필요합니다.**

1. 컴포넌트가 다른 컴포넌트를 래핑할 때 `JSX`를 하위 컴포넌트로 받아들이도록 합니다.
   - 래퍼 컴포넌트가 자체 상태를 업데이트하면 `React`는 자식 컴포넌트가 다시 렌더링할 필요가 없다는 것을 알 수 있습니다.
2. 지역 상태(변수)를 선호하고 필요 이상으로 **상태를 끌어올리지 마세요.**
   - `form` or `hover` 여부와 같은 일시적인 상태를 트리의 상단이나 전역 상태 라이브러리에 유지하지 마세요.
3. **렌더링 로직을 순수**하게 유지하세요.
   - 컴포넌트를 리렌더링할 때 문제가 발생하거나 시각적으로 어색하다면 컴포넌트에 버그가 있는 것입니다!
   - 메모이제이션하는 대신 버그를 수정합니다.
4. **상태를 업데이트하는 불필요한 `useEffect`**는 피하세요.
   - 대부분의 성능 문제는 컴포넌트를 리렌더링하게 만드는 **`useEffect`** 에서 비롯된 연속적 업데이트 때문입니다.
5. `useEffect`에서 불필요한 종속성을 제거하세요.
   - 메모이제이션 대신 객체나 함수를 `useEffect` 내부 or 컴포넌트 외부로 이동하는 것이 더 간단할 때가 많습니다.

<br>

- 특정 상호작용이 여전히 느리다면 `React Developer Tools`를 사용합니다.
- 위 원칙은 컴포넌트를 더 쉽게 디버깅하고 이해할 수 있게 합니다.

<br>

## 메모이제이션된 콜백에서 상태 업데이트하기

- 메모이제이션된 콜백에서 상태를 업데이트하는 경우에 대해 살펴보겠습니다.
- 아래 `handleAddTodo` 함수는 `todos`를 종속성 배열에 지정하여 다음 `todos`를 계산합니다.

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback(
    (text) => {
      const newTodo = { id: nextId++, text };
      setTodos([...todos, newTodo]);
    },
    [todos]
  );
  // ...
}
```

<br>

- 일반적으로 메모이제이션된 함수는 가능한 적은 종속성 배열을 사용해야 합니다.
- 다음 상태를 계산하기 위해 일부 상태만 읽어야 하는 경우 **업데이트 함수**를 전달하여 해당 종속성을 제거합니다.

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos((todos) => [...todos, newTodo]);
  }, []); // ✅ todos 종속성 배열이 필요 없습니다.
  // ...
}
```

<br>

## `useEffect`가 자주 사용되지 않도록 하기

- 가끔 `useEffect` 내부에서 함수를 호출할 때가 있습니다.

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  function createOptions() {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId,
    };
  }

  useEffect(
    () => {
      const options = createOptions();
      const connection = createConnection();
      connection.connect();
      // ...
    } /* ... */
  );
}
```

<br>

- **모든 반응형 값은 `useEffect`의 종속성 배열에 선언해야 합니다.**
- 그러나 `createOptions`를 종속성 배열에 추가하면 `useEffect`가 `ChatRoom`에 계속 다시 연결합니다.

```jsx
useEffect(() => {
  const options = createOptions();
  const connection = createConnection();
  connection.connect();
  return () => connection.disconnect();
  // 🔴 문제: 매 렌더링마다 종속성이 변경됩니다.
}, [createOptions]);
// ...
```

<br>

- 이를 해결하려면 `useEffect`에서 호출해야 하는 함수를 `useCallback`으로 래핑합니다.

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const createOptions = useCallback(() => {
    return {
      serverUrl: 'https://localhost:1234',
      roomId: roomId,
    };
    // ✅ roomId가 변경될 때만 변경됩니다.
  }, [roomId]);

  useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
    // ✅ createOptions이 변경될 때만 변경됩니다.
  }, [createOptions]);
  // ...
}
```

<br>

- `roomId`가 동일한 경우 리렌더링할 때마다 `createOptions` 함수가 동일하게 유지됩니다.
- 하지만 종속성 배열을 없애는 것이 더 좋습니다.
- 함수를 `useEffect` 내부로 옮깁니다.

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    function createOptions() {
      // ✅ useCallback이나 함수 종속성 배열을 사용할 필요가 없습니다!
      return {
        serverUrl: 'https://localhost:1234',
        roomId: roomId,
      };
    }

    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
    // ✅ roomId가 변경될 때만 감지합니다.
  }, [roomId]);
  // ...
}
```

<br>

- 코드가 더 간결해지고 `useCallback`이 필요하지 않습니다.

<br>

## 커스텀 Hook 최적화

- 커스텀 `hook`을 사용할 때는 반환하는 모든 함수를 `useCallback`으로 래핑합니다.

```jsx
function useRouter() {
  const { dispatch } = useContext(RouterStateContext);

  const navigate = useCallback(
    (url) => {
      dispatch({ type: 'navigate', url });
    },
    [dispatch]
  );

  const goBack = useCallback(() => {
    dispatch({ type: 'back' });
  }, [dispatch]);

  return {
    navigate,
    goBack,
  };
}
```

- `hook`의 사용자가 필요할 때 자신의 코드를 최적화할 수 있습니다.

<br>

## 트러블슈팅

### 컴포넌트가 렌더링될 때마다 `useCallback`이 다른 함수를 반환합니다.

- 2번째 인수로 종속성 배열을 지정했는지 확인하세요!
- 종속성 배열이 없다면 `useCallback`은 매번 새로운 함수를 반환합니다.

```jsx
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }); // 🔴 종속성 배열이 없으므로 매번 새로운 함수 반환
  // ...
}
```

<br>

- 수정 후

```jsx
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback(
    (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    },
    [productId, referrer]
  ); // ✅ 불필요한 함수 반환이 없습니다.
  // ...
}
```

<br>

- 효과가 없다면 종속성이 이전 렌더링과 다르다는 의미입니다.
- 종속성을 수동으로 로깅하여 문제를 디버그합니다.

```jsx
const handleSubmit = useCallback(
  (orderDetails) => {
    // ..
  },
  [productId, referrer]
);

console.log([productId, referrer]);
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

- 메모이제이션을 방해하는 종속성을 찾았다면 제거할 방법을 찾거나 함께 메모이제이션합니다.

<br>

### 각 반복마다 `useCallback`을 호출해야 할 때는 어떻게 해야 하나요?

- `Chart` 컴포넌트가 `memo`로 감싸져 있다고 가정해 봅시다.
- `ReportList` 컴포넌트가 다시 렌더링될 때 배열의 모든 `Chart`를 리렌더링하는 것은 좋지 않습니다.
- 하지만 반복문에서 `useCallback`을 호출할 수 없습니다.

```jsx
function ReportList({ items }) {
  return (
    <article>
      {items.map((item) => {
        // 🔴 반복문에서 useCallback을 호출할 수 없습니다.
        const handleClick = useCallback(() => {
          sendReport(item);
        }, [item]);

        return (
          <figure key={item.id}>
            <Chart onClick={handleClick} />
          </figure>
        );
      })}
    </article>
  );
}
```

<br>

- 대신 각 `item`에 대한 컴포넌트를 추출하고 거기에 `useCallback`을 추가합니다.

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
  // ✅ 최상단에 존재하는 useCallback
  const handleClick = useCallback(() => {
    sendReport(item);
  }, [item]);

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
}
```

<br>

- 마지막 코드에서 `useCallback`을 제거하고 `Report` 컴포넌트를 `memo`로 감싸는 방법도 있습니다.
- `item` `prop`이 변경되지 않으면 `Report`가 리렌더링 되지 않으므로 `Chart`도 리렌더링 되지 않습니다.

```jsx
function ReportList({ items }) {
  // ...
}

const Report = memo(function Report({ item }) {
  function handleClick() {
    sendReport(item);
  }

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
});
```
