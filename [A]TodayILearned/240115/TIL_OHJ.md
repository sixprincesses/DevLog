# 2024.01.15 TIL

## 1. React Router 설정 방식 논의
    1. BrowserRouter 활용
    2. createBrowserRouter & RouterProvider 활용
- 1번 방식은 기존에 리액트에서 쭉 활용하던 방식이고, 2번 방식은 2023년에 새롭게 추가된 방식이다.
- 한눈에 보기엔 1번 방식이, 명시적으로 구분이 가는 건 2번 방식이라고 생각함
- 2번 방식을 채택하기로 결정

## 2. vscode 재설치
- 파일에서 'vscode'열기가 안 뜨는 현상을 해결하기 위해 vscode를 제거하고 다시 설치하며 설정을 건드림

## 3. vscode night owl theme 적용
- 타입스크립트에서 함수와 변수들을 더 명확히 구분하기 위해 vscode night owl theme을 적용
```
>color theme -> night owl
```

## 4. 무료 폰트 5가지 조사
- 조건: 한국어/영어 모두 적용, 무료여야함
```
@import url('https://fonts.cdnfonts.com/css/handwriting');
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@100;300;400;500;700&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Nanum+Pen+Script&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Rampart+One&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Caveat&display=swap');

```

## 5. 서비스 이름 고안
- Inked
- Queueing

## 6. 버튼 스타일
```

.btn {
  display: block;
  position: relative;
  float: left;
  width: 120px;
  padding: 0;
  margin: 10px 20px 10px 0;
  font-weight: 600;
  text-align: center;
  line-height: 50px;
  color: #8c5a76;
  background: #d9a9b6;
  border: none;
  border-radius: 5px;
  transition: all 0.2s;
  cursor: pointer;
}

.btn-fade:hover {
  color: #d9a9b6;
  background: #8c5a76;
}

.btn-float {
  background: none;
  box-shadow: 0px 0px 0px 0px rgba(0, 0, 0, 0.5);
}

.btn-float:before {
  content: "Float";
  display: block;
  position: absolute;
  top: 0;
  left: 0;
  width: 120px;
  height: 50px;
  border-radius: 5px;
  color: #d9a9b6;
  background: #8c5a76;
  transition: all 0.2s;
}

.btn-float:before {
  box-shadow: 0px 0px 0px 0px rgba(0, 0, 0, 0.4);
}

.btn-float:hover:before {
  margin-top: -2px;
  margin-left: 0px;
  transform: scale(1.1, 1.1);
  -ms-transform: scale(1.1, 1.1);
  -webkit-transform: scale(1.1, 1.1);
  box-shadow: 0px 5px 5px -2px rgba(0, 0, 0, 0.25);
}

.btn-push {
  box-shadow: 0px 5px 0px 0px #8c5a76;
}

.btn-push:hover {
  margin-top: 15px;
  margin-bottom: 5px;
  box-shadow: 0px 0px 0px 0px #8c5a76;
}

.btn-toggle {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 50px;
}

.play-btn {
  width: 30px;
}

.button {
  position: relative;
  padding: 0;
  width: 200px;
  height: 200px;
  border: 4px solid #888888;
  outline: none;
  background-color: #f4f5f6;
  border-radius: 40px;
  box-shadow: -6px -20px 35px #ffffff, -6px -10px 15px #ffffff, -20px 0px 30px #ffffff, 6px 20px 25px rgba(0, 0, 0, 0.2);
  transition: .13s ease-in-out;
  cursor: pointer;
  font-family: 'Handwriting', sans-serif;
}

.button:active {
  box-shadow: none;
}

.button:active .button__content {
  box-shadow: none;
  transform: translate3d(0px, 0px, 0px);
}

.button__content {
  position: relative;
  display: grid;
  padding: 20px;
  width: 80%;
  height: 80%;
  grid-template-columns: 1fr 1fr 1fr 1fr;
  grid-template-rows: 1fr 1fr;
  box-shadow: inset 0px -8px 0px #dddddd, 0px -8px 0px #f4f5f6;
  border-radius: 40px;
  transition: .13s ease-in-out;
  z-index: 1;
}

.button__icon {
  position: relative;
  display: flex;
  transform: translate3d(0px, -4px, 0px);
  grid-column: 4;
  align-self: start;
  justify-self: end;
  width: 32px;
  height: 32px;
  transition: .13s ease-in-out;
}

.button__icon svg {
  width: 32px;
  height: 32px;
  fill: #aaaaaa;
}

.button__text {
  position: relative;
  transform: translate3d(0px, -4px, 0px);
  margin: 0;
  align-self: end;
  grid-column: 1/5;
  grid-row: 2;
  text-align: center;
  font-size: 50px;
  background-color: #888888;
  color: transparent;
  text-shadow: 2px 2px 3px rgba(255, 255, 255, 0.5);
  -webkit-background-clip: text;
  -moz-background-clip: text;
  background-clip: text;
  transition: .13s ease-in-out;
  /* font-family: 'Handwriting', sans-serif; */
  /* font-family: 'Noto Sans KR', sans-serif; */
  /* # 선택 1 */
  /* font-family: 'Nanum Pen Script', cursive; */
  /* # 선택 2 */
  /* font-family: 'Rampart One', cursive; */
  /* # 선택 3 */
  font-family: 'Caveat', cursive;
}

```