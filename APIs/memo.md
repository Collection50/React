# memo

- `memo`를 사용하면 컴포넌트의 `prop`이 변경되지 않은 경우 리렌더링을 생략할 수 있습니다.

```jsx
const MemoizedComponent = memo(SomeComponent, arePropsEqual?)
```

## 레퍼런스

### `memo(Component, arePropsEqual?)`

- 컴포넌트를 `memo`로 감싸면 해당 컴포넌트의 메모이제이션 버전을 사용할 수 있습니다.
- 메모이제이션된 컴포넌트는 `prop`가 변경되지 않는 한 리렌더링되지 않습니다.
- 메모이제이션은 성능 최적화를 **위한** 것이지 성능 최적화를 보장하는 것은 아닙니다.

<br>

```jsx
import { memo } from 'react';

const SomeComponent = memo(function SomeComponent(props) {
  // ...
});
```

<br>

### 매개변수

- `Component`
  - 메모이제이션하려는 컴포넌트입니다.
  - `memo`는 메모이제이션된 새 컴포넌트를 반환합니다.
  - 함수나 `forwardRef` 등 모든 유효한 `React` 컴포넌트를 사용할 수 있습니다.
- `arePropsEqual` (**옵션**)
  - 컴포넌트의 이전 `prop`과 새 `prop`을 인수로 받는 함수입니다.
  - 컴포넌트가 이전 `prop`과 새 `prop`이 동일한 경우 `true`를 반환합니다.
  - 동일하지 않으면 `false`를 반환합니다.
  - 보통 이 함수를 지정하지 않습니다.
  - 기본적으로 `prop`은 `Object.is` 메서드를 통해 비교됩니다.

<br>

### 반환값

- `memo`는 새로운 `React` 컴포넌트를 반환합니다.
- 새로운 컴포넌트는 `memo`에 제공된 컴포넌트와 동일하게 동작하지만 `prop`이 변경되지 않는 한 리렌더링하지 않습니다.

<br>

## 사용법

### `prop`이 변경되지 않은 경우 리렌더링 건너뛰기

- `React`는 일반적으로 상위 컴포넌트가 리렌더링될 때마다 컴포넌트를 리렌더링합니다.
- `memo`를 사용하면 `prop`이 동일한 경우 리렌더링하지 않는 컴포넌트를 만들 수 있습니다.
- 이러한 컴포넌트를 **메모이제이션된** 컴포넌트라고 합니다.

<br>

- 컴포넌트를 메모이제이션하려면 컴포넌트를 `memo`로 래핑하고 반환하는 값을 사용합니다.

```jsx
const Greeting = memo(function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
});

export default Greeting;
```

<br>

- `React` 컴포넌트는 항상 순수한 렌더링 로직을 가져야 합니다.
- `prop`, state, 컨텍스트가 변경되지 않은 경우 동일한 출력을 반환해야 합니다.
- `memo`를 사용하면 컴포넌트가 순수하다는 것을 `React`에 알리는 것이므로 `prop`이 변경되지 않는 한 `React`는 리렌더링할 필요가 없습니다.
- `memo`를 사용하더라도 상태가 변경되거나 컨텍스트가 변경되면 컴포넌트가 리렌더링됩니다.

<br>

- 아래 예시는 `name`이 변경될 때마다 `Greeting` 컴포넌트가 리렌더링되지만 주소가 변경될 때는 렌더링되지 않습니다.
  - (주소가 `prop`으로 전달되지 않기 때문에)

```jsx
import { memo, useState } from 'react';

export default function MyApp() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  return (
    <>
      <label>
        Name{': '}
        <input value={name} onChange={(e) => setName(e.target.value)} />
      </label>
      <label>
        Address{': '}
        <input value={address} onChange={(e) => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log('Greeting was rendered at', new Date().toLocaleTimeString());
  return (
    <h3>
      Hello{name && ', '}
      {name}!
    </h3>
  );
});
```

<br>

### 참고

- **`memo`는 성능 최적화를 위해서만 사용합니다.**
- `memo` 없이 코드가 작동하지 않는다면 근본적인 문제를 찾아서 먼저 수정합니다.
- 이후 `memo`를 추가하여 성능을 개선합니다.

<br>

## Deep Dive - 모든 곳에 `memo`를 추가해야 하나요?

- 광범위한(페이지 또는 전체 섹션 교체 등) 상호 작용이 대부분이라면 필요하지 않습니다.
- 반면 드로잉 에디터나 도형 이동과 같이 세분화되어 있는 경우라면 메모이제이션이 매우 유용합니다.

<br>

- `memo`를 통한 최적화는 컴포넌트가 동일한 `prop`으로 자주 리렌더링되거나 고비용 리렌더링 연산에 유용합니다.
- 리렌더링될 때 눈에 띄는 지연이 없다면 `memo`는 불필요합니다.
- 렌더링 중에 정의된 객체나 일반 함수를 전달하는 경우처럼 `prop`이 항상 다른 경우에는 `memo`는 전혀 쓸모가 없습니다.
- 그렇기 때문에 `memo`와 함께 [`useMemo`](https://github.com/Collection50/React/blob/master/Hooks/useMemo.md)와 [`useCallback`](https://github.com/Collection50/React/blob/master/Hooks/useCallback.md)이 필요한 경우가 많습니다.

<br>

- 이외의 경우에는 `memo`을 사용하는 이점이 없습니다.
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

- 특정 상호작용이 여전히 느리다면 `React DevTools`의 `Profiler`를 사용해 어떤 컴포넌트가 메모이제이션를 통해 가장 큰 이점을 얻을 수 있는지 확인합니다.
- 이러한 원칙은 컴포넌트를 더 쉽게 디버깅하고 이해할 수 있게 해주므로 원칙을 따르는 것이 좋습니다.

<br>

### 상태를 사용하여 메모이제이션된 컴포넌트 업데이트하기

- 컴포넌트가 메모이제이션되어 있어도 상태가 변경되면 리렌더링됩니다.
- 메모이제이션은 상위 컴포넌트로부터 전달된 `prop`에만 적용됩니다.

```jsx
import { memo, useState } from 'react';

export default function MyApp() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  return (
    <>
      <label>
        Name{': '}
        <input value={name} onChange={(e) => setName(e.target.value)} />
      </label>
      <label>
        Address{': '}
        <input value={address} onChange={(e) => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log('Greeting was rendered at', new Date().toLocaleTimeString());
  const [greeting, setGreeting] = useState('Hello');
  return (
    <>
      <h3>
        {greeting}
        {name && ', '}
        {name}!
      </h3>
      <GreetingSelector value={greeting} onChange={setGreeting} />
    </>
  );
});

function GreetingSelector({ value, onChange }) {
  return (
    <>
      <label>
        <input
          type='radio'
          checked={value === 'Hello'}
          onChange={(e) => onChange('Hello')}
        />
        Regular greeting
      </label>
      <label>
        <input
          type='radio'
          checked={value === 'Hello and welcome'}
          onChange={(e) => onChange('Hello and welcome')}
        />
        Enthusiastic greeting
      </label>
    </>
  );
}
```

<br>

- 상태 변수를 현재 값으로 설정하면 `memo`가 없어도 `React`는 컴포넌트의 리렌더링을 건너뜁니다.
- 컴포넌트 함수가 1번 더 호출되지만 결과는 버려집니다.

<br>

### 컨텍스트를 사용하여 메모이제이션된 컴포넌트 업데이트하기

- 컴포넌트가 메모이제이션되어 있어도 컨텍스트가 변경되면 리렌더링됩니다.
- 메모이제이션은 상위 컴포넌트로부터 전달되는 `prop`에만 적용됩니다.

```jsx
import { createContext, memo, useContext, useState } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  const [theme, setTheme] = useState('dark');

  function handleClick() {
    setTheme(theme === 'dark' ? 'light' : 'dark');
  }

  return (
    <ThemeContext.Provider value={theme}>
      <button onClick={handleClick}>Switch theme</button>
      <Greeting name='Taylor' />
    </ThemeContext.Provider>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log('Greeting was rendered at', new Date().toLocaleTimeString());
  const theme = useContext(ThemeContext);
  return <h3 className={theme}>Hello, {name}!</h3>;
});
```

<br>

- 컨텍스트 일부가 변경될 때만 리렌더링되도록 하려면 컴포넌트를 둘로 분할합니다.
- 외부 컴포넌트에서 컨텍스트를 참조하고 메모이제이션된 하위 컴포넌트에게 `prop`으로 전달합니다.

<br>

### `props` 변경 최소화하기

- `memo`를 사용하면 `prop`이 다를 때마다 리렌더링합니다.
- `React`는 컴포넌트의 모든 `prop`을 `Object.is` 메서드를 사용해 이전과 비교합니다.
- `Object.is(3, 3)`는 `true`지만 `Object.is({}, {})`는 `false`라는 점에 유의합니다.

<br>

- `memo`를 최대한 활용하려면 `prop`이 변경되는 횟수를 최소화합니다.
- `prop`이 객체인 경우 `useMemo`를 사용하여 상위 컴포넌트가 매번 객체를 다시 생성하지 않도록 합니다.

```jsx
function Page() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);

  const person = useMemo(() => ({ name, age }), [name, age]);

  return <Profile person={person} />;
}

const Profile = memo(function Profile({ person }) {
  // ...
});
```

<br>

- `prop` 변경을 최소화하는 더 좋은 방법은 컴포넌트가 최소한의 정보만 `prop`이 사용하도록 하는 것입니다.
- 예를 들어 객체 대신 개별 값을 허용합니다.

```jsx
function Page() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);
  return <Profile name={name} age={age} />;
}

const Profile = memo(function Profile({ name, age }) {
  // ...
});
```

<br>

- 컴포넌트가 값 자체보다는 값의 존재를 나타내는 불리언 값을 받습니다.

```jsx
function GroupsLanding({ person }) {
  const hasGroups = person.groups !== null;
  return <CallToAction hasGroups={hasGroups} />;
}

const CallToAction = memo(function CallToAction({ hasGroups }) {
  // ...
});
```

<br>

- 메모이제이션된 컴포넌트에 함수를 전달해야 하는 경우 컴포넌트 외부에 함수를 선언하여 변경되지 않도록 하거나, 리렌더링 간 함수를 캐시하기 위해 `useCallback`을 사용합니다.

<br>

## 커스텀 비교 함수 지정

- 드물지만 메모이제이션된 컴포넌트의 `prop` 변경을 최소화하는 것이 불가능할 수 있습니다.
- 커스텀 비교 함수를 사용하면 `prop`을 비교하는 데 사용할 수 있습니다.
- 이 함수는 `memo`에 2번째 인수로 전달됩니다.
- 새 `prop`가 이전 `prop`이 동일한 경우에만 `true`를 반환하며 그렇지 않으면 `false`를` 반환합니다.

```jsx
const Chart = memo(function Chart({ dataPoints }) {
  // ...
}, arePropsEqual);

function arePropsEqual(oldProps, newProps) {
  return (
    oldProps.dataPoints.length === newProps.dataPoints.length &&
    oldProps.dataPoints.every((oldPoint, index) => {
      const newPoint = newProps.dataPoints[index];
      return oldPoint.x === newPoint.x && oldPoint.y === newPoint.y;
    })
  );
}
```

<br>

- 이 경우 브라우저 개발자 도구의 퍼포먼스 패널을 사용하여 비교 기능이 실제로 컴포넌트를 리렌더링하는 것보다 빠른지 확인하세요.

<br>

- 성능을 측정할 때는 `React`가 배포 모드에서 실행 중인지 확인합니다.

<br>

## 주의

- **커스텀 함수인 `arePropsEqual`를 제공하는 경우 함수를 포함하여 모든 `prop`을 비교해야 합니다.**
- 함수는 상위 컴포넌트의 `prop`과 상태에 따라 달라집니다.
- `oldProps.onClick !== newProps.onClick`일 때 `true`를 반환하면 컴포넌트가 `onClick` 핸들러는 이전 렌더링의 `prop`과 상태를 계속 **가리키게** 되므로 매우 혼란스러운 버그가 발생할 수 있습니다.

<br>

- 데이터 구조의 깊이가 제한되어 있다는 것을 100% 확신하지 않는 한 `arePropsEqual` 내부에서 깊은(`deep`) 비교 연산을 수행하지 않습니다.
- 깊은 비교 연산은 매우 느려질 수 있으며 데이터 구조를 변경하면 몇 초 동안 정지될 수 있습니다.

<br>

## 트러블슈팅

### `prop`이 객체, 배열 또는 함수일 때 컴포넌트가 리렌더링합니다.

- `React`는 얕은(`shallow`) 비교, 즉 각 새로운 `prop`이 이전 `prop`과 동등한지 고려하여 `prop`을 비교합니다.
- 상위 컴포넌트가 리렌더링될 때마다 새로운 객체나 배열을 생성하면 컴포넌트가 동일하더라도 `React`는 여전히 변경된 것으로 간주합니다.
- 마찬가지로 상위 컴포넌트를 렌더링할 때 새 함수를 생성하면 함수의 정의가 동일하더라도 `React`는 함수가 변경된 것으로 간주합니다.
- 이를 방지하려면 상위 컴포넌트에서 `prop`을 단순화하거나 `prop`을 메모이제이션합니다.
