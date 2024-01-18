## 백준15651번

```jsx
const fs = require("fs");
const input = fs
  .readFileSync(
    process.platform === "linux"
      ? "/dev/stdin"
      : "./input.txt"
  )
  .toString()
  .trim()
  .split("\n");

const [N, M] = input.shift().split(" ").map(Number);

perm = [];
result = [];

const Perm = (depth) => {
  if (depth === M) {
    result.push([...perm]);
    return;
  }

  for (let i = 1; i <= N; i++) {
    perm.push(i);
    Perm(depth + 1);
    perm.pop();
  }
};

Perm(0);

console.log(result.map((line) => line.join(" ")).join("\n"));
```

- 전형적인 순열 문제로 따로 특별한 것은 없었다.

## React

- 왜 어렵나?
    - 새로운 언어라는 막연함
        - 개발자로서 피할 수 없는 것으로 받아들일 줄 알아야 함
        - 리액트만의 이야기가 아님
    - 문법의 어색함
        - html과 크게 보면 비슷한 vue와 비교하여 js에 좀더 비슷한 형태를 가짐
- CRA
    - babel, webpack과 같은 필수적인 툴들이 포함되어 있어서 사용
    - npx create-react-app (project 명)
- Node.js
    - NPM을 쓰기 위하여 설치 ⇒ 최신 버전 유지 권장
- 실습
    - 변수로부터 값을 가져오려면 중괄호{}로 묶어줌
    - 훅
        - useState
            - [객체, 대체값]으로 구성
            - 대체값으로만 변경
    - props(property)
        - 파라미터 형태로 값을 전달
- mui.com, DataGrid
    - 그리드 구현을 도와주는 라이브러리

## ToastUI editor ⇒ viewer 값 주기

- initialValue로는 안됨

[React - 버튼클릭으로 원하는 컴포넌트 렌더링하기](https://velog.io/@moolbum/체크한것-컴포넌트-렌더하기)

- 위 방식으로 구현 한 번은 가능
    
    ```jsx
    const [testString, setTestString] = useState();
    
    const handleSave = () => {
      const markDownContent = editorRef.current?.getInstance().getMarkdown();
      if (markDownContent.length < 10) {
        editorRef.current?.getInstance().focus(); // editor에 focus
        return;
      }
      console.log(markDownContent);
      setTestString(markDownContent);
    };
    
    // 중략
    
    {testString && <Viewer initialValue={testString} plugins={[gitPlugin]} />}
    ```
    
    - testString의 boolean 값을 활용하여 렌더링 시점을 컨트롤 할 수 있다
        - 이 방법도 한 번밖에 안 되어서 새로운 데이터를 보낼 때마다 새로 할 수 있는 방법이 있는지 생각해봐야 됨
