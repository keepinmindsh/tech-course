# Calendar 기본 과제 

![Calendar](https://github.com/keepinmindsh/tech-course/blob/main/project/calendar/calendar_example.png)

- [ * 기간 ] 에서 시작일자, 종료일자를 선택하고 [조회] 버튼을 클릭하면 선택한 기간 만큼으로 pre, next로 이동할 수 있다. 시작일자부터 [2020년 5월] 부분에 시작일자의 년/월이 선택된다.
- Calendar에서 날짜를 선택하면 [선택된 일자]의 우측으로 값이 표시된다.
- [조회] 버튼을 클릭 시 조회한 기간 만큼이 우측의 테이블에 출력되어야 한다.
- 페이징은 테이블의 행이 50개가 넘어갈 경우 페이징 개수만큼 숫자로 나뉘어진다.
- Calendar와 테이블에 토/일이 각기 색으로 표시된다.
  - 토요일 : 파란색, 일요일 빨간색
- 테이블의 국경일은 국경일 여부에 대해서만 예/아니오로 표시한다. 

***

- 프로젝트 구성 방식 
  - BackEnd 구성은 API를 이용한다. 
    - Calendar의 일자를 계산하는 로직을 Java를 이용해서 구현한다. 
    - Java 코드를 작성시 Test 코드를 작성한다. 
  - UI는 자유롭게 구성한다. 언어 무관, 개별 프로젝트로 구성한다. 
  - Java 프로젝트는 github를 이용해서 github organizations을 구성해 하나의 소스 코드 내에서 구성한다. 
    - git branch를 구성한다. 
      - develop
      - main
    - 개발은 develop branch에서 진행하고, main으로 pull request 시에는 code review 후에 진행한다. 