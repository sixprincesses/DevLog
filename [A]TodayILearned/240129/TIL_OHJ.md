# 2024.01.29 TIL

## 메인 페이지 Flip Clock 구현
- 사이즈를 크게 줄이느라, 관련된 모든 클래스의 시계 사이즈와 글자 크기를 수정하는 과정 거침

## 공부 페이지 구성
- 스톱워치 구현

- 인스턴스 코드미러 구현
    - react ace로 구현

- 레이아웃 완성
    - overflow-y가 적용 안되는 현상 발생
        - div에 상위 div를 한 개 더 부여하면서 해결함. 아직 왜 해결된건지 잘 이해 못함.

- 아직 부족한 점들
    - 드롭다운 적용
    - 마크다운, 깃허브 인스턴스 적용
    - 깃허브와 코드미러는 각각 컴포넌트/언어 선택할 수 있도록 수정
    - 미리보기 구현
    - 최종 미리보기 페이지 구현

## 회원가입 페이지
- 디자인 완성
    - 아직 부족한 점들
        - 유저 정보 리덕스 저장
        - 정보 입력 성공 시 경로 변경
        - fetch 말고 axios 활용
        - axios 요청할 시 타입 세세히 주기