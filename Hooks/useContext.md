# useContext

- `useContext`는 컴포넌트에서 컨텍스트를 참조하고 구독하는 `React` `hook`입니다.

```jsx
const value = useContext(SomeContext);
```

## 레퍼런스

### `useContext(SomeContext)`

- 컴포넌트 최상단에서 `useContext`를 호출하여 컨텍스트를 참조하고 구독합니다.

```jsx
import { useContext } from 'react';

function MyComponent() {
  const theme = useContext(ThemeContext);
  // ...
}
```

<br>

### 매개변수

- `someContext`
  - `createContext`로 생성한 컨텍스트입니다.
  - 컨텍스트 자체는 정보를 보유하지 않고 컴포넌트에서 제공하거나 참조할 수 있는 정보의 종류를 나타냅니다.

<br>

### 반환값

- `useContext`는 컴포넌트의 컨텍스트 값을 반환합니다.
- 반환값은 트리에서 가장 가까운 `SomeContext.Provider`에 전달된 `value`에 의해 결정됩니다.
- 이러한 프로바이더가 없는 경우 반환값은 `createContext`에 전달한 `defaultValue`가 됩니다.
- 반환된 값은 항상 최신 값입니다.
- 컨텍스트가 변경되면 `React`는 컨텍스트를 참조하는 컴포넌트를 자동으로 리렌더링합니다.

<br>

### 주의사항

- `useContext` 호출은 **동일한** 컴포넌트에서 반환된 프로바이더의 영향을 받지 않습니다.
- 상응하는 `<Context.Provider>`(컨텍스트의 프로바이더)는 `useContext` 호출하는 컴포넌트 **위**에 있어야 합니다. (트리 상단)

<br>

- `React`는 다른 `value`을 받는 프로바이더부터 시작해서 특정 컨텍스트를 사용하는 모든 하위 컴포넌트를 **자동으로 리렌더링합니다.**
- `Object.is` 메서드를 통해 변경사항을 비교합니다.
- `memo`로 리렌더링을 건너뛰어도 하위 컴포넌트가 새로운 컨텍스트 값을 받는 것을 **막을 수 없습니다.**

<br>

- 빌드 시스템이 중복 모듈을 생성하는 경우 컨텍스트가 망가질 수 있습니다.
- 컨텍스트를 통해 무언가를 전달하는 경우, 컨텍스트를 제공하는 데 사용하는 `SomeContext`와 컨텍스트를 참조하는 데 사용하는 `SomeContext`가 `===`(일치 비교 연산자)에 의해 결정되는 **정확히 동일한 객체**인 경우에만 작동합니다.

<br>

## 사용법

### 렌더 트리 깊은 곳까지 데이터 전달

- 컴포넌트 최상단에서 `useContext`를 호출하여 컨텍스트를 참조하고 구독합니다.

```jsx
import { useContext } from 'react';

function Button() {
  const theme = useContext(ThemeContext);
  // ...
}
```

<br>

- `useContext`는 매개변수로 전달한 **컨텍스트**의 **컨텍스트 값**을 반환합니다.
- 컨텍스트 값을 결정하기 위해 `React`는 컴포넌트 트리를 검색하고 **위에서 가장 가까운 컨텍스트 프로바이더**를 찾습니다.

<br>

- 컨텍스트를 `Button`에 전달하려면 `Button` 또는 상위 컴포넌트를 컨텍스트 프로바이더로 래핑합니다.

```jsx
function MyPage() {
  return (
    <ThemeContext.Provider value='dark'>
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  // ... 내부에 버튼을 렌더링합니다 ...
}
```

<br>

- 프로바이더와 `Button` 사이에 컴포넌트 개수는 중요하지 않습니다.
- `Form` 내부의 `Button`이 `useContext(ThemeContext)`를 호출하면 `'dark'`를 값으로 받습니다.

<br>

### 주의

- `useContext`는 위에서 가장 가까운 프로바이더를 찾습니다.
- 위쪽으로 검색하며 **`useContext`를 호출하는 컴포넌트의 프로바이더는 고려하지 않습니다.**

<br>

```jsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext.Provider value='dark'>
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  return (
    <Panel title='Welcome'>
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  );
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return <button className={className}>{children}</button>;
}
```

<br>

### 컨텍스트를 활용한 데이터 업데이트

- 컨텍스트를 업데이트하려면 컨텍스트를 **상태**와 결합합니다.
- 상위 컴포넌트에서 상태 변수를 선언하고 **컨텍스트 값**으로 프로바이더에 전달합니다.

```jsx
function MyPage() {
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext.Provider value={theme}>
      <Form />
      <Button
        onClick={() => {
          setTheme('light');
        }}
      >
        Switch to light theme
      </Button>
    </ThemeContext.Provider>
  );
}
```

<br>

- 프로바이더 내부의 모든 `Button`이 현재 `theme` 값을 받게 됩니다.
- 프로바이더의 `theme` 값을 변경하기 위해 `setTheme`를 호출하면 모든 `Button` 컴포넌트가 새로운 값(`'light'`)으로 리렌더링됩니다.

<br>

### `fallback`의 기본값 지정

- 상위 트리에서 **컨텍스트**의 프로바이더를 찾을 수 없는 경우 `useContext`가 반환하는 **컨텍스트 값**은 해당 **컨텍스트를 생성할 때 전달**한 **기본값**입니다.

```jsx
const ThemeContext = createContext(null);
```

<br>

- **기본값은 변경되지 않습니다.**
- 컨텍스트를 업데이트하려면 **상태**를 사용합니다.

<br>

- 보통 `null` 대신 의미 있는 값을 사용합니다.

```jsx
const ThemeContext = createContext('light');
```

<br>

- 실수로 해당 프로바이더 없이 컴포넌트를 렌더링해도 중단되지 않습니다.
- 또한 많은 프로바이더를 설정하지 않고 용이하게 테스트 환경을 구축할 수 있습니다.

<br>

### 트리의 일부에서 컨텍스트 오버라이딩

- 트리 일부분을 다른 프로바이더로 래핑하여 컨텍스트를 오버라이딩합니다.

```jsx
<ThemeContext.Provider value='dark'>
  ...
  <ThemeContext.Provider value='light'>
    <Footer />
  </ThemeContext.Provider>
  ...
</ThemeContext.Provider>
```

<br>

- 원하는만큼 프로바이더를 중첩하거나 오버라이딩할 수 있습니다.

<br>

### 객체와 함수를 전달할 때 리렌더링 최적화

- 객체와 함수를 포함한 **모든 유형의 값**을 컨텍스트를 통해 전달할 수 있습니다.

```jsx
function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  function login(response) {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }

  return (
    <AuthContext.Provider value={{ currentUser, login }}>
      <Page />
    </AuthContext.Provider>
  );
}
```

<br>

- **컨텍스트 값**은 2개 프로퍼티를 가진 자바스크립트 객체이며 그 중 1개는 함수입니다.
- `MyApp`이 리렌더링할 때마다(예: 라우팅 변경) 다른 함수를 가리키는 다른 객체가 됩니다.
- 따라서 `React`는 `useContext(AuthContext)`를 호출하는 트리 깊은 곳에 있는 모든 컴포넌트도 리렌더링합니다.

<br>

- 작은 앱에서는 문제 되지 않습니다.
- 그러나 `currentUser`와 같은 기본값이 변경되지 않았다면 리렌더링할 필요가 없습니다.
- 변경되지 않은 값을 활용할 수 있도록 `login` 함수를 `useCallback`으로 래핑하고 객체 생성을 `useMemo`로 래핑합니다.
- 성능 최적화를 위한 방법입니다.

```jsx
import { useCallback, useMemo } from 'react';

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(
    () => ({
      currentUser,
      login,
    }),
    [currentUser, login]
  );

  return (
    <AuthContext.Provider value={contextValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

<br>

- `MyApp`이 리렌더링하는 경우에도 `currentUser`가 변경되지 않는 한 `useContext(AuthContext)`를 호출하는 컴포넌트는 리렌더링하지 않습니다.
- [useMemo](https://github.com/Collection50/React/blob/master/Hooks/useMemo.md)와 [useCallback](https://github.com/Collection50/React/blob/master/Hooks/useCallback.md)에 대해 더 알아보기

<br>

## 트러블슈팅

### 컴포넌트에 프로바이더의 값이 표시되지 않습니다.

- 이런 일이 발생하는 몇 가지 이유가 있습니다.

<br>

1. `useContext`를 호출하는 곳과 동일한 컴포넌트(또는 하위)에서 `<SomeContext.Provider>`를 렌더링하고 있습니다.
   - `<SomeContext.Provider>`를 `useContext`를 호출하는 컴포넌트보다 상위로 이동합니다.
2. 컴포넌트를 `<SomeContext.Provider>`로 래핑하는 것을 잊었거나 다른 위치에 존재하는 경우입니다.
   - `React DevTools`를 사용해 구조가 올바른지 확인합니다.
3. 제공하는 컴포넌트의 `SomeContext`와 참조하는 컴포넌트의 `SomeContext`가 다른 2개 객체가 되는 경우입니다.
   - 심볼릭 링크를 사용하는 경우 이런 문제가 발생할 수 있습니다.
   - 이를 확인하려면 `window.SomeContext1` 및 `window.SomeContext2`와 같은 전역 변수에 할당하고 콘솔에서 `window.SomeContext1` === `window.SomeContext2`인지 확인합니다.
   - 동일하지 않은 경우 빌드 수준에서 해당 문제를 수정하세요.

<br>

### 기본값이 다른데도 항상 컨텍스트에서 `undefined`를 얻습니다.

- 트리에 `value`가 없는 프로바이더가 있을 수 있습니다.

```jsx
// 🚩 작동하지 않음: `value` prop 없음
<ThemeContext.Provider>
  <Button />
</ThemeContext.Provider>
```

<br>

- `value` 지정을 잊어버린 경우 `value={undefined}`를 전달하는 것과 같습니다.

<br>

- 실수로 `prop` 이름을 틀렸을 수도 있습니다.

```jsx
// 🚩 작동하지 않음: prop의 이름은 "value"입니다.
<ThemeContext.Provider theme={theme}>
  <Button />
</ThemeContext.Provider>
```

<br>

- 2가지 경우 모두 `React`에서 경고가 표시될 것입니다.
- 이를 고치려면 `value` `prop`을 사용합니다.

<br>

```jsx
// ✅ `value` prop 전달
<ThemeContext.Provider value={theme}>
  <Button />
</ThemeContext.Provider>
```

<br>

- `createContext(defaultValue)`의 기본값은 상위 트리에 프로바이더가 없는 경우에만 사용됩니다.
- 상위 트리 어딘가에 `<SomeContext.Provider value={undefined}>` 컴포넌트가 있는 경우 `useContext(SomeContext)`를 호출하는 컴포넌트는 `undefined`를 **컨텍스트 값**으로 사용합니다.
