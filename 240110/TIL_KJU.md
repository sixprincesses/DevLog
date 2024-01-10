# 들어가며

해당 문서에서는 깃허브의 소셜 회원가입과정을 정리하였습니다.

## 1. Github 사전 준비하기

가장 먼저 [깃허브](https://github.com/)의 계정이 있어야합니다.

그 다음 다음과 같은 과정으로 `Github App`을 생성합니다.

<img width="800" src="https://github.com/KIMSEI1124/CI_CD_Study/assets/74192619/6221e717-a81f-4084-b5cd-80e3ca8f9e50" />

1. 사이트 오른쪽 상단에 있는 프로필 사진을 클릭하여 사이드바를 클릭합니다.
2. 하단에 있는 `Settings`를 클릭한 다음 오른쪽 사이드바 하단에 존재하는 `Developer Settings` 를 선택합니다.
3. `Github App`에서 `New Github App`을 선택하여 생성합니다.

<img width="800" src="https://github.com/KIMSEI1124/CI_CD_Study/assets/74192619/7c502ff9-aa9a-4819-a737-ed086c2e0d12" />

4. 위 사진과 같이 URL을 작성합니다. 실제로 운영을한다면 도메인을 작성하시면 됩니다.

<img width="800" src="https://github.com/KIMSEI1124/CI_CD_Study/assets/74192619/45e5b425-5112-40e3-b0ce-292d24f0fd2f" />

5. 이후 `Client_id`를 발급이 되었고 `Client_secrets`도 버튼을 클릭하여 발급받습니다.

이로써 준비는 끝났습니다.

## 2. 실습 환경

- Java 17.0.9
- Spring Boot 3.1.7
- Window10 or MacOS

위와 같은 환경을 구성하였습니다.

## 3. GithubClient 구성하기

먼저 인터페이스를 작성하도록 하겠습니다. 그 이유는 추후 `github` 말고 다른 소셜로그인을 도입하게 되어도 해당 구현은 반드시 필요하기 때문입니다.

```java
public interface OAuthClient {
    String getAccessToken(String code);

    OAuthUserInfoResponse getUserInfo(String accessToken);
}
```

각 함수에 대한 설명은 다음과 같습니다.

- `getAccessToken()` : 사용자의 인증 요청 코드(callback)을 전달받아 엑세스토큰을 구합니다.
- `getUserInfo()` : 발급 받은 엑세스 토큰으로 사용자의 정보를 조회합니다.

위 두 함수는 다른 소셜로그인도 공통으로 사용되기 때문에 필요할 때 구현하시면 좋습니다.
