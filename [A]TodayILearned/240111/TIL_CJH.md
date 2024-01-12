# 2024.01.11 TodayILearned

### [DataBase] 설계

임시 피드에 대한 DB 설계

- Member와 1:1 관계
- 피드와 임시피트의 Feature를 동일하게 유지
- 임시피드의 공부시간 로그를 임시저장할 때, 끝 시간을 null로 처리 혹은 업데이트

### [Diagram] Studying Feed

https://github.com/sixprincesses/DevLog/blob/main/%5BBE%5DDiagram/%5B240111%5DFlowChart-%EA%B3%B5%EB%B6%80-%ED%94%BC%EB%93%9C.drawio.png 

### [SpringBoot] ImageServer 구성

1. DB에 Image에 대한 ImageName, Extension, Path 정보 저장
2. Upload
    - ImageController - ImageService - StorageRepository - DB
3. Download
    - DB - StorageRepository - ImageService - ImageController

```properties
# AWS S3
cloud.aws.credentials.access-key=접근키
cloud.aws.credentials.secret-key=비밀키

## S3 Region
cloud.aws.region.static=지역정보

## S3 Server Domain
cloud.aws.s3.domain=서버종류

## S3 Server Name
cloud.aws.s3.bucket=서버명

## cloud inital setting
cloud.aws.stack.auto=false
cloud 세팅을 기본적으로 실시하므로 AWS를 사용할 경우 미리 FALSE로 세팅
```

