## 팔로우 / 언팔로우 기능 구현

## 실습 환경

- Java 17.0.9
- Spring Boot 3.2.1
- Lombok
- JPA

## 1. Entity 구성

```java
// Member.java

@JsonIgnore
@OneToMany(mappedBy = "to")
private List<Follow> followings = new ArrayList<>();

@JsonIgnore
@OneToMany(mappedBy = "from")
private List<Follow> followers = new ArrayList<>();
```

```java
// Follow.java

@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "follow")
public class Follow extends BaseEntity {

    @Id
    @GeneratedValue
    @Column(name = "follow_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "following_id")
    private Member to;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "follower_id")
    private Member from;

    @Builder
    public Follow(Member to, Member from) {
        relateFollowerFollowing(to, from);
    }

    public void relateFollowerFollowing(Member to, Member from) {
        this.to = to;
        this.from = from;
        to.getFollowers().add(this);
        from.getFollowings().add(this);
    }
}
```

### 1. 지연 로딩 (FetchType.LAZY)

지연 로딩은 연관된 엔티티를 필요할 때까지 로드하지 않는 방식입니다 

애플리케이션의 성능을 향상시키고 불필요한 데이터베이스 액세스를 줄일 수 있습니다

### 2. JsonIgnore

Member 엔티티를 JSON으로 직렬화할 때 followings와 followers 필드는 무시됩니다

Member 엔티티가 Follow 엔티티를 참조하고, Follow 엔티티가 다시 Member 엔티티를 참조하는 무한 루프를 방지합니다
