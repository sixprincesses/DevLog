# 2024.01.10 TodayILearned

### [Database] 순환참조

> 순환 참조란?

참조하는 대상이 서로 물려 있어 참조할 수 없게 되는 현상

> 문제점

JPA에서 양방향으로 연결된 Entity를 그대로 조회하는 경우, 서로의 정보를 순환하면서 조회하다가 StackOverflow가 발생

> 순환 참조 해결방법

1. @JsonIgnore 
- JSON 데이터에 해당 프로퍼티는 null로 변경됨

2. @JsonManagedReference 와 @JsonBackReference
- 부모 클래스 필드에 @JsonManagedReference를, 자식 클래스 필드에 @JsonBackReference를 추가

3. @JsonIgnoreProperties
- 부모 클래스필드에 @JsonIgnoreProperties 붙여주기

4. DTO 사용
- Entity 자체를 response로 리턴하지 않고, entity 자체를 return 하지 말고, DTO 객체를 만들어 필요한 데이터만 옮겨담아 리턴

5. 매핑 재설정
- 양방향 매핑을 단방향 매핑으로 전환

### [SpringBoot] ImageServer 구성

1. AWS S3 Cloud Storage

2. Local Storage
