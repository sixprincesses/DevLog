# 들어가며

팔로우 목록 조회 시 Querydsl을 활용하여 조회하고자 하였습니다.

## 개발 환경

- Java 17.0.9
- Spring Boot 3.2.1
- Lombok
- JPA
- Querydsl

## Custom Repository

```java
public interface FollowRepositoryCustom {
    Page<FollowsResponse> followersLoad(Long memberId, Pageable pageable);
}
```

```java
@Override
public Page<FollowsResponse> followersLoad(Long memberId, Pageable pageable) {

    List<FollowsResponse> followers = followsLoad(
            memberId, pageable, follow.from.nickname, follow.to.id);
    JPAQuery<Long> countQuery = followCount(memberId, follow.to.id);

    return PageableExecutionUtils.getPage(followers, pageable, countQuery::fetchOne);
}
private List<FollowsResponse> followsLoad(Long memberId, Pageable pageable, StringPath nickname, NumberPath<Long> id) {
    return queryFactory
            .select(new QFollowsResponse(
                    nickname
            )).from(follow)
            .where(id.eq(memberId))
            .offset(pageable.getOffset())
            .limit(pageable.getPageSize())
            .fetch();
}
private JPAQuery<Long> followCount(Long memberId, NumberPath<Long> id) {
    return queryFactory
            .select(follow.count())
            .from(follow)
            .where(id.eq(memberId));
}
```