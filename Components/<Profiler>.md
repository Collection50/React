# `<Profiler>`

- `<Profiler>`를 사용하면 프로그래밍 방식으로 `React` 트리의 렌더링 성능을 측정할 수 있습니다.

```jsx
<Profiler id='App' onRender={onRender}>
  <App />
</Profiler>
```

<br>

## 레퍼런스

### `<Profiler>`

- 컴포넌트 트리를 `<Profiler>`로 래핑하여 렌더링 성능을 측정합니다.

```jsx
<Profiler id='App' onRender={onRender}>
  <App />
</Profiler>
```

<br>

### `Props`

- `id`
  - 측정 중인 UI를 식별하는 문자열입니다.
- `onRender`
  - 프로파일링된 트리 내의 컴포넌트가 업데이트될 때마다` React`가 호출하는 **콜백** 함수입니다.
  - 렌더링 내용과 소요된 시간에 대한 정보를 받습니다.

<br>

### 주의사항

- 프로파일링은 약간의 오버헤드가 존재하므로 배포 환경에서는 비활성화되어 있습니다.
- 배포 환경에서 프로파일링을 사용하려면 특수 배포 빌드를 사용합니다.

<br>

## `onRender` 콜백 함수

- `React`는 렌더링 정보와 함께 `onRender` 콜백을 호출합니다.

```jsx
function onRender(
  id,
  phase,
  actualDuration,
  baseDuration,
  startTime,
  commitTime
) {
  // 렌더링 타이밍 집계 또는 기록...
}
```

<br>

### 매개변수

- `id`
  - 방금 커밋한 `<Profiler>` 트리의 문자열 아이디입니다.
  - 여러 프로파일러를 사용하는 경우 식별할 수 있습니다.
- `phase`
  - **마운트**, **업데이트**, **중첩 업데이트** 여부
  - 처음 마운트되었는지, 리렌더링되었는지 알 수 있습니다.
- `actualDuration`
  - `<Profiler>`와 하위 트리를 렌더링하는 데 소요된 시간(밀리초)입니다.
  - 서브 트리가 메모이제이션(`memo`, `useMemo`)을 얼마나 잘 사용하는지 나타냅니다.
  - 하위 컴포넌의 특정 `prop`이 변경되는 경우에만 리렌더링되기 때문에 이 값은 첫 마운트 이후에는 크게 감소합니다.
- `baseDuration`
  - 최적화 없이 전체 `<Profiler>` 하위 트리를 리렌더링하는 데 걸리는 시간입니다.
  - 각 컴포넌트의 최근 렌더링 시간을 합산합니다.
  - 이 값은 최악의 렌더링 비용(예: 첫 마운트 or 메모이제이션이 없는 트리)을 추정합니다.
  - 실제 지속 시간과 비교하여 메모이제이션이 작동하는지 확인합니다.
- `startTime`
  - `React`가 현재 업데이트 렌더링을 시작한 시점에 대한 타임스탬프입니다.
- `endTime`
  - `React`가 현재 업데이트를 커밋한 시점에 대한 타임스탬프입니다.
  - 이 값은 모든 프로파일러 간에 공유되므로 그룹화할 수 있습니다.

<br>

## 사용법

### 프로그래밍 방식으로 렌더링 성능 측정

- `<Profiler>` 컴포넌트를 래핑하여 렌더링 성능을 측정합니다.

```jsx
<App>
  <Profiler id='Sidebar' onRender={onRender}>
    <Sidebar />
  </Profiler>
  <PageContent />
</App>
```

<br>

- 트리 내의 컴포넌트가 업데이트를 **커밋**할 때마다 `id`와 `onRender` 콜백함수가 필요합니다.

<br>

### 참고

- `<Profiler>`를 사용하면 프로그래밍 방식으로 측정값을 수집할 수 있습니다.
- 대화형 프로파일러를 찾고 있다면 `React DevTools`의 프로파일러 탭을 사용합니다.

<br>

### 애플리케이션의 다양한 부분 측정

- 여러 `<Profiler>` 컴포넌트를 사용하여 애플리케이션의 여러 부분을 측정할 수 있습니다.

```jsx
<App>
  <Profiler id='Sidebar' onRender={onRender}>
    <Sidebar />
  </Profiler>
  <Profiler id='Content' onRender={onRender}>
    <Content />
  </Profiler>
</App>
```

<br>

- `<Profiler>` 컴포넌트를 중첩할 수도 있습니다.

```jsx
<App>
  <Profiler id='Sidebar' onRender={onRender}>
    <Sidebar />
  </Profiler>
  <Profiler id='Content' onRender={onRender}>
    <Content>
      <Profiler id='Editor' onRender={onRender}>
        <Editor />
      </Profiler>
      <Preview />
    </Content>
  </Profiler>
</App>
```

<br>

- `<Profiler>`는 가벼운 컴포넌트지만 필요한 경우에만 사용합니다.
- 사용할 때마다 약간의 CPU 및 메모리 오버헤드가 추가되기 떄문입니다.
