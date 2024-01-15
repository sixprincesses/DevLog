# 2024.01.15 TIL

## 1. 실습 환경
- Java 17.0.9
- Spring Boot 3.2.1
- Lombok
- Spring Security

## 2. 회원 검증 구현

### 1. 검증 흐름

1. 사용자 로그인(username, password) 검증
2. 로그인 검증 통과 시 인증 객체 생성
```java
UsernamePasswordAuthenticationToken authenticationToken =
                new UsernamePasswordAuthenticationToken(memberCreationDto.getSocialNickname(),
                        memberCreationDto.getNodeId());
Authentication authentication = authenticationManager.authenticate(authenticationToken);
```
3. 인증 객체 기반 JWT 생성
4. access token과 refresh token(Redis에 따로 저장) 반환 
5. 인증이 필요한 서비스 접근 시 access token으로 인증
6. claim 기반 검증 후 인증 객체 SecurityContext에 저장
```java
String token = jwtService.resolveToken(request);
try {
    if (token != null && jwtService.validateToken(token)){
        Authentication authentication = jwtService.getAuthentication(token);
        SecurityContextHolder.getContext().setAuthentication(authentication);
    }
} catch (RedisConnectionException e){
    SecurityContextHolder.clearContext();
    throw new RedisException("Redis connection failed");
} catch (Exception e){
    throw new JwtServiceException("token is not valid");
}
```
7. 로그아웃 시 Redis에 저장된 refresh token 삭제

### Spring Security 변경점

```java 
.addFilterBefore(new JwtFilter(jwtService), UsernamePasswordAuthenticationFilter.class);
```

Custom Filter를 만들고 apply() 하는 방식에서 위와 같이 변경