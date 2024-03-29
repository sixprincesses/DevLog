# 2024.01.17 TIL

## 타이머 추가 설계
- 시간, 분 타이머가 59초에서 00초로 넘어갈 때가 아니라 00초에서 01초로 넘어갈 때 flip 된다는 점이 문제
    - seconds를 dependancy로 하는 useEffect 활용, 초가 58초일 때 setTimeout -> 1초 후에 flip 되도록 하는 로직 구현
```javascript
useEffect(() => {
    if (minutes === 59 && seconds === 58) {
    setTimeout(()=> {
        setHours((prevHours) => prevHours + 1)
        setMinutes(0)
    }, 1000)
    } else if (seconds === 58) {
    setTimeout(() => {
        setMinutes((prevMinutes) => prevMinutes + 1)
    }, 1000)
    }
}, [seconds, minutes])
```

## 깃허브 로그인
- github 로그인 flow에 대한 이해 부족으로 서버 통신 간 혼선이 발생했었음
    - 애초에 서버에 요청 보내는 로직이 callback 컴포넌트에 있었고, 해당 컴포넌트로 callback url 및 App 라우터 설정을 완료했음에도 인터넷 블로그 글의 코드만 맹신하다가 에러 겪음


## 회원가입 페이지
- json 데이터와 file을 함께 보낼 땐 multipart/form-data를 활용해야 한다는 사실을 깨달음. 
    - file은 엄연히 간단한 string들로 이루어진 다른 데이터들과 구분해서 보내야 함. 그렇지 않으면 서버 단에서 용량 등의 이유로 문제 발생시킴
    - 한꺼번에 json으로 묶어 보내는 **415에러** 발생했었음

> 해결 코드
```javascript
const formData = new FormData();
const jsonData = {
          nickname: values.nickname,
          name: values.Name,
          email: values.email,
          phone: values.phone,
          gender: values.gender,
          // image_id: values.profileImage,
        };
 formData.append("json", 
          new Blob([JSON.stringify(jsonData)], {type: "application/json"})
        );
        formData.append("image_id", values.profileImage)
const response = await fetch("http://70.12.247.83:8080/member/1", {
            method: "POST",
            body: formData,
})
```

> 알게된 사실
- headers의 default는 application/json이다.
- json과 file은 multipart/form-data를 활용하자.

