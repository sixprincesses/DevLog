# 2024.01.12 TIL

## 1. 실습 환경
- Java 17.0.9
- Spring Boot 3.2.1
- Lombok
- Spring Security

## 2. OAuth를 활용한 Github 로그인 흐름

### 1. 클라이언트에서 Github로의 요청

`https://github.com/login/oauth/authorize?client_id={client_id}`

해당 URL로 GET 요청을 보내면 사전에 설정한 callback URL에 code를 쿼리 파라미터 양식으로 붙여 보내줍니다

### 2. code를 활용한 Github Access Token 발급 및 사용자 정보 반환

```java
// Github Access Token 반환
    public String getAccessToken(String code) {
        OAuthAccessTokenRequestDto requestDto = new OAuthAccessTokenRequestDto(clientId, clientSecret, code);
        Optional<OAuthAccessTokenResponseDto> oAuthAccessTokenResponseDto = Optional.ofNullable(
                restTemplate.postForObject(
                        "https://github.com/login/oauth/access_token",
                        requestDto,
                        OAuthAccessTokenResponseDto.class
                )
        );

        return oAuthAccessTokenResponseDto
                .orElseThrow(RuntimeException::new)
                .getAccessToken();
    }
    
// Github 사용자 정보 반환
    public OAuthGithubUserInfoResponseDto getUserInfo(String accessToken){
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + accessToken);

        HttpEntity<String> entity = new HttpEntity<>("parameters", headers);
        ResponseEntity<OAuthGithubUserInfoResponseDto> userInfoResponseDto = restTemplate.exchange(
            "https://api.github.com/user",
            HttpMethod.GET,
            entity,
            OAuthGithubUserInfoResponseDto.class
        );
        
        return userInfoResponseDto.getBody();
    }
```

## 3. Spring Security + JWT

### 1. 로그인 플로우

1. Github로부터 받은 사용자 정보에서 id, login, node_id를 추출합니다.
2. Spring Security에 등록되는 username은 login으로, password는 node_id로 설정합니다.
3. 해당 정보를 기반으로 사용자를 h2 DB에 등록하고 JWT를 발급해줍니다.
4. JWT refresh token은 username(login)을 key로 하여 REDIS에 저장합니다.

### 2. REDIS

- Jedis 대신 Lettuce를 쓰는 이유
    - Redis를 Java에서 사용하기 위한 클라이언트 라이브러리
        1. 비동기 지원
            - Jedis는 블로킹 연산을 사용하므로 요청이 완료될 때까지 스레드가 차단
            - 많은 스레드를 사용할 때 확장성 문제
            - Lettuce는 비동기 및 논블로킹 모드를 지원
            - 스레드로 여러 요청을 처리하고 응답을 기다리지 않고 다른 작업을 수행가능
        2. 커넥션풀링
            - Jedis는 내장 커넥션 풀을 가지고 있지 않음
                - 커넥션 관리에 대한 사용자가 직접 처리
            - Lettuce는 커넥션 풀링을 내장
        3. 스레드 안정성
            - 스레드 안전성을 보장

`implementation 'org.springframework.boot:spring-boot-starter-data-redis'`

```
data:
  redis:
    host: localhost
    port: 6379
```

```java
@Configuration
public class RedisConfig {

    @Value("${data.redis.port}")
    private int port;

    @Value("${data.redis.host}")
    private String host;

    @Bean
    public RedisConnectionFactory redisConnectionFactory(){
        return new LettuceConnectionFactory(host, port);
    }

    @Primary
    @Bean
    public RedisTemplate<String, String> redisTemplate(){
        // redisTemplate를 받아와서 set, get, delete 사용
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<>();
        // setKeySerializer, setValueSerializer 설정
        // redis-cli을 통해 직접 데이터를 조회 시 알아볼 수 없는 형태로 출력되는 것을 방지
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        redisTemplate.setConnectionFactory(redisConnectionFactory());

        return redisTemplate;
    }
}
```

refresh token 저장

```java
    public String refresh(Authentication authentication){
        UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();

        Claims claims = Jwts.claims().setSubject(userDetails.getUsername());
        Date now = new Date();
        Date expireDate = new Date(now.getTime() + refreshTokenExpiration);

        String refreshToken = Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(now)
                .setExpiration(expireDate)
                .signWith(SignatureAlgorithm.HS256, accessSecret)
                .compact();

        // redis에 저장
        redisTemplate.opsForValue().set(
                userDetails.getUsername(),
                refreshToken,
                refreshTokenExpiration,
                TimeUnit.MILLISECONDS
        );

        return refreshToken;
    }
```

### 3. DOCKER

```
docker run -d --name redis-container \
--restart=on-failure \
-p 6379:6379 \
-v redis-volume:/data \
-e TZ=Asia/Seoul \
redis
```
- `docker run -d --name redis-container \` : 컨테이너 생성 (-d background 실행)
- `--restart=on-failure \` : 구동 실패시 재구동
- `-p 6379:6379 \` : 포트지정 (왼쪽: Local Port/ 오른쪽: 컨테이너 내 포트)
- `-v redis-volume:/data \` : 해당 컨테이너의 저장공간 (업데이트 해도 유지)
- `-e TZ=Asia/Seoul \` : 시간 조정
- `redis` : 이미지 명

### 4. 결과

![스크린샷 2024-01-14 오후 7 10 58](https://github.com/sixprincesses/Test-LoginFlow/assets/116432941/d3f10797-793c-45a0-8a1c-34b246d1c2a8)

![스크린샷 2024-01-14 오후 7 11 23](https://github.com/sixprincesses/Test-LoginFlow/assets/116432941/994f58ca-ae41-4d98-8579-9b5bf1656e88)

```
127.0.0.1:6379> keys *
1) "sirlyun"
127.0.0.1:6379> get sirlyun
"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJzaXJseXVuIiwiaWF0IjoxNzA1MjI3MDQ5LCJleHAiOjE3MDU4MzE4NDl9.5X_j___04DaW0mwNHHTeMnjOEyNm9lmdek8rTk0grZc"
```
