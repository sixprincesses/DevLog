# 2024-01-12 TIL

## 프론트-백 Image 송신, 수신
- axios가 아닌 fetch로 간단히 로직 구현
- cors 문제 생겨 백에게 해결 요청
- post로 form-data에 이미지를 넣어 보내는 건 정상적으로 되었는데, post로 이미지를 받는 건 계속 cors 문제가 발생 -> 결과적으로 왜 이게 해결이 안 되었는지는 백이 설명해줘야 할 듯?


## Github 로그인 구현
- 2가지 component로 구성
    - 1차적으로 깃허브 로그인 페이지로 이동시켜주는 컴포넌트
    - 깃허브가 콜백 url로 authentication code를 돌려줬을 때, 이를 받아 서버에 access token 요청하는 컴포넌트


## Docker 활용해 백서버 

```
sh deply.sh
``` 
- 스프링 서버 측에서 위 코드를 실행했을 시 이미지를 생성해 자동으로 도커가 가상 서버를 실행할 수 있도록 하는 코드를 스프링 측에서 작성
- 프론트 쪽 callback url과 github 쪽 callback url을 다르게 설정해 에러가 발생했었음


## 공부해야한다고 느낀 것들
- 도커 + 컨테이너 개념
- JDK, IntelliJ 설치
- React Router 설계하는 여러 가지 방법론, 기술
- 쿠버네티스 개념
