## 식별자
식별자(Identifier): 하나의 엔티티타입에서 각각의 엔티티를 구분할 수 있는 결정자 
- 모든 엔티티타입에는 반드시 하나 이상의 식별자가 있어야 한다. 
- 데이터베이스로 구현했을 때 데이터 처리 기준이 되는 PK, FK 역할을 수행한다. 

### 특징 
- 식별자에 의해 엔티티타입 내 모든 엔티티들이 유일하게 구분되어야 한다. 
- 특정 엔티티타입에 식별자가 지정되면 그 식별자는 변하지 않아야 한다. 
- 주식별자의 경우 식별자가 지정되면 주식별자 속성에 반드시 데이터값이 있어야 한다. 

### 구분

![identifier classification](identifier-classification.png)

**주식별자/보조 식별자**
- 주식별자: 엔티티타입의 대표성을 나타내는 유일한 식별자
    - 엔티티타입 하나에 하나
    - 물리 테이블에서는 PK 역할
- 보조 식별자: 주식별자를 대신해 보조적으로 엔티티를 식별할 수 있게 함
    - 엔티티타입 하나에 두 개 이상일 수 있음
    - PK로 생성되지 않고 유니크 인덱스로 지정되어 사용됨

엔티티를 유일하게 식별할 수 있는 속성이 두 개 이상일 때, 해당 업무에 적합한 속성을 주식별자로 두고 다른 속성은 보조 식별자로 활용한다. 

**내부 식별자/외부 식별자**
- 내부 식별자: 자신의 엔티티타입 내에서 스스로 생성되어 존재하는 식별자
- 외부 식별자: 다른 엔티티타입으로부터 관계에 의해 주식별자 속성을 상속받아 자신의 속성에 포함되는 식별자
    - 주식별자 영역에 포함 가능
    - 자신의 엔티티타입으로부터 다른 엔티티타입을 찾아가는 FK 역할 

**단일 식별자/복합 식별자**
- 단일 식별자: 주식별자의 구성이 한 가지 속성으로만 이루어짐
- 복합 식별자: 주식별자의 구성이 두 개 이상의 속성으로 구성됨

**원조 식별자/대리 식별자**
- 원조 식별자
- 대리 식별자: 주식별자가 복합 식별자인 경우 여러 개의 속성을 묶어 하나의 속성으로 만들어 주식별자로 활용하는 경우
    - 원래의 주식별자를 일반 속성으로 내리고 일련번호 형태의 주식별자를 사용하는 경우도 있음
    - ex) 사업소코드+주문일자+일련번호 -> 주문번호 