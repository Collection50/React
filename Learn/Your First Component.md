# Your First Component

- 컴포넌트 === `React`의 핵심 개념 중 하나
- 사용자 인터페이스(UI)를 구축하는 기반

## 컴포넌트: UI 빌딩 블록

- `HTML`의 기본 태그 집합을 사용하여 구조화된 문서 생성 가능

```html
<article>
  <h1>My First Component</h1>
  <ol>
    <li>Components: UI Building Blocks</li>
    <li>Defining a Component</li>
    <li>Using a Component</li>
  </ol>
</article>
```

- 위 `HTML`은 `<article>, <h1>, <ol>, <li>`으로 표현되어 있다
- `CSS`, `JavaScript와` 결합된 마크업은 웹의 모든 UI 뒤에 존재한다
  <br><br>

- `React`를 사용하면 `HTML`, `CSS`, `JavaScript`를 `컴포넌트`로 결합하여
- 모든 페이지에서 렌더링할 수 있는 `<TableOfContents />` 컴포넌트로 변환 가능
- 내부적으로는 여전히 `<article>`, `<h1>` 등과 같은 `HTML` 태그 사용
- `HTML` 태그와 마찬가지로 컴포넌트를 활용하여 전체 페이지를 디자인 가능

```html
<PageLayout>
  <NavigationHeader>
    <SearchBar />
    <Link to="/docs">Docs</Link>
  </NavigationHeader>
  <Sidebar />
  <PageContent>
    <TableOfContents />
    <DocumentationText />
  </PageContent>
</PageLayout>
```

## 컴포넌트 정의

- 전통적으로 웹에서는 콘텐츠에 일부 `JavaScript`를 사용하여 상호 작용 해왔습니다.
- `React`는 동일한 기술을 사용하면서 상호 작용을 우선시합니다.
- `React` 컴포넌트는 마크업을 뿌릴 수 있는 `JavaScript` 기능입니다.

```jsx
export default function Profile() {
  return <img src='https://i.imgur.com/MK3eW3Am.jpg' alt='Katherine Johnson' />;
}
```

### 1단계 컴포넌트 내보내기

- `export default` 접두사는 표준 JavaScript 구문입니다.
- 다른 파일에서 가져올 수 있도록 파일의 기본 기능을 표시할 수 있습니다.

### 2단계 함수 정의

- `Profile` 이름으로 `JavaScript` 함수를 정의합니다.
- 주의점: `React` 컴포넌트는 일반 `JavaScript` 함수이지만 이름은 대문자로 시작해야 합니다.
- 그렇지 않으면 작동하지 않습니다!

### 3단계 마크업 추가

- 컴포넌트는 `src`, `alt`, `<img />`를 반환합니다.
- `<img />`는 `HTML`처럼 작성되었지만 실제로는 `JavaScript`입니다!
- 이 문법을 `JSX` 라고 하며 `JavaScript` 내부에 마크업을 삽입할 수 있습니다.
- 반환문을 한 줄에 작성할 수 있습니다.

```jsx
return <img src='https://i.imgur.com/MK3eW3As.jpg' alt='Katherine Johnson' />;
```

- 그러나 마크업이 모두 키워드와 같은 줄에 있지 않으면 다음과 return같이 한 쌍의 괄호로 묶어야 합니다.

```jsx
return (
  <div>
    <img src='https://i.imgur.com/MK3eW3As.jpg' alt='Katherine Johnson' />
  </div>
);
```

- 주의: 괄호가 없으면 모든 코드는 `return`이 무시됩니다!

## 컴포넌트 사용

- `Profile` 컴포넌트를 정의 했으므로 다른 컴포넌트 안에 중첩할 수 있습니다.
- 예를 들어 여러 개의 `Profile` 컴포넌트를 사용하는 `Gallery`를 외부로 내보낼 수 있습니다.

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

## 브라우저가 보는 것

- 대소문자에서 차이점을 확인할 수 있습니다.
  <br><br>

- `<section>`은 소문자이므로 우리가 `HTML` 태그를 사용하는 것을 알고 있습니다.
- `<Profile />`는 대문자로 시작하므로 `Profile`이라는 **컴포넌트**를 사용하는 것을 알고 있습니다.
- 그리고 `Profile`은 더 많은 `<img />` `HTML`을 포함합니다.
- 결국 브라우저에 표시되는 내용은 다음과 같습니다.

```jsx
<section>
  <h1>Amazing scientists</h1>
  <img src='https://i.imgur.com/MK3eW3As.jpg' alt='Katherine Johnson' />
  <img src='https://i.imgur.com/MK3eW3As.jpg' alt='Katherine Johnson' />
  <img src='https://i.imgur.com/MK3eW3As.jpg' alt='Katherine Johnson' />
</section>
```

## 컴포넌트 중첩 및 구성

- 컴포넌트는 `JavaScript` 함수이므로 동일한 파일에 여러 컴포넌트를 보관할 수 있습니다.
- 컴포넌트가 상대적으로 작거나 밀접하게 관련되어 있을 때 편리합니다.
- `Profile`은 파일이 복잡해지면 언제든지 별도의 파일로 이동할 수 있습니다.
  <br><br>

- `Profile` 컴포넌트는 `Gallery` 내부에서 렌더링 됩니다.
- 따라서 `Gallery` 컴포넌트는 `Profile` 컴포넌트를 **자식**으로 렌더링 하는 상위 컴포넌트입니다.
- 이것은 `React` 마법의 일부입니다.
- 컴포넌트는 여러 위치에서 사용할 수 있습니다.
  <br><br>

- 주의: 컴포넌트는 다른 컴포넌트를 렌더링할 수 있지만 함수 정의를 중첩해서는 안 됩니다.

```jsx
export default function Gallery() {
  // 🔴 다른 컴포넌트 안에 또 다른 컴포넌트를 정의하지 마세요!
  function Profile() {
    // ...
  }
  // ...
}

export default function Gallery() {
  // ...
}

// ✅ 모든 컴포넌트는 최상위 레벨에서 정의됩니다.
function Profile() {
  // ...
}
```

- 하위 컴포넌트가 상위 컴포넌트의 데이터를 필요로 하는 경우 중첩 정의 대신 `props`로 전달합니다.

## Deep Dive

- `React` 앱은 **root** 컴포넌트에서 시작합니다.
- 루트 컴포넌트는 일반적으로 새 프로젝트를 시작할 때 자동으로 생성됩니다.
- 예를 들어 `CodeSandbox` 또는 `Create React App`을 사용하는 경우 루트 컴포넌트는 `src/App.js`에 정의됩니다.
- `Next.js` 프레임워크를 사용하는경우 루트 컴포넌트는 `pages/index.js`에 정의됩니다
- 이 예에서는 루트 컴포넌트를 `exporting` 했습니다.
  <br><br>

- 대부분의 `React` 앱은 컴포넌트를 맨 아래까지(`All the way down`) 사용합니다.
- 이는 버튼과 같은 재사용 가능한 부분에 컴포넌트를 사용할 뿐만 아니라
- 사이드바, 목록 등의 전체 페이지와 같은 큰 부분에도 컴포넌트를 사용하게 됩니다!
  <br><br>

- 컴포넌트는 한 번만 사용되는 경우에도 UI 코드 및 마크업을 구성하는 편리한 방법입니다.
- `Next.js`와 같은 프레임워크는 이를 한 단계 더 발전시킵니다.
- 빈 `HTML` 파일을 사용하고 `React`가 `JavaScript`로 페이지를 관리하도록 하는 방법 대신 (`before`)
- `React` 컴포넌트에서 자동으로 `HTML`을 생성합니다. (`after`)
- 이렇게 하면 `JavaScript` 코드가 로드되기 전, 앱에서 일부 콘텐츠를 표시할 수 있습니다.
  <br><br>

- 여전히 많은 웹사이트는 `React`를 사용하여 상호작용을 첨가합니다.
- 전체 페이지에 대한 단일 컴포넌트 대신 많은 루트 컴포넌트가 있습니다.
- 필요한 만큼 `React`를 사용할 수 있습니다.

## 요약

- `React`를 사용하면 재사용 가능한 UI 컴포넌트를 만들 수 있습니다.
- 모든 UI는 컴포넌트입니다.
- `React` 컴포넌트는 다음 특징을 제외하면 일반 `JavaScript` 함수입니다.
  1. 이름은 항상 **대문자**로 시작합니다.
  2. `JSX` **마크업을 반환**합니다.
