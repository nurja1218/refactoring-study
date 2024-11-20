- 7.1 레코드 캡슐화하기
  - 캡슐화를 하는 이유?
    - 객체를 사용하면 어떻게 저장했는지를 숨긴 채 값을 각각의 메서드로 제공할 수 있다
      - 사용자는 무엇이 저장된 값이고 무엇이 계산된 값인지 알 필요가 없다
    - 캡슐화하면 이름을 바꿀 때도 좋다. 필드 이름을 바꿔도 기존 이름과 새 이름 모두를 각각의 메서드로 제공할 수 있어서 사용자 모두가 새로운 메서드로 옮겨갈 때까지 점진적으로 수정할 수 있다.
      - 클래스(server) 내 필드나 함수(api)들의 변경사항이 사용자(client)들에게 영향을 끼치지 않는다
      - 중첩된 리스트나 해시맵을 받아서 JSON이나 XML같은 포맷으로 직렬화 할 때가 많다. 이런 구조 역시 캡슐화할 수 있는데, 그러면 나중에 포맷을 바꾸거나 추적하기 어려운 데이터를 수정하기가 수월해진다
        - 예시 코드
        ```
        "1920":{
           name:"김요한",
           id:"1920",
           usages:{
             "2022":{
               "1": 50,
               "2":55
             },
             "2021":{
               "1":70,
               "2":75,
             }
           }
         }
        ```
        - 변환 코드
        ```
        class CustomerData {
          constructor(data) {
            this._data = data;
          }
          setUsage(customerId, year, month, amount) {
            this._data[customerId].usages[year][month] = amount;
          }
          get rawData() {
            return _.cloneDeep(this.data);
          }
        }
        ```
        - 코드 읽는 방법1 : 읽는 코드를 모두 독립 함수로 추출
        ```
        usageXXXX(customerId, year, month) {
            return this._data[customerId].usages[year][month];
          }

        usageYYYY(customerId) {
            return this._data[customerId].usages[year][month];
          }

        const later = getCustomerData().usageXXXX(customerID, laterYear, month);

        // custormer의 모든 쓰임을 명시적인 API로 제공
        // 하지만 읽는 패턴이 다양하면 그만큼 작성할 코드가 늘어남
        // 리스트-해시 데이터 형태로 넘겨주는 것도 가능
        ```
        - 코드 읽는 방법2 : 읽는 코드를 원본 데이터를 복사해서 반환하는 방식
        ```
        const later = getCustomerData().rawData[customerID].usage(customerID, laterYear, month);

        // 데이터 구조가 클수록 복제 비용이 커져서 성능이 느려질 수 있다
        ```
- 7.2 컬렉션 캡슐화하기
  - 컬렉션 변수로의 접근을 캡슐화하면서 게터가 컬렉션 자체를 반환하도록 한다면, 그 컬렉션을 감싼 클래스가 눈치채지 못하는 상태에서 컬렉션의 원소들이 바뀌어버릴 수 있다
    - 예시) 컬렉션을 감싼 클래스에 흔히 add(), remove()라는 이름의 컬렉션 변경자 메서드를 만든다
    - 컬렉션 게터가 원본 컬렉션을 반환하지 않게 만들어서 클라이언트가 실수로 컬렉션을 바꿀 가능성을 차단한다
      - 내부 컬렉션을 직접 수정하지 못하게 막는다
      - 컬렉션을 읽기 전용으로 만든다
      - 컬렉션 게터를 활용하되 내부 컬렉션의 복제본을 반환
