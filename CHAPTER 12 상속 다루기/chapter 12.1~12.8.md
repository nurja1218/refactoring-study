- 12.1 메서드 올리기

  - 중복되는 코드가 존재하는 경우 활용한다.
  - 슈퍼클래스에서 정의하여 서브클래스에서는 상속받아 활용할 수 있도록 한다.
  - 전체 흐름은 비슷하지만 세부 내용이 다른 경우 [템플릿 메서드](https://refactoring.com/catalog/formTemplateMethod.html)를 사용한다.
    ⇒ 템플릿 메서드 : 결국 공통 부분을 분리하여 메서드를 찢는 내용입니다.

  ```tsx
  // AS-IS
  class Employee {}

  class SalesPerson extends Employee {
    get name() {}
  }

  class Engineer extends Employee {
    get name() {}
  }
  ```

  ```tsx
  // TO-BE
  class Employee {
    get name() {}
  }

  class SalesPerson extends Employee {}

  class Engineer extends Employee {}
  ```

- 12.2 필드 올리기

  - 중복되는 필드가 존재하는 경우 사용한다.

  ```tsx
  // AS-IS
  class Employee {}

  class SalesPerson extends Employee {
    private name: string;
  }

  class Engineer extends Employee {
    private name: string;
  }
  ```

  ```tsx
  // TO-BE
  class Employee {
    private name: string;
  }

  class SalesPerson extends Employee {}

  class Engineer extends Employee {}
  ```

- 12.3 생성자 본문 올리기

  - 복수 개의 서브클래스 생성자에서 동일하게 초기화를 시키는 로직이 있는 경우 사용한다.

  ```tsx
  // AS-IS
  class Employee {}

  class SalesPerson extends Employee {
    constructor(name, managingProductList) {
      super();
      this.name = name;
      this.managingProductList = managingProductList;
    }
  }

  class Engineer extends Employee {
    constructor(name, scrumName) {
      super();
      this.name = name;
      this.scrumName = scrumName;
    }
  }
  ```

  ```tsx
  // AS-IS
  class Employee {
    private name: string;
    constructor(name) {
      this.name = name;
    }
  }

  class SalesPerson extends Employee {
    constructor(name, managingProductList) {
      super(name);
      this.managingProductList = managingProductList;
    }
  }

  class Engineer extends Employee {
    constructor(name, scrumName) {
      super(name);
      this.scrumName = scrumName;
    }
  }
  ```

    <aside>
    💡
    
    항상 이때마다 아쉬운 것은 @nestjs/swagger에서 지원해주는 PickType이 아쉬운 것 같아요.
    이전에 코드를 열어봤을 때 내부에서 생성자는 가져오지 않도록 신규 클래스가 생성되어 반환되더라구요.
    
    </aside>

- 12.4 메서드 내리기

  - 특정 서브클래스에서만 필요한 메서드는 슈퍼클래스에서 제외한다.
  - 단, 메서드 내리기 리팩토링을 시도하는 경우는 다른 서브클래스들이 무엇을 호출하고 있는지를 정확하게 알고 있을 때에만 수행한다.

  ```tsx
  // AS-IS
  class Employee {
    get salesGoal() {}
  }

  class SalesPerson extends Employee {}
  class Engineer extends Employee {}
  ```

  ```tsx
  // TO-BE
  class Employee {}

  class SalesPerson extends Employee {
    get salesGoal() {}
  }
  class Engineer extends Employee {}
  ```

- 12.5 필드 내리기

  - 특정 서브클래스에서만 사용하는 필드는 슈퍼클래스에서 제외한다.

  ```tsx
  // AS-IS
  class Employee {
    private lastSalesRecord: string;
  }

  class SalesPerson extends Employee {}

  class Engineer extends Employee {}
  ```

  ```tsx
  // TO-BE
  class Employee {}

  class SalesPerson extends Employee {
    private lastSalesRecord: string;
  }

  class Engineer extends Employee {}
  ```

- 12.6 타입 코드를 서브 클래스로 바꾸기
  - 다형성을 적용할 수 있다면 서브 클래스를 적극적으로 활용하자.
  - 케이스 별로 하나씩 캡슐화 시키고 서브 클래스를 생성한다.
  ```tsx
  // AS-IS
  function createEmployee(name, type) {
    return new Employee(name, type);
  }
  ```
  ```tsx
  // TO-BE
  function createEmployee(name, type) {
    switch (type) {
      case "engineer":
        return new Engineer(name);
      case "salesPerson":
        return new SalesPerson(name);
    }
  }
  ```
- 12.7 서브클래스 제거하기

  - 더이상 서브클래스의 기능이 명확하지 않거나 사용되지 않는 서브 클래스는 슈퍼클래스의 필드로 대체한다.

  ```tsx
  // AS-IS
  class Person {
    get genderCode() {
      return "X";
    }
  }

  class Male extends Person {
    get genderCode() {
      return "M";
    }
  }

  class Female extends Person {
    get genderCode() {
      return "F";
    }
  }
  ```

  ```tsx
  // TO-BE
  class Person {
    private gender: string;

    get genderCode() {
      return this.genderCode;
    }
  }
  ```

- 12.8 슈퍼클래스 추출하기

  - 비슷한 일을 하는 복수 개의 클래스가 있다면 슈퍼클래스를 생성하여 상속받는 구조로 변경하자
    <aside>
    💡

    중복되는 동작을 상속으로 해결할 지 위임(12.11절)으로 해결할 지는 선택해야한다.
    다만, 슈퍼클래스 추출하기 방법이 리팩토링하기 좀 더 쉽다.

    </aside>

    ```tsx
    // AS-IS
    class Department {
      get totalAnnualCost() {}
      get name() {}
      get headCount() {}
    }

    class Employee {
      get annualCost() {}
      get name() {}
      get id() {}
    }
    ```

    ```tsx
    // TO-BE
    class Party {
      get name() {}
      get annualCost() {}
    }

    class Department extends Party {
      get headCount() {}
    }

    class Employee extends Party {
      get id() {}
    }
    ```
