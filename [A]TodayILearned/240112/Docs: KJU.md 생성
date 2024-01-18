# 들어가며
---
[로그 및 SQL 진입점 정보 추가 여정 ~ 우아한 기술블로그](https://techblog.woowahan.com/13429/?ref=codenary)를 보던 중 `Caffeine`이라는 단어가 나왔습니다.
평소에 보지 못한 키워드라 검색을 해보았더니 성능 향상에서 주로 사용하는 캐시관련 기술인 것을 확인하였고 현재(2023.09.21)에 성능에 관심이 많아 해당 기술에 대해 알아보고자 합니다.
## Caffeine 이란?
---
`Caffeine`은 High performance 캐싱 라이브러리 입니다.<sub>3)</sub>

<img src="https://raw.githubusercontent.com/ben-manes/caffeine/master/wiki/logo.png" width="200">

`Caffeine`은 다음과 같은 특징이 있습니다.
1. `max size`를 설정해두고, 해당 사이즈가 넘어갈 경우 내보냅니다. 
2. 마지막 접근(`read based`) 혹은 최소 쓰기 (`write based`)에 따라 만료 시간 설정 가능합니다.
3. `refreshAfterWrite` 로 자동 `refresh` 가능합니다.
4. `key, value`가 자동적으로 `weak reference`로 이동되기 때문에 `GC`를 통해 삭제 가능합니다.
5. `cache access`에 대한 `statistic` 제공하므로 모니터링 제공합니다.
## Quick Start
---
`Caffeine`를 사용하는 간단한 예제를 확인해 보겠습니다.

예제 환경은 다음과 같습니다.
- Java 17
- Spring Boot 3.0.x
- Lombok, Spring Data JPA, Spring Web

### Dependency 추가
---

```groovy
/* Caffeine */  
implementation 'org.springframework.boot:spring-boot-starter-cache'  
implementation 'com.github.ben-manes.caffeine:caffeine'
```

`build.gradle`에 다음과 같이 추가할 수 있습니다.

### Configuration 설정
---
먼저 `Caffeine`를 사용하기 위해서 `Cache`설정을 가진 `Enum`을 생성합니다.
아래 예제에서는 캐시 이름, 만료 시간, 저장 가능한 최대 갯수를 정의하였고 다른 설정이 필요하다면 추가할 수 있습니다.

```java
@RequiredArgsConstructor  
@Getter  
public enum CaffeineCacheType {  
    USER("user", 10, 10000),  
    ;  
  
    private final String cacheName;  
    private final int expireTime;
    private final int maximumSize;  
}
```

`Config` 클래스를 만든 뒤 `@EnableCaching` 어노테이션으로 캐싱을 활성시켜줍니다. 그리고 `CaffeineCacheType`에 등록한 캐시들을 객체로 생성 후 `SimpleCacheManager` 객체에 등록합니다.
```java
@EnableCaching  
@Configuration  
public class CaffeineConfig {  
  
    @Bean  
    public CacheManager cacheManager() {  
    List<CaffeineCache> caches = Arrays.stream(CaffeineCacheType.values())  
    .map(cache -> new CaffeineCache(cache.getCacheName(), Caffeine.newBuilder().recordStats()  
                                .expireAfterWrite(cache.getExpireTime(), TimeUnit.SECONDS)
                                .maximumSize(cache.getMaximumSize())  
                                .build()  
    )  
    )  
    .toList();  
    SimpleCacheManager simpleCacheManager = new SimpleCacheManager();  
    simpleCacheManager.setCaches(caches);  
    return simpleCacheManager;  
    }  
}
```

다음과 같이 설정하면 캐시를 사용할 준비가 완료되었습니다.

### `Caffeine` 사용하기

---
간단한 예제를 통해서 사용 방법을 알아보겠습니다.

아래 예제와 같이 `Domain`을 구성하였습니다.

```java
/* Member */
@NoArgsConstructor(access = AccessLevel.PROTECTED)  
@AllArgsConstructor  
@Builder  
@Getter  
@Entity  
public class Member {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private int id;  
    private String name;  
}

/* MemberRepository */
public interface MemberRepository extends JpaRepository<Member, Integer> {  
}
```

이제 `Cache`를 사용하기 위한 함수 설정을 해보겠습니다.
다음과 같이 `Caching`을 원하는 함수에 다음과 같이 설정을 하게 된다면 모든 준비는 끝났습니다.

```java
@RequiredArgsConstructor  
@Service  
public class MemberService {  
    private final MemberRepository memberRepository;  
  
    @Cacheable(cacheNames = "userCache")  
    public Member findById(int id) {  
    return memberRepository.findById(id)  
    .orElseThrow();  
    }  
}
```

실제 테스트를 통하여 확인을 해보겠습니다.

```java
@Slf4j  
@SpringBootTest  
class MemberServiceTest {  
    @Autowired  
    private MemberService memberService;  
    @Autowired  
    private MemberRepository memberRepository;  
  
    @DisplayName("카페인 캐시 조회를 테스트한다.")  
    @Test  
    void cacheTest() {  
    /* Given */  
    Member saveMember = memberRepository.save(Member.builder()  
    .name("카페인")  
    .build());  
  
    /* When */  
    log.info("Search By Repository");  
    Member findMember = memberService.findById(saveMember.getId());  
  
    log.info("Search By Cache");  
    Member findMemberWithCache = memberService.findById(saveMember.getId());  
  
    /* Then */  
    assertThat(findMember).isEqualTo(findMemberWithCache);  
    }  
}
```

테스트 결과 `SQL` 문이 한번만 동작한 것을 확인할 수 있습니다.

```text
Hibernate: 
    insert 
    into
        member
        (id, name) 
    values
        (default, ?)
2023-09-23T20:29:31.729+09:00  INFO 45015 --- [    Test worker] c.s.c.c.m.service.MemberServiceTest      : Search By Repository
Hibernate: 
    select
        m1_0.id,
        m1_0.name 
    from
        member m1_0 
    where
        m1_0.id=?
2023-09-23T20:29:31.788+09:00  INFO 45015 --- [    Test worker] c.s.c.c.m.service.MemberServiceTest      : Search By Cache

```

## 결론
---

이렇게 `Cache` 를 사용하게 된다면 `Cache`에 해당  값이 존재한다면 DB 쿼리가 필요없이 `Cache`에서 조회할 수 있어 속도가 빠릅니다.

캐싱을 사용할 때는 정말 조심해서 써야하고 성능 향상을 위해서는 관리를 잘 해야 합니다.

## Ref
---
1. [Spring Boot 에서 Cache 사용하기 ~ 뱀귤 블로그](https://bcp0109.tistory.com/385)
2. [[Spring] Caffeine ~ koiil](https://velog.io/@_koiil/Caffeine)
3. [caffeine ~ ben-manes](https://github.com/ben-manes/caffeine)
4. [Spring Caching](https://docs.spring.io/spring-boot/docs/3.0.11/reference/html/io.html#io)
