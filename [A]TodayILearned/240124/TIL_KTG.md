# React Legacy : createRef

```
createRef 는 임의의 값을 포함할 수 있는 ref 객체를 생성한다.
```

!!주의 : `createRef` 는 일반적으로 `class components` 에서 사용되므로 `function components` 에서는 `useRef` 를 사용하는 것이 옳다.

| 참조          | 설명                                                                       |
| ------------- | -------------------------------------------------------------------------- |
| `createRef()` | `class component` 내부에서 `ref` 를 선언하기 위해 `createRef` 를 호출한다. |

| 파라미터 | 설명                           |
| -------- | ------------------------------ |
| -        | `createRef` 는 파라미터가 없다 |

| 반환값  | 설명                                                                                                                                                                             |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| current | 초기값으로 `null` 이 설정되며, 추후 다른 값으로 설정할 수 있다. <br> `JSX node` 의 `ref` 속성에 ref 객체를 할당하게 된다면, 리액트는 `current` 속성에 JSX node 를 설정하게 된다. |

## 주의사항

1. `createRef` 는 항상 다른 객체를 반환한다. 즉, `{ current: null }` 을 직접 타이핑 하는 것과 같은 효과이다.
2. `function component` 에서는 항상 같은 객체를 반환하는 `useRef` 를 사용하는 것이 옳다.
3. `const ref = useRef()` 는 `const [ref, _] = useState(() => createRef(null))` 과 같다.

## 사용법

1. `class component` 에서 ref 선언하기

```js
import { Component, createRef } from "react";

export default class Form extends Component {
  inputRef = createRef();

  handleClick = () => {
    this.inputRef.current.focus();
  };

  render() {
    return (
      <>
        <input ref={this.inputRef} />
        <button onClick={this.handleClick}>Focus the input</button>
      </>
    );
  }
}
```

## useRef 로 바꾸기

```js
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
