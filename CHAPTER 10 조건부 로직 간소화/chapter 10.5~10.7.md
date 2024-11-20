- 10.5 특이 케이스 추가하기
    <aside>
    💡
    
    똑같은 구조를 가지고 있는건 함수로 모아서 처리하자 
    
    </aside>
    
    - 특수한 경우의 공통 동작을 한곳에 모으자.
    - 리팩터링 과정에서 코드를 일괄적으로 바꿔야 한다면 해당 함수에 에러처리 구문을 넣어, 실수가 있는지 확인하자.
    - 특이 케이스의 경우라면 특이 케이스 객체를 반환하여 기본값 처리를 하자.
    - 데이터 구조를 변환하는 함수
        - enrich: 부가 정보 추가
        - transform :형태를 바꾸는 경우
    - 특수한 경우의 공통 동작을 한곳에 모으자.
    - 
    
    ```jsx
    const site = acquireSiteData()
    const aCustomer = site.customer
    if (aCustomer === '미확인 고객')
      // 1. 반복되는 검사 케이스 확인
    
      function isUnknown(aCustomer) {
        return aCustomer === '미확인 고객'
      } // 2. 함수로 추출
    
    // 3. 객체 자체에 unKnown 관련 정보와 특이케이스일 때의 정보를 삽입
    function enrichSite(site) {
      const result = _.cloneDepp(site)
    
      if (isUnknown(result.customer)) {
        result.customer = unknownCustomer
      }
    }
    
    ```

- 10.6 어서션 추가하기
  - 항상 참이라고 가정하는 조건부 문장
  - 오류 찾기에 활용 가능
  - 특정 로직이 실행 될 때 필요한 프리컨디션을 나타냄
    <aside>
    💡
    
    - 특정 코드 앞에 항상 참이어야 하는 조건을 명시하고, 이를 테스트 코드로 대체할 수 있다.
    </aside>
    
    > 우리의 경우, 이를 활용하여 값을 검증하고 필요시 에러를 발생시킨다.
    > 
    
    ```jsx
    applyDiscount(aNumber){
        return aNumber - (this.discountRate * aNumber);
    } // 1. discountRate는 항상 양수라는 가정이 깔려 있음.
    
    assert(this.discountRate >= 0) // 2. 이런 어서션을 추가 할 수 있다.
    // 3. 필요하다면 discountRate의 게터에 추가도 가능하다.
    // 4. '반드시 참이어야 하는 것' 에 달자.
    ```

- 10.7 제어 플래그를 탈출문으로 바꾸기
  - 반복문을 작성하다보면 isFind 같은 제어 플래그를 사용하는 경우가 있다.
  - 이런 친구들은 삭제하고 break, return 으로 해당 반복문을 끝내도록 하자.
  - 물론 이러기 위해서는 작게 추출되어야 하는 작업이 선행되어야 한다.
  -
  **배경**
  제어 플래그의 정의를 먼저 보자.
  제어 플래그는 코드의 동작을 바꾸는 곳에서 나온다.
  조건절에서는 제어 플래그를 통해 검사하고 어떤 문에서는 계산을 통해 제어 플래그의 값을 바꾸는 구조로 되어있다.
  제어 플래그를 사용하면 코드를 이해하기 어려워지므로 이를 리팩토링해서 쉽게 바꾸자.
  제어 플래그의 주 서식지는 반복문이다.
  break 문이나 continue 문에 익숙하지 않은 사람이거나, 함수의 return 을 적절하게 이용하지 못하면 나온다.
  **절차**
  1. 제어 플래그를 사용하는 코드를 함수로 추출할지 고려한다.
  2. 제어 플래그를 갱신하는 코드 각각을 적절한 제어문으로 바꾼다. 하나씩 바꿀 때마다 테스트한다.
  3. 모두 수정했다면 제어 플래그를 제거한다.
  **예시**
  다음 예시는 사람 목록을 훑으면서 악당 (miscreant) 을 찾는 코드다.
  악당 이름은 하드코딩 되어있다.
  ```
  boolean found = false;

  for (Person p : people) {
      if (!found) {
          if (p.name.equals("조거")) {
              sendAlert();
              found = true;
          }
          if (p.name.equals("사루만")) {
              sendAlert();
              found = true;
          }
      }
  }
  ```
  여기서 found 플래그는 제어 플래그다.
  리팩토링을 하기 위해서 제어 플래그를 사용하는 부분을 따로 뽑아보자.
  ```
  public void checkForMiscreants(List<Person> people) {
      boolean found = false;

      for (Person p : people) {
          if (!found) {
              if (p.name.equals("조거")) {
                  sendAlert();
                  found = true;
              }
              if (p.name.equals("사루만")) {
                  sendAlert();
                  found = true;
              }
          }
      }
  }
  ```
  코드를 보니 제어 플래그가 참이면 sendAlert() 메소드를 호출하고 딱히 하는 일이 없다. 그러므로 리턴을 해주면 될 것 같다.
  ```
  public void checkForMiscreants(List<Person> people) {
      boolean found = false;

      for (Person p : people) {
          if (!found) {
              if (p.name.equals("조거")) {
                  sendAlert();
                  return;
              }
              if (p.name.equals("사루만")) {
                  sendAlert();
                  return;
              }
          }
      }
  }
  ```
  모두 이렇게 리턴하도록 하자.
  그런다음 제어 플래그를 제거하자.
  ```java
  public void checkForMiscreants(List<Person> people) {
      for (Person p : people) {
          if (p.name.equals("조거")) {
              sendAlert();
              return;
          }
          if (p.name.equals("사루만")) {
              sendAlert();
              return;
          }
      }
  }
  ```
