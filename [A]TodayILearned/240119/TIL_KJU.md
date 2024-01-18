# 1. 들어가며

---

스프링 부트 애플리케이션을 도커 위에다가 배포하려고 합니다.

해당 포스트에서는 `DockerFiler` 을 어떻게 작성하고 배포하는지에 대해서 알아보도록 하겠습니다.

# 2. DockerFile 작성

---

> **💫 Info** **실습 환경**
> MacOS
> Spring Boot 3.1.x
> Java 17.0.x

```Dockerfile
FROM openjdk:17 AS Builder  
  
COPY gradlew .  
COPY gradle gradle  
COPY build.gradle .  
COPY settings.gradle .  
COPY src src  
  
RUN chmod +x ./gradlew  
RUN ./gradlew clean build  
  
FROM openjdk:17  
  
ARG JAR_FILE=build/libs/*.jar  
COPY ${JAR_FILE} app.jar  
  
ENTRYPOINT ["java","-jar","/app.jar"]
```

먼저 다음과 같이 작성을 하였습니다. 두 단계로 이루어져 있으며 하나씩 분석해보도록 하겠습니다.

**첫 번째 단계: 빌더(Builder)**
- `openjdk:17` 이미지를 기반으로 시작합니다.
- `gradlew`, `gradle`, `build.gradle`, `settings.gradle`, 그리고 `src` 디렉터리를 작업 디렉터리에 복사합니다.
- `gradlew` 파일의 실행 권한을 변경합니다.
- `./gradlew clean build` 명령을 실행하여 Java 애플리케이션을 빌드합니다.

**두 번째 단계: 최종 이미지**
- 다시 `openjdk:17` 기반 이미지를 사용합니다.
- `JAR_FILE` 인자를 사용하여 (기본값으로 `build/libs/*.jar`가 설정되어 있음) 이전 단계에서 생성된 jar 파일을 `app.jar`로 최종 이미지에 복사합니다.
- 컨테이너의 엔트리 포인트를 `java -jar /app.jar`로 설정합니다.

다음과 같이 진행한 후 `DockerImage` 를 생성합니다.

`docker build <DockerFile위치> -t <이미지 이름>`

<img width="800" src="https://github.com/KIMSEI1124/CI_CD_Study/assets/74192619/204206f3-3e10-4073-a2e8-3dd73601df94" />
위와 같이 이미지를 생성할 수 있습니다.

# 3. 컨테이너 실행하기

---

먼저 이미지가 정상적으로 생성되었는지 확인하려면 다음과 같이 명령어를 작성합니다.

`docker image ls <이미지 이름>`

![도커 이미지 확인](https://github.com/KIMSEI1124/CI_CD_Study/assets/74192619/954cd3ac-070f-493d-9f2c-92edf72f3223)

위 사진과 같이 정상적으로 나오게 된다면 이미지 생성에 성공하였습니다. 이후 해당 이미지를 실행시킵니다.

```sh
docker run -d \
--name <컨테이너 이름> \
-p 8080:8080 \
<이미지 이름>
```

이후 확인해보면 정상적으로 동작하는 것을 확인할 수 있습니다.

`docker ps -f name=<컨테이너 이름>`
<img width="800" src="https://github.com/KIMSEI1124/CI_CD_Study/assets/74192619/2d120830-a958-4b0a-8cba-97d6f79b382a" />
# 4. 정리

---

스프링 부트 애플리케이션을 손쉽게 `dockerfile` 로 도커 이미지로 만든 후 컨테이너로 배포를 해봤습니다.
처음에는 시간이 걸렸지만 손 쉽게 작성할 수 있어서 도커의 장점을 하나 더 배운 것 같습니다. 이후에는 도커를 사용하여 `CI-CD`환경을 구성하여 무중단 배포까지 노려보도록 하겠습니다.
