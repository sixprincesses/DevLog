# 들어가며

---

같이 스터디 하는 팀원의 코드를 리뷰하던 중 다음과 같은 질문이 있었습니다.

```java
public abstract class RequestMemberDto {
    @NotBlank(message = "비밀번호는 필수 입력 값 입니다.")
    @Pattern(regexp = "(?=,*[0-9])(?=.*[a-zA-Z])(?=.*\\W)(?=\\S+$).{8,16}", message = "비밀 번호는 8~16자 영문 대, 소문자, 숫자, 특수 문자를 사용해주세요.")
    protected String password;

    @NotBlank(message = "이름은 필수 입력 값입니다.")
    protected String name;

    @NotBlank(message = "생년월일은 필수 입력 값 입니다.")
    @DateTimeFormat(pattern = "yyyy-mm-dd", iso = DateTimeFormat.ISO.DATE)
    protected LocalDate birthDate;

    public abstract Member toMember();
}
```

> 여기서 만약 `@pattern` 으로 Email과 PhoneNumber의 유효성 검사를 진행 한다면 Embedded Class에서 추가로 검증 기능이 필요한 건지 궁금합니다.

라고 질문을 하였습니다.

그래서 해당 질문에 대한 제 생각과 과연 어떤 방식이 좋을지에 대해서 조사를 해보았습니다.

# 내 생각

---

개인적인 생각으로는 `Request`나 `Entity`에서 한 곳에서 검증하라고 하면 일단 `Entity`에서 검증을 하도록 할 것 같습니다.
그 이유는 `Request`마다 모든 `Valid`를 설정해주었는데 추후 스펙이 변경되면 모든 곳을 찾아서 수정해야합니다.
하지만 한 곳에서만 검증을 진행한다면 스펙이 변경될 때 한 곳에서만 진행하면 되니 검증을 진행해야 하는 곳은 `Entity`라고 생각을 하였습니다.

아래는 제 생각대로 구현한 예제 코드입니다.

```java
package com.multi.mariage.member.domain.embedded;  
  
import com.multi.mariage.member.exception.MemberErrorCode;  
import com.multi.mariage.member.exception.MemberException;  
import jakarta.persistence.Column;  
import jakarta.persistence.Embeddable;  
import lombok.AccessLevel;  
import lombok.Getter;  
import lombok.NoArgsConstructor;  
import org.springframework.security.crypto.password.PasswordEncoder;  
  
import java.util.regex.Pattern;  
  
@Getter  
@NoArgsConstructor(access = AccessLevel.PROTECTED)  
@Embeddable  
public class Password {  
    private static final int MIN_LENGTH = 8;  
    private static final int MAX_LENGTH = 16;  
    private static final String PASSWORD_FORMAT = 
	    "^(?=.*[a-z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]+$";  
    private static final Pattern PASSWORD_PATTERN = Pattern.compile(PASSWORD_FORMAT);  
    @Column(name = "password", nullable = false)  
    private String value;  
  
    private Password(String value) {  
	    this.value = value;  
    }  
  
    public static Password encrypt(String value, PasswordEncoder passwordEncoder) {  
	    validateLengthInRange(value);  
	    validatePatternIsValid(value);  
	    return new Password(passwordEncoder.encode(value));  
    }  
  
    private static void validateLengthInRange(String value) {  
	    int length = value.length();  
	    if (length < MIN_LENGTH || MAX_LENGTH < length) {  
		    throw new MemberException(MemberErrorCode.PASSWORD_CANNOT_BE_OUT_OF_RANGE);  
	    }  
    }  
  
    private static void validatePatternIsValid(String value) {  
	    if (isNotValid(value)) {  
		    throw new MemberException(MemberErrorCode.PASSWORD_PATTERN_MUST_BE_VALID);  
	    }  
    }  
  
    private static boolean isNotValid(String value) {  
	    return !PASSWORD_PATTERN.matcher(value).matches();  
    }  
}
```

> 해커톤때 사용했던 비밀번호 검증 로직 코드

다음과 같이 정적 팩토리 메소드를 사용하여 검증도 한번에 처리하였습니다.

> 💫 Info
> 정적 팩토리 메소드는 이펙티브 자바의 아이템1인 **생성자 대신 정적 팩토리 메서드를 고려하라**를 참고하였습니다.

위와 같이 코드를 작성하면 요구사항이 변경되었을 때 해당 검증 로직만 수정하면 된다고 생각하여 유지보수성이 올라간다고 생각하였는데 다른 사람들의 의견도 찾았습니다.

# 다른 사람들의 답변

---

해당 자료에 대해서 이미 고민하던 글이 있어서 참고를 하였습니다.
## [Spring Rest API validation should be in DTO or in entity?](https://stackoverflow.com/questions/42280355/spring-rest-api-validation-should-be-in-dto-or-in-entity)

---

다음 글에서는 `Rest API`를 사용할 때 어떤 계층에서 검증을 해야하는지에 대한 질문이였고 `DTO`에서 검증을 하고 `Entity`에서 다시 검증을 하면 코드의 중복이 나오는 것 아니냐는 질문이였습니다.

이에 대한 답변으로는 다음과 같았습니다.

**우리는 항상 모든 애플리케이션의 여러 단계에서 유효성 검사를 수행하기 위해 노력해야 합니다.**

- 입력된 값 개체가 유효한지 확인했지만, 엔티티의 값은 유효한지 검증하지 않았습니다.
- 개발자의 실수를 방지하는 것도 중요합니다.

위와 같은 이유로 `DTO` 뿐만 아니라 `Entity` 에서도 유효성 검사를 적용하면 테스트 케이스가 광범위한 기능 유형성 검사에 집중할 수 있고 데이터 저장소 상태가 애플리케이션에 의해 손상되지 않도록 보장할 수 있다고 합니다.

위와 같이 입력된 값이 비즈니스 로직에 의해 수정된 결과가 `Entity`에서 검증할 때 잘못된 값이 저장될 수 도 있는 것을 방지할 수 있습니다.

그 외의 다른 내용들에서도 한 곳만 할 이유가 없고 두곳 다 해야한다고 주장하고 있는 것을 확인할 수 있습니다.

# 정리

---

검증은 개발을 할 때 가장 중요한 작업중 하나입니다. 그래서 검증은 최대한 자세하게 하는 것이 좋을 것 같고 `DTO`나 `Entity` 한 곳에서만 하는 것이 아닌 두 곳에서 하는 것이 좋다고 하여 추후 개발을 할 때 두 곳에서 검증을 하는 것을 적극 채용할 것 입니다.

# Ref

---

1.  [Spring Rest API validation should be in DTO or in entity?](https://stackoverflow.com/questions/42280355/spring-rest-api-validation-should-be-in-dto-or-in-entity)
2.  [Spring DTO validation in Service or Controller?](https://stackoverflow.com/questions/19100317/spring-dto-validation-in-service-or-controller)
3.  [Should I validate dtos or entities?](https://softwareengineering.stackexchange.com/questions/387763/should-i-validate-dtos-or-entities)
