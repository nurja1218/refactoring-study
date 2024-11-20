- **11.8 생성자를 팩터리 함수로 바꾸기**
  - 기본 생성자는 반드시 해당 클래스의 인스턴스를 반환해야 하므로 다른 종류의 객체를 반환하거나, 객체 생성 과정에서 좀 더 복잡한 처리를 수행하기 어렵다
    ex. `new Employee()` → 항상 `Employee` 클래스의 인스턴스가 생성
  - 팩터리 함수는 일반적인 함수로 함수를 제약없이 생성하고 다양한 처리를 수행할 수 있다
  - 기본 생성자 이상의 동작을 구현하고 싶을 때
  ```tsx
  // before
  leadEngineer = new Employee(document.leadEngineer, "E");
  // after
  leadEngineer = createEngineer(document.leadEngineer);

  // ---------

  // before 2 - 일반/당일 배송
  class Shipping {
    public cost: number;
    public estimatedDeliveryDays: number;

    constructor(type: string) {
      if (type === "standard") {
        this.cost = 5;
        this.estimatedDeliveryDays = 5;
      } else if (type === "express") {
        this.cost = 15;
        this.estimatedDeliveryDays = 1;
      } else {
        throw new Error("Invalid shipping type");
      }
    }
  }

  // after 2 - 일반/당일 배송
  class Shipping extends BaseEntity {
    constructor(public cost: number, public estimatedDeliveryDays: number) {
      super();
    }
  }

  class StandardShipping extends Shipping {
    constructor() {
      super(2500, 3); // 비용 2500, 5일 소요
    }
  }

  class ExpressShipping extends Shipping {
    constructor() {
      super(3500, 1); // 비용 3500, 1일 소요
    }
  }

  // 팩터리 함수
  function createShipping(type: string): Shipping {
    if (type === "standard") {
      return new StandardShipping();
    } else if (type === "express") {
      return new ExpressShipping();
    } else {
      throw new Error("Invalid shipping type");
    }
  }

  // 사용 예시
  const shipping = createShipping("express");
  ```
- **11.9 함수를 명령으로 바꾸기**
  - 명령 객체(Command)는 하나의 함수를 **객체로 캡슐화**한 것을 의미한다
  - 이 객체는 함수에 필요한 데이터를 **상태**로 저장하고, **행동**(실행할 함수)을 객체의 메서드로 정의한다
  - 명령은 평범한 함수 메커니즘보다 훨씬 유연하게 함수를 제어하고 표현할 수 있다
  - 일급 함수를 지원하지 않는 언어를 사용할때 일급 함수의 기능 대부분을 흉내 낼 수 있다
  - 단, 유연성은 (언제나 그렇듯) 복잡성을 키우고 얻는 대가임을 잊지 말아야 한다
  - 일급 함수를 지원하는 언어라면 가능한 일급 함수를 사용한다
  - 일급함수
    - 함수가 변수처럼 다뤄질 수 있는 기능
    - **변수에 저장**될 수 있고, 다른 함수의 인자로 전달될 수 있고, **함수에서 반환값**으로 사용할 수 있다
  ```tsx
  // before
  function score(candidate, medicalExam, scoringGuide) {
    let result = 0;
    let healthLevel = 0;
    // ...
    // 건강 상태에 따라 점수 계산
  	if (medicalExam.isSmoker) {
  	    healthLevel -= 10;
  	}
  	if (scoringGuide.state === "CA") {
  	    result -= 5;
  	}
  	// 다른 계산 로직들
  	return { result, healthLevel };
  }

  // after
  class Scorer {

    constructor(candidate, medicalExam, scoringGuide) {
      this._candidate = candidate;
      this._medicalExam = medicalExam;
      this._scoringGuide = scoringGuide;
      this._result = 0;
      this._healthLevel = 0;
      this._previousState = null;
    }

    execute() {
      // 이전 상태 저장
      this._previousState = { result: this._result, healthLevel: this._healthLevel };

      this._result = 0;
      this._healthLevel = 0;

      // 점수 계산 로직
      if (this._medicalExam.isSmoker) {
          this._healthLevel -= 10;
      }

      if (this._scoringGuide.state === "CA") {
          this._result -= 5;
      }

      // ...
      return { this._result, this._healthLevel }
    }

    undo() {
        if (this._previousState) {
            // 이전 상태로 되돌리기
            this._result = this._previousState.result;
            this._healthLevel = this._previousState.healthLevel;
            return this._previousState;
        }
    }
  }
  ```
- **11.10 명령을 함수로 바꾸기**
  - 명령 객체는 복잡한 연산을 다를 수 있는 강력한 메커니즘을 제공한다
  - 명령의 이런 능력은 공짜가 아니다, 로직이 크게 복잡하지 않다면 명령 객체는 장점보다 단점이 크니 평범한 함수로 바꿔주는 게 낫다
  - 불필요한 명령을 제거하여 복잡성을 낮춘다
  - 명령의 로직이 크게 복잡하지 않은 경우
  ```tsx
  // before - 요금을 계산하는 단순한 작업을 명령 객체로 구현
  class ChargeCalculator {
    constructor(customer, usage) {
      this._customer = customer;
      this._usage = usage;
    }
    execute() {
      return this._customer.rate * this._usage;
    }
  }
  // after - 동일한 계산을 하고, 명령 객체가 가져오는 복잡성을 제거
  function charge(customer, usage) {
    return customer.rate * usage;
  }
  ```
- **11.11 수정된 값 반환하기**
  - 데이터가 어떻게 수정되는지를 추적하는 일은 코드에서 이해하기 가장 어려운 부분 중 하나다
  - 데이터가 수정된다면 그 사실을 명확히 알려주어서 함수가 무슨 일을 하는지 쉽게 알 수 있게 하는게 중요하다
  - 가장 좋은 방법으로는 변수를 갱신하는 함수라면 수정된 값을 반환하게 하는 방법이 있다
  - 데이터가 수정됨을 명확히 알려주어 코드를 이해하기 쉬워진다
  - 값 하나를 계산한다는 분명한 목적이 있는 함수들에 효과적이다
  ```tsx
  // before
  let totalAscent = 0;
  calculateAscent();

  function calculateAscent() {
    for (let i = 1; i < points.length; i++) {
      const verticalChange = points[i].elevation - points[i - 1].elevation;
      totalAscent += verticalChange > 0 ? verticalChange : 0;
    }
  }
  // after
  const totalAscent = calculateAscent();

  function calculateAscent() {
    let result = 0;
    for (let i = 1; i < points.length; i++) {
      const verticalChange = points[i].elevation - points[i - 1].elevation;
      result += verticalChange > 0 ? verticalChange : 0;
    }
    return result;
  }
  ```
  - 수정된 값을 반환하지 않는 경우는?
    - 객체의 내부 상태 관리 → 외부 변수를 내부 함수에서 직접 수정
    - 로그 기록
    - db update → 이것도 결국 서비스 함수 내에서 업데이트 결과를 조회해서 반환하는 경우가 많음
      ```tsx
      class Counter {
        private count = 0;

        increment() {
          this.count++; // 상태만 수정하고 값을 반환하지 않음
        }
        getCount() {
          return this.count; // 외부에서 count 값을 확인할 수 있는 메서드 제공
        }
      }
      ```
- **11.12 오류 코드를 예외로 바꾸기**
  - 예전에는 오류 코드를 사용하는게 보편적이었으나, 이 경우 오류 코드를 매번 검사해서 콜스택으로 전파하거나 직접 처리하는 처리가 필요하다
  - 예외는 프로그래밍 언어에서 제공하는 독립적인 오류 처리 메커니즘이다
  - 오류가 발견되면 예외를 던지고 적절한 예외 핸들러를 찾을 때까지 콜스택을 타고 위로 전파되어 오류 코드를 일일이 검사하지 않아도 되어 편리하다
  - 오류 발생에 따른 복잡한 상황에 대처하는 코드를 작성하거나 읽을 일이 없게 해준다
  - 예외는 프로그램의 정상 동작 범주에 들지 않는 오류를 나타낼 때만 쓰여야 한다
  - 괜찮은 경험 법칙으로는 예외를 던지는 코드를 프로그램 종료 코드로 바꿔도 프로그램이 여전히 정상 동작할지를 따져보는 것이다
  - 정상 동작하지 않을 것 같다면 예외 대신 오류를 검출하여 프로그램을 정상 흐름으로 되돌리게끔 처리해야 한다
  ```tsx
  before;
  if (data) return new ShippingRules(data);
  else return -23;
  after;
  if (data) return new ShippingRules(data);
  else throw new OrderProcessingError(-23);
  ```
- **11.13 예외를 사전확인으로 바꾸기**
  - 예외라는 개념은 프로그래밍 언어의 발전에 의미 있는 한걸음이었다
  - 하지만 좋은 것들이 늘 그렇듯, 예외도 과용되곤 한다. 예외는 '뜻밖의 오류' 라는, 말 그대로 예외적으로 동작할 때만 쓰여야 한다
    → **예외를 남용하지 않고, 사전에 조건을 확인하는 방식**을 권장
  - 함수 수행 시 문제가 될 수 있는 조건을 함수 호출 전에 검사할 수 있다면, 예외를 던지는 대신 호출하는곳에서 조건을 검사해야 한다
  - 예외가 정말로 의미있는 부분에서만 사용될 수 있도록 한다
  - 예외 대신 사전확인이 가능할 때
  ```tsx
  // before
  double getValueForPeriod (int periodNumber) {
    try {
      return values[periodNumber];
    } catch (ArrayIndexOutOfBoundsException e) {
      return 0;
    }
  }
  // after
  double getValueForPeriod (int periodNumber) {
    return (periodNumber >= values.length) ? 0 : values[periodNumber];
  }

  // mono repo
  ...
  async uploadImage(
      @Body() body: ImageUploadRequestDto,
    ): Promise<ImageUploadResultDto> {
      try {
        return await this.imageUploadService.uploadSquareImage(body.file);
      } catch (e) {
        if (e instanceof NotSquareImageError) {
          throw new BadRequestException(
            HTTP_RESPONSE_ERROR_CODE.UPLOAD_IMAGE.UPLOADIMAGE004,
          );
        } else {
          throw e;
        }
      }
    }
  ```
