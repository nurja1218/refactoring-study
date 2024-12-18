- 12.9 계층 합치기
  계층 구조도가 진화하면서 어떤 클래스와 그 부모가 비슷해져서 더는 독립적으로 존재할 이유가 사라지는 경우 둘을 합친다

  ```jsx
  to-be
  class Employee {...}
  class Saleperson extends Employee {...}

  as-is
  class Employee {...}
  ```

  - 절차
    - 두 클래스 중 제거할 클래스 선택
    - 필드 올리기와 메서드 올리기 혹은 필드 내리기와 메서드 내리기를 적용하여 하나의 클래스오 옮긴다
    - 제거할 클래스를 참조하던 모든 코드가 남겨질 클래스를 참조하도록 변경
    - 빈 클래스 제거
    - 테스트

- 12.10 서브클래스를 위임으로 바꾸기
  공통 데이터와 동작은 모두 슈퍼클래스에 두고 서브클래스는 자신에 맞게 기능을 추가하거나 오버라이드한다
  상속의 단점은 한 번만 사용할 수 있고, 상속은 클래스들의 관계를 아주 긴밀하게 결합하여 부모를 수정하면 이미 존재하는 자식들의 기능을 해치기가 쉽기 때문에 각별히 주의가 필요하다.
  위임은 위 두 단점을 해결, 위임은 객체 사이의 일반적인 관계이므로 상호작용에 필요한 인터페이스를 명확히 정의할 수 있다.

  ```jsx
  as - is;
  class Order {
    get daysToShip() {
      return this._warehouse.daysToShip;
    }
  }

  class PriorityOrder extends Order {
    get daysToShip() {
      return this._priorityPlan.daysToShip;
    }
  }

  to - be;
  class Order {
    get daysToShip() {
      return this._priorityDelegate
        ? this._priorityDelegate.daysToShip
        : this._warehouse.daysToShip;
    }
  }

  class PriorityOrderDelegate {
    get daysToShip() {
      return this._priorityPlan.daysToShip;
    }
  }
  ```

  - 절차
    1. 생성자를 호출하는 곳이 많다면 생성자를 팩터리 함수로 바꾼다
    2. 위임으로 활용할 빈 클래스를 만든다.
    3. 위임을 저장할 필드를 슈퍼클래스에 추가한다.
    4. 서브클래스 생성 코드를 수정하여 위임 인스턴스를 생성하고 위임 필드에 대입해 초기화한다.
    5. 서브클래스의 메서드 중 위임 클래스로 이동할 것을 고른다.
    6. 함수 옮기기를 적용해 위임 클래스로 옮긴다.
    7. 서브클래스 외부에도 원래 메서드를 호출하는 코드가 있다면 서브클래스의 위임 코드를 슈퍼클래스로 옮긴다.
    8. 테스트한다.
    9. 서브클래스의 모든 메서드가 옮겨질 때까지 5~8 과정 반복
    10. 서브클래스들의 생성자를 호출하는 코드를 찾아서 슈퍼클래스의 생성자를 사용하도록 수정한다.
    11. 테스트한다.
    12. 서브클래스를 삭제한다.(죽은코드 삭제)

- 12.11 슈퍼클래스를 위임으로 바꾸기
  상속을 위임으로 전환
  객체 지향 프로그래밍에서 상속은 기존 기능을 재활용하는 강력하고 손쉬운 수단
  기존 클래스를 상속하여 입맛에 맞게 오버라이드하거나 새 기능을 추가하면 되지만 상속이 혼란과 복잡도를 키우는 경우도 발생
  자바의 스택 클래스
  리스트의 연산 중 스택에는 적용되지 않는 게 많음에도 그 모든 연산이 스택 인터페이스에 그대로 노출
  제대로된 상속이라면 서브클래스가 슈퍼클래스의 모든 기능을 사용함은 물론, 서브클래스의 인스턴스를 슈퍼클래스의 인스턴스로도 취급할 수 있어야한다.
  위임을 이용하면 기능 일부만 빌려올 뿐 서로 별개인 개념이 명확해진다.

  ```jsx
  as-is
  class List {...}
  class Stack extends List {...}

  to-be
  class Stack {
  	constructor() {
  		this._storage = new List();
  	}
  }
  class List() {...}
  ```

  - 절차
    - 슈퍼클래스 객체를 참조하는 필드를 서브클래스에 만든다.
    - 슈퍼클래스의 동작 각각에 대응하는 전달 함수를 서브클래스에 만든다. 서로 관련된 함수끼리 그룹으로 묶어 진행하며, 그룹을 하나씩 만들때마다 테스트한다.
    - 슈퍼클래스의 동작 모두가 전달 함수로 오버라이드되었다면 상속 관계를 끊는다.
