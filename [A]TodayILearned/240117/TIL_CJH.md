# 2024.01.17 TodayILearned

### [ImageServer] Client, FE, BE, DB, CLOUD 데이터 순환 설계

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
  
### [LiveChatServer] Client, FE, BE, Socket, DB 데이터 순환 설계

> 로그인

1. WebSocket 연결
2. Notification을 받을 전용 Channel로 Subscribe

> 실시간 채팅(Client A, Client B)

1. A와 B만의 채팅을 위한 Channel 생성
2. UUID를 ChannelDB에 저장과 동시에 A와 B에게 전송
3. A와 B는 해당 UUID를 subscription

> 로그아웃 후 재로그인 사이 새로운 채팅 처리

1. WebSocket 연결
2. Notification을 받을 전용 Channel로 Subscribe
3. A와 B만의 Channel에 A가 없으면 읽지않은 메세지 별로 Notification에 처리
