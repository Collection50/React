# Writing Markup with JSX

- `JSX`는 `JavaScript`를 확장한 문법으로, `JavaScript` 파일을 `HTML`과 비슷하게 마크업을 작성할 수 있도록 해줍니다.
- 컴포넌트를 작성하는 다른 방법도 있지만, 대부분의 `React` 개발자는 `JSX`의 간결함을 선호하며, 대부분의 코드베이스에서도 `JSX`를 사용합니다.

## 배우게 될 것

- `React`에서 렌더링 로직과 마크업을 함께 사용하는 이유
- `HTML`과 `JSX`의 차이점
- `JSX`로 렌더링하는 방법

## JSX: JavaScript에 마크업 넣기

- Web은 `HTML`, `CSS`, `JavaScript`를 기반으로 만들어져왔습니다.
- 웹 개발자는 `HTML`로 내용을, `CSS`로 디자인을, `JavaScript`로 로직을 작성합니다.
- 보통은 `HTML`, `CSS`, `JavaScript`를 각각 분리된 파일로 관리합니다!
- 페이지 로직은 `JavaScript` 안에서 분리되어 동작하는 동안, `HTML` 안에서는 내용이 마크업됩니다.

| HTML              | JS                |
| ----------------- | ----------------- |
| `<div>`, `<head>` | `isLoggedIn() {}` |
| `<p>`, `<form>`   | `onClick() {}`    |

- 그러나 Web은 상호작용하는 작업이 점점 많아지면서, 로직에서 내용을 결정하는 일이 많아졌습니다.
- 즉 `JS`가 `HTML`을 동작하는 것을 의미합니다.
- 이것이 `React`에서 렌더링 로직과 마크업이 같은 곳에 함께 있게 된 이유입니다.
- 즉, **컴포넌트**에서 말이죠.

```js
// Sidebar.js 컴포넌트
Sidebar() {
  if (isLoggedInt()) {
    <p>Welcome</p>
  } else {
    <From />
  }
}
```

- 렌더링 로직과 마크업이 함께 있으면, 변화가 생길 때마다 서로 조화를 이룰 수 있습니다.
- 반대로, **버튼**의 마크업과 **사이드바**의 마크업처럼 서로 관련이 없는 항목끼리는 각각 자체적으로 변경하는 편이 더 안전하기 때문에 서로 분리되도록 합니다.
  <br><br>

- 각각의 `React` 컴포넌트는 브라우저에 마크업을 렌더링하는 `JavaScript` 함수입니다.
- `React` 컴포넌트는 `JSX`라는 확장된 문법을 사용하여 마크업을 표현합니다.
- `JSX`는 `HTML`과 매우 비슷하지만, 조금 더 엄격하고 **동적**인 정보를 표시할 수 있습니다.
- `JSX`를 이해하는 가장 좋은 방법은 `HTML` 마크업을 `JSX` 마크업으로 변환해보는 것입니다.
- NOTE: `JSX`와 `React`는 서로 다른 개념이기에, 각각 독립적으로 사용 가능합니다.

## HTML을 JSX로 변환하기

```html
<h1>Hedy Lamarr's Todos</h1>
<img src="https://i.imgur.com/yXOvdOSs.jpg" alt="Hedy Lamarr" class="photo" />
<ul>
  <li>Invent new traffic lights</li>
  <li>Rehearse a movie scene</li>
  <li>Improve the spectrum technology</li>
</ul>
```

- 이제 이것을 컴포넌트로 만들어볼 겁니다.

```js
export default function TodoList() {
  return (
    // something..
  )
}
```

- HTML을 그대로 복사하여 붙여넣는다면 동작하지 않을 겁니다.

```jsx
export default function TodoList() {
  return (
    // 이것은 동작하지 않습니다!
    <h1>Hedy Lamarr's Todos</h1>
    <img
      src="https://i.imgur.com/yXOvdOSs.jpg"
      alt="Hedy Lamarr"
      class="photo"
    >
    <ul>
      <li>Invent new traffic lights
      <li>Rehearse a movie scene
      <li>Improve the spectrum technology
    </ul>
  );
}
```

- 왜냐하면 `JSX`는 엄격한 규칙이 몇 가지 더 있기 때문입니다!

## JSX 규칙

### 1. 하나의 루트 엘리먼트로 반환하기

- 한 컴포넌트에서 여러 엘리먼트를 반환하려면, 하나의 부모 태그로 감싸야 합니다.
- 예를 들어, 다음과 같이 `<div>`를 사용할 수 있습니다.

```html
<div>
  <h1>Hedy Lamarr's Todos</h1>
  <img src="https://i.imgur.com/yXOvdOSs.jpg" alt="Hedy Lamarr" class="photo" />
  <ul>
    ...
  </ul>
</div>
```

- 마크업에 `<div>`를 추가하고 싶지 않다면, `<>`와 `</>`로 대체할 수 있습니다.

```html
<>
  <h1>Hedy Lamarr's Todos</h1>
  <img
    src="https://i.imgur.com/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
  >
  <ul>
    ...
  </ul>
</>
```

- 이 빈 태그를 *React fragment*라고 합니다.
- *React fragments*는 브라우저상의 `HTML` 트리 구조에서 **흔적을 남기지 않고 그룹화**해줍니다.
  <br><br>

### Deep Dive

- `JSX`는 `HTML`처럼 보이지만 내부적으로는 일반 `JS` 객체로 변환됩니다.
- 배열로 감싸지 않은 하나의 함수는 두 개의 객체를 반환할 수 없습니다.
- 따라서 또 다른 태그나 *fragment*로 감싸지 않으면 2개의 `JSX` 태그를 반환할 수 없습니다.

### 2. 모든 태그는 닫아주기

- `JSX`에서는 태그를 명시적으로 닫아야 합니다.
- `<img>`처럼 자동으로 닫아주는 태그는 반드시 `<img />` 형태로 작성해야 하며, 래핑 태그도 `<li>oranges</li> `형태로 작성해야 합니다.
  <br><br>
- 다음과 같이 Hedy Lamarr의 이미지와 리스트의 항목들을 닫아줍니다.

```js
<>
  <img src='https://i.imgur.com/yXOvdOSs.jpg' alt='Hedy Lamarr' class='photo' />
  <ul>
    <li>Invent new traffic lights</li>
    <li>Rehearse a movie scene</li>
    <li>Improve the spectrum technology</li>
  </ul>
</>
```

### 3. <s>모두</s> 대부분 카멜 케이스로!

- `JSX`는 `JS`로 바뀌고 `JSX`에서 작성된 어트리뷰트는 `JS` 객체의 **키**가 됩니다.
- 컴포넌트에서는 종종 어트리뷰트를 변수로 읽고 싶은 경우가 있습니다.
- 그러나 `JS`는 변수명에 제한이 있습니다.
- 예를 들면, 변수명에 대시를 포함하거나 `class`처럼 예약어를 사용할 수 없습니다.
  <br><br>
- 이것이 바로 `React`에서 `HTML`과 `SVG`의 어트리뷰트 대부분이 **카멜 케이스**로 작성되는 이유입니다.
- 예를 들면, `stroke-width` 대신 `strokeWidth`로 사용합니다.
- `class`는 예약어이기 때문에, `React`에서는 DOM의 프로퍼티에 대응한 이름을 따서 `className`으로 대신 작성합니다.

```js
<img
  src='https://i.imgur.com/yXOvdOSs.jpg'
  alt='Hedy Lamarr'
  className='photo'
/>
```

- 주의: 역사적 이유로, `aria-*`와 `data-*`의 어트리뷰트는 `HTML`과 동일하게 대시를 사용하여 작성합니다.

## 전문가 팁: JSX 변환기 사용하기

- 기존 마크업에서 모든 어트리뷰트를 변환하는 것은 지루할 수 있습니다!
- 변환기를 사용하여 `HTML`과 `SVG`를 `JSX`로 변환하는 것을 추천합니다.
- 변환기는 매우 유용하지만 그래도 `JSX`를 어떻게 쓰는지 이해하는 것도 중요합니다.

```js
// App.js
export default function TodoList() {
  return (
    <>
      <h1>Hedy Lamarr's Todos</h1>
      <img
        src='https://i.imgur.com/yXOvdOSs.jpg'
        alt='Hedy Lamarr'
        className='photo'
      />
      <ul>
        <li>Invent new traffic lights</li>
        <li>Rehearse a movie scene</li>
        <li>Improve the spectrum technology</li>
      </ul>
    </>
  );
}
```

## 요약

- `JSX`가 존재하는 이유와 컴포넌트에서 `JSX`를 사용하는 방법을 알았습니다.
- `React` 컴포넌트는 서로 관련된 마크업과 렌더링 로직을 함께 그룹화합니다.
- `JSX`는 HTML과 유사하지만 몇 가지 차이점이 있습니다.
- 필요한 경우 변환기를 사용할 수 있습니다.
- 오류 메시지는 종종 마크업을 수정하는 올바른 방향을 알려줍니다.
