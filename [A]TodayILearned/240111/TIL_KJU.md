# 들어가며

실습환경은 다음과 같습니다.

- Java 17
- Spring Boot 3.1.7
- Lombok
- Spring Data MongoDB
- Spring Web
- IntelliJ IDEA
- Window 10 & macOS

# 1. MongoDB 설치하기

먼저 `Docker`을 활용하여 설치를 하도록 하겠습니다.
이미지를 확인하기 위해서 [DockerHub - MongoDB](https://hub.docker.com/_/mongo)를 조회합니다.

```shell
# MongoDB 설치
docker run -d \
-p 27017:27017 \
--name mongo-container \
-v mongo-volume:/data/db \
mongo
```

각 옵션에 대한 설명은 아래와 같습니다.

- `-p` : 내부 포트와 외부 포트를 27017로 설정합니다.
- `--name` : 컨테이너의 이름을 `mongo-container`로 설정합니다.
- `-v` : 컨테이너가 삭제되어도 데이터를 유지하기 위해서 볼륨으로 연결을 합니다.

> 🌟 **Info**
>
> `MongoDB`의 기본 포트는 [제타위키 - 몽고DB](https://zetawiki.com/wiki/MongoDB_%EA%B8%B0%EB%B3%B8_%ED%8F%AC%ED%8A%B8) 에서 확인해본 결과
> "27017"인 것을 확인하였습니다.

다음과 같이 제대로 설치된 것을 확인할 수 있습니다.

<img width="800" alt="mongo install with docker" src="https://github.com/KIMSEI1124/CI_CD_Study/assets/74192619/a49d7dee-c1fe-43cf-9b8a-9565df5b5d18" />

## 1.1. DB 유저 생성하기

컨테이너를 생성하였으면 유저를 생성해보겠습니다.

```shell
docker exec -it <mongoDB 컨테이너> /bin/bash
```

위의 명령어를 사용하여 컨테이너에 접속합니다.

이후 아래의 명령어를 입력하여 DB 에 접근할 수 있습니다.

```shell
mongosh 
```

<img width="800" alt="mongosh" src="https://github.com/KIMSEI1124/CI_CD_Study/assets/74192619/fbb1e661-db20-4c01-bd20-fb276a3e2aec" />

> 🌟 **Info**
>
> 이전에는 `mongo`로도 접속을 하였지만 일부 버전에서는 `mongosh`로 접속합니다.

```shell
# 사용자 관리 데이터베이스 접근
use admin

# 계정 생성
db.createUser(
  {
    user: "root",
    pwd:  "1234",
    roles: [
    	{ "role" : "root", "db" : "admin" },
	]
  }
)
```

위와 같이 입력하고 `{ ok: 1 }`을 반환하면 정상적으로 동작한 것 입니다.

<img width="800" alt="create user" src="https://github.com/KIMSEI1124/CI_CD_Study/assets/74192619/0419359b-c684-4c5f-890f-8e692a204418" />

# 2. 스프링 부트 설정

위에서 설치한 `MongoDB`와 스프링 부트의 설정을 연결하기 위한 `yml`을 다음과 같이 작성합니다.

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://root:1234@localhost:27017/devlog?authSource=admin
#      uri: mongodb://<USERNAME>:<PASSWORD>@<URL>:<PORT>/<DB이름>?authSource=admin
```

# 3. Intellij IDEA 연결하기

MongoDB 와 IDE 를 연결을 해보도록 하겠습니다.

<img width="800" alt="ide" src="https://github.com/KIMSEI1124/CI_CD_Study/assets/74192619/ad89148c-65d6-44d2-b4de-641c53517da4" />

위 사진과 같이 진행하면 연결을 할 수 있었습니다.

> ⚠️ **Warning**
>
> ```text
> com.mongodb.MongoSecurityException: Exception authenticating MongoCredential{mechanism=SCRAM-SHA-1, 
> userName='username', source='db_name', password=<hidden>, mechanismProperties=<hidden>}
> Command failed with error 18 (AuthenticationFailed): 'Authentication failed.' on server localhost:27017. The full 
> response is {"ok": 0.0, "errmsg": "Authentication failed.", "code": 18, "codeName": "AuthenticationFailed"}.
> ```
> 위와 같이 오류가 발생하는 경우가 있어서 저 같은 경우에는 `MongoURL` 뒤 파라미터로 "authSource=admin" 을 추가하였습니다.<sub>1)</sub>

# 4. 예제 코드 작성

`Spring Boot`에서 사용할 예정이므로 도메인 영역과 테스트 코드를 작성합니다.

```java
// Feed
@Data
@Document(collation = "feed")
public class Feed {
    @Id
    private String id;

    private String content;

    public Feed(String content) {
        this.content = content;
    }
}

// FeedRepository
public interface FeedRepository extends MongoRepository<Feed, String> {
}

// Test
@SpringBootTest
class FeedRepositoryTest {


    @Autowired
    private FeedRepository feedRepository;

    @Test
    void saveFeed() {
        /* Given */
        Feed feed = new Feed("content");

        /* When */
        Feed savedFeed = feedRepository.save(feed);

        /* Then */
        assertThat(savedFeed).isEqualTo(feed);
    }
}
```

이후 DB를 확인하면 정상적으로 저장이 된 것을 확인할 수 있습니다.

<img width="800" alt="save" src="https://github.com/KIMSEI1124/CI_CD_Study/assets/74192619/9aebd888-ee7b-4277-81f0-9cd5384e3f93">

# Ref

1. [[DataGrip] MongoDB 연결 실패시 - minhyuk yoon (joseph)](https://velog.io/@ymh92730/MongoDB-%EC%97%B0%EA%B2%B0-%EC%8B%A4%ED%8C%A8%EC%8B%9C)
