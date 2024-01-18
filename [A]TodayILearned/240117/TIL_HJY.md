## 백준 15666번

```jsx
const fs = require("fs");
const input = fs
  .readFileSync(process.platform === "linux" ? "/dev/stdin" : "./input.txt")
  .toString()
  .trim()
  .split("\n");

const [N, M] = input.shift().split(" ").map(Number);
const visited = new Array(9).fill(false);

const list = input.shift().split(" ").map(Number);

list.sort((a, b) => a - b);

let perm = [];
let result = new Set();

const Perm = (idx, depth) => {
  if (depth === M) {
    result.add([...perm].join(" "));
    return;
  }

  for (let i = idx; i < N; i++) {
    perm.push(list[i]);
    Perm(i, depth + 1);
    perm.pop();
  }
};

Perm(0, 0);

console.log(Array.from(result).join("\n"));
```

- Set에서 배열의 중복 체크가 안되어서 join 메서드를 통해 문자열로 추가하며 중복성 체크

[[React] 리액트 JSX에서 이미지가 엑박으로 나올 때](https://gomgomkim.tistory.com/11)

## JSX에 이미지 넣기

- Vue 때도 있었던 에러인데 경로를 img태그의 src로 등록하여 넣으면 엑박이 뜨는 현상이 자꾸 발생
    
    ```jsx
    <img src='./images/icon.png'/> // 엑박 발생
    
    // require 사용으로도 해결 안됨 => import 모듈 사용으로 해결
    import icon from './images/icon.png'
    <img src = {icon}/>
    ```
    

## React-icons

- 게시판 ui를 짜는 중에 아이콘을 사용해야 하는 상황이 생겨 사용하게 됨
- 설치
    
    ```jsx
    npm i react-icons --save
    ```
    
- 설치 후 홈페이지에 들어가서 마음에 드는 아이콘 선택 후 import하고 사용
    
    <img width="300" alt="Untitled" src="https://github.com/sixprincesses/DevLog/assets/87738361/e436a727-903b-43a6-8647-58fba4e1ba13">
    
    ```jsx
    import { MdArrowForwardIos } from "react-icons/md";
    
    // 중략
    return (
    	<div>
    	</div>
    	<MdArrowForwardIos />
    );
    ```
    

## TS 타입 지정

- 웬만해서는 타입지정 오류 발생 안 하지만 객체의 메서드나 프로퍼티의 접근 하는 일이 있으면 발생
    
    ```jsx
    const onChangeComment = (e: React.ChangeEvent<HTMLInputElement>) => {
      setComment(e.target.value);
    };
    
    // onChange의 툴팁을 읽어보면 React.ChangeEventHandler<HTMLInputElement>라고
    // 타입이 뜨지만 event를 받는 parameter의 타입은 Handler가 빠진 React.ChangeEvent임에 주의
    ```
