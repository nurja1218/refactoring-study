- 7.3 기본형을 객체로 바꾸기

  - 단순한 정보(문자열이나 숫자)로 표현되던 값이 출력 이상의 기능이 필요해지는 순간 클래스로 바꾼다
  - 프로그램이 커지면 커질 수록 유용하다. 특별한 동작이 필요해지면 확장이나 응용에 좋은 도구가 된다.

  ```tsx
  highPriorityCount =
  		orders.filter(o => "high" === o.priority || "rush" === o.priority).length;

  ------------------------------------------------------------------------------

    constructor(value) {
      if (value instanceof Priority) return value;
      if (Priority.legalValues().includes(value))
        this._value = value;
      else
        throw new Error(`<${value}>는 유효하지 않은 우선순위입니다.`);
    }

    toString() {return this._value;}
    get _index() {return Priority.legalValues().findIndex(s => s === this._value);}
    static legalValues() {return ['low', 'normal', 'high', 'rush'];}
    equals(other) {return this._index > other._index;}
    higherThan(other) {return this._index > other._index;}
    lowerThan(other) {return this._index < other._index;}


   highPriorityCount =
  		 orders.filter(o => o.priority.higherThan(new Priority("normal"))).length;
  ```

  - priority 를 분리하여 priority 를 위한 동작을 추가할 수 있고 그렇게 되면 클라이언트 코드를 더 의미 있게 작성할 수 있게된다.
  - 가독성뿐 아니라 역할 분리도 확실하게 되는 느낌

- 7.4 임시 변수를 질의 함수로 바꾸기

  - 임시 변수를 사용하면 코드 반복을 줄이고 의미를 설명할 수 있어서 유용하다.
  - 아예 함수로 만들어 사용하는 편이 나을 때도 있다.
  - 부자연스러운 의존관계나 부수효과를 찾고 제거하는데 도움된다.
  - 비슷한 로직을 재사용할 수 있어 코드 중복이 줄어들고 관리가 용이해진다.

  ```tsx
  constructor(quantity, item){
      this._quantity = quantity;
      this._item = item;
    }

    get price(){
      var basePrice = this._quantity * this._item.price;
      var discountFactor = 0.98;

      if (basePrice > 1000) discountFactor -= 0.03;
      return baseprice * discountFactor;
    }


    -------------------------------------------------------------------

    get basePrice(){
      return this._quantity * this._item.price;
    }

    get discountFactor(){
      var discountFactor = 0.98;
      if (this.basePrice > 1000) discountFactor -= 0.03;
      return discountFactor;
    }


    get price(){
      return this.basePrice * this.discountFactor;
    }
  ```

- 7.5 클래스 추출하기

  - 클래스는 명확하게 추상화하고 소수의 주어진 역할만 처리해야 한다
  - 메서드와 데이터가 너무 많은 클래스는 이해하기 힘들고 관리가 어렵다
  - 개발을 하다보면 클래스의 기능이 점점 비대해지고 복잡해진다
  - 메서드와 데이터를 따로 묶을 수 있거나,
    함께 변경되는 일이 많거나 의존하는 데이터,
    특정 데이터나 메서드를 제거해도 논리적으로 문제가 없는 경우
    는 분리해도 된다는 신호이다.

        ```tsx
        Class Person
        get name()  {return this._name;}
        set name(arg) {this._name = arg;}
        get telephoneNumber() {return `(${this.officeAreaCode}) ${this.officeNumber}`;}
        get officeAreaCode()  {return this._officeAreaCode;}
        set officeAreaCode(arg) {this._officeAreaCode = arg;}
        get officeNumber() {return this._officeNumer}
        set officeNumber(arg) {this._officeNumber = arg;}

        ---------------------------------------------------------

        Class TeleponeNumber
        get areaCode()  {return this._areaCode;}
        set areaCode(arg) {this._areaCode = arg;}
        get number()  {return this._number;}
        set number(arg) {return this._number = arg;}
        toString()  {return `$({this.areaCode}) ${this.number}`;}

        Class Person
        get officeAreaCode()  {return this._telephoneNumber.areaCode;}
        set officeAreaCode(arg)  {this._telephoneNumber.areaCode = arg;}
        get officeNumber()  {return this._telephoneNumber.number;}
        set officeNumber(arg)  {this._telephoneNumber.number = arg;}
        get telephoneNumber() {return this._telephoneNumber.toString();}
        ```
