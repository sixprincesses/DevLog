# 24.02.08 TIL

## 로그인이 제대로 안 되는 이슈?
- 로컬에서는 로그인이 잘 되는데, 배포서버에서는 로그인이 안된다?
    ### 문제 정리
    - 배포된 서버에서는 깃허브에서 authorization code를 받아오는 시간이 로컬보다 길다
    - callback url이 렌더링 될 때 서버에 code와 함께 토큰 요청을 보냄
    - 렌더링 될 때 code를 아직 깃허브가 안 줘서 요청을 못 보내고 있었던 것
    - url이 렌더링 될 때가 아닌, code가 들어올 때 요청을 바꾸도록 수정하니 문제 해결

## 앞으로 해야할 일
- 메인페이지
    - 새로고침 문제 2개
        - 로그인 광클 시 깃허브 막힘
        - 검색 부분 새로고침 할때마다 정신사나운 것
    - 깃허브 로그인 버튼 반응형 처리

- 회원가입 페이지
    - 스킵 버튼
    - 이용약관 세부사항

- 피드
    - 댓글과 대댓글

- 공부페이지
    - 저장 눌렀을 시 데이터 처리

- 업로드 페이지
    - 디자인 및 이동
    - 데이터 처리

- SSE 
    - 각 데이터에 대해 클릭 시 해당 링크로 이동 해결

- 전반적
    - 도킹 처리
    - 코드 간결화 및 스타일 문제
    - 디자인 손보기