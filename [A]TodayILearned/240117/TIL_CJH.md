# 2024.01.17 TodayILearned

### [ImageServer] Client, FE, BE, DB, CLOUD 데이터 순환과정

> 회원 이미지 등록
1. Client → FE
2. FE → BE(Member Server)
  - Multipart 형식으로 데이터 전송
3. BE(MemberServer) → BE(StorageServer)
  - Multipart 형식으로 Member Id와 함께 데이터 전송
4. BE(StorageServer) → DB
  - StorageDB에 Member Id, 생성된 UUID, 확장자, 생성시간 등을 저장
5. BE(StorageServer) → Cloud(AWS S3)
  - 생성된 UUID로 데이터 저장
  
