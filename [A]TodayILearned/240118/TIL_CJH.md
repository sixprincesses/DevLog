# 2024.01.18 TodayILearned

1. JSON으로 데이터를 송수신할 때, DTO를 통해 전달받으려면 생성자가 필요하다

2. BaseEntity를 통해 각 Entity의 공통 feature를 관리할 수 있다

`extends BaseEntity`를 통해 Entity에 상속

3. Backend Package Architecture

```java
@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseEntity {
    @CreatedDate
    private LocalDateTime createTime;

    @LastModifiedDate
    private LocalDateTime updateTime;
}
```

```
\---server
    +---global
    |   +---config
    |   |       WebsocketConfig.java
    |   |
    |   +---domain
    |   |       Data1BaseEntity.java
    |   |       Data2BaseEntity.java
    |   |
    |   \---exception
    \---project
        +---client
        |       AuthenticationClient.java
        |       GithubClient.java
        |       StorageClient.java
        |
        +---controller
        +---data
        |   +---request
        |   |       DataRequest.java
        |   |
        |   +---response
        |   |       DataResponse.java
        |   |
        |   \---vo
        |           DataVO.java
        |
        +---domain
        +---repository
        \---service
                DataLoadService.java
                DataSaveService.java
                DataService.java
```
