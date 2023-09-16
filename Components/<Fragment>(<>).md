# `<Fragment>` (<>...</>)

- `<>...</>` 문법을 통해 자주 사용되는 `<Fragment>`를 사용하면 래퍼 노드 없이 그룹화할 수 있습니다.

```jsx
<>
  <OneChild />
  <AnotherChild />
</>
```

<br>

## 레퍼런스

### `<Fragment>`

- 단일 요소가 필요한 상황에서 요소를 `<Fragment>`로 그룹화할 수 있습니다.
- `<Fragment>`를 사용하여 그룹화해도 DOM에는 아무런 영향을 미치지 않으며 그룹화하지 않은 것과 동일합니다.
- 빈 `JSX` 태그 `<></>`는 `<Fragment></Fragment>`의 축약어입니다.

<br>

### `props`

- `key` (옵션)
  - `<Fragment>`는 `key`를 가질 수 있습니다.

<br>

### 주의사항

- `Fragment`에 `key`를 전달하려면 `<>...</>`를 사용하면 안 됩니다.
  - `react`에서 `Fragment`를 `import`하여 `<Fragment key={yourKey}>...</Fragment>`를 렌더링합니다.
- `React`는 아래 2개 경우에서 상태를 재설정하지 않습니다.
  1. `<><Child /></>`에서 `[<Child />]`로 이동하거나 뒤로가기하는 경우
  2. `<><Child /></>`에서 `<Child />`로 이동했다가 뒤로가기하는 경우
  - 위와 같은 동작은 1단계 `depth`에서만 작동합니다.
  - 예를 들어 `<><><Child /></></>`에서 `<Child />`로 이동하면 상태가 재설정됩니다.
  - 정확한 동작은 [여기](https://gist.github.com/clemmy/b3ef00f9507909429d8aa0d3ee4f986b)를 참조합니다.

<br>

## 사용법

### 여러 요소 반환

- `Fragment` 또는 `<>...</>` 문법을 사용하여 여러 요소를 그룹화합니다.
- 모든 위치에 여러 요소를 넣을 수 있습니다.
- 컴포넌트는 하나의 요소만 반환할 수 있지만 `Fragment`를 사용하면 여러 요소를 그룹화하여 반환할 수 있습니다.

```jsx
function Post() {
  return (
    <>
      <PostTitle />
      <PostBody />
    </>
  );
}
```

<br>

- `Fragment`가 유용한 이유는 `Fragment`로 그룹화해도 레이아웃이나 스타일에 영향을 미치지 않기 때문입니다.
- 개발자 도구를 살펴보면 `<h1>`, `<article>`과 같은 모든 DOM 노드가 래퍼 없이 표시됩니다.

```jsx
export default function Blog() {
  return (
    <>
      <Post title='An update' body="It's been a while since I posted..." />
      <Post title='My new blog' body='I am starting a new blog!' />
    </>
  );
}

function Post({ title, body }) {
  return (
    <>
      <PostTitle title={title} />
      <PostBody body={body} />
    </>
  );
}

function PostTitle({ title }) {
  return <h1>{title}</h1>;
}

function PostBody({ body }) {
  return (
    <article>
      <p>{body}</p>
    </article>
  );
}
```

<br>

## Deep Dive - 특별한 문법 없이 `Fragment`를 작성하는 방법은 무엇인가요?

- 위 예시는 `React`에서 `Fragment`를 `import`하는 것과 같습니다.

```jsx
import { Fragment } from 'react';

function Post() {
  return (
    <Fragment>
      <PostTitle />
      <PostBody />
    </Fragment>
  );
}
```

<br>

- 일반적으로 `Fragment`에 `key`를 전달하는 경우가 아니라면 필요하지 않습니다.

<br>

### 변수에 여러 요소 할당하기

- `Fragment` 요소를 변수에 할당하고 `prop`으로 전달하는 작업을 수행할 수 있습니다.

```jsx
function CloseDialog() {
  const buttons = (
    <>
      <OKButton />
      <CancelButton />
    </>
  );
  return (
    <AlertDialog buttons={buttons}>
      Are you sure you want to leave this page?
    </AlertDialog>
  );
}
```

<br>

### 텍스트와 함께 요소 그룹화하기

- `Fragment`를 사용하여 텍스트와 컴포넌트를 함께 그룹화합니다.

```jsx
function DateRangePicker({ start, end }) {
  return (
    <>
      From
      <DatePicker date={start} />
      to
      <DatePicker date={end} />
    </>
  );
}
```

<br>

### `Fragment` 배열 렌더링

- 아래의 코드는 `<></>` 문법을 사용하는 대신 `Fragment`를 명시적으로 작성해야 하는 경우입니다.
- 반복문에서 여러 요소를 렌더링할 때는 `key`를 할당합니다.
- 반복문의 `Fragment`에 `key`를 할당하는 경우 아래와 같이 사용합니다.

```jsx
function Blog() {
  return posts.map((post) => (
    <Fragment key={post.id}>
      <PostTitle title={post.title} />
      <PostBody body={post.body} />
    </Fragment>
  ));
}
```

<br>

- DOM을 검사하여 `Fragment`의 하위 컴포넌트에 래퍼 요소가 없는지 확인할 수 있습니다.

```jsx
import { Fragment } from 'react';

const posts = [
  { id: 1, title: 'An update', body: "It's been a while since I posted..." },
  { id: 2, title: 'My new blog', body: 'I am starting a new blog!' },
];

export default function Blog() {
  return posts.map((post) => (
    <Fragment key={post.id}>
      <PostTitle title={post.title} />
      <PostBody body={post.body} />
    </Fragment>
  ));
}

function PostTitle({ title }) {
  return <h1>{title}</h1>;
}

function PostBody({ body }) {
  return (
    <article>
      <p>{body}</p>
    </article>
  );
}
```
