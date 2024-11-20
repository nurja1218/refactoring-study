- 6.9 여러 함수를 클래스로 묶기
    <aside>
    💡 데이터를 갱신할 가능성이 있다면 클래스로 캡슐화하라!    ex. Entity, ResponseDto
    
    </aside>
    
    - 클래스 묶기의 장점
        - 공통 환경에서의 명확한 표현
        - 간결한 함수 호출
        - 일관적인 관리
    - SOLID 단일원칙접근(SRP) 적용하기 ⇒ getter 활용하기
        - function 활용하기
            
            ```tsx
            // 클래스
            class Ticket{
            	function calculateBaseCharge(){
            		return baseRate(this.month, this.year) * this.quantity;
            	}
            }
            ```
            
            ```tsx
            // 사용처(Client)
            const ticket = new Ticket();
            const basicChargeAmount = ticket.calculateBaseCharge();  //함수를 통해 계산한다는 것을 client에서 유추할 수 있음
            ```
            
        - getter 활용하기
            
            ```tsx
            // 클래스
            class Ticket{
            	get baseCharge(){
            		return baseRate(this.month, this.year) * this.quantity;
            	}
            }
            ```
            
            ```tsx
            // 사용처(Client)
            const ticket = new Ticket();
            const basicChargeAmount = ticket.baseCharge;  //client에서는 계산된 필드인지 유추할 수 없음
            ```
            
            <aside>
            💡 Client 입장에서는 인스턴스의 필드가 실제 필드인지 계산된 값인지 알 수 없다.
            
            </aside>

- 6.10 여러 함수를 반환 함수로 묶기
  - 변환 함수로 묶는 방법 2가지
    - 검색 및 갱신을 일관된 함수에서 진행하여 `중복`을 막는다.
    - 클래스로 묶는다.
    - 2가지 방식의 차이점 : 원본 데이터가 코드 내에서 갱신되는 경우 클래스로 묶는다. ⇒ 일관성을 유지할 수 있다.
  - example
    ```tsx
    // AS-IS
    function productLikeCount() { ... };
    function updateEsProduct() { ... };
    ```
    ```tsx
    // TO-BE
    function updateEsProductLikeCount(){
    	const count = this.productLikeCount();
    	updateEsProduct({
    		...,
    		likeCount: count,
    	})
    }
    ```
      <aside>
      💡 유저의 상품 좋아요, 상품 좋아요 취소 시 필요한 업데이트 과정을 중복하지 않고 하나의 함수로 진행 가능
      
      </aside>

- 6.11 단계 쪼개기
    <aside>
    💡 코드에서는 하나에 한 가지만 다룰 것을 명심하기!
    **단계를 명확히 분리하는 것이 핵심이다.**
    
    </aside>
    
    ```tsx
    // 주문한 상품의 총 금액을 구하는 기능
    const orderData = orderString.split(/\s+/);
    const productPrice = priceList[orderData[0].split("-")[1]];
    const orderPrice = parseInt(orderData[1] * productPrice);  //문제점 : 파싱도하고 금액도 계산한다.
    ```
    
    ```tsx
    const order = parseOrder(orderString);  //개선점 : 파싱하는 역할의 기능이 분리되었다.
    const orderPrice = price(order, priceList);  // 개선점 : 총 주문 금액을 계산하는 기능이 분리되었다.
    
    // 기능 1 : string으로 받아온 주문 건 파싱하기
    function parseOrder(orderString){
    	const values = order.split(/\s+/);
    	
    	return({
    		productId: values[0].split("-")[1],
    		quantity: parseInt(values[1]),
    	})
    }
    
    // 기능 2 : 총 주문 금액 계산하기
    function price(order, priceList){
    	return order.quantity * priceList[order.productId];
    }
    ```
    
    - 책 에서는 아래와 같이 6가지 단계가 나오는데…
        1. 기능 2가지 중 두 번째 기능에 대한 함수를 추출한다. 이 때 매개변수는 고려하지 않는다.
        2. 테스트한다.
        3. 매개변수가 많다면 중간 구조를 만든다.
            
            ```tsx
            // 빈 객체 만들기(중간구조 만들기)
            const numbers = {};
            function sum(a, b, c){ ... };
            ```
            
        4. 테스트한다.
        5. 중간구조에 매개변수를 객체 형태로 담아 처리한다.
            
            ```tsx
            // 중간구조에 매개변수 담기
            const numbers = {a, b, c};
            function sum(numbers){ ... };
            ```
            
        6. 첫 번째 기능에 대한 함수도 추출한다.
        
        <aside>
        💡 결국, 기능 별 단계를 명확히하고 매개변수 줄이며 기능을 쪼개라 인 것으로 보입니다.
        여기서 리팩토링 시 중요한 점은, 2번과 4번의 테스트한다 가 중요할 것 같아요.
        중간중간 확인하고 리팩토링 성공 여부에 대한 기준은 `테스트 코드`이다.
        
        </aside>
