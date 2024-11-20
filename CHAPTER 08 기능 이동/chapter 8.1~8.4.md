- 8.1 함수 옮기기
    <aside>
    💡
    
    기능 모듈화하기 = 연관된 요소 묶기 → 일부만 이해하여도 수정 가능해야한다.
    
    </aside>
    
    - 절차
        1. 타깃 함수를 원하는 위치에 복사하여 신규 함수를 생성한다.
            
            ```tsx
            // AS-IS
            class Account {
            	get 은행이자(){ 
            			// 은행 이자 계산
            			...
            			return baseCharge + 계좌타입별초과인출이자();
            	};
            	get 계좌타입별초과인출이자() { ... 계좌 타입별로 초과 인출 이자 계산 ...}; => AccountType 클래스로 이동
            }
            
            // TO-BE
            class Account {
            	get 은행이자(){
            			// 은행 이자 계산
            			...
            			return baseCharge + 계좌타입별초과인출이자();
            	};
            	get 계좌타입별초과인출이자() { ... 계좌 타입별로 초과 인출 이자 계산 ...}; => AccountType 클래스로 이동
            }
            
            class AccountType {
            	get 초과인출이자() { ... 초과 인출 이자 계산 ... };
            }
            ```
            
        2. 새 보금자리에 맞게 신규 함수를 수정한다.
        3. 타깃 함수를 수정하여 새 메서드를 호출하도록 한다.
            
            ```tsx
            // TO-BE
            class Account {
            	constructor(){
            		this.type = new AccountType();
            	}
            
            	get 은행이자(){
            			// 은행 이자 계산
            			...
            			return baseCharge + 계좌타입별초과인출이자();
            	};
            	get 계좌타입별초과인출이자() { 
            			return this.type.초과인출이자;    // AccountType 메소드 호출
            	};
            }
            
            class AccountType {
            	get 초과인출이자() { ... 초과 인출 이자 계산 ... };
            }
            ```
            
        4. 인라인 여부를 결정하여 반영한다.
            
            ```tsx
            // 만약 인라인한다면,
            class Account {
            	constructor(){
            		this.type = new AccountType();
            	}
            
            	get 은행이자(){
            			// 은행 이자 계산
            			...
            			return baseCharge + this.type.초과인출이자();    // AccountType 메소드 호출
            	};
            }
            
            class AccountType {
            	get 초과인출이자() { ... 초과 인출 이자 계산 ... };
            }
            ```

- 8.2 필드 옮기기
    <aside>
    💡
    
    데이터 구조가 적절치 않다면 곧바로 수정해야한다.
    
    </aside>
    
    - 절차
        1. 옮기려고 하는 클래스에 필드와 접근자 메서드(getter, setter)를 생성한다.
        2. 기존 클래스에서 새롭게 생성된 필드를 참조할 수 있도록 변경한다.
        3. 테스트한다.
        4. 기존 클래스의 필드를 삭제한다.
    - example
        
        ```tsx
        // AS-IS
        class Customer {
        	get plan() {return this.plan; }
        	get discountRate() { return this.discountRate; }
        }
        
        // TO-BE
        class Customer {
        	get plan() {return this.plan; }
        	get discountRate() { return this.plan.discountRate; }  // 데이터 구조가 변경됨
        }
        ```

- 8.3 문장을 함수로 옮기기
    <aside>
    💡
    
    중복을 제거하여 깨끗한 코드를 유지해야한다. → 사실 꼭 중복만을 위한 것은 아니고 모듈화에 가까운 것 같음
    
    </aside>
    
    - 절차
        1. 새로운 함수를 만들어 동일한 기능을 추가한다.(단, 함수명은 당장 신경쓰지 않고 기억하기 쉬운 함수명을 선택한다.)
        2. 클라이언트에서 새로운 함수를 호출할 수 있도록 한다.
        3. 테스트한다.
        4. 새로운 함수명을 명확하게 변경한다.
    - example
        
        ```tsx
        // AS-IS
        result.push(`제목 : ${title}`);
        result.concat(photoData(person.photo));
        
        function photoData(photo){
        	return [
        		`위치 : ${location}`,
        		`날짜 : ${new Date()}`
        	];
        }
        
        // TO-BE
        result.concat(photoData(person.photo));
        
        function photoData(photo){
        	return [
        		`제목 : ${title}`,
        		`위치 : ${location}`,
        		`날짜 : ${new Date()}`
        	];
        }
        ```

- 8.4 문장을 호출한 곳으로 옮기기 ↔ 8.3 문장을 함수로 옮기기와 반대
    <aside>
    💡
    
    기존에는 한 가지 일만을 수행하는 함수였지만 기능의 분리가 필요하여 두 가지 일을 하는 함수가 되어버렸을 경우 사용
    → 일부 호출자에서는 다르게 동작하는 경우
    
    </aside>
    
    1. 남길 코드를 새로운 함수로 추출한다.(단, 함수명은 당장 신경쓰지 않고 기억하기 쉬운 함수명을 선택한다.)
        
        ```tsx
        result.concat(photoData(person.photo));
        
        function photoData(photo){
        	return [
        		`제목 : ${title}`,
        		`위치 : ${location}`,
        		`날짜 : ${new Date()}`
        	];
        }
        
        function aaa(photo){       // 함수 추출
        	return [
        		`위치 : ${location}`,
        		`날짜 : ${new Date()}`
        	];
        }
        ```
        
    2. 기존 함수에서 새로운 함수를 호출한다.
        
        ```tsx
        function photoData(photo){
        	const result: string[] = []
        	result.push(`제목 : ${title}`);
        
        	return result.push(...aaa(photo));
        }
        ```
        
    3. 변경이 필요한 클라이언트에서 새로운 함수와 분리된 기능을 추가한다.
        
        ```tsx
        result.push(`제목 : ${title}`);
        result.concat(aaa(person.photo));
        ```
        
    4. 테스트한다.
    5. 기존 함수를 지우고 신규 함수명을 명확하게 변경한다.
        
        ```tsx
        result.push(`제목 : ${title}`);
        result.concat(photoInfo(person.photo));
        
        // photoData 삭제
        ~~function photoData(photo){
        	const result: string[] = []
        	result.push(`제목 : ${title}`);
        
        	return result.push(...aaa(photo));
        }~~
        
        function photoInfo(photo){
        	return [
        		`위치 : ${location}`,
        		`날짜 : ${new Date()}`
        	];
        }
        ```
