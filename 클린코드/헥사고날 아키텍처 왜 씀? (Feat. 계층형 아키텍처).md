## 0️⃣ 배경

기존에 `계층형 아키텍처`로 진행중이던 프로젝트가 있었다. 프로젝트 POC 단계 까진 문제가 없었다. 하지만 유지보수를 위한 `테스트 코드`, `외부 기술들의 변화`, `복잡한 비즈니스 로직` 등 다양한 문제를 직면하게 되었다.

이러한 문제들을 해결하기 위한 방법 중 하나로 `헥사고날 아키텍처`를 알게 되었고 공부하게 되었다.

책 `만들면서 배우는 클린 아키텍처`을 읽으며 공부하고 프로젝트에 적용하면서 느낀점을 위주로 헥사고날 아키텍처에 대해 정리했다.

## 1️⃣ 헥사고날 아키텍처란?

소프트웨어 설계에 사용되는 아키텍처 패턴 중 하나이며, 사전적 의미로는 육각형 건축물을 말한다. 사실, 이름만 들어서는 전혀 어떤 아키텍처인지 감이 안온다.

![image](https://github.com/JeongHunHui/TIL/assets/108508730/0161400b-c8f6-44a1-9b61-538ae218de93)

위 그림을 보면 육각형이 있고, 육각형 경계를 기준으로 영역을 내부와 외부로 나눌 수 있다. 헥사고날 아키텍처는 **내부의 도메인 비즈니스로직이 외부요소에 의존하지 않도록 설계된 아키텍처**이다.

영역의 외부에는 어댑터가, 내부는 유스케이스와 엔티티, 그리고 그 경계는 포트로 이루어져있다. 각 요소에 대해 설명하며 어떻게 헥사고날 아키텍처가 내부와 외부의 결합도를 낮추고, 그렇게 하면 뭐가 좋은지 알아보겠다.

## 2️⃣ 포트와 어댑터 아키텍처

헥사고날 아키텍처는 다른 말로 포트와 어댑터 아키텍처라고도 한다. 그만큼 헥사고날 아키텍처를 이해하기 위해선 포트와 어댑터가 무엇이고, 이들의 역할에 대해 이해할 필요가 있다.

### 포트와 어댑터는 뭘까?

- **포트**
    - 포트는 외부 영역과 내부 영역 사이의 연결을 추상화한 인터페이스이다. 포트는 내부 영역 사용을 위해 노출된 인바운드 포트와, 내부 영역에서 외부 영역을 사용하기 위한 아웃 바운드 포트로 구분할 수 있다.
- **어댑터**
    - 어댑터는 외부 시스템과 상호작용 하는 역할을 한다. 어댑터도 마찬가지로 인바운드와 아웃바운드로 나뉜다. 예를 들면, HTTP 요청을 받는 인바운드 웹 어댑터, DB에 접근하여 데이터를 읽고 쓸 수 있는 아웃바운드 영속성 어댑터 등이 있다.

### 흐름 예시

이렇게 설명만 봐서는 감이 안오니까, 송금 기능을 이용하는 과정을 예시로 들며 흐름을 설명하겠다.

![image](https://github.com/JeongHunHui/TIL/assets/108508730/3676eb33-c41a-4906-8c86-62d57b055e01)

위 그림은 송금 기능을 이용하는 과정을 그림으로 그린 것이다. 예시 코드는 [여기](https://github.com/wikibook/clean-architecture)를 참고하면 된다.

1. 사용자의 HTTP 요청을 인바운드 어댑터인 `AccountController`가 받고, 인바운드 포트인 `SendMoneyUseCase`를 사용하여 내부 로직에 접근한다.
    
    ```java
    @WebAdapter
    @RestController
    @RequiredArgsConstructor
    class AccountController {
    	private final SendMoneyUseCase sendMoneyUseCase;
    
    	@PostMapping(path = "/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}")
    	void sendMoney(
    			@PathVariable("sourceAccountId") Long sourceAccountId,
    			@PathVariable("targetAccountId") Long targetAccountId,
    			@PathVariable("amount") Long amount
    	) {
    		sendMoneyUseCase.sendMoney(sourceAccountId, targetAccountId, amount);
    	}
    }
    ```
    
    ```java
    public interface SendMoneyUseCase {
    	boolean sendMoney(Long sourceAccountId, Long targetAccountId, Long amount);
    }
    ```
    
2. 인바운드 포트 `SendMoneyUseCase`를 구현한 `SendMoneyService`는 비즈니스 로직을 처리하고, 아웃바운드 포트인 `LoadAccountPort`를 통해 결과를 외부로 전달한다.
    
    ```java
    @RequiredArgsConstructor
    @UseCase
    @Transactional
    public class SendMoneyService implements SendMoneyUseCase {
    	private final LoadAccountPort loadAccountPort;
    
    	@Override
    	public boolean sendMoney(Long sourceId, Long targetId, Long amount) {
    		LocalDateTime baseDate = LocalDateTime.now().minusDays(10);
    		Account sourceAccount = loadAccountPort.loadAccount(sourceId, baseDate);
    		Account targetAccount = loadAccountPort.loadAccount(targetId, baseDate);
    		return true;
    	}
    }
    ```
    
    ```java
    public interface LoadAccountPort {
    	Account loadAccount(Long accountId, LocalDateTime baseDate);
    }
    ```
    
3. `LoadAccountPort`를 구현한 아웃바운드 어댑터인 `AccountPersistenceAdapter`는 받은 결과를 DB에 저장한다.
    
    ```java
    @RequiredArgsConstructor
    @PersistenceAdapter
    class AccountPersistenceAdapter implements LoadAccountPort {
    	private final SpringDataAccountRepository accountRepository;
    	private final ActivityRepository activityRepository;
    	private final AccountMapper accountMapper;
    
    	@Override
    	public Account loadAccount(Long accountId, LocalDateTime baselineDate) {
    		AccountJpaEntity account = accountRepository.findById(accountId)
    		
    		List<ActivityJpaEntity> activities =
    				activityRepository.findByOwnerSince(accountId, baseDate);
    
    		Long withdrawalBalance = orZero(
    			activityRepository.getWithdrawalBalanceUntil(accountId, baseDate)
    		);
    
    		Long depositBalance = orZero(
    			activityRepository.getDepositBalanceUntil(accountId, baseDate)
    		);
    
    		return accountMapper.mapToDomainEntity(
    			account, activities, withdrawalBalance, depositBalance
    		);
    	}
    
    	private Long orZero(Long value){
    		return value == null ? 0L : value;
    	}
    }
    ```
    

## 3️⃣ 계층형 아키텍처 VS 헥사고날 아키텍처

이제 헥사고날 아키텍처가 무엇인지 알았을 것이다. 그렇다면, 기존에 많이 사용하던 계층형 아키텍처와 비교해서 어떤 점이 좋은걸까?

### 계층형 아키텍처의 문제점

![image](https://github.com/JeongHunHui/TIL/assets/108508730/0c62e51b-348b-4e89-9eee-46d325c0c250)

우선 계층형 아키텍처는 웹, 도메인, 영속성 계층으로 이루어진 아키텍처이다. 각 계층은 하위 계층에 의존한다. 이러한 계층형 아키텍처는 아래와 같은 문제점들이 있다.

1. **데이터베이스 주도 설계를 유도한다.**
    - 웹 계층은 도메인 계층에, 도메인 계층은 영속성 계층에 의존하기 때문에 자연스럽게 모든 것이 데이터베이스에 의존하기 쉬워진다. 즉, 데이터베이스의 구조를 먼저 생각하고, 이를 토대로 도메인 로직을 구현하는 것이다.
    - 결국 아래와 같이 도메인 계층과 영속성 계층간에 강한 결합이 생기고, 이는 순수한 도메인 로직 구현을 방해한다.
        
        ![image](https://github.com/JeongHunHui/TIL/assets/108508730/716208c9-b108-48e3-83e9-211194d60071)
        
2. **외부 시스템의 변화에 취약하다.**
    - 해당 내용은 1번 내용과도 연결되는 내용이고, 직접 경험해본 내용이기도 하다. 퀴즈 서비스를 개발하는 중에 요구사항의 변경으로 기존에 `MySQL`을 사용하여 퀴즈 정보를 저장하던 것을 `Redis`를 사용하도록 리팩토링을 하게 되었다. 하지만 1번과 같이 도메인 로직이 영속성 관점과 섞여있는 상태에서 이를 리팩토링하는 것은 상당히 힘들었다.
3. **테스트가 어렵다.**
    - 서비스의 기능이 확장됨에 따라 서비스는 영속성 계층에 많은 의존성을 갖게 되고, 웹 계층도 이러한 서비스들에 의존하게된다. 그렇기 때문에 웹 계층을 테스트하기 위해선 도메인 계층은 물론 영속성 계층도 Mocking해야 하므로 테스트가 어려워진다.
4. **동시 작업이 어렵다.**
    - 만약 어플리케이션에 새로운 유스케이스를 추가한다고 생각해보자. 개발자가 총 3명이라고 했을 때 한 명은 웹 계층, 한 명은 도메인 계층, 한 명은 영속성 계층을 담당해서 하면 될까? 계층형 아키텍처에선 이는 불가능하다. 모든 계층이 영속성 계층에 의존하기 때문에 영속성 계층을 먼저 개발해야하고, 그 다음에는 도메인, 마지막으로 웹 계층을 만들어야한다.

### 헥사고날 아키텍처와 비교 - 장점

그렇다면, 헥사고날 아키텍처는 계층형 아키텍처와 비교했을 때 어떤 장점이 있을까?

1. **도메인 중심 설계가 쉬워진다.**
    - 계층형 아키텍처에서는 상위 계층이 하위 계층에 강한 결합이 있었다. 특히 영속성 계층과의 강한 결합으로 인해 다양한 문제가 발생했다.
    - 하지만 헥사고날 아키텍처는 내부 영역이 외부 영역에 의해 영향을 받지 않는다.
        
        ```java
        @RequiredArgsConstructor
        @UseCase
        @Transactional
        public class SendMoneyService implements SendMoneyUseCase {
        	private final LoadAccountPort loadAccountPort;
        
        	@Override
        	public boolean sendMoney(Long sourceId, Long targetId, Long amount) {
        		LocalDateTime baseDate = LocalDateTime.now().minusDays(10);
        		Account sourceAccount = loadAccountPort.loadAccount(sourceId, baseDate);
        		Account targetAccount = loadAccountPort.loadAccount(targetId, baseDate);
        		return true;
        	}
        }
        ```
        
        위와 같은 `SendMoneyService` 입장에서는 외부에서 데이터를 어떻게 가져오는지 알 필요가 없다. 이로 인해 외부 시스템에 구애받지 않고 도메인 중심 설계를 할 수 있다.
        
    - 도메인 중심 설계를 하게되면, 도메인 객체를 중심으로 비즈니스 로직이 짜여지므로 코드의 재사용성이 증가하고, 더 자연스러운 코드를 짤 수 있게된다.
2. **외부 요소의 변경이 쉬워진다.**
    - 계층형 아키텍처에서는 도메인 계층과 영속성 계층간의 강한 결합으로 인해 외부 요소의 변경이 일어나면 대부분의 코드를 다시 작성해야했다.
    - 하지만 헥사고날 아키텍처에서는 요구사항의 변경으로 외부 요소를 변경(ex: MySQL를 Redis로 변경)하는 경우 외부 영역의 어댑터만 바꿔주면 되기 때문에 외부 요소의 변경이 쉽다.
3. **테스트가 쉬워진다.**
    - 헥사고날 아키텍처에서는 외부 요소와는 포트를 통해 연결되기 때문에, 비즈니스 로직 테스트 시 포트의 Mock 객체를 통해 쉽게 테스트를 진행할 수 있다.
4. **동시 작업이 쉽다.**
    - 계층형 아키텍처에서는 하나의 유스케이스를 여러명의 개발자가 동시에 개발하는 것이 어려웠다.
    - 하지만 헥사고날 아키텍처에서는 어댑터 부분, 도메인 부분으로 나눠서 여러 개발자가 동시에 작업이 가능하다.

### 헥사고날 아키텍처와 비교 - 단점

그렇다면 헥사고날 아키텍처는 `은탄환`일까? 절대 아니다. 헥사고날 아키텍처는 아래와 같은 단점들이 있다.

1. **코드가 많아진다.**
    - 내부 영역이 외부 영역과 완전히 분리되어야 하기 때문에 각 영역별로 추가적인 코드 작성이 필요해서 코드가 많아진다.
    - 예시로 계층형은 도메인 객체와 ORM에 사용되는 엔티티의 구분이 없지만, 헥사고날은 ORM용 엔티티, 도메인 객체, 상호 변환을 위한 Mapper까지 필요하다.
2. **진입 장벽이 높다.**
    - 아키텍처를 도입하기 위해서 포트, 어댑터 등 알아야할 개념이 추가로 생기고, 초반에 프로젝트를 구성하는 데에 시간이 오래 걸린다.

## 4️⃣ 결론

항상 완벽한 아키텍처는 없다. 사실 처음 헥사고날 아키텍처를 들었을 때는 그냥 실속없이 이름만 멋진 개념인 줄 알았다. 하지만, 계층형 아키텍처로 프로젝트를 진행하면서 마주한 문제점들을 해결하기 위해 찾다가 헥사고날 아키텍처를 알게 되었다. 처음에는 반신반의 했지만, 책을 읽어보고 직접 프로젝트에 적용해보며 현재 상황에 잘 맞는 아키텍처라는 생각이 들게되었고, 결국 헥사고날 아키텍처로 리팩토링하게 되었다.

무조건 유행한다고 해보기 보다는 현재 진행하는 프로젝트의 성격과 잘 비교해서 적용하면 좋을 것 같다.
