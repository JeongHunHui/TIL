# QueryDSL

### 참조

- QueryDSL 쿼리문 작성 방법
    
    [https://ykh6242.tistory.com/m/107](https://ykh6242.tistory.com/m/107)
    
- QueryDSL 세팅 및 기본 문법
    
    [https://velog.io/@shlee327/Querydsl-기본문법-학습하기](https://velog.io/@shlee327/Querydsl-%EA%B8%B0%EB%B3%B8%EB%AC%B8%EB%B2%95-%ED%95%99%EC%8A%B5%ED%95%98%EA%B8%B0)
    
- 스프링 부트에 QueryDSL을 사용해보자
    
    [https://tecoble.techcourse.co.kr/post/2021-08-08-basic-querydsl/](https://tecoble.techcourse.co.kr/post/2021-08-08-basic-querydsl/)
    

Spring Data JPA를 사용하더라도 결국 JPQL을 작성해야 한다. 간단한 로직을 작성하는데 큰 문제는 없으나, 복잡한 로직의 경우 개행이 포함된 쿼리 문자열이 상당히 길어진다. 또한 JPQL로 동적인 쿼리를 처리하거나, JPQL문에 오타가 있을경우 잡아주기 힘들다.

→ QueryDSL이 이런 문제들을 해결하는데 도움을 준다.

### QueryDSL

장점

1. 문자가 아닌 코드로 쿼리를 작성함으로써, 컴파일 시점에 문법 오류를 쉽게 확인 가능하다.
2. 자동 완성 등 IDE의 도움을 받을 수 있다.
3. 동적인 쿼리 작성이 편리하다.
4. 쿼리 작성 시 제약 조건 등을 메서드 추출을 통해 재사용할 수 있다.

단점

1. 다소 번거로운 Gradle 설정
2. 따로 사용법을 익혀야함

### 세팅

build.gradle

```java
// querydsl 추가
buildscript {
    ext {
        queryDslVersion = "5.0.0"
    }
}

plugins {
    id 'org.springframework.boot' version '2.7.4'
    id 'io.spring.dependency-management' version '1.0.14.RELEASE'
    // querydsl 추가
    id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
    id 'java'
}

group = 'com.deca'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-web'

    //querydsl 추가
    implementation "com.querydsl:querydsl-jpa:${queryDslVersion}"
    annotationProcessor "com.querydsl:querydsl-apt:${queryDslVersion}"

    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'

    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    //JUnit4 추가
    testImplementation("org.junit.vintage:junit-vintage-engine") {
        exclude group: "org.hamcrest", module: "hamcrest-core"
    }
}

test {
    useJUnitPlatform()
}

//querydsl 추가(끝까지)
def querydslDir = "$buildDir/generated/querydsl"

querydsl {
    jpa = true
    querydslSourcesDir = querydslDir
}

sourceSets {
    main.java.srcDir querydslDir
}

configurations {
    querydsl.extendsFrom compileClasspath
}

compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
```

gradle 설정 후

Gradle - Tasks - other - compileQuerydsl 실행

![image](https://user-images.githubusercontent.com/108508730/196136084-479e082a-8a9f-482e-acb2-d36b0f4b702c.png)

터미널에 ./gradlew clean compileQuerydsl 입력

![image](https://user-images.githubusercontent.com/108508730/196136146-f2053026-5511-482e-b94b-305a35ef107a.png)

Gradle - Tasks - other - compileJava 실행

![image](https://user-images.githubusercontent.com/108508730/196136258-9b2bda2c-7ab6-45a1-b6e2-3f2eaeef1dbb.png)

→ Q클래스 생성 완료

![image](https://user-images.githubusercontent.com/108508730/196136297-e13d7991-3cad-423c-91e4-2a63a6594568.png)

설정파일 생성 - QueryDslConfig

```java
package com.deca.SpringJpaPractice.config;

import com.querydsl.jpa.impl.JPAQueryFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

@Configuration
public class QueryDslConfig {
  @PersistenceContext private EntityManager em;

  @Bean
  public JPAQueryFactory jpaQueryFactory() {
    return new JPAQueryFactory(em);
  }
}
```

→ JPAQueryFactory를 Bean으로 등록

---

### QueryDSL 문법

**Q-Type 활용**

```java
QMember qMember = new QMember("m"); //별칭 직접 지정
QMember qMember = QMember.member; //기본 인스턴스 사용
```

**검색조건 쿼리**

```java
객체.속성.ne("값") // 속성 != "값"
객체.속성.eq("값") // 속성 != "값"
객체.속성.isNotNull() // 속성이 is not null
객체.속성.in(10, 20) // 속성 in(10, 20)
객체.속성.notIn(10, 20) // 속성 not in(10, 20)
객체.속성.between(10, 30) // between 10, 30
객체.속성.goe(30) // 속성 >= 30
객체.속성.gt(30) // 속성 > 30
객체.속성.loe(30) // 속성 <= 30
객체.속성.lt(30) // 속성 < 30
객체.속성.like("값%") // like 검색
객체.속성.contains("값") // like '%값%' 검색
객체.속성.startsWith("값") // like '값%' 검색
```

- 각 검색조건은 .and(검색조건) .or(검색조건) 으로 연결 가능
    
    예시: 이름이 member거나 나이가 20 이상이거나
    
    ```java
    member.username.eq("member").or(member.age.goe(20))
    ```
    
- 각 검색조건은 파라미터로 and연결 가능
    - ex) where(검색조건1, 검색조건2) = where(검색조건1.and(검색조건2))
- 참고로 검색조건은 `com.querydsl.core.types.dsl.BooleanExpression` 클래스이므로 따로 빼서 메소드를 만들어 줄수도 있다.
    
    ```java
    private BooleanExpression statusEq(OrderStatus statusCond) {
        if (statusCond == null){
          return null;
        }
        return order.status.eq(statusCond);
    }
    // 활용
    // where(statusEq(status1).and(order.id.eq(1)))
    // -> 주문의 상태가 status1이고, 주문의 id가 1인 경우 검색
    ```
    

**쿼리 메소드**

- select()
- from()
- selectFrom()
- where()
- update()
- set()
- delete()

**조인**

- join | innerJoin(member.team, team): member 테이블과 member.team과 연관된 team테이블 내부 join
- .on(조건): join뒤에 붙혀서 join대상 필터링
- leftJoin, rightJoin: left, right 외부 join

**집합 함수**

- sum, avg, min, max, count 사용 가능
- groupBy 사용 가능, having으로 결과 제한 가능

**결과 조회**

- fetch(): 리스트 조회, 데이터가 없으면 빈 리스트 반환
- fetchOne(): 단건 조회, 결과가 없으면 null, 둘 이상이면 에러

**정렬**

- orderBy(테이블1.속성1.정렬방식, 테이블1.속성1.정렬방식, …)
- 정렬방식: asc() - 오름차순, desc() - 내림차순

**서브 쿼리**

- `com.querydsl.jpa.JPAExperssions` 를 사용하여 서브쿼리 작성 가능
    
    ```java
    Member result = queryFactory
                    .selectFrom(member)
                    .where(member.age.eq(
                            JPAExpressions
                                    .select(memberSub.age.max())
                                    .from(memberSub))
                            )
                    .fetchOne();
    ```
    

**예시 - OrderRepository**

```java
import ...

@Repository
@RequiredArgsConstructor
public class OrderRepository {
  private final EntityManager em;

  private final JPAQueryFactory jpaQueryFactory;
	
	// ...
  
	public List<Order> findAll(OrderSearch orderSearch) {

    QOrder order = QOrder.order;
    QMember member = QMember.member;

    // 검색 조건에 따라서 동적인 쿼리가 필요
    // ex) 이름 = ??, 주문상태 = ?? 등으로 필터링 가능
    return jpaQueryFactory
            // order 테이블에서(select(order).from(order)와 동일)
            .selectFrom(order)
            // order와 order와 연관된 member를 join
            .join(order.member, member)
            // 검색 조건의 주문상태가 order의 주문상태와 같은가?
            .where(statusEq(orderSearch.getOrderStatus()),
                    // 검색 조건의 이름이 회원의 이름과 같은가?
                    nameLike(orderSearch.getMemberName()))
            // 검색결과 1000개로 제한
            .limit(1000)
            // 결과 조회
            .fetch();
  }

  private BooleanExpression statusEq(OrderStatus statusCond) {
    if (statusCond == null){
      return null;
    }
    return order.status.eq(statusCond);
  }

  private BooleanExpression nameLike(String nameCond) {
    if (!StringUtils.hasText(nameCond)){
      return null;
    }
    return member.name.like(nameCond);
  }
}
```

→ 이런식으로 동적 쿼리 작성도 가능
