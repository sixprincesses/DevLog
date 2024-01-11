## 문제
- 공부 시간 로그 남기기 방식
    - 사용자가 수동으로 임시 저장하는 경우
        - Contents와 Time 데이터 둘다 필요
    - 시스템 자체에서 임시 저장을 자동으로 하는 경우
        - Contents 데이터만 필요

## 해결 방안
`List<startTime, endTime>`
startTime null 불가능
endTime null 가능
다음 startTime 들어오면 이전 endTime Update

## 해결해야 할 것
- 공부 창을 두 개 띄웠을 때의 충돌 문제
- 공부 창과 피드 창을 띄우는 것은 가능하게 개선