- IO 작업이 많은 어플리케이션은 MVC 스레드 모델로는 기술적 한계가 있음
- 요청당 스레드 모델: 요청을 하나의 스레드에 할당. 해당 스레드는 할당 받은 요청만 처리. IO 작업의 경우에는 스레드는 blocking 됨. 
- 리액티프 앱: async, non-blocking. 스레드는 IO 작업이 발생하면 기다리지 않고 다른 작업 수행. 동일한 양의 계산 리소스로 더 많은 사용자엑 서비스 제공.


## 리액터 & 스프링의 비동기, 비차단 아키텍처

- reactive manifesto는 reactive 시스템을 "반응성, 복원성, 탄력성이 높고 메시지가 주도하는 시스템"이라고 함.

- thread-per-request:
	- 하나의 스레드가 하나의 요청을 담당.
	- IO 인터럽트 발생 시 스레드는 차단된 상태로 대기. 
	- 스레드에 할당된 자원은 차단된 동안 활용하지 못함. 
- event-loop:
	- 스레드를 특정 요청에 독점적으로 할당 X
	- 콜백을 등록해 놓고 데이터가 준비될 때 마다 알림이 전송되고 스레드 중 하나가 콜백을 수행.
	- 어플리케이션의 확장성에 제약 X: 동시에 처리할 수 있는 요청의 수를 늘리기 위해서 스레드 수를 늘리는 것이 아니기 때문.
- 이벤트 루프의 내부 메커니즘을 알 필요 없음?

- 확장성과 비용 최적화를 위해서 리액티프 패러다임을 선택하는 것은 좋은 선택. 

- 리액티브 어플리케이션의 필수 기능: 제어흐름. non-blocking backpressure
	- 데이터를 처리하는 쪽에서 수신 데이터 양을 제어해 자신이 처리할 수 있는 것보다 많은 데이터가 들어오는 것에 의해 발생할 수 있는 위험을 경감
	- thread-per-request 모델에서는 동시 처리 능력을 높이기 위해서 더 많은 스레드가 필요 -> 시스템이 응답하지 않거나 느려지는 등의 문제 발생

- 리액티프 어플리케이션의 단점: 패러다임의 복잡성
	- 이벤트 중심 방식으로 사고 전환이 필요
	- 비동기 IO로 인한 디버깅이 어려움.

### 리엑터

- 프로젝트 리엑터: JVM에서 비동기식 비차단 어플리케이션 구축을 위함 프레임워크. reactive streams의 구현체.
- reactive streams는 개념적으로 java stream api와 유사하지만 전자는 push 방식으로 생성자로부터 통보 받는 비동기적 방식이고 후자는 pull 방식으로 동기적으로 데이터를 처리하는 동기적 방식
- publisher: 데이터 생성 담당. Mono(하나 이하), Flux(다수 or 0개의 데이터 생산). 
- subscriber: subscription을 하면서 backpressure를 정의한다. 자기 자신이 한 번에 처리할 수 있는 데이터양을 알려줌.
	- 처리 속도를 생산자가 아닌 소비자가 정의하는 방식은 소비자가 너무 많은 데이터를 전달 받아서 데이터를 처리하지 못하는 상황을 방지
- java stream api는 fluent api를 사용해서 `map, flatMap, filter`와 같은 연산자로 데이터 처리. 이 때 불변의 상태를 유지하면서 새로운 Stream 객체 생성
	- 리액티브 스트림에서도 비슷하게 동작
- 리액어테서는 retryWhen, timeout등의 연산자를 사용해 배압을 적용하고 오류를 처리.


### 스프링 리액티브 스택

- 서블릿 스택: 서블릿 컨테이너, 서블릿 API, 서블릿 MVC
- 리액티브 스택: Netty, 리액터, 스프링 웹플럭스.

### R2DBC

- JDBC: 리액티브 X. 
- R2DBC: RDBMS를 리액티브 방식으로 사용하게 하는 드라이버. 

- 이후로는 코드 설명

## spring web client

- RestTemplate: API기반으로 차단 방식의 HTTP client
	- 몇 버전부터는 Webclient방식으로 동작
- Webclient: 차단 및 비차단 IO를 제공


## reactive spring을 통한 복원력 높은 어플리케이션

- 복원력: 장애가 발생하더라도 시스템을 계속 사용할 수 있게 유지하면서 서비스를 제공할 수 있는 속성. 
- 실패는 언제라도 일어날 수 있고 모든 실패를 막을 방법은 없다. 그렇기 때문에 높은 내결함성을 갖는 어플리케시연 설계가 중요. 

- 복원력을 달성하는 데에 중요한 점은 문제가 해결될 때까지 구성 요소를 격리하는 것. crack propagation을 막을 수 있다. 

- Resilience4J, Spring cloud Circuit Breaker: 복원력 높은 어플리케이션을 구축하기 위해서 사용
- 리액티브 스택의 토대인 리엑터도 복원력을 높이기 위한 기능 제공

- 타임 아웃
	- 적절한 시간 내에 응답이 오지 않아도 어플리케이션의 응답성을 유지하기 위한 도구
	- 클라이언트가 기다리는 시간을 제한해서 너무 많은 자원이 차단되는 것을 방지
	- 연결 타임아웃, 연결 풀 타임아웃, 읽기 타임아웃
	- 타임아웃 시간을 적절히 설정하는 것은 어려움. 읽기 작업은 멱등성이 보장되기 때문에 작업을 여러번 수행해도 문제 없지만 쓰기 작업의 경우에는 그렇지 않다. 타임아웃을 적절히 설정하고 시간이 초과하면 사용자에게 작업 결과를 올바르게 제공하는 것을 포함해야 한다
- 재시도
	- 특정 시간 내에 응답이 없거나 순간 요청을 처리하지 못하고 서버 오류가 발생한 경우에 클라이언트가 다시 요청을 보내는 기능
	- But 계속해서 재시도 공격은 시스템을 불안정하게 만든다
- 지수 백오프
	- 재시도 횟수를 시간이 지남에 따라 간격을 증가시키는 것.
	- 재시도가 늘어날 수록 점점 더 많은 시간을 기다리게 해서 서비스가 회복되고 응답하기 위한 시간을 충분히 준다
- 멱등적이지 않은 동작을 재시도하는 것은 지양해야함 
- 전체 워크플로우에서 사용자가 참여하고 있다면 복원력과 사용자 경험사이에서 균형을 잡아야 한다
	- 요청 재시도로 인해 사용자를 너무 기다리게 해서는 안된다. 
	- 재시도가 필요하다고 생각되면 사용자에게 알리는 방식을 사용해야 함

- 폴백
	- 오류나 예외가 발생하면 폴백 함수를 수행할 수 있지만 오류나 예외가 모두 동일한 것은 아니다
	- 어떤 오류는 비즈니스 로직 관점에서 의미가 있다. e.g. 특정 책을 주문 불가능할 때
	- 프로젝트 리액터는 폴백을 정의하기 위한 onErrorResume을 제공
	- 핵심 목표는 실패가 발생허더라도 사용자가 이를 알아차리지 못하고 서비스를 계속 사용할 수 있게 해서 복원력 높은 시스템을 설계하는 것
