# React Hooks : useRef

```
useRef 는 렌더링 할 필요 없이 값을 참조할 수 있는 리액트 훅이다.
```

| 파라미터     | 설명                                                                                                        |
| ------------ | ----------------------------------------------------------------------------------------------------------- |
| initialValue | ref 객체의 초기값으로 원하는 값을 지정할 수 있다. 모든 타입으로 지정할 수 있으며, 렌더링 이후에는 무시된다. |

| 반환값  | 설명                                                                                                                                                                                                                                                       |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| current | 사용자가 설정한 `initialValue` 로 초기화 되며, 추후 다른 값으로 변경이 가능하다. <br> `JSX node` 의 `ref` 속성에 ref 객체를 할당하게 된다면, 리액트는 `current` 속성에 JSX node 를 설정하게 된다. <br> **다음 렌더링에도 useRef 는 같은 객체를 반환한다.** |

## 주의사항

1. state 와 다르게 `rec.current` 속성은 변환이 가능하다. 단, **렌더링되는 오브젝트가 할당되어 있는 경우에는 변환시키지 말아야 한다.**

2. ref 객체는 자바스크립트 객체이기 때문에 `ref.current` **속성을 변화시킬 때 React는 변화**를 알지 못한다. 따라서 컴포넌트를 리랜더링 하지 않는다.

3. 컴포넌트를 예측 불가능하게 하기 때문에, 초기화 이외에 렌더링 도중 `ref.current` 속성을 **읽거나 쓰지 말아야 한다.**

4. `Strict Mode` 에서 리액트는 컴포넌트 함수를 두번 호출한다. 이 때 ref 객체는 두번 생성되지만, 하나는 버려진다. 컴포넌트 함수에 문자가 없다면 이런 상황은 아무런 영향을 주지 않는다.

## 사용법

1. 값 참조

```jsx
import { useRef } from "react";

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert("You clicked " + ref.current + " times!");
  }

  return <button onClick={handleClick}>Click me!</button>;
}
```

2. DOM 조작

```jsx
import { useRef } from "react";

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>Focus the input</button>
    </>
  );
}
```

3. ref 객체 재생성 방지

```jsx
function Video() {
  const playerRef = useRef(null);

  function getPlayer() {
    if (playerRef.current !== null) {
      return playerRef.current;
    }
    const player = new VideoPlayer();
    playerRef.current = player;
    return player;
  }

  // other codes...
}
```

## 커스텀 컴포넌트에서 ref 객체 불러오기

```jsx
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

기본적으로 컴포넌트는 `내부 ref` 를 `DOM node` 에 노출하지 않기 때문에, 커스텀 컴포넌트에서 ref를 지정하려고 하면 다음과 같은 에러가 나타난다.

```console
Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
```

따라서, 컴포넌트에 `forwardRef` 를 사용하여 컴포넌트의 참조를 얻을 수 있도록 해줘야 한다.

```jsx
// MyInput.jsx
import { forwardRef } from "react";

const MyInput = forwardRef(({ value, onChange }, ref) => {
  return <input value={value} onChange={onChange} ref={ref} />;
});
```
