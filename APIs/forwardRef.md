# forwardRef

- `forwardRef`를 사용하면 컴포넌트가 `ref`를 사용하여 상위 컴포넌트에 DOM 노드를 노출할 수 있습니다.

```jsx
const SomeComponent = forwardRef(render);
```

<br>

## 레퍼런스

### `forwardRef(render)`

- `ref`를 하위 컴포넌트로 전달하려면 `forwardRef`를 사용합니다.

<br>

### 매개변수

- `render`
  - 컴포넌트의 렌더링 함수입니다.
  - `React`는 상위 컴포넌트에서 받은 `props`와 `ref`를 가지고 이 함수를 호출합니다.

<br>

### 반환값

- `forwardRef`는 `JSX`에서 렌더링할 수 있는 `React` 컴포넌트를 반환합니다.
- 일반 함수로 정의된 `React` 컴포넌트와 달리 `forwardRef`가 반환하는 컴포넌트는 `ref` 프로퍼티를 수신할 수 있습니다.

<br>

### 주의사항

- `Strict Mode`에서는 **실수로 발생한 부수효과를 찾기 위해** `React`가 **함수를 두 번 호출**합니다.
  - 개발 모드에서만 이뤄지며 배포 환경에는 영향을 미치지 않습니다.
  - 함수가 순수하다면 영향을 미치지 않습니다.
  - 함수의 결과 중 1개는 무시됩니다.

<br>

## `render` 함수

- `forwardRef`는 렌더 함수를 인수로 받습니다.
- `React`는 이 함수를 `props`, `ref`와 함께 호출합니다.

```jsx
const MyInput = forwardRef(function MyInput(props, ref) {
  return (
    <label>
      {props.label}
      <input ref={ref} />
    </label>
  );
});
```

<br>

### 매개변수

- `props`
  - 상위 컴포넌트가 전달한 `prop`입니다.
- `ref`
  - 상위 컴포넌트가 전달한 `ref` 어트리뷰트입니다.
  - `ref`는 객체나 함수입니다.
  - 상위 컴포넌트가 `ref`를 전달하지 않은 경우 `null`입니다.
  - 전달 받은 `ref`를 다른 컴포넌트에 전달하거나 `useImperativeHandle`에 전달해야 합니다.

<br>

### 반환값

- `forwardRef`는 `JSX`에서 렌더링할 수 있는 `React` 컴포넌트를 반환합니다.
- 일반 함수로 정의된 `React` 컴포넌트와 달리 `forwardRef`가 반환하는 컴포넌트는 `ref` 프로퍼티를 수신할 수 있습니다.

<br>

## 사용법

### 상위 컴포넌트에 DOM 노드 노출하기

- 기본적으로 각 컴포넌트의 DOM 노드는 비공개입니다.
- 하지만 상위 컴포넌트에 DOM 노드를 노출하는 것이 유용할 수 있습니다.
- 이 때 컴포넌트를 `forwardRef`로 래핑합니다.

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} />
    </label>
  );
});
```

<br>

- `props` 다음에 2번째 인수로 `ref`를 받게 됩니다.
- 이를 노출하려는 DOM 노드에 전달합니다.

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});
```

<br>

- 상위 `Form` 컴포넌트가 `MyInput`의 `<input>` 노드에 접근할 수 있습니다.

```jsx
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label='Enter your name:' ref={ref} />
      <button type='button' onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

<br>

- `Form` 컴포넌트는 `MyInput` 컴포넌트에 `ref`를 전달합니다.
- `MyInput` 컴포넌트는 해당 `ref`를 `<input>` 태그에 전달합니다.
- 그 결과 `Form` 컴포넌트는 해당 `<input>` 태그의 `focus`를 호출할 수 있습니다.

<br>

- 컴포넌트 내부 DOM 노드에 `ref`를 노출하면 컴포넌트의 내부를 변경하기가 더 어려워진다는 점에 유의하세요.
- 일반적으로 `button`이나 `input`과 같이 재사용 가능한 Low 레벨(저수준) DOM 노드를 노출하지만 `Avatar`나 `Comment`와 같은 애플리케이션 레벨 노드는 노출하지 않습니다.

<br>

## 여러 컴포넌트를 통과하여 `ref` 전달

- `ref`를 DOM 노드로 전달하지 않고 `MyInput` 컴포넌트로 전달할 수 있습니다.

```jsx
const FormField = forwardRef(function FormField(props, ref) {
  // ...
  return (
    <>
      <MyInput ref={ref} />
      ...
    </>
  );
});
```

<br>

- `MyInput` 컴포넌트가 `<input>`에 대한 `ref`를 전달하면 `FormField`에 대한 `ref`가 해당 `<input>`을 제공합니다.

```jsx
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <FormField label='Enter your name:' ref={ref} isRequired={true} />
      <button type='button' onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

<br>

- `Form` 컴포넌트는 `ref`를 정의하고 이를 `FormField`에 전달합니다.
- `FormField` 컴포넌트는 해당 `ref`를 `MyInput`에 전달하고 `ref`는 브라우저 `<input>` DOM 노드로 전달합니다.
- 이것이 `Form`이 해당 DOM 노드에 접근하는 방식입니다.

```jsx
// App.js
import { useRef } from 'react';
import FormField from './FormField.js';

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <FormField label='Enter your name:' ref={ref} isRequired={true} />
      <button type='button' onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

```jsx
// FormField.js
import { forwardRef, useState } from 'react';
import MyInput from './MyInput.js';

const FormField = forwardRef(function FormField({ label, isRequired }, ref) {
  const [value, setValue] = useState('');
  return (
    <>
      <MyInput
        ref={ref}
        label={label}
        value={value}
        onChange={(e) => setValue(e.target.value)}
      />
      {isRequired && value === '' && <i>Required</i>}
    </>
  );
});

export default FormField;
```

```jsx
// MyInput.js
import { forwardRef } from 'react';

const MyInput = forwardRef((props, ref) => {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});

export default MyInput;
```

<br>

### DOM 노드 대신 명령형 핸들 노출하기

- 전체 DOM 노드를 노출하는 대신 **제한된** 메서드 집합을 사용하여 **명령형 핸들(커스텀 객체)**을 노출할 수 있습니다.
- 이렇게 하려면 DOM 노드를 보유할 별도의 `ref`를 정의해야 합니다.

```jsx
const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  // ...

  return <input {...props} ref={inputRef} />;
});
```

<br>

- 전달 받은 `ref`를 [`useImperativeHandle`](https://github.com/Collection50/React/blob/master/Hooks/useImperativeHandle.md)에 전달하고 `ref`에 노출할 값을 지정합니다.

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        focus() {
          inputRef.current.focus();
        },
        scrollIntoView() {
          inputRef.current.scrollIntoView();
        },
      };
    },
    []
  );

  return <input {...props} ref={inputRef} />;
});
```

<br>

- 일부 컴포넌트가 `MyInput`에 대한 `ref`를 받으면 DOM 노드 대신 `{ focus, scrollIntoView }` 객체만 전달 받습니다.
- 이를 통해 DOM 노드에 노출하는 정보를 최소한으로 제한할 수 있습니다.

```jsx
// App.js
import { useRef } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
    // 아래 특성은 DOM 노드가 노출되지 않기 때문에 작동하지 않습니다.
    // ref.current.style.opacity = 0.5;
  }

  return (
    <form>
      <MyInput placeholder='Enter your name' ref={ref} />
      <button type='button' onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

```jsx
// MyInput.js
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        focus() {
          inputRef.current.focus();
        },
        scrollIntoView() {
          inputRef.current.scrollIntoView();
        },
      };
    },
    []
  );

  return <input {...props} ref={inputRef} />;
});

export default MyInput;
```

<br>

### 주의

- `ref`를 과도하게 사용하지 않습니다.
- 스크롤, 포커싱, 애니메이션 트리거, 텍스트 선택 등 `prop`으로 표현할 수 없는 필수적인 동작에만 `ref`를 사용합니다.

<br>

- `prop`으로 표현할 수 있는 것은 `ref`를 사용하면 안 됩니다.
- 예를 들어 `Modal` 컴포넌트에서 `{ open, close }`와 같은 명령형 핸들링을 대신 `<Modal isOpen={isOpen} />`와 같이 `prop`을 사용하는 것이 좋습니다.
- `useEffect`를 사용하면 `prop`을 통해 필수 동작을 노출할 수 있습니다.

<br>

## 트러블슈팅

### 컴포넌트가 `forwardRef`로 래핑되어 있지만 `ref`는 항상 `null`입니다.

- `ref`의 사용을 잊어버렸음을 의미합니다.

<br>

- 예를 들어 아래 컴포넌트는 `ref`로 아무 작업도 하지 않습니다.

```jsx
const MyInput = forwardRef(function MyInput({ label }, ref) {
  return (
    <label>
      {label}
      <input />
    </label>
  );
});
```

<br>

- 해결하려면 `ref`를 DOM 노드나 다른 컴포넌트로 전달합니다.

```jsx
const MyInput = forwardRef(function MyInput({ label }, ref) {
  return (
    <label>
      {label}
      <input ref={ref} />
    </label>
  );
});
```

<br>

- 조건부 렌더링인 경우 `MyInput`에 대한 참조가 `null`일 수도 있습니다.

```jsx
const MyInput = forwardRef(function MyInput({ label, showInput }, ref) {
  return (
    <label>
      {label}
      {showInput && <input ref={ref} />}
    </label>
  );
});
```

<br>

- `showInput`이 `false`면 `ref`가 어떤 노드로도 전달되지 않으며 `MyInput`에 대한 `ref`는 비어 있게 됩니다.
- 아래 예제의 `Panel`처럼 조건문이 다른 컴포넌트 안에 숨겨져 있는 경우 실수하기 쉽습니다.

```jsx
const MyInput = forwardRef(function MyInput({ label, showInput }, ref) {
  return (
    <label>
      {label}
      <Panel isExpanded={showInput}>
        <input ref={ref} />
      </Panel>
    </label>
  );
});
```
