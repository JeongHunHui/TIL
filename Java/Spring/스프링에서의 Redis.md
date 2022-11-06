스프링에서 Redis로 세션 클러스터링 하는법

- 참조
    
    [Springboot + Redis 연동하는 예제 (2) 세션 클러스터링](https://oingdaddy.tistory.com/311)
    
- 세션 클러스터링이란?
    
    [세션 클러스터링이란?](https://kku-jun.tistory.com/44)
    

@EnableRedisHttpSession

[https://github.com/f-lab-edu/daangn-market-used-trading/pull/6/commits/d004614f67673edc8a17cffe4d0c0b1a0f5ac691](https://github.com/f-lab-edu/daangn-market-used-trading/pull/6/commits/d004614f67673edc8a17cffe4d0c0b1a0f5ac691)

- springSessionRepositoryFilter라는 이름을 가진 Filter를 구현한 스프링 빈을 생성
- springSessionRepositoryFilter는 HttpSession을 SpringSession이 지원하는 커스텀 구현으로 대체하는 역할을 수행하게 되는데 이 경우 SpringSession을 Redis에서 지원하게 됨

Redis client

- jedis, lettuce등이 있고, 스프링에서는 lettuce를 사용한다.
    - lettuce 사용 이유
        
        [Jedis 보다 Lettuce 를 쓰자](https://jojoldu.tistory.com/418)
        
- jedis는 thread-safe 하지 않기 때문에 멀티 스레드 환경에서 Jedis를 사용하기 위해서 Connection Pool을 이용하는 방식을 사용하고 있다.
- 그로인해 동시에 처리할 수 있는 요청의 개수가 Connection Pool의 개수이며, Connection Pool의 개수만큼 생성되어야 하는 Jedis 인스턴스 당 물리적인 연결 비용이 발생한다.
- 반면 Netty 기반의 Lettuce 는 thread-safe 하기 때문에 여러 스레드에서 공유해서 사용할 수 있고, 이로인해 Lettuce와 상호 작용하는 스레드의 개수와 상관없이 Redis와 단일 연결을 사용할 수 있다.
- 또한, 비동기로 요청을 처리하기 때문에 Jedis에 비해서 성능상의 이점도 가지고 있다.
- 때문에 Spring Boot 2.0 부터는 Jedis대신에 Lettuce를 Redis의 기본 클라이언트로써 사용하고 있다.
- 참고
    - 스레드 안전(thread safe)은 여러 스레드로부터 동시에 접근이 이루어져도 프로그램의 실행에 문제가 없음을 뜻함.
    - 커넥션 풀(Connection Pool)
        
        [[Spring] 커넥션 풀(Connection pool)이란?](https://linked2ev.github.io/spring/2019/08/14/Spring-3-%EC%BB%A4%EB%84%A5%EC%85%98-%ED%92%80%EC%9D%B4%EB%9E%80/)
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fb4fc79e-40a4-4765-95f0-f3bd09cabc74/Untitled.png)
        
        - 웹 컨테이너(WAS)가 실행되며 DB와 미리 connection 해놓은 객체들을 pool에 저장해뒀다가 요청이 오면 connection을 빌려주고 처리가 끝나면 반납받아 다시 pool에 저장하는 방식

`RedisTemplate`

`RedisConnectionFactory`

`LettuceConnectionFactory`

`RedisStandaloneConfiguration`

`StringRedisSerializer`

`GenericJackson2JsonRedisSerializer`
