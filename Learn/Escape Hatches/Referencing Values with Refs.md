# Referencing Values with Refs

- 컴포넌트에 데이터를 저장하고 싶지만 리렌더링을 원하지 않을 때가 있습니다.
- 이때 `ref`를 사용할 수 있습니다.

## 배우게 될 것

- 컴포넌트에 `ref` 추가하는 방법
- `ref`의 값을 업데이트 하는 방법
- `ref`와 상태의 차이점
- `ref`를 안전하게 사용하는 방법

## 컴포넌트에 `ref` 추가하기

- `useRef`를 `import`하여 컴포넌트에 `ref`를 추가할 수 있습니다.

```js
import { useRef } from 'react';
```

<br>

- 컴포넌트 내부에서 `useRef` `Hook`을 호출하고 참조할 초기 값을 유일한 인수로 전달합니다.
- 예를 들어 값이 `0`인 `ref`가 있습니다.

```js
const ref = useRef(0);
```

<br>

- `ref`는 아래와 같은 객체를 반환합니다.

```js
{
  current: 0;
}
```

<br>

- `ref.current` 프로퍼티를 통해 `ref`의 현재 값을 참조할 수 있습니다.
- 이 값은 의도적으로 변경가능한 값이기에 읽기와 쓰기 모두 가능합니다.
- `React`가 관리하지 않는 컴포넌트의 비밀 주머니입니다.
- (단방향 데이터 흐름에서의 **escape hatch**입니다!)

<br>

- 아래 버튼은 클릭할때마다 `ref.current`를 증가합니다.

```js
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current += 1;
    alert(`You clicked  ${ref.current}  times!`);
  }

  return <button onClick={handleClick}>Click me!</button>;
}
```

<br>

- `ref`는 숫자를 가리킵니다.
- 하지만 상태처럼 문자열, 객체, 함수 등 어떤 것이든 가리킬 수 있습니다.
- 상태와는 다르게, `ref`는 읽기 쓰기가 모두 가능한 순수 `JS` 객체입니다.

<br>

- **버튼을 클릭할 때마다 컴포넌트는 리렌더링 되지 않습니다.**
- 상태와 같이 `ref`의 값은 리렌더링 동안 유지됩니다.
- 하지만 상태의 변경은 컴포넌트를 리렌더링합니다.
- 반면 `ref`의 변경은 리렌더링 하지 않습니다.

## 스톱워치 예제

- 단일 컴포넌트에서 `ref`와 상태를 조합하여 사용할 수 있습니다.
- 버튼을 눌러 시작하거나 정지할 수 있는 스톱워치를 만들어봅시다.
- `Start` 버튼을 누른 후 경과된 시간을 화면에 표시하려면 언제, 얼마나 `Start` 버튼을 눌렀는지 확인해야 합니다.
- **이 데이터는 렌더링에 사용되며 상태에서 관리됩니다.**

```js
const [startTime, setStartTime] = useState(null);
const [now, setNow] = useState(null);
```

<br>

- `Start` 버튼을 눌렀을 때, `10ms`마다 업데이트 하기 위해 `setInterval` 함수를 사용할 겁니다.

```js
import { useState } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);

  function handleStart() {
    // 카운팅 시작
    setStartTime(Date.now());
    setNow(Date.now());

    setInterval(() => {
      // 매 10ms마다 현재 시간 업데이트
      setNow(Date.now());
    }, 10);
  }

  const secondsPasses = startTime && now ? (now - startTime) / 1000 : 0;

  return (
    <>
      <h1>Time Passed: {secondPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>Start</button>
    </>
  );
}
```

<br>

- `Stop` 버튼이 클릭되었을 때는 `now` 상태의 업데이트를 중지하여 `interval` 함수의 동작을 취소해야 합니다.
- `clearInterval` 함수를 활용하면 됩니다.
- 하지만 이를 활용하려면 `Start` 버튼을 클릭할 때 `setInterval` 함수가 반환한 `interval ID`를 사용해야 합니다.
- 즉 `interval ID`를 저장해야 한다는 것입니다.
- `interval ID`는 렌더링 도중 필요하지 않기 때문에 `ref`로 관리합니다.

```js
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    // 아래 함수도 동일 기능
    // handleStop();
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  const secondsPassed = startTime && now ? (now - startTime) / 1000 : 0;

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>Start</button>
      <button onClick={handleStop}>Stop</button>
    </>
  );
}
```

<br>

- 렌더링 도중 사용되는 데이터는 상태로 관리됩니다.
- 리렌더링이 필요하지 않거나 이벤트 핸들러에서만 필요한 데이터는 `ref`를 사용하는 것이 효율적입니다.

## `ref` vs 상태

- `ref`를 상태보다 덜 엄격하다고 생각할 수 있습니다.
- 매번 새롭게 설정되어야 하는 상태 대신 **변경**할 수 있기 때문이죠.
- 하지만 대부분의 경우 상태를 사용하빈다.
- `ref`는 자주 사용되지 않는 **escape hatch**입니다.

| `refs`                                                                   | 상태                                                                                                      |
| ------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- |
| `useRef(initialValue)`는<br> `{current: initialValue}`<br>를 반환합니다. | `useState(initialValue)`는<br> 상태 변수의 초기값과 상태 변경 함수를 반환합니다.<br>(`[value, setValue]`) |
| 데이터를 변경해도 리렌더링하지 않습니다.                                 | 데이터를 변경한다면 리렌더링합니다.                                                                       |
| 변경 가능합니다.<br>`current` 값을 언제나 변경할 수 있습니다.            | 변경 불가능합니다.<br>변경하기 위해선 상태 설정 함수를 사용해야 합니다.                                   |
| 렌더링 도중엔 `current` 값을 읽거나 쓸 수 없습니다.                      | 언제나 상태를 참조할 수 있습니다.<br> 각 렌더링은 상태의 스냅샷을 기억합니다.                             |

<br>

- 아래 카운터 버튼은 상태로 구현되었습니다.

```js
import { useState } from 'react';

export default function Coutner() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <>
      <button onClick={handleClick}>You clicked {count} times</button>
    </>
  );
}
```

<br>

- `count` 값을 화면에 표시하기 때문에 상태를 사용하는 것이 합리적입니다.
- `Counter`의 값이 `setCount` 함수를 통해 변경된다면 `React`는 컴포넌트를 리렌더링하고 새로운 `count`를 화면에 표시합니다.
  <br><br>
- `ref`를 활용해 구현하는 경우 `React`는 컴포넌트를 리렌더링하지 않기 때문에 `count`의 변화를 확인할 수 없습니다.
- 아래 버튼을 클릭해도 **텍스트는 변하지 않습니다.**

```js
import { useRef } from 'react';

export default function Coutner() {
  const [countRef] = useRef(0);

  function handleClick() {
    // 컴포넌트를 리렌더링하지 않습니다.
    countRef.current += 1;
  }

  return (
    <>
      <button onClick={handleClick}>You clicked {countRef} times</button>
    </>
  );
}
```

- 렌더링 도중 `ref.current`를 참조하면 신뢰할 수 없는 코드가 되는 이유입니다.

## Deep Dive - `useRef`는 어떻게 동작하나요?

- `useState`, `useRef` 모두 `React`에서 제공되는 함수지만 일반적으로 `useRef`는 `useState` **상위에 구현됩니다.**
- `React` 내부를 상상해본다면 `useRef`는 아래와 같습니다.

```js
// React 내부
function useRef(initialValue) {
  const [ref, unused] = useState({ current: initialValue });
  return ref;
}
```

- 첫번째 렌더링에서, `useRef`는 `{current: initialValue}`를 반환합니다.
- 이 객체는 `React`에 저장되어 있기에 다음 렌더링 동안 같은 객체가 반환됩니다.
- 이 예제에선 상태 설정 함수가 사용되지 않는 방식을 확인하세요.
- 상태 설정 함수가 필요없는 이유는 `useRef` 함수는 항상 같은 객체를 반환해야 하기 때문입니다.
  <br><br>
- `React`는 `useRef` 함수가 기본 내장되어 있습니다.
- 하지만 `useRef`를 상태 설정 함수가 없는 상태라고 생각할 수 있습니다.
- 객체 지향 프로그래밍에 익숙하다면, `refs`는 인스턴스 필드라고 생각할 수 있습니다.

## `refs`를 사용할 때

- 컴포넌트가 `React` **외부** API와 통신해야 할 때 `ref`를 사용합니다.
- (컴포넌트의 시각화에 영향을 미치지 않는 브라우저 API)
- 아래는 일부 특별한 상황입니다.

1. `timeout ID`를 저장할 때
2. 다음 페이지를 표현하는 `DOM` 엘리먼트르를 조작하거나 저장할 때
3. `JSX` 구성에 필요하지 않은 객체를 저장할 때

- 컴포넌트가 어떤 값을 저장해 사용해야 한다면 렌더링 로직과 충돌하지 않는 방법은 `refs`를 사용하는 것입니다.

## 좋은 refs 사용 예시

- 아래 규칙을 따르면 예측 가능한 컴포넌트를 만들 수 있습니다.

1. `refs`를 **escape hatch**처럼 다루세요.

   - `refs`는 외부 시스템이나 브라우저 API에서 사용하는 것이 유용합니다.
   - `refs`에 부여된 어플리케이션 로직과 데이터 흐름이 너무 많다면 접근 방식을 바꿔야 합니다.

2. 렌더링 도중 `ref.current`를 읽거나 쓰지 마세요.

   - 렌더링 도중 데이터가 필요하다면 **상태**를 사용하면 됩니다.
   - `React`는 `ref.current`가 언제 바뀌는지 모르기 때문에, 렌더링하는 동안 데이터를 참조하는 것도 컴포넌트의 동작을 예측 불가능하게 만듭니다.
   - (`if (!ref.current) ref.current = new Thing()`과 같은 코드는 렌더링 중 1번만 실행되기 때문에 예외적으로 괜찮습니다.)
     <br><br>

- `React`의 상태 제한이 `refs`에 적용되지는 않습니다.
- 예를 들어 상태는 **모든 렌더링에 대해 스냅샷을 사용**하고, **동기적으로 업데이트되지 않는 것**처럼 동작합니다.
- 하지만 `ref.current`를 변경한다면 `ref`의 값은 즉시 변경합니다.

```js
ref.current = 5;
console.log(ref.current); // 5
```

<br>

- `refs`는 그 자체로 `JS`의 객체이기 때문입니다.
  <br><br>
- 또한 `refs`를 사용하면서 변경에 대해 걱정할 필요 없습니다.
- 변경된 객체가 렌더링에 사용되지 않는다면 `React`는 `ref`에 대해 상관하지 않습니다.

## refs와 DOM

- `ref`는 어떤 값을 할당받을 수 있습니다.
- 하지만 `ref`는 일반적으로 `DOM` 요소 접근에 사용됩니다.
- 예를 들어 `input`을 프로그래밍 관점에서 접근하는 경우 유용합니다.
- `<div ref={myRef}>`와 같이 `JSX`의 어트리뷰트에 `ref`를 전달할 때 `React`는 `DOM` 요소를 `myRef.current`에 할당합니다.

## 요약

- `ref`는 렌더링에 사용되지 않는 값을 저장하는 **escape hatch**입니다.
- `ref`는 `current` 프로퍼티를 갖고 있는 순수 `JS` 객체이며 읽거나 쓸 수 있습니다.
- `useRef` `Hook`을 사용하여 `ref`를 사용합니다.
- 상태와 같이 `ref`는 컴포넌트의 렌더링 사이에서 데이터를 저장합니다.
- 상태와는 다르게 `ref.current`를 변경하는 것은 리렌더링 하지 않습니다.
- 렌더링 도중 `ref.current`를 읽거나 쓰면 안 됩니다.
  - 이는 컴포넌트의 예측 가능성을 보장하지 않습니다.
