# useId

- `useId`는 접근성 어트리뷰트의 고유 **ID**를 생성하는 `React` `hook`입니다.

```jsx
const id = useId();
```

<br>

## 레퍼런스

- 컴포넌트 최상단에서 `useId`를 호출하여 고유 **ID**를 생성합니다.

```jsx
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  // ...
}
```

<br>

### 매개변수

- `useId`는 매개변수를 갖지 않습니다.

<br>

### 반환값

- 특정 컴포넌트에서 특정 `useId` 호출과 연관된 고유 **ID** 문자열을 반환합니다.

<br>

### 주의사항

- `useId`는 `hook`이므로 **컴포넌트 최상단**이나 커스텀 `hook`에서만 호출할 수 있습니다.
- 반복문, 조건문에서는 호출할 수 없습니다.
- 필요한 경우 컴포넌트를 추출하고 상태를 옮깁니다.

<br>

- `useId`를 배열의 **`key` 생성에 사용하면 안 됩니다.**
- `key`는 데이터에 존재해야 합니다.

<br>

## 사용법

### 접근성 어트리뷰트에 대한 고유 ID 생성하기

- 컴포넌트 최상단에서 `useId`를 호출하여 고유 **ID**를 생성합니다.

```jsx
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  // ...
}
```

<br>

- 생성된 **ID**를 다른 어트리뷰트에 전달합니다.

```jsx
<>
  <input type="password" aria-describedby={passwordHintId} />
  <p id={passwordHintId}>
</>
```

<br>

- 유용하게 사용되는 예시를 살펴보겠습니다.

<br>

- `aria-describedby`와 같은 `HTML` 접근성 어트리뷰트를 사용하면 두 태그가 관련되어 있음을 지정합니다.
- 예를 들어 `input`태그를 `p` 태그가 설명하도록 지정할 수 있습니다.

<br>

- 일반 `HTML`에서는 다음과 같이 작성합니다.

```jsx
<label>
  Password:
  <input
    type="password"
    aria-describedby="password-hint"
  />
</label>
<p id="password-hint">
  The password should contain at least 18 characters
</p>
```

<br>

- 하지만 이와 같이 **ID**를 하드코딩하는 것은 좋지 않습니다.
- 컴포넌트는 페이지에서 2번 이상 렌더링될 수 있지만 **ID**는 고유해야 합니다!
- **ID**를 하드코딩하는 대신 `useId`로 고유한 **ID**를 생성합니다.

```jsx
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  return (
    <>
      <label>
        Password:
        <input type='password' aria-describedby={passwordHintId} />
      </label>
      <p id={passwordHintId}>
        The password should contain at least 18 characters
      </p>
    </>
  );
}
```

<br>

- 아래와 같이 `PasswordField`가 여러 번 렌더링되어도 생성된 **ID**는 고유합니다.

<br>

```jsx
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  return (
    <>
      <label>
        Password:
        <input type='password' aria-describedby={passwordHintId} />
      </label>
      <p id={passwordHintId}>
        The password should contain at least 18 characters
      </p>
    </>
  );
}

export default function App() {
  return (
    <>
      <h2>Choose password</h2>
      <PasswordField />
      <h2>Confirm password</h2>
      <PasswordField />
    </>
  );
}
```

<br>

### 주의사항

- `SSR`의 경우 `useId`를 사용하려면 서버와 클라이언트에 동일한 컴포넌트 트리가 필요합니다.
- 서버와 클라이언트에서 렌더링하는 트리가 일치하지 않으면 생성된 **ID**가 일치하지 않기 때문입니다.

<br>

## Deep Dive - `useId`가 증감 연산자보다 나은 이유는 무엇인가요?

- `useId`가 `nextId++`와 같은 전역 변수 증감보다 나은 이유를 설명하겠습니다.

<br>

- `useId`의 가장 큰 장점은 `SSR`을 사용할 수 있다는 것입니다.
- `SSR` 중에 컴포넌트는 HTML을 생성합니다.
- 클라이언트에서는 생성된 HTML에 이벤트 핸들러를 첨부하는 `hydration`이 이뤄집니다.
- `hydration`되려면 클라이언트 HTML과 서버 HTML이 일치해야 합니다.

<br>

- 증감 연산자를 사용하는 경우 클라이언트에서 `hydration`되는 순서와 서버 HTML의 순서가 일치하지 않을 수 있으므로 불안정합니다.
- `useId`는 `hydration` 작동 여부를 확인할 수 있고 서버와 클라이언트 간에 출력이 일치하는지도 확인할 수 있습니다.

<br>

- `useId`는 호출하는 컴포넌트의 **상위 경로**에서 생성됩니다.
- 따라서 클라이언트와 서버 트리가 동일하면 렌더링 순서에 관계없이 **상위 경로**가 일치합니다.

<br>

### 관련 요소에 대한 ID 생성

- 여러 개의 관련 요소에 **ID**를 부여해야 하는 경우 `useId`를 호출하여 공유 접두사를 생성할 수 있습니다.

```jsx
import { useId } from 'react';

export default function Form() {
  const id = useId();
  return (
    <form>
      <label htmlFor={id + '-firstName'}>First Name:</label>
      <input id={id + '-firstName'} type='text' />
      <hr />
      <label htmlFor={id + '-lastName'}>Last Name:</label>
      <input id={id + '-lastName'} type='text' />
    </form>
  );
}
```

<br>

- 이렇게 하면 고유 **ID**가 필요한 모든 요소에 대해 `useId`를 호출하지 않아도 됩니다.

<br>

### 생성된 모든 **ID**에 공유 접두사 지정하기

- 1개 페이지에서 여러 개의 독립적인 `React` 애플리케이션을 렌더링하는 경우 `createRoot` or `hydrateRoot`에 `identifierPrefix`를 옵션으로 전달합니다.
- 이렇게 하면 `useId`로 생성된 모든 식별자가 고유 접두사로 시작하므로 서로 다른 앱에서 생성된 **ID**가 충돌하지 않습니다.

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>My app</title>
  </head>
  <body>
    <div id="root1"></div>
    <div id="root2"></div>
  </body>
</html>
```

```jsx
// App.js
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  console.log('Generated identifier:', passwordHintId);
  return (
    <>
      <label>
        Password:
        <input type='password' aria-describedby={passwordHintId} />
      </label>
      <p id={passwordHintId}>
        The password should contain at least 18 characters
      </p>
    </>
  );
}

export default function App() {
  return (
    <>
      <h2>Choose password</h2>
      <PasswordField />
    </>
  );
}
```

```jsx
// Index.js
import { createRoot } from 'react-dom/client';
import App from './App.js';
import './styles.css';

const root1 = createRoot(document.getElementById('root1'), {
  identifierPrefix: 'my-first-app-',
});
root1.render(<App />);

const root2 = createRoot(document.getElementById('root2'), {
  identifierPrefix: 'my-second-app-',
});
root2.render(<App />);
```
