# 들어가며

---

멀티 모듈을 가지고 실제 설계를 할 때 어떠한 점들을 고민하고 작성하는지에 대해서 자료를 조사한 후 정리해보았습니다.

# Naver - 인프콘 2022

---

[실전! 멀티 모듈 프로젝트 구조와 설계 | 인프콘 2022](https://www.youtube.com/watch?v=ipDzLJK-7Kc) 에서의 정보들을 정리해봤습니다.

## 1. "WHY" 멀티 모듈 프로젝트 구조가 중요할까요?

---

아키텍처 관련으로는 2가지 이야기가 있습니다.

- 아키텍처는 프로젝트 초기에 이루어져야 하는 일련의 설계 결정이다. - 마틴 파울러, 랄프 존슨
- 아키텍처는 요소의 구조(structure)와 그 관계(relationship)에 관한 것 - 소프트웨어 엔지니어링 연구소

위에 있는 아키텍처를 멀티 모듈 프로젝트로 바꿔서 이해를 한다면 결국 나중에 변경하기 어렵다는 것을 설명하기 위한 것 입니다.

프로젝트 구조를 **나중에 변경하게 되면 나중에 변경하기 어렵습니다.** 그러나 프로젝트 초기에 시작한다면 **리스크를 줄이기 위한 시작입니다.**

<img width="443" alt="2023-11-13_10-50-22" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/a7abcfa2-3513-43fe-9455-ca1b2311094c">

본 발표에서 나온 발표 자료중 하나며 초기에는 정상적으로 동작하였지만 중복을 제거하기 위해서 공통적인 구현은 `core`와 `common`에 구현을 하다보니 점점 거대해 지는 현상을 발견 하였습니다.

이로써 발생하는 문제점들은 소개해보도록 하겠습니다.

### **Too Many Connections**

---

`Core` & `Common` 사용하는 의존성 프로젝트는 커넥션 풀을 모두 할당 받는다.

### "Java.lang.NoClassDefFoundError"

---

특정한 모듈이 하위 버전 라이브러리를 참조하는 경우 업그레이드 난해함

### "Copy & Paste" 

---

참조하는 곳이 너무 많아 일단 `if..else` 분기 처리 `copy & paste`

**AS-IS**

`return userService.getById(id);`

**TO-BE**

```java
User user = userService.getById(id);
if (type = "A") {
	user = userService.getById2(id); // -> getById(id) 참조 후 수정
}
return user;
```

깨진 창문 효과처럼 무수히 많아지는 `copy & paste`가 발생합니다.

### "All Build & Deploy"

---

**TO-BE**

```java
User user = userService.getById(id);
if (type == "A") {
	user = userService.getById2(id); /// -> getByid(id) 참조 후 수정
}
return user;
```

다음과 같이 오탈자가 있어서 전체 빌드와 배포를 하게 된다면 매우 고통스럽고 비용이 많이 들 수 있습니다.
## 2. "WHAT"을 기준으로 멀티 모듈 프로젝트 구조를 나눠어야 할까요?

---

> 가장 중요한 내용이라고 생각!

위에서 봤던 비효율적인 구조를 다음과 같은 구조로 변경을 해보았습니다. 하지만 진짜로 좋은 구조일까요?

<img width="700" alt="2023-11-13_10-59-26" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/05ea8479-4dcb-429f-8a2c-d889598f01f2">

기술 베이스 지향적인 모듈 구조로 설계를 하게 된다면 다음과 같은 문제가 발생하게 됩니다.

<img width="700" alt="2023-11-13_11-01-33" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/8299abe6-ae9d-4bb5-8ed9-449eaac8322d">

점점 어려워 지고 복잡하게 됩니다. 그리고 유관부서 및 업체연동 모듈을 추가해보도록 하겠습니다.

<img width="700" alt="2023-11-13_11-23-26" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/4ee65c0a-d3e7-4a28-b8d0-fbc88d83d55e">

이렇게 되면 점점 복잡해지며 많은 연결을 볼 수 있습니다.

기능들을 나열해서 보도록 하겠습니다. 빨간색 영역들의 모듈들이 늘어나게 되면서 멀티 모듈들이 2배 이상으로 늘어나게 됩니다.

<img width="700" alt="2023-11-13_11-23-07" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/3ace6e3f-ee9f-4037-898f-8b7b624a9e8b">

### 우선 CORE, COMMON 모듈은 삭제하고, 그래도 CORE, COMMON이 정말 필요한지? 한번 더 고민하기

---

중복은 제거되어야 하는 것에 크게 공감합니다. 

그렇다고 중복을 제거하기 위헤서 모든 것을 한 곳에 구현하는 것은 좋지 않다고 생각을 하였습니다.

해당 부분은 앙이 방대하므로 해당 부분보다는 설계 부분에 더 집중하겠습니다.

### BOUNDED CONTEXT - 경계 나누기

---

<img width="502" alt="2023-11-13_11-27-14" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/42315eb3-4fed-45ba-a6fe-e60aa029572d">

[바운디드 컨텍스트 ~ 마틴 파울러](https://martinfowler.com/bliki/BoundedContext.html) 

경계를 잘 나누는 것이 멀티 모듈 프로젝트를 설계할 때 가장 중요하다고 생각합니다.

### 멀티 모듈 그룹 경계 나누기

---

기준이 되는 멀티 그룹 4개는 다음과 같습니다.

**Boot Server**
`batch`, `admin`, `api` 
**변화가 잦은 서버 모듈**입니다.

**Data(Domain)**
`mate`, `user`, `chart`등 
서버 모듈과 밀접한 관계를 가지고 있으며 도메인으로 데이터 스토어를 직접 핸들링 하는 부분입니다.

**Infra**
`and`, `vod`, `photo`, `billing`
유관 부서 및 업체 연동을 위한 그룹 모듈입니다.
구현하게 되면 변화는 없지만 버젼업 등이 되면 큰 변화가 일어나는 부분입니다.

**CLOUD(SYSTEM)**
`config`, `gateway`, `discovery`, `aws`, `gcp`, `azure`
서버 관리를 위한 그룹 모듈이며 컨테이너 환경과 트래픽 제어를 위한 시스템 관련 그룹입니다.

## 3. "HOW" 실전 멀티 모듈 프로젝트 구현을 해야 될까요?

---

4가지 멀티 그룹을 가지고 어떻게 구현을 해야하는지 이야기를 해보도록 하겠습니다.

- Java
- Spring
- Gradle

를 가지고 작성을 한 예제입니다.

멀티 그룹 성격에 맞게 각각의 프로젝트를 구성합니다.

<img width="474" alt="2023-11-13_11-36-30" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/fbc27f94-e2a6-4af3-8397-bda20fd86922">


이렇게 구성하게 되면 일단 가독성이 좋아지는 것을 볼 수 있고 `Build.gradle`같은 빌드 도구를 사용하여 빌드 도구들을 일괄적으로 선언할 수 있습니다.

### 멀티 모듈 그룹 분리하기

---

<img width="678" alt="2023-11-13_11-38-34" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/9832b8be-4a03-4041-ae74-579bb018033a">

가독성이 좋아지고 정리되는 것을 볼 수 있습니다.
그런데 현재 상태로 운영하다 보면 또 다른 문제를 마주치게 됩니다.
"**계속 늘어나는 모듈들 -> 복잡도 증가 -> 늘어나는 빌드 시간 + ...**"

<img width="700" alt="2023-11-13_11-41-29" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/50957790-b847-4f31-bf87-9644871442d0">

다음과 같이 `boot` 와 `data` 는 연관되는 부분이 많기 때문에 한 곳으로 몰아뒀습니다.
이후 `cloud`에서 `webhook`이 일어나는 부분을 따로 분리하여 `cloud`를 구성합니다.
마지막으로 `infra` 부분을 구성하게 됩니다.

이렇게 구성함으로써 더 명확하게 분리하게 된 것을 확인할 수 있습니다.

### Data -> Infra 멀티 모듈 프로젝트 관계 구현

---

<img width="700" alt="2023-11-13_14-10-29" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/c0a65325-8c25-4ba9-853f-73459f2b70f0">


위의 사진 처럼 코드를 작성하고 봤을 때 정리된 것 처럼 보였습니다. 그런데 좋은 듯 했지만 문제가 발생했습니다. 해당 `infra`를 많이 참조하게 된다면 "Many To Connection"이 발생하게 되었습니다.

그래서 다음과 같이 해결하였습니다.

<img width="700" alt="2023-11-13_14-12-13" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/50ef9c68-92c0-4c9c-a169-eace052675c6">

### Boot -> Data 멀티 모듈 프로젝트 관계 구현

---

이 둘 사이의 관계를 구현하게 되면 서비스 구현체를 어디에다가 둬야 하는지 고민을 많이 하게 됩니다.

`Java`의 `Spring` 에서 개발을 하다보면 `Service`는 어디에서 구현을 해야하는지 고민이 됩니다.
이러면 `Boot Module`와 `Data Module`중 어디에 둬야 하는지 고민을 해보았습니다.

<img width="700" alt="2023-11-13_11-50-14" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/bef199b4-e5c3-4e55-bc27-095ca8fa03e7">

결론은 **모듈 별로 책임과 역활에 각각 구현되어야 하고 서로 협력**하여야 합니다.
`data-meta`에서는 `ServletRequest`까지 내려가는 것은 역활과 책임, 협력 관계가 올바르지 않다고 생각합니다.

웹과 의존적인 클래스들을 전달하는 행동은 하지 많아야 한다고 생각합니다. `cookie`, `session`와 같은 것도 동일합니다.

### Cloud -> Boot 멀티 프로젝트 관계 구현

---
<img width="569" alt="2023-11-13_11-55-17" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/d0a4c9ad-4370-47ff-b61c-b20431036075">

시스템 버젼업으로 인하여 시스템 부분이 재빌드, 배포가 되는 것을 보며 분리해야 한다고 생각했습니다.

<img width="696" alt="2023-11-13_11-56-41" src="https://github.com/KIMSEI1124/Spring_Study/assets/74192619/431f8cb2-aaad-44a3-9dad-840eed721f08">

> 해당 부분은 `K8S` 환경을 구현할 때 자료를 더 조사해보겠습니다.

# 정리

---

## 1. "왜" 멀티 모듈 프로젝트 구조가 중요할까요?

---

- 잘못 구성되면 나중에 변경하기 고통스럽다.
- 프로젝트 초기에 이루어져야 하는 일련의 설계 과정이다.
- 개발, 생산성에 막대한 영향을 미친다.

## 2. "무엇"을 기준으로 멀티 모듈 프로젝트 구조를 나눠야 할까요?

---

- 경계안에서 의미를 갖을 수 있는 그룹을 정의하는(나누는)것이 가장 중요하다. (Bounded Context)
- 역활, 책임, 협력 관계가 올바른지 다시 한번 생각한다.
- `Boot(Server)`, `Infra`, `Data(Domain)`, `System(Cloud)`

## 3. "어떻게" 실전 멀티 모듈 프로젝트 구현을 해야 할까요?

---

- 프로젝트가 커지고 있다면 다시 경계를 나누고 그 기준으로 소스 저장소를 분리한다.
- `Infra(외부)` 라이브러에는 `Data` 관련 구현을 지향한다.
- 서비스 구현은 각자 역활에 맞게 각각 구현될 수 있다. ( 공통으로 한쪽에 구현하지 않는다. )
- 시스템 레벨 구현이 실제 서비스 애플리케이션과 밀접하게 연관되지 않도록 격리하거나 전환(`istio`) 한다.

## 배운점

---

영상 길이는 36분짜리지만 이해하면서 들으려고 하다보니 2시간 이상 걸렸습니다.
그것도 마지막 부분에서는 재대로 이해하지도 못하고 단순 정리만 하였습니다.

하지만 설계가 중요하고 어느정도 감을 잡게 되어서 다음 프로젝트를 진행하게 된다면 설계에 대한 고민을 많이 하여 효율적인 코드를 작성할 수 있는 능력을 키웠다고 생각합니다.

# Ref

---

1. [실전! 멀티 모듈 프로젝트 구조와 설계 | 인프콘 2022](https://www.youtube.com/watch?v=ipDzLJK-7Kc) 
