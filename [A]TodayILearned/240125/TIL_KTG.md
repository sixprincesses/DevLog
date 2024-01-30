# React APIs : forwardRef

```
forwardRef 를 사용하면 ref 를 이용해서 부모 컴포넌트에게 컴포넌트의 DOM node 를 노출시킬 수 있다.
```

| 참조                 | 설명                                                                                       |
| -------------------- | ------------------------------------------------------------------------------------------ |
| `forwardRef(render)` | `forawrdRef()` 를 호출하여 컴포넌트에 ref 를 받아 자식 컴포넌트에게 전달할 수 있도록 한다. |

| 파라미터 | 설명                                                                                                                                         |
| -------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `render` | 컴포넌트를 렌더링 하기 위한 함수. 리액트는 컴포넌트가 부모에게 받은 `props` 와 `ref` 를 함께 호출한다. 반환된 JSX 가 컴포넌트의 출력이 된다. |

| 반환값          | 설명                                                                                                                                               |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| React component | JSX 형태로 렌더링한 컴포넌트를 반환한다. 일반적인 함수로 만든 React 컴포넌트와 다르게 `forwardRef` 로 반환된 컴포넌트는 `ref` 를 상속받을 수 있다. |

## 주의사항

1. 개발 중에 `Strict Mode` 에서는 React 가 렌더링 함수를 두번 호출한다. 만약 함수에 문제가 없다면 한번의 호출은 무시되기 때문에, 컴포넌트의 로직에 아무런 이상이 없을 것이다.

## `render` 함수

`forwardRef` 함수는 React 에서 `props`, `ref` 와 함께 호출하는 `render` 함수를 인자로 받는다.

```js
const MyInput = forwardRef(function MyInput(props, ref) {
  return (
    <label>
      {props.label}
      <input ref={ref} />
    </label>
  );
});
```

| 파라미터 | 설명                                                                                                                                                                                                            |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `props`  | 부모 컴포넌트로 부터 상속받은 인자.                                                                                                                                                                             |
| `ref`    | `ref` 는 객체나 함수이고, 부모 컴포넌트로 부터 상속받는다. 만약 부모 컴포넌트가 `ref` 를 넘기지 않았다면, `null` 이 된다. 상속받은 `ref` 는 다른 컴포넌트에 다시 넘기거나, `useImperativeHandle`에 넘겨야 한다. |

| 반환값 | 설명                                                                                                                                                                 |
| ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|        | `forwardRef` 는 JSX로 랜더링 할 수 있는 React 컴포넌트를 반환한다. 일반적인 React 컴포넌트와 다르게 `forwardRef` 에 의해 반환된 컴포넌트는 `ref`를 상속받을 수 있다. |

## 사용법

1. 부모 요소에게 DOM node 노출시키기

```js
// MyInput.js
import { forwardRef } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
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

```js
// App.js
import { useRef } from "react";
import MyInput from "./MyInput.js";

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

2. 여러 컴포넌트 사이에 `ref` 포워딩 하기

```js
// MyInput.js
import { forwardRef } from "react";

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

```js
// FormField.js
import { forwardRef, useState } from "react";
import MyInput from "./MyInput.js";

const FormField = forwardRef(function FormField({ label, isRequired }, ref) {
  const [value, setValue] = useState("");
  return (
    <>
      <MyInput
        ref={ref}
        label={label}
        value={value}
        onChange={(e) => setValue(e.target.value)}
      />
      {isRequired && value === "" && <i>Required</i>}
    </>
  );
});

export default FormField;
```

```js
// App.js
import { useRef } from "react";
import FormField from "./FormField.js";

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <FormField label="Enter your name:" ref={ref} isRequired={true} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

3. DOM node 대신 `imperative handle` 노출시키기

```js
// MyInput.js
import { forwardRef, useRef, useImperativeHandle } from "react";

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

```js
// App.js
import { useRef } from "react";
import MyInput from "./MyInput.js";

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
    // This won't work because the DOM node isn't exposed:
    // ref.current.style.opacity = 0.5;
  }

  return (
    <form>
      <MyInput placeholder="Enter your name" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```
