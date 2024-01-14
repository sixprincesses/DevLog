## Recoil

- react 상태관리를 위한 라이브러리

[](https://seungahhong.github.io/blog/2022/03/2022-03-22-recoil/)

- 설치
    
    ```jsx
    npm install recoil
    ```
    
- atom
    - 상태(state)의 일부
    - useRecoilState()를 통해 읽고 쓰기 가능
        
        ```jsx
        import { atom } from 'recoil'
        
        export type Todo = {
          key: number
          content: string
          isCompleted: boolean
        }
        
        export const todosState = atom<Todo[]>({
          key: 'todosState',
          default: [
            {
              key: 0,
              content: '예제 입니다',
              isCompleted: false,
            },
          ],
        })
        ```
        
    - 타입스크립트에서의 타입과 인터페이스
        
        [타입스크립트 type과 interface의 공통점과 차이점](https://yceffort.kr/2021/03/typescript-interface-vs-type)
        
- selector
    - Vue에서의 computed같은 역할
    - selector는 쉽게 말해서 기존에 선언한 atom을 전/후 처리하여 새로운 값을 리턴하거나 기존 atom의 값을 수정하는 역할을 수행한다. 그리고 참조한 atom의 값이 최신화되면 자동으로 selector의 값도 최신화하므로, 관리하기에도 간편하다.
        
        [React) Recoil selector 만져보기](https://velog.io/@2ast/React-Recoil-selector-만져보기)
        
    - 순수함수
        
        [순수함수란?](https://velog.io/@chdb57/ㅇㅇㅇㅇㅇㅇㅇㅇ)
        
    
    ```jsx
    import { selector } from 'recoil'
    import { todosState } from './atoms'
    
    export const unCompletedTodos = selector({
      key: 'unCompletedTodos',
      get: ({ get }) => {
        const todos = get(todosState)
        return todos.filter((todo) => !todo.isCompleted)
      },
    })
    ```
    
- 로직의 분리
    
    ```jsx
    // useTodos.ts => 컴포넌트에서는 UI, UX적인 문제만 최대한 다루도록 노력
    import { useRecoilState, useRecoilValue } from 'recoil'
    import { todosState } from './atoms'
    import { unCompletedTodos } from './selectors'
    
    const useTodos = () => {
      const [todos, setTodos] = useRecoilState(todosState)
      const uct = useRecoilValue(unCompletedTodos)
    
      const onClickTodoAddButton = () => {
        setTodos((prev) => {
          return [
            ...prev,
            {
              key: Math.random() * 2424,
              content: `예제 입니다`,
              isCompleted: true,
            },
          ]
        })
      }
    
      return {
        todos,
        setTodos,
        onClickTodoAddButton,
        unCompletedTodos: uct,
      }
    }
    
    export default useTodos
    ```
    

## useState의 원리 (약식)

```jsx
let todosState: any[] = ['바보']

const render = () => {}

const setTodos = (params: ((prevState: any[]) => void) | any[]) => {
  if (typeof params === 'function') {
		// function일 경우에는 인자로 state value를 넘겨줌
    const newState = params(todosState)
    todosState = [...todosState, newState]
    render()
  } else {
    const newState = params
    todosState = [...todosState, newState]
    render()
  }
}

setTodos((prevState) => {
  return […prevState, {}]
})
```
