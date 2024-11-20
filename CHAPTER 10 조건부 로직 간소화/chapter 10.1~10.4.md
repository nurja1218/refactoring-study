- 10.1 조건문 분해하기
  - 배경
    - 긴 함수는 그자체로 읽기 어렵지만, 조건문은 그 어려움을 한층 가중시킨다.
    - 코드를 부위별로 분해한 다음 해체된 코드 덩어리들을 각 덩어리의 의도를 살린 이름의 함수 호출로 바꿔주면 전체적인 의도가 더 확실히 드러남
  - 절차
    - as-is
      ```jsx
      let charge;

      if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
        charge = quantity * plan.summerRate;
      else charge = quantity * plan.regularRate + planRegularSeviceCharge;

      return charge;
      ```
      - 조건식만 봐서는 의도하는 바를 파악하는데 꽤 오랜시간이 걸린다.
    - to-be
      ```jsx
      if (isSummerPlan(plan)) return summerCharge();
      else return regularCharge();
      ```
      - 조건식과 그 조건식에 딸린 조건절 각각을 함수로 추출한다.
      - 조건식의 의도를 훨씬 쉽게 알 수 있는 상태가 된다.
- 10.2 조건식 통합하기
  - 배경
    - 비교하는 조건은 다르지만 결과로 수행하는 동작은 똑같은 코드들이 있을때
    - 여러 조각으로 나뉜 조건들을 하나로 통합함으로써 내가 하려는 일이 더 명확해진다.
    - 이후 복잡한 조건식을 함수로 추출하면 코드의 의도가 훨씬 분명하게 드러나는 경우가 많다.
    \*\* 하나의 검사라고 생각할 수 없는 검사들이라고 판단되면 리팩터링을 해선 안된다.
  - 절차
      <aside>
      💡
      
      1. 해당 조건식들 모두에 `부수효과가 없는지 확인`한다. 
      (부수효과가 있는 조건식들에는 11.1 질의 함수와 변경 함수 분리하기를 먼저 사용한다.)
      2. 조건문 두개를 선택하여 두 조건문의 조건식들을 논리 연산자로 결합한다. 
      (`순차적으로 이뤄지는 조건문은 or로 결합하고, 중첩된 조건문은 and로 결합한다.`)
      3. 테스트한다.
      4. 조건이 하나만 남을때까지 2~3 과정을 반복한다.
      5. 하나로 합쳐진 조건식을 함수로 추출할지 고려해본다.
      </aside>
      
      - as-is
          
          ```jsx
          if (anEmployee.seniority < 2) return 0;
          if (anEmployee.monthsDisabled > 12) return 0;
          if (anEmployee.isPartTime) return 0;
          ```
          
      - to-be
          
          ```jsx
          if (isNotElligibleForDisability()) return 0;
          
          function isNotElligibleForDisability() {
            return (
              anEmployee.seniority < 2 ||
              anEmployee.monthsDisabled > 12 ||
              anEmployee.isPartTime
            );
          }
          ```

- 10.3 중첩 조건문을 보호 구문으로 바꾸기
  - 배경
    - 조건문의 종류
      - 참인 경로와 거짓인 경로 모두 정상동작인 경우 → `if - else 사용`
      - 한쪽만 정상인 경우 → `비정상 조건(혹은 정상조건)을 검사하여 함수에서 빠져나옴`
  - 절차
      <aside>
      💡
      
      1. 교체해야 할 조건 중 가장 바깥 것을 선택하여 보호 구문으로 바꾼다.
      2. 테스트
      3. 1~2반복
      4. 모든 보호 구문이 같은 결과를 반환한다면 보호 구문들의 조건식을 통합(10.2)한다.
      </aside>
      
      - as-is
          
          ```jsx
          function getPayAmount() {
            let result
            if (isDead) {
              result = deadAmount()
            } else {
              if (isSeparated) {
                result = separatedAmount()
              } else {
                if (isRetired) {
                  result = retiredAmount()
                } else {
                  result = normalAmount()
                }
              }
            }
            return result
          }
          ```
          
      - to-be
          
          ```jsx
          function getPayAmount() {
            if (isDead) return deadAmount();
            if (isSeparated) return separatedAmount();
            if (isRetired) return retiredAmount();
            return normalPayAmount();
          }
          ```
          
          - 순차적으로 보호구문을 만들다 보면 가변 변수를 제거해도 되는 상황까지 와서 일석이조
          
          ** 리팩터링을 수행할 때 조건식을 반대로 만들어 적용하는 경우도 많다

- 10.4 조건부 로직을 다형성으로 바꾸기
  - 배경
    - 복잡한 조건부 로직은 해석하기 가장 난해한 대상이다.
    - 종종 더 높은 수준의 개념을 도입해서 분리할 수 있다.
    - 클래스와 다형성을 이용하면 더 확실하게 분리할 수도 있다.
    - 예) switch - case 문의 경우 case별로 클래스를 하나씩 만들어 공통 switch 로직의 중복을 없앨 수 있다.
    - 모든 조건부 로직을 다형성으로 대체하는건 추천하지 않는다.
  - 절차
      <aside>
      💡
      
      1. 다형적 동작을 표현하는 클래스를 만든다. 조건에 따라 알맞은 인스턴스를 반환하는 팩터리 함수도 만든다.
      2. 팩터리 함수를 사용하도록 코드를 변경한다.
      3. 리팩터링 대상인 조건부 로직을 함수로 추출하고, 수퍼클래스의 메서드로 옮긴다.
      4. 서브 클래스에서 수퍼클래스의 조건부 로직 메서드를 오버라이드 한다.
          
          조건부에서 해당 서브클래스에 해당하는 조건절을 서브클래스 메서드에서 적절히 수정한다.
          
      5. 같은 방식으로 각 조건절을 해당 서브클래스에서 메서드로 구현한다.
      6. 수퍼클래스 메서드에서는 기본 동작만 남긴다. 
      필요에 따라 추상클래스로 선언하거나, 서브클래스에서 처리해야함을 알리도록 에러를 던진다.
      </aside>
      
      - as-is
          
          ```jsx
          function plumage (bird) {
          	switch (bird.type) {
          	  case "유럽 제비":
          	    return "보통이다";
          	  case "아프리카 제비":
          	    return (bird.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
          	  case "노르웨이 파랑 앵무":
          	    return (bird.voltage > 100) ? "그을렸다" : "예쁘다";
          	  default:
          	    return "알 수 없다";
          	}
          }
          ```
          
          - 깃털 상태를 알기 위해 각 새의 종에 따라 조건문이 복잡하게 구성되어 있다.
      - to-be
          
          ```jsx
          function createBird(bird) {
          	swtich (bird.type) {
          		... return new 해당 새의 종 클래스
          	}
          }
          ```
          
          ```jsx
          // 수퍼클래스
          class Bird {
          	get plumage() {...}
          }
          
          // 각 새 종별 서브클래스
          class EuropeanSwallow extends Bird {
            get plumage() {
              return "보통이다";
            }
          class AfricanSwallow extends Bird {
            get plumage() {
               return (this.numberOfCoconuts > 2) ? "지쳤다" : "보통이다";
            }
          class NorwegianBlueParrot extends Bird {
            get plumage() {
               return (this.voltage > 100) ? "그을렸다" : "예쁘다";
            }
          ```
          
          - 종별 서브클래스를 생성하고 적합한 인스턴스를 반환하는 팩토리 함수를 준비한다.
          - 깃털을 얻는 조건부 메서드를 처리하기 위해 해당 서브클래스에서 오버라이드 한다.
          - 수퍼클래스의 조건문에 throw문을 통해 정의하지 않은 클래스의 동작을 방지할 수 있다.
          
          ** (주관적 견해) 덕타이핑은 별로 좋아보이지 않음
          
      
      ---
      
      - as-is
          - 신용 평가 기관에서 선박의 항해 투자 등급을 계산하는 코드
          (위험요소로 항해 경로의 자연조건과 선장의 항해 이력을 고려)
          
          ```jsx
          function rating(voyage, history) { // 투자 등급
          	const vpf = voyageProfitFactor(voyage,history);
          	const vr = voyageRisk(voyage);
          	const chr = captainHistoryRisk(voyage, history);
          	if (vpf * 3 > (vr + chr * 2 )) return "A";
          	else return "B";
          }
          
          function voyageRisk(voyage) { // 항해 경로 위험요소
          	let result = 1;
          	if (voyage.length > 4) result += 2;
          	if (voyage.length > 8) result += voyage.length - 8;
          	if (["중국", "동인도"].includes(voyage.zone)) result += 4;
          	return Math.max(result, 0);
          }
          
          function captainHistoryRisk(voyage, history) { // 선장의 항해 이력 위험 요소
            let result = 1;
            if (history.length < 5) result += 4;
            result += history.filter(v => v.profit < 0).length;
            if (voyage.zone === '중국' && hasChina(history)) result -= 2;
          
          	return Math.max(result, 0);
          }
          
          function hasChina(history) {} // 중국을 경유하는가?
          
          function voyageProfitFactor(voyage, history) { // 수익 요인
            let result = 2;
            if (voyage.zone === '중국') result += 1;
            if (voyage.zone === '동인도') result += 1;
            if (voyage.zone === '중국' && hasChina(history)) {
              result += 3;
          
              if (history.length > 10) result += 1;
              if (voyage.length > 12) result += 1;
              if (voyage.length > 18) result -= 1;
            } else {
              if (history.length > 8) result += 1;
              if (voyage.length > 14) result -= 1;
            }
          
            return result;
          }
          ```
          
          - 다형성을 적용하기 전 함수를 클래스로 묶기 먼저 적용한다.
          - voyageProfitFactor 에서 조건부 블록 전체를 함수로 추출한다.
      - to-be
          
          ```jsx
          function createRating(voyage, history) {
          	if (voyage.zone === "중국" && history.some(v => "중국" === v.zone))
          		return new ExperiencedChinaRating(voyage, history);
          	else return new Rating(voyage, history);
          }
          
          function rating(voyage, history) {
          	return createRating(voyage, history).value;
          }
          
          class Rating {
            ...
            
            get value () {...}
            
            get voyageRisk () {...}
          
            get captainHistoryRisk() {
              let result = 1;
              if (this.history.length < 5) result += 4;
              result += this.history.filter(v => v.profit < 0).length;
              return Math.max(result, 0);
            }
          
            get voyageProfitFactor() {
              let result = 2;
              if (this.voyage.zone === '중국') result += 1;
              if (this.voyage.zone === '동인도') result += 1;
              result += this.historyLengthFactor;
              result += this.voyageLengthFactor;
              return result;
            }
          
            get historyLengthFactor() {
              return this.history.length > 8 ? 1 : 0;
            }
          
            get voyageLengthFactor(): number {
              return this.voyage.length > 14 ? -1 : 0;
            }
          }
          
          ---------------------------------------------------
          
          // 특수 로직을 오버라이드 하는 서브클래스
          class ExperiencedChinaRating extends Rating {
          
            get captainHistoryRisk() {
              const result = super.captainHistoryRisk - 2;
              return Math.max(result, 0);
            }
            
            get voyageProfitFactor() {
              return super.voyageProfitFactor + 3;
            }
          
            get historyLengthFactor() {
              return this.history.length > 10 ? 1 : 0;
            }
          
            get voyageLengthFactor() {
              let result = 0;
              if (this.voyage.length > 12) result += 1;
              if (this.voyage.length > 18) result -= 1;
              return result;
            }
          }
          ```
          
          - 기본 Rating 클래스에는 중국 항해 경험이 있는지와 관련된 복잡한 조건에서 벗어난 기본 로직만 남아있다.
          - 중국 항해 경험이 있을 때를 담당하는 클래스는 기본 클래스와의 차이만 담고 있다.
          - 더 가다듬기
              - 변형 동작은 슈퍼클래스와의 차이를 표현해야 하는 서브클래스에서만 신경 쓰면 된다.
              - And는 메서드가 두 가지 독립된 일을 수행한다고 소리친다. (history와 voyage 별로 분리)
