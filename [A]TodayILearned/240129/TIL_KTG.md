# React, Ref 전달하기

## Ref 란?

리액트에서 render 시 생성된 DOM element에 접근하는 방법을 제공하는 것.

### Ref 사용 예시

1. UI 관리(Focus, 텍스트 선택, 미디어 재생 등)
2. 애니메이션 시작 유도
3. Third Party DOM 라이브러리 통합

_주의사항 : 대부분의 경우, ref 를 통해 특정 동작이 되도록 하는 것 보다 상위 컴포넌트에서 state 를 통해 관리하는 것이 맞으므로, ref 를 사용하기 전에 필요성을 반드시 체크해야 한다._

### Ref 만들기

1. useRef

2. createRef
