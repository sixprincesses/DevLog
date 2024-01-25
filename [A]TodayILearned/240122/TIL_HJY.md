# Part1: array, object state 변경하는 법 ~ Part1: map : 많은 div들을 반복문으로 줄이고 싶은 충동이 들 때

## array, object state 변경하는 법

- array의 x번째 항목 변경
    
    ```jsx
    array자료[X] = '바꿀값'
    ```
    
- array, object 자료 다룰 때는 원본 데이터 조작 x ⇒ 기본값은 보존해주는 방식이 좋은 관습
    
    ```jsx
    let [글제목, 글제목변경] = useState(['남자 코트 추천', '강남 우동맛집', '파이썬 독학']);
    
    let copy = [...글제목]
    copy[0] = '여자 코트 추천';
    글제목변경(copy)
    ```
    
- state 변경함수 동작원리
    - 기존 state === 신규 state 검사 후 같으면 state 변경 x
    - array와 object의 식별자에는 배열과 객체의 값들이 아닌 주소를 저장
        - 복사한 array와 object의 값을 바꾸면 원본 array와 object의 값 또한 변경
    - 스프레드 연산자를 통하여 copy해주지 않으면 같은 주소를 참조하여서 변경함수 동작x
        
        ```jsx
        let copy = 글제목 // 같은 주소를 참조하게 됨
        let copy = [...글제목] // 다른 주소를 참조하게 됨
        ```
        

## Component : 많은 div들을 한 단어로 줄이고 싶으면

- html 코드 짤 때, return 안에 두 개의 html태그 나란히 적기 금지
    - 나란히 적고 싶으면 또 하나의 <div></div>로 감싸던가 <></>도 가능 (fragment 문법)
- Component ⇒ 축약한 HTML 덩어리
    - Component 만들 때, 주의점
        1. component 작명할 땐 영어 대문자로 보통 작명
        2. function App(){} 내부에서 만들면 안됨
            1. component 안에 component 만들지 않기
        3. <컴포넌트></컴포넌트> or <컴포넌트/> 둘 다 가능
- Component를 만들면 좋을 때,
    - 사이트에 반복적으로 출현하는 HTML 덩어리들
    - 내용이 매우 자주 변경될 것 같은 HTML 부분
    - 다른 페이지를 만들고 싶다면 그 페이지의 HTML 내용을 하나의 Component
    - 다른 팀원과 협업 시, 웹페이지를 Component 단위로 나눠서 작업을 분배도 가능
- Component의 단점
    - 자바스크립트의 변수 범위는 함수 단위이기 때문에 다른 Component의 변수 사용 불편

## 리액트 환경에서 동적인 UI 만드는 법 (모달창만들기)

- 리액트에서 동적인 UI 만드는 step
    1. html css로 미리 UI 디자인을 다 해놓고
    2. UI의 현재 상태를 state로 저장해두고
    3. state에 따라서 UI가 어떻게 보일지 조건문 등으로 작성
- JSX에서의 조건문 작성
    - JSX 안에서는 if else 문법 불가 ⇒ 삼항 연산자 사용
        
        ```jsx
        조건식 ? 조건식 참일 때 실행할 코드 : 조건식 거짓일 때 실행할 코드
        ```
        
    - html 남기기 싫을 때, null 자주 사용

## map : 많은 div들을 반복문으로 줄이고 싶은 충동이 들 때

- map 문법을 사용하여 반복문으로 코드를 간결하게 구현
