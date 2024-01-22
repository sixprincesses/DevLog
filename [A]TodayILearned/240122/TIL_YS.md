## 1. 실습 환경

- Java 17.0.9
- Spring Boot 3.2.1
- Lombok
- Redis
- MySQL
- JWT

## 2. OAuth를 활용한 Github 로그인 흐름

### 1. 클라이언트에서 Github로의 요청

`https://github.com/login/oauth/authorize?client_id={client_id}`

해당 URL로 GET 요청을 보내면 사전에 설정한 callback URL에 code를 쿼리 파라미터 양식으로 붙여 보내줍니다

### 2. code를 활용한 Github Access Token 발급 및 사용자 정보 반환

```java
/**
     * Github 에 등록된 사용자 정보를 가져오기 위한 access_token 요청
     *
     * @param githubCodeResponse OAuth Callback Code
     * @return 사용자의 access_token
     */

    public GithubTokenResponse getAccessToken(GithubCodeResponse githubCodeResponse) {

        GithubTokenRequest githubTokenRequest = GithubTokenRequest.builder()
                .clientId(clientId)
                .clientSecret(clientSecret)
                .code(githubCodeResponse.code())
                .build();

        return restTemplate.postForObject(
                "https://github.com/login/oauth/access_token",
                githubTokenRequest,
                GithubTokenResponse.class
        );
    }

    /**
     * Github에 등록된 사용자 정보 반환
     *
     * @param githubTokenResponse Github access_token
     * @return 사용자의 id와 login 정보
     */
    public GithubUserInfoResponse getUserInfo(GithubTokenResponse githubTokenResponse) {

        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + githubTokenResponse.accessToken());

        HttpEntity<String> entity = new HttpEntity<>("parameters", headers);

        return restTemplate.exchange(
                "https://api.github.com/user",
                HttpMethod.GET,
                entity,
                GithubUserInfoResponse.class
        ).getBody();
    }
```

## 3. JWT

### 1. 로그인 플로우

1. Github로부터 받은 사용자 정보에서 id, login을 추출합니다.
2. 해당 정보를 기반으로 사용자를 MySQL에 등록하고 JWT를 발급해줍니다.
    1. 회원가입은 member_id를 함께 반환해 회원 정보 수정 페이지로 이동 가능하도록 해줍니다.
3. JWT refresh token은 social_nickname(login)을 key로 하여 REDIS에 저장합니다.

### 2. REDIS

`implementation 'org.springframework.boot:spring-boot-starter-data-redis'`

```
data:
  redis:
    host: localhost
    port: 6379

```

```java
@RedisHash(value = "refresh_token", timeToLive = 60 * 60 * 24 * 7)
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class RefreshToken {

    @Id
    private Long socialId;
    private String refreshToken;

    @Builder
    public RefreshToken(Long socialId, String refreshToken) {
        this.socialId = socialId;
        this.refreshToken = refreshToken;
    }
}

public interface RefreshTokenRepository extends CrudRepository<RefreshToken, Long> {
}
```

refresh token 저장

```java
public TokenMarker jwtSave(MemberTokenRequest memberTokenRequest){

        Claims claims = Jwts.claims()
                .setSubject(String.valueOf(memberTokenRequest.socialId()));
        Date now = new Date();
        Date accessExpireDate = new Date(now.getTime() + accessTokenExpiration);
        Date refreshExpireDate = new Date(now.getTime() + refreshTokenExpiration);
        Key hmacKey = new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), SignatureAlgorithm.HS256.getJcaName());

        String accessToken = Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(now)
                .setExpiration(accessExpireDate)
                .signWith(hmacKey, SignatureAlgorithm.HS256)
                .compact();

        String refreshToken = Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(now)
                .setExpiration(refreshExpireDate)
                .signWith(hmacKey, SignatureAlgorithm.HS256)
                .compact();

        // Redis 에 저장
        refreshTokenRepository.save(RefreshToken.builder()
                .socialId(memberTokenRequest.socialId())
                .refreshToken(refreshToken)
                .build());

        if (memberTokenRequest.memberId() != null){
            return SignupTokenResponse.builder()
                    .accessToken(accessToken)
                    .refreshToken(refreshToken)
                    .memberId(memberTokenRequest.memberId())
                    .build();
        }
        return LoginTokenResponse.builder()
                .accessToken(accessToken)
                .refreshToken(refreshToken)
                .build();
    }
```

### 3. DOCKER

```bash
docker run -d --name redis-container \
--restart=on-failure \
-p 6379:6379 \
-v redis-volume:/data \
-e TZ=Asia/Seoul \
redis
```

```bash
docker run --name mysql-container \
-e MYSQL_ROOT_PASSWORD={password} -d \
-p 3306:3306 \
mysql
```

- `docker run -d --name redis-container \` : 컨테이너 생성 (-d background 실행)
- `-restart=on-failure \` : 구동 실패시 재구동
- `p 6379:6379 \` : 포트지정 (왼쪽: Local Port/ 오른쪽: 컨테이너 내 포트)
- `v redis-volume:/data \` : 해당 컨테이너의 저장공간 (업데이트 해도 유지)
- `e TZ=Asia/Seoul \` : 시간 조정
- `redis` : 이미지 명
