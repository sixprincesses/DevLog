## 백준 16953번

```jsx
const fs = require("fs");
const input = fs
  .readFileSync(process.platform === "linux" ? "/dev/stdin" : "./input.txt")
  .toString()
  .trim()
  .split("\n");

const [A, B] = input.shift().split(" ").map(Number);
const visited = new Set();

const BFS = () => {
  const queue = [A];
  visited.add(A);

  let cnt = 0;
  while (queue.length) {
    const size = queue.length;
    for (let i = 0; i < size; i++) {
      const v = queue.shift();

      if (v * 2 === B || v * 10 + 1 === B) {
        return cnt + 2;
      }

      if (!visited.has(v * 2) && v * 2 < B) {
        queue.push(v * 2);
        visited.add(v * 2);
      }

      if (!visited.has(v * 10 + 1) && v * 10 + 1 < B) {
        queue.push(v * 10 + 1);
        visited.add(v * 10 + 1);
      }

      if (!queue.length) {
        return -1;
      }
    }
    cnt++;
  }
};

const result = BFS();

console.log(result);
```

- BFS를 활용하여 풀었는데 다른 사람들의 풀이를 보니 B → A로 역순으로 풀면 더 간단하게 풀 수 있는 문제인 것 같다.

## 마크다운 문제 발생

- custom plugin에서 마크다운 언어가 안되는 문제
    - 아직 해결 못했음
- 줄바꿈이 안 먹는 문제
    - \n을 <br/>로 치환해주어서 해결
    
    ```jsx
    const gitPlugin = () => {
      const toHTMLRenderers = {
        Git(node: any) {
          let body = node.literal;
          // body = body.split("\n");
          // body = body.join("<br/>");
          body = body.replace(/\n/g, "<br/>");
          // 정규표현식 활용하여 치환
          return [
            {
              type: "openTag",
              tagName: "div",
              outerNewLine: true,
              attributes: { style: `color:hotpink; background-color:black` },
            },
            { type: "html", content: body },
            { type: "closeTag", tagName: "div", outerNewLine: true },
          ];
        },
      };
    
      return { toHTMLRenderers };
    };
    ```
    

## 회고

- 기능 구현 확인에 힘써서 아직 설계가 모자란 부분 존재 (피그마)
- 피그마 집중적으로 주말동안 마무리 해보도록 노력할 것
- 피그마 할 때, 커뮤니티 돌아다니며 UI Kit 모아 놓고 하는 거 추천

## CORS에러 발견되면 볼 것

[[CORS]access-control-allow-origin 오류 해결방법](https://developer111.tistory.com/18)
