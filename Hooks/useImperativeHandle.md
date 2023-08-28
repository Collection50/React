# useImperativeHandle

- `useImperativeHandle`은 `ref`의 핸들링을 커스터마이징할 수 있는 `React` `hook`입니다.

```jsx
useImperativeHandle(ref, createHandle, dependencies?)
```

<br>

## 레퍼런스

### `useImperativeHandle(ref, createHandle, dependencies?)`

- 컴포넌트 최상단에서 `useImperativeHandle`을 호출하여 `ref` 핸들링을 커스터마이징합니다.

```jsx
import { forwardRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  useImperativeHandle(
    ref,
    () => {
      return {
        // ... 함수 ...
      };
    },
    []
  );
  // ...
});
```

<br>

### 매개변수

- `ref`
  - 위의 `forwardRef` 함수에서 2번째 인수인 `ref`입니다.
- `createHandle`
  - 인수를 받지 않고 `ref` 핸들링을 반환하는 함수입니다.
  - `ref` 핸들링은 모든 값을 가질 수 있습니다.
  - 일반적으로 메서드가 있는 객체를 반환합니다.
- `dependencies`(옵션)
  - `createHandle` 코드 내부에서 참조하는 모든 반응형 값의 목록입니다.
  - 반응형 값에는 `prop`, state, 컴포넌트에서 선언된 모든 변수와 함수가 포함됩니다.
  - 린터 사용 시 모든 반응형 값이 종속성에 올바르게 지정되었는지 확인합니다.
  - 종속성 목록에는 일정한 수의 항목이 있어야 하며 `[dep1, dep2, dep3]`와 같이 인라인으로 작성해야 합니다.
  - `Object.is` 메서드를 사용하여 비교합니다.
  - 리렌더링으로 인해 종속성이 변경되었거나 이 매개변수를 생략한 경우 `createHandle` 함수가 다시 실행되고 새로 생성된 핸들링이 `ref`에 할당됩니다.

<br>

### 반환값

- `useImperativeHandle`은 `undefined`를 반환합니다.

<br>

## 사용법

### 상위 컴포넌트에 커스텀 `ref` 핸들링 노출하기

- 기본적으로 상위 컴포넌트에 DOM 노드를 노출하지 않습니다.
- 예를 들어 `MyInput`의 상위 컴포넌트가 `<input>` DOM 노드에 액세스할 수 있도록 하려면 `forwardRef`를 사용해야 합니다.

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  return <input {...props} ref={ref} />;
});
```

<br>

- 위 코드에서 **`MyInput`의 `ref`에는 `<input>` DOM 노드가 할당됩니다.**
- 하지만 커스텀 값을 노출할 수 있습니다.
- 노출된 핸들링을 커스터마이징하려면 `useImperativeHandle`을 사용합니다.

```jsx
import { forwardRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  useImperativeHandle(
    ref,
    () => {
      return {
        // ... your methods ...
      };
    },
    []
  );

  return <input {...props} />;
});
```

<br>

- 위 코드는 `ref`가 더 이상 `<input>`으로 전달되지 않습니다.

<br>

- 예를 들어 모든 `<input>` 태그를 노출하지 않고 `focus`와 `scrollIntoView`만 노출한다고 가정해 보겠습니다.
- 실제 브라우저 DOM을 별도의 `ref`에 보관해야 합니다.
- 이후 `useImperativeHandle`을 사용하여 상위 컴포넌트가 호출할 메서드만 포함된 핸들링을 노출합니다.

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

- 이제 상위 컴포넌트가 `MyInput`에 대한 `ref`를 받으면 `focus`와 `scrollIntoView` 메서드를 호출할 수 있습니다.
- 하지만 `<input>` DOM 노드에 대한 모든 권한은 갖지 못합니다.

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
      <MyInput label='Enter your name:' ref={ref} />
      <button type='button' onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

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

export default MyInput;
```

<br>

### 나만의 명령형 메서드 노출하기

- 명령형 핸들링을 통해 노출하는 메서드가 DOM 메서드일 필요는 없습니다.
- 아래 `Post` 컴포넌트는 명령형 핸들링을 통해 `scrollAndFocusAddComment` 메서드를 노출합니다.
- 버튼을 클릭하면 상위 `Page`가 댓글 목록을 스크롤하고 `input` 태그에 `focus`할 수 있습니다.

```jsx
// App.js
import { useRef } from 'react';
import Post from './Post.js';

export default function Page() {
  const postRef = useRef(null);

  function handleClick() {
    postRef.current.scrollAndFocusAddComment();
  }

  return (
    <>
      <button onClick={handleClick}>Write a comment</button>
      <Post ref={postRef} />
    </>
  );
}
```

```jsx
// Post.js
import { forwardRef, useRef, useImperativeHandle } from 'react';
import CommentList from './CommentList.js';
import AddComment from './AddComment.js';

const Post = forwardRef((props, ref) => {
  const commentsRef = useRef(null);
  const addCommentRef = useRef(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        scrollAndFocusAddComment() {
          commentsRef.current.scrollToBottom();
          addCommentRef.current.focus();
        },
      };
    },
    []
  );

  return (
    <>
      <article>
        <p>Welcome to my blog!</p>
      </article>
      <CommentList ref={commentsRef} />
      <AddComment ref={addCommentRef} />
    </>
  );
});

export default Post;
```

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const CommentList = forwardRef(function CommentList(props, ref) {
  const divRef = useRef(null);

  useImperativeHandle(
    ref,
    () => {
      return {
        scrollToBottom() {
          const node = divRef.current;
          node.scrollTop = node.scrollHeight;
        },
      };
    },
    []
  );

  let comments = [];
  for (let i = 0; i < 50; i++) {
    comments.push(<p key={i}>Comment #{i}</p>);
  }

  return (
    <div className='CommentList' ref={divRef}>
      {comments}
    </div>
  );
});

export default CommentList;
```

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const AddComment = forwardRef(function AddComment(props, ref) {
  return <input placeholder='Add comment...' ref={ref} />;
});

export default AddComment;
```

<br>

### 주의

- **`ref`를 과도하게 사용하지 않습니다.**
- 노드로 스크롤하기, 노드에 포커스 맞추기, 애니메이션 트리거하기, 텍스트 선택하기 등 `prop`으로 표현할 수 없는 필수적인 동작에만 `ref`를 사용합니다.

<br>

- `prop`으로 표현할 수 있는 것은 `ref`를 사용하면 안 됩니다.
- 예를 들어 `Modal` 컴포넌트에서 `{ open, close }`와 같은 명령형 핸들링을 대신 `<Modal isOpen={isOpen} />`와 같이 `prop`을 사용하는 것이 좋습니다.
- `useEffect`를 사용하면 `prop`을 통해 필수 동작을 노출할 수 있습니다.
