# 2024.01.11 TodayILearned

### [SpringBoot] ImageServer 구성

1. 파일 업로드
    - BE - FE -SERVER
    - UUID로 파일명을 Key값으로 수정하여 저장
    - Frontend에서 직접 업로드를 진행할 경우 Frontend 보안이 취약할 시 S3 서버 보안 취약성 생성
    
2. 파일 다운로드
    - FE - BE - SERVER
    - Frontend에게 Key만 전달
    - Frontend에서 S3에 직접 GET으로 파일 받음

### [REDIS] Jedis 대신 Lettuce를 쓰는 이유
- Redis를 Java에서 사용하기 위한 클라이언트 라이브러리
1. 비동기 지원
   - Jedis는 블로킹 연산을 사용하므로 요청이 완료될 때까지 스레드가 차단
   - 많은 스레드를 사용할 때 확장성 문제
   - Lettuce는 비동기 및 논블로킹 모드를 지원
   - 스레드로 여러 요청을 처리하고 응답을 기다리지 않고 다른 작업을 수행가능
2. 커넥션풀링
    - Jedis는 내장 커넥션 풀을 가지고 있지 않음
        - 커넥션 관리에 대한 사용자가 직접 처리
    - Lettuce는 커넥션 풀링을 내장
4. 스레드 안정성
    - 스레드 안전성을 보장

### STOMP(Simple Text Oriented Messaging Protocol)
- 간단하고 가벼운 텍스트 기반의 프로토콜

### [DOCKER] REDIS
```
docker run -d --name redis-container \
--restart=on-failure \
-p 6379:6379 \
-v redis-volume:/data \
-e TZ=Asia/Seoul \
redis
```
- `docker run -d --name redis-container \` : 컨테이너 생성 (-d background 실행)
- `--restart=on-failure \` : 구동 실패시 재구동
- `-p 6379:6379 \` : 포트지정 (왼쪽: Local Port/ 오른쪽: 컨테이너 내 포트)
- `-v redis-volume:/data \` : 해당 컨테이너의 저장공간 (업데이트 해도 유지)
- `-e TZ=Asia/Seoul \` : 시간 조정
- `redis` : 이미지 명
