
JSON으로 데이터를 송수신할 때, DTO를 통해 전달받으려면 생성자가 필요하다.

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
