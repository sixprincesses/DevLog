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
