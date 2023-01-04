# Importing and Exporting Components

- 컴포넌트의 가장 큰 장점은 재사용성으로 컴포넌트를 조합해 또 다른 컴포넌트를 만들 수 있습니다.
- 컴포넌트를 여러 번 중첩하게 되면 다른 파일로 분리해야 하는 시점이 생깁니다.
- 이렇게 분리하면 나중에 파일을 찾기 더 쉽고 재사용하기 편리해집니다.

## Root 컴포넌트란

첫 컴포넌트에서 만든 `Profile` 컴포넌트와 `Gallery` 컴포넌트는 아래와 같이 렌더링 됩니다.

```jsx
function Profile() {
  return <img src='https://i.imgur.com/MK3eW3As.jpg' alt='Katherine Johnson' />;
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```jsx
img { margin: 0 10px 10px 0; height: 90px; }
```

- 이 예제의 모든 컴포넌트는 `App.js`라는 `root` 컴포넌트 파일에 존재합니다.
- `Create React App`에서는 앱 전체가 `src/App.js`에서 실행됩니다.
- 설정에 따라 `root` 컴포넌트가 다른 파일에 위치할 수도 있습니다.
- `Next.js`처럼 파일 기반의 라우팅 프레임워크는 페이지별로 `root` 컴포넌트가 다를 수 있습니다.

## 컴포넌트를 import 하거나 export 하는 방법

- 랜딩 화면을 변경하게 되어 과학자들이 아니라 과학책으로 변경하거나 프로필 사진을 다른 곳에서 사용하게 된다면 Gallery 컴포넌트와 Profile 컴포넌트를 root 컴포넌트가 아닌 다른 파일로 옮기는 게 좋습니다.
- 그렇게 변경하면 재사용성이 높아 컴포넌트를 모듈로 사용할 수 있습니다.
- 컴포넌트를 다른 파일로 이동하려면 세 가지 단계가 있습니다.

1. 컴포넌트를 추가할 `JS` 파일을 생성합니다.
2. 새로 만든 파일에서 함수 컴포넌트를 `export` 합니다. (default 또는 named export 방식을 사용합니다)
3. 컴포넌트를 사용할 파일에서 `import` 합니다. (적절한 방식을 선택해서 default 또는 named로 import 합니다)

- 아래 예제를 보면 `App.js` 파일에서 `Profile`과 `Gallery` 컴포넌트를 빼서 새로운 `Gallery.js` 파일로 옮겼습니다.
- 이제 `Gallery`는 `Gallery.js`에서 import 해서 사용할 수 있습니다.

```jsx
import Gallery from './Gallery.js';

export default function App() {
  return <Gallery />;
}
```

```jsx
// Gallary.js
function Profile() {
  return <img src='https://i.imgur.com/QIrZWGIs.jpg' alt='Alan L. Hart' />;
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```jsx
img { margin: 0 10px 10px 0; height: 90px; }
```

- 이제 이 예제에서는 컴포넌트들이 두 파일로 나뉘게 되었습니다.

1. `Gallery.js`:
   - `Profile` 컴포넌트를 정의하고 해당 파일에서만 사용되기 때문에 `export` 되지 않습니다.
   - **Default** 방식으로 `Gallery` 컴포넌트를 `export` 합니다.
2. `App.js`:
   - **Default** 방식으로 `Gallery`를 `Gallery.js`로부터 **import** 합니다.
   - `Root App` 컴포넌트를 **default** 방식으로 **export** 합니다.

가끔 `.js`와 같은 파일 확장자가 없는 때도 있습니다.

```js
import Gallery from './Gallery';
```

- React에서는 `'./Gallery.js'` 또는 `'./Gallery'` 둘 다 사용할 수 있지만 전자의 경우가 **native ES Modules** 사용 방법에 더 가깝습니다.

### Deep Dive

- Deafult vs named exports
  <br><br>

- `JavaScript`에서는 `default`와 `named exports`라는 두 가지 방법으로 값을 `export` 합니다.
- 지금까지는 `default export`만 사용했지만 두 방법 모두 1개 파일에서 사용할 수 있습니다.
- 다만 1개 파일에서는 하나의 `default export`만 존재할 수 있고 `named exports`는 **여러 개**가 존재할 수 있습니다.
  <br><br>

- export 방식에 따라 import 하는 방식이 정해집니다.
- **default export** 한 값을 **named import** 하면 에러가 발생합니다.
- 아래 표에는 각각의 경우의 문법이 정리되어 있습니다.

| 문법    | Export statement                      | Import statement                        |
| ------- | ------------------------------------- | --------------------------------------- |
| default | `export default function Button() {}` | `import Button from './button.js';`     |
| named   | `export function Button() {}`         | `import { Button } from './button.js';` |

- **default import**를 사용하는 경우 **다른 이름**으로 명명할 수 있습니다.
- 예를 들어 `import Banana from './button.js'` 라고 쓰더라도 같은 `default export` 값을 가져오게 됩니다.
- 반대로 **named import**를 사용할 때는 양쪽 파일의 이름이 **같아야 해서** **named import**라고 불립니다.

- 보편적으로 **1개 파일에서 1개 컴포넌트만 export** 할 때 **default export** 방식을 사용합니다.
- **여러 컴포넌트를 export** 할 경우엔 **named export** 방식을 사용합니다.
- 어떤 방식을 사용하든 컴포넌트와 파일의 이름을 의미 있게 명명하는 것은 중요합니다.
- `export default () => {}`처럼 이름 없는 컴포넌트는 나중에 디버깅하기 어렵기 때문에 권장하지 않습니다.
  <br><br>

## 한 파일에서 여러 컴포넌트를 import 하거나 export 하는 방법

- `Gallary` 전체가 아니라 `Profile` 1개만 사용하고 싶을 때 `Profile` 컴포넌트만 export 하면 됩니다.
- 하지만 `Gallery.js` 파일에는 이미 하나의 **default export**가 존재하기 때문의 두 개의 **default export**를 정의할 수 없습니다.
- 이런 경우 새로운 파일 하나를 더 생성해서 **default export**를 사용하거나 **named export**로 `Profile` 컴포넌트를 export 할 수 있습니다.
- 한 파일에서는 단 하나의 default export만 사용할 수 있지만 named export는 여러 번 사용할 수 있습니다.
  <br><br>

먼저 **named export** 방식을 사용해서 `Gallery.js` 파일에서 `Profile` 컴포넌트를 export 합니다.  
(default 키워드를 사용하지 않습니다)

```js
export function Profile() {
  // something...
}
```

<br>

- 이후 `named import`를 사용합니다. (중괄호 사용)
- `Gallary.js`에 존재하는 `Profile`을 `App.js`로 `import`합니다.

```js
import { Profile } from './Gallery.js';
```

<br>

- 마지막으로, `App` 컴포넌트에 존재하는 `<Profile />`을 렌더링합니다.

```js
export default function App() {
  return <Profile />;
}
```

<br>

- 현재 `Gallery.js`는 2개의 `exports`를 소유하고 있습니다.
- (**default** `Gallary`, **named** `Profile`)
- `App.js`는 그 2개를 `import`합니다.

```js
// App.js
import Gallery from './Gallery.js';
import { Profile } from './Gallery.js';

export default function App() {
  return <Profile />;
}
```

```js
// Gallary.js
export function Profile() {
  return <img src='https://i.imgur.com/QIrZWGIs.jpg' alt='Alan L. Hart' />;
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

```js
img { margin: 0 10px 10px 0; height: 90px; }
```

<br>

- 이제 **default export**와 **named export**를 함께 사용하고 있습니다.

* `Gallery.js`:
  - **named export** 방식으로 `Profile`이라는 이름의 컴포넌트를 export 합니다.
  - **default export** 방식으로 `Gallery` 컴포넌트를 export 합니다.
* `App.js`:
  - `Gallery.js`에서 **named import** 방식으로 `Profile` 컴포넌트를 import 합니다.
  - `Gallery.js`에서 **default import** 방식으로 `Gallery` 컴포넌트를 import 합니다.
  - **Default export** 방식으로 `App` 컴포넌트를 export 합니다.

## 요약

이 페이지에서 배운 내용:

- 루트 구성 요소 파일이란?
- 구성 요소를 가져오고 내보내는 방법
- 기본 및 명명된 가져오기 및 내보내기를 사용하는 시기와 방법
- 동일한 파일에서 여러 구성 요소를 내보내는 방법
