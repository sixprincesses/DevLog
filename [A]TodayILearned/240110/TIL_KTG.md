# 오늘 한 일

1. Github REST API 검색 및 구현 완료 (Repository: test-Github)
2. 기능명세 작성 완료
3. ERD 초안 완료

## Github REST API 검색 및 구현

### 목적

프로젝트 기능 중 하나인 공부 기록의 편의성을 위한 `깃허브 커밋 내역을 자동으로 가져올 수 있도록 하는 서비스`를 개발하고자 테스트를 진행한다.

### 사용 기술

1. 언어 : Typescript
2. 프레임워크 : React
3. 라이브러리 : Axios

### API

1. Github REST API

   | API 주소                                                                                       | 개요                                                   | 정보                                          |
   | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------ | --------------------------------------------- |
   | https://api.github.com/users/\${userName}                                                      | 특정사용자에 대한 정보를 가져온다                      | -                                             |
   | https://api.github.com/users/\${userName}/repos                                                | 사용자의 레포지토리 리스트를 가져온다                  | name = selectedRepo                           |
   | https://api.github.com/repos/\${userName}/\${selectedRepo}                                     | 사용자의 특정 레포지토리를 가져온다                    | -                                             |
   | https://api.github.com/repos/\${userName}/\${selectedRepo}/branches                            | 사용자 레포지토리의 브랜치 리스트를 가져온다           | name = selectedBranch                         |
   | https://api.github.com/repos/\${userName}/\${selectedRepo}/branches/\${selectedBranch}         | 사용자 레포지토리의 특정 브랜치를 가져온다             | -                                             |
   | https://api.github.com/repos/\${userName}/\${selectedRepo}/commits?sha=${selectedBranch}       | 사용자 레포지토리 특정 브랜치의 커밋 리스트를 가져온다 | sha = selectedCommit                          |
   | https://api.github.com/repos/\${userName}/\${selectedRepo}/commits/\${selectedBranch}          | 사용자 레포지토리 특정 브랜치의 최신 커밋을 가져온다   | -                                             |
   | https://api.github.com/repos/\${userName}/\${selectedRepo}/commits/\${selectedCommit}          | 사용자 레포지토리의 특정 커밋을 가져온다               | files.map((file) => file.patch) === 파일 목록 |
   | https://raw.githubusercontent.com/\${userName}/\${selectedRepo}/\${selectedCommit}/\${fileDir} | 로우 파일 경로                                         | -                                             |

## 2. 기능명세 작성 완료

### 개요

프로젝트에서 구현할 핵심적인 기능을 정리하여 명세로 작성

## 3. ERD 초안 완료

### 개요

프로젝트에서 사용할 DB 구조 초안 작성

### 토론 및 토의

1. 커리큘럼-챕터-피드 구조의 DB 구조화 작업
    - 의견 
        1. 1:N 중계테이블을 커리큘럼-챕터, 챕터-피드로 2개 만들어서 연결하는 것이 좋다.
        2. 1:N 중계테이블을 커리큘럼-(챕터, 피드)로 1개 만들어서 연결하는 것이 좋다.
    - 이유
        1. 더 깔끔한 데이터 베이스 설계가 된다.
        2. Join을 줄여서 성능이 좋다.
    - 컨설턴트 의견
        - 1번은 더 정규화가 잘 된 테이블이고, 2번은 성능이 좋을 수 있다. 일반적으로는 정규화가 잘 된 1번을 선택한다.

2. 통계 테이블 분리
    - 의견
        1. 통계 테이블을 분리해야 하는가?
        2. 통계 테이블이 필요 없는가?
    - 이유
        1. 통계 테이블을 따로 관리하는 것이 탐색 횟수가 훨씬 적어 효율적이다.
        2. 통계 테이블이 없어도 계산하는 쿼리를 이용하여 구현할 수 있다.  
    - 컨설턴트 의견
        - 두 의견 모두 많이 쓰이지만, 정확성이 중요한 자료의 경우는 2번을 채택하고, 아니라면 1번을 채택한다.

3. DB 사이클 주의
    - DB 테이블 관계에 사이클이 생길 경우, 무한 참조가 발생할 수 있으니 조심해야 한다.
    - 만약 발생했을 경우, 참조를 차단할 수 있도록 함수를 짜거나, DB 구조를 바꿔야 한다.
