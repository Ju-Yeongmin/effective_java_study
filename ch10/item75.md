# Item 75
## 예외의 상세 메시지에 실패 관련 정보를 담으라

예외를 잡지 못해 프로그램이 실패하면 자바 시스템은 그 예외의 추적 정보를 자동으로 출력한다
#### 보통은 이 정보가 프로그래머 혹은 엔지니어가 얻는 유일한 정보인 경우가 많다
- 스택 추적은 예외 객체의 toString 메서드를 호출해 얻는 문자열이다
- 보통은 예외 클래스 이름 뒤에 상세 메시지가 붙는 형태
- 따라서 예외의 toString 메서드에 실패에 관한 정보를 가능한한 많이 담아야 한다
- 사후 분석을 위해 실패 순간의 상황을 정확히 포착해 예외의 상세 메시지에 담아야 한다

#### 실패 순산을 포착하려면 예외 메시지에 매개변수와 필드의 값을 담자
예를들어 IndexOutOfBoundsException의 상세메시지는?
- 범위의 최솟값과 최댓값
- 그 범위를 벗어났다는 인덱스의 값
- 이 정보들 중에서 셋 중 한 두개 혹은 셋 모두가 잘못됐을 수도 있다
- 가능한한 많은 정보를 주어야 무엇을 고쳐야할지 분석하는데 도움이 된다

#### 상세 메시지에 비밀번호나 암호 키 같은 정보까지 담으면 안된다
당연한 얘기겠지만 보안과 관련된 정보는 주의해서 다뤄야 한다

#### 장황하게 적을 필요는 없다
예외의 상세 메시지와 사용자에게 보여줄 오류 메시지를 혼동하지 마라
- 문제를 분석하는 사람들은 소스코드를 함께 분석한다
- 예외가 발생한 파일의 이름, 줄번호 등을 기입하지만 코드에서 확인할 정보를 장황하게 적을 필요 없다
- 최종 메시지는 친절하게, 예외 메시지는 담긴 내용이 더 중요하다

```java
핵심!
프로그래머나 엔지니어에게 에러의 원인을 분석하는 것은 매우 중요하다. 따라서 에러에는
실패에 관련된 정보를 담아야한다. 예외를 일으킨 순간의 매개변수와 필드 값, 파일의 이름,
줄 번호 등을 기입하되 소스코드에서 확인할 수 있는 정보를 장황하게 적어서 가독성을
떨어뜨리지 말자
```