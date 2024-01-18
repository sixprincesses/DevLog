# 들어가며

---
**`.properties`, `.yaml` 등 이런 파일이 스프링에서 설정 정보를 보관할 수 있도록 만든 원리는 무엇일까?** 에 대한 내용에 대해서 찾아보고 정리합니다.

실제로 개발을 진행할 때 `ymal` 파일을 자주 사용하는데 관심이 있었던 주제이기 때문에 자세하게 찾아보며 코드도 찾아봅니다.

## Property

---

기존에 `Spring` 에서는 `XML`로 모든 설정을 관리했는데 `Spring Boot`에서는 정적인 값을 `key-value`형식으로 관리합니다.

> `Properties`, `YMAL` 

그러면 `Spring Boot`는 어떻게 해당 값들을 사용하는 것 일까요?

## Auto Configuration

---

해답은 `@SrpingBootApplication` 어노테이션에 있습니다.

```java
@SpringBootApplication  // 이 어노테이션
public class ConfigApplication {  
    public static void main(String[] args) {  
    SpringApplication.run(ConfigApplication.class, args);  
    }  
}
```

해당 어노테이션이 어떻게 구성되어 있는지 확인하면 다음과 같습니다.

```java
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)  
@Documented  
@Inherited  
@SpringBootConfiguration  
@EnableAutoConfiguration  
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),  
       @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) }) 
public @interface SpringBootApplication {
```

**위의 어노테이션 중 `@EnableAutoConfiguration` 이 자동 환경 설정을 도와줍니다.**
`Spring Boot` 에서 개선된 점이며 `Spring Boot`가 가지는 가장 큰 장점입니다.

> 자동으로 환경을 설정해주기 때문에 번거롭게 설정을 할 필요가 없어져 개발자들은 개발에만 집중할 수 있게 됩니다.

이 어노테이션 하나로 `Web`, `H2`, `JDBC` 를 비롯한 수 많은 자동 설정을 제공합니다.

> 만약, `H2 DB`의 의존성이 클래스 경로에 존재한다면 자동으로 `In-Memory DB`에 접근하게 됩니다.
> 이러면 개발자가 `H2 DB`의 설정을 파일에 따로 입력하지 않아도 해당 애플리케이션이 `In-Memory DB`에 접근합니다.

혹시나 더 자세한 코드를 확인하고 싶은 분들은 `AutoConfigurationPackages.java` 관련 추상 클래스를 확인해보시면 좋을 것 같습니다. 설정 파일이 존재하지 않으면 예외를 던지고 있으며 설정파일을 등록, 추가 하는 기능이 존재합니다.

### Spring Boot 의 기본 포트가 8080인 이유

---

`Embedded Tomcat`에서 기본 설정 값으로 `8080` 포트를 사용하게 되는데 이것도 `@EnableAutoConfiguration`의 영향 때문 입니다.

```json
{  
  "name": "server.port",  
  "type": "java.lang.Integer",  
  "description": "Server HTTP port.",  
  "sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties",  
  "defaultValue": 8080  
},
```

> 해당 설정은 `IntelliJ IDEA` 기준으로 `외부 라이브러리 -> Gradle: org.springframework.boot:spring-boot-autoconfigure:${spring_boot_version} -> META-INF -> spring-configuration-metadata.json` 에서 확인할 수 있습니다.
> 
> <img width="500" alt="2023-09-29_23-25-27" src="https://user-images.githubusercontent.com/74192619/271638876-8fc995b9-a122-4a8a-83b6-1d1462b3130b.png">

### 중복되는 빈 설정에 대한 처리 방법

---

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {  
    if (!isEnabled(annotationMetadata)) {  
       return EMPTY_ENTRY;  
    }  
    AnnotationAttributes attributes = getAttributes(annotationMetadata);  
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);  
    configurations = removeDuplicates(configurations);  // 중복되는 설정 제거
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);  
    checkExcludedClasses(configurations, exclusions);  
    configurations.removeAll(exclusions);  
    configurations = getConfigurationClassFilter().filter(configurations);  
    fireAutoConfigurationImportEvents(configurations, exclusions);  
    return new AutoConfigurationEntry(configurations, exclusions);  
}
```

`Spring Boot`에서는 자동 설정할 클래스들을 먼저 불러오는데 만약 중복되는 `Bean`이 설정될 경우가 생기는데 이를 대비하여 처리하는 로직이 있습니다.

바로 `Set`을 이용하여 제외할 설정을 저장합니다. 여기서 제외할 설정들은 중복된 설정 `removeDuplicates` 메소드를 이용해 추려지게 됩니다. 마지막으로 이 중에서 프로젝트에 사용 되는 `Bean`만 `Import`할 자동 설정 대상으로 선택됩니다.

> 더 자세한 코드는 `AutoConfigurationImportSelector.java` 파일에서 확인할 수 있습니다.
## YAML

---

위의 글에서는 `Properties`를 예시로 들었습니다. 하지만 저는 `YAML`을 더 많이 사용하고 있고 `Spring Project`팀에서도 `YAML`을 사용하고 있다고 합니다.

원래 `YAML`을 사용하려면 `SnakeYAML` 모듈을 별도로 설치해야 하지만, 이제는 `Spring Boot Starter`에 이 모듈이 기본으로 내장되어 있기 때문에 별도로 의존성을 추가할 필요 없이 `application.properties`를 `application.yml`로 이름을 바꾸면 됩니다.

> 만약 `properties` 와 `ymal` 파일이 두 개 다 존재하면 `ymal` 파일만 오버라이드 되어 적용됩니다.

## Ref
---
1. [[Spring boot] 환경 설정의 동작 원리와 애플리케이션 환경 나누기](https://blog.neonkid.xyz/216)
