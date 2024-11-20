- 8.5 인라인 코드를 함수 호출로 바꾸기
  ```jsx
  AS-IS

  let appliesToMass = false;
  for(const s of states) {
      if (s === 'MA') appliesToMass = true;
  }

  --------------------------------------------

  TO-BE

  appliesToMass = states.includes('MA');
  ```
  - 함수는 여러 동작을 하나로 묶어준다
  - 함수의 이름이 코드 동작 방식보다는 목적을 말해주기 때문에 코드를 이해하기 더 쉬워진다
  - 동작을 변경할때도 비슷해 보이는 코드를 일일이 찾아 수정 대신 함수 하나만 수정하면 된다 → 모듈화
- 8.6 문장 슬라이드하기
  ```jsx
  AS-IS

  const pricingPlan = retrievePricingPlan();
  const order = retreiveOrder();
  let charge;
  const chargePreUnit = pricingPlan.unit;

  ---------------------------------------------

  TO-BE

  const pricingPlan = retrievePricingPlan();
  const chargePreUnit = pricingPlan.unit;
  const order = retreiveOrder();
  let charge;
  ```
  - 관련된 코드들이 가까이 모여 있으면 이해하기 더 쉽다
  - 조건문이 있을 때의 슬라이드
    ```jsx
    let result;
    if (avaliableResoureces.length === 0) {
      result = createResourece();
      allocatedResources.push(result);
    } else {
      result = avaliableResource.pop();
      allocatedResources.push(result);
    }
    return result;
    ```
  - 중복된 문장들을 조건문 밖으로 슬라이드 할 수 있다.
  ```jsx
  let result;
  if (avaliableResoureces.length === 0) {
    result = createResourece();
  } else {
    result = avaliableResource.pop();
  }
  allocatedResources.push(result);
  return result;
  ```
- 8.7 반복문 쪼개기
  ```jsx
  AS-IS

  let averageAge = 0;
  let totalSalary = 0;
  for (const p of people) {
      averageAge += p.age;
      totalSalary += p.salary;
  }
  averageAge = averageAge / people.length;

  -----------------------------------------

  TO-BE

  let totalSalary = 0;
  for (const p of people) {
      totalSalary += p.salary;
  }

  let averageAge = 0;
  for (const p of people) {
      averageAge += p.age;
  }

  averageAge = averageAge / people.length;
  ```
  - 반복문 하나에 두 가지 일을 수행 → 반복문 수정해야 할 때 마다 두 가지 일 모두 잘 이해하고 진행해야한다
  - 반복문을 분리해두면 수정할 동작 하나만 이해해도 된다
  - 리팩토링과 최적화를 구분
    - 최적화는 코드를 깔끔히 정리한 이후 수행
    - 반복문 두 번 실행하는 것이 병목이라면 그 때 다시 하나로 합치면 된다
  - 절차
    1. 반복문을 복제해 두 개로 만든다
    2. 반복문이 중복되어 생기는 부수효과를 파악하여 제거
    3. 테스트
    4. 완료되면 각 반복문을 함수로 추출할지 고민
- 8.8 반복문을 파이프라인으로 바꾸기
  ```jsx
  AS-IS

  const names = [];
  for (const i of input) {
      if (i.job === 'programer')
          names.push(i.name);
  }

  -------------------------------

  TO-BE

  const names = input
      .filter(i => i.job === 'programer')
      .map(i => i.name);
  ```
  - 절차
    1. 반복문에서 사용하는 컬렉션을 가리키는 변수를 하나 만든다
    2. 반복문의 첫 줄부터 시작해서, 각각의 단위 행위를 적절한 컬렉션 파이프라인 연산으로 대체한다
       - 컬렉션 파이프라인은 1에서 만든 반복문 컬렉션 변수에서 시작하여, 이전 연산의 결과를 기초로 연쇄적으로 수행된다. 하나를 대체할 때 마다 테스트한다.
    3. 반복문의 모든 동작을 대체했다면 반복문 자체를 지운다
- 8.9 죽은 코드 제거하기
  ```jsx
  if (false) {
  	doSomethingThatUsedToMatter();
  }
  --------------------------------
  ```
  - 사용되지 않는 코드가 있다면 동작 이해하는 데 큰 걸림돌이 될 수 있다
  - 절차
    1. 죽은 코드를 외부에서 참조할 수 있는 경우라면(예컨대 함수 하나가 통째로 죽었을 때) 혹시라도 호출하는 곳이 있는지 확인한다.
    2. 없다면 죽은 코드를 제거한다.
    3. 테스트한다.
