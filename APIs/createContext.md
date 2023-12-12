# createContext

- `createContext`를 사용하면 컴포넌트가 제공하거나 참조할 수 있는 컨텍스트를 생성합니다.

```jsx
const SomeContext = createContext(defaultValue);
```

<br>

## 레퍼런스

### `createContext(defaultValue)`

- 컴포넌트 외부에서 `createContext`를 호출하여 컨텍스트를 생성합니다.

```jsx
import { createContext } from 'react';

const ThemeContext = createContext('light');
```

<br>

### 매개변수

- `defaultValue`
  - 컴포넌트의 상위 트리에 프로바이더가 없을 때 컨텍스트의 기본값입니다.
  - 기본값이 없는 경우 `null`을 지정합니다.
  - 기본값은 **최후 수단**으로 사용됩니다.
  - 기본값은 정적인 값이며 변경되지 않습니다.

<br>

### 반환값

- `createContext`는 컨텍스트 객체를 반환합니다.

<br>

- **컨텍스트 객체 자체는 어떠한 정보도 보유하지 않습니다.**
- 컨텍스트는 다른 컴포넌트가 참조하거나 제공하는 컨텍스트를 표시합니다.
- 일반적으로 상위 컴포넌트에서 `SomeContext.Provider`를 사용해 컨텍스트 값을 지정하고 하위 컴포넌트에서 `useContext(SomeContext)`를 호출해 컨텍스트 값을 참조합니다.
- 컨텍스트 객체에는 몇 가지 프로퍼티가 있습니다.

<br>

- `SomeContext.Provider`를 사용하면 컴포넌트에 컨텍스트 값을 제공합니다.
- `SomeContext.Consumer`는 값을 참조하는 대체 방법이며 거의 사용되지 않습니다.

<br>

## `SomeCotext.Provier`

- 컴포넌트를 컨텍스트 프로바이더로 래핑하여 모든 컴포넌트에서 컨텍스트 값을 지정합니다.

```jsx
function App() {
  const [theme, setTheme] = useState('light');
  // ...
  return (
    <ThemeContext.Provider value={theme}>
      <Page />
    </ThemeContext.Provider>
  );
}
```

<br>

### `Props`

- `value`
  - 프로바이더 내에서 컨텍스트를 참조하는 모든 컴포넌트에 전달할 값입니다.
  - 컨텍스트 값은 모든 유형의 값을 사용할 수 있습니다.
  - 프로바이더 내부에서 `useContext(SomeContext)`를 호출하는 컴포넌트는 상위의 컨텍스트 프로바이더의 값을 참조합니다.

<br>

## `SomeCotext.Consumer`

- `useContext`가 존재하기 전에는 컨텍스트를 참조하는 다른 방법이 있었습니다.

```jsx
function Button() {
  // 🟡 레거시 버전 (권장하지 않음)
  return (
    <ThemeContext.Consumer>
      {(theme) => <button className={theme} />}
    </ThemeContext.Consumer>
  );
}
```

<br>

- 이 방식은 여전히 작동하지만 **`useContext()`를 사용하여 컨텍스트를 참조하는 방법을 권장합니다.**

<br>

```jsx
function Button() {
  // ✅ 권장
  const theme = useContext(ThemeContext);
  return <button className={theme} />;
}
```

<br>

### `Props`

- `children` (**함수**)
  - `useContext()`와 동일한 알고리즘에 의해 결정된 현재 컨텍스트 값을 활용하여 함수를 호출합니다.
  - 이후 함수에서 반환한 결과를 렌더링합니다.
  - `React`는 상위 컴포넌트의 컨텍스트가 변경될 때마다 이 함수를 다시 실행하고 UI를 업데이트합니다.

<br>

## 사용법

### 컨텍스트 생성하기

- 컨텍스트를 사용하면 컴포넌트가 `prop`을 명시적으로 전달하지 않고도 **정보를 깊숙이 전달**할 수 있습니다.

<br>

- 컴포넌트 외부에서 `createContext`를 호출하여 1개 이상의 컨텍스트를 생성합니다.

```jsx
import { createContext } from 'react';

const ThemeContext = createContext('light');
const AuthContext = createContext(null);
```

<br>

- `createContext`는 **컨텍스트 객체**를 반환합니다.
- 컴포넌트는 컨텍스트를 `useContext()`에 전달하여 컨텍스트를 **참조**합니다.

```jsx
function Button() {
  const theme = useContext(ThemeContext);
  // ...
}

function Profile() {
  const currentUser = useContext(AuthContext);
  // ...
}
```

<br>

- 기본적으로 `useContext`의 반환값은 컨텍스트를 만들 때 지정한 기본값입니다.
- 그러나 기본값은 변경되지 않으므로 그 자체로는 유용하지 않습니다.

<br>

- 컨텍스트는 컴포넌트에서 다른 **동적 값을 제공할 수 있기 때문에 유용**합니다.

```jsx
function App() {
  const [theme, setTheme] = useState('dark');
  const [currentUser, setCurrentUser] = useState({ name: 'Taylor' });

  // ...

  return (
    <ThemeContext.Provider value={theme}>
      <AuthContext.Provider value={currentUser}>
        <Page />
      </AuthContext.Provider>
    </ThemeContext.Provider>
  );
}
```

<br>

- `Page` 컴포넌트와 그 안에 있는 모든 컴포넌트는 아무리 깊은 `depth`이더라도 컨텍스트 값을 **참조할 수** 있습니다.
- 컨텍스트 값이 변경되면 `React`는 컨텍스트를 참조하는 컴포넌트도 리렌더링합니다.

<br>

### 파일에서 컨텍스트 `import` 및 `export`

- 다른 파일에 있는 컴포넌트가 동일한 컨텍스트를 참조하는 경우가 있습니다.
- 그러므로 컨텍스트를 별도 파일에 선언하는 것이 일반적입니다.
- `export` 문을 사용하여 다른 파일에서 컨텍스트를 사용할 수 있도록 설정합니다.

```jsx
// Contexts.js
import { createContext } from 'react';

export const ThemeContext = createContext('light');
export const AuthContext = createContext(null);
```

<br>

- 다른 파일에서 선언된 컴포넌트는 `import` 문을 사용하여 컨텍스트를 참조하거나 제공합니다.

<br>

```jsx
// Button.js
import { ThemeContext } from './Contexts.js';

function Button() {
  const theme = useContext(ThemeContext);
  // ...
}
```

```jsx
// App.js
import { ThemeContext, AuthContext } from './Contexts.js';

function App() {
  // ...
  return (
    <ThemeContext.Provider value={theme}>
      <AuthContext.Provider value={currentUser}>
        <Page />
      </AuthContext.Provider>
    </ThemeContext.Provider>
  );
}
```

- 컴포넌트 `import` 및 `export`와 유사하게 작동합니다.

<br>

## 트러블슈팅

### 컨텍스트 값을 변경하는 방법을 찾을 수 없습니다.

- 아래 코드는 기본 컨텍스트 값을 지정합니다.

```jsx
const ThemeContext = createContext('light');
```

<br>

- 위 기본값은 절대 변하지 않습니다.
- `React`는 상위 트리에서 일치하는 프로바이더를 찾을 수 없는 경우에만 이 값을 `fallback`으로 사용합니다.

<br>

- 컨텍스트를 변경하려면 컨텍스트 프로바이더에 `state`를 추가하고 컴포넌트를 래핑합니다.
