# 2024.01.11 TIL

## 1. 회원가입/로그인 서비스 플로우

![FlowChart-로그인-토큰발급 drawio](https://github.com/sixprincesses/DevLog/assets/116432941/3628f9df-eb38-4a4c-a5cb-26ef6f138586)


## 2. 스프링 데이터 JPA

### 3. 실습 환경

- Java 17.0.9
- Spring Boot 3.2.1

위와 같은 환경으로 구성하였습니다.

### 4. 기본 구조

```java
package study.datajpa.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import study.datajpa.entity.Team;

public interface TeamRepository extends JpaRepository<Team, Long> {
}
```

- JpaRepository<{Entity}, {pk}>

Repository 구현 시 인터페이스만 잡아 놓으면 구현 클래스를 자동으로 주입해줍니다.

기본적인 메서드 구현에 소요되는 시간을 단축할 수 있습니다.

### 5. 쿼리 메서드

```java
package study.datajpa.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import study.datajpa.entity.Team;
import java.util.List;

public interface TeamRepository extends JpaRepository<Team, Long> {
    // 쿼리 메서드로 메서드 기반 쿼리 생성 가능
    List<Member> findByUsername(String username);
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

- 제공되는 쿼리 메서드 기능
    - 조회: find ... By, read ... By, query ... By, get ... By
    - Count: count ... By
    - Exists: exists ... By
    - 삭제: delete ... By, remove ... By
    - Distinct: findDistinct, findMemberDistinctBy 등
    - Limit: findFirst3, findFirst, findTop3 등

[Ref] https://docs.spring.io/spring-data/jpa/docs/current-SNAPSHOT/reference/html/#reference

### NamedQuery

`Entity`
```java
package study.datajpa.entity;

import jakarta.persistence.*;
import lombok.*;

@Entity
@Getter @Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@NamedQuery(
        name = "Member.findByUsername",
        query = "select m from Member m where m.username = :username"
)
public class Member {
    ...
}
```

`Repository`
```java
package study.datajpa.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import study.datajpa.entity.Member;

import java.util.List;

public interface MemberRepository extends JpaRepository<Member, Long> {
    @Query(name = "Member.findByUsername")
    List<Member> findByUsername(@Param("username") String username);
}
```