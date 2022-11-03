### 참조

- JUnit5 완벽가이드
    
    [https://donghyeon.dev/junit/2021/04/11/JUnit5-완벽-가이드/](https://donghyeon.dev/junit/2021/04/11/JUnit5-%EC%99%84%EB%B2%BD-%EA%B0%80%EC%9D%B4%EB%93%9C/)
    
- JUnit5, Mockito
    
    [https://blog.naver.com/jae_choel/222233495278](https://blog.naver.com/jae_choel/222233495278)
    

### given-when-then-Style

- **Given**
    
    테스트에서 구체화하고자 하는 행동을 시작하기 전에 테스트 상태를 설명하는 부분ㅍ
    
- **When**
    
    구체화하고자 하는 그 행동
    
- **Then**
    
    어떤 특정한 행동 때문에 발생할거라고 예상되는 변화에 대한 설명
    

**예시**

```java
@Test
@DisplayName("해당 이메일로 가입된 회원이 존재하는 경우 정상적으로 사용자를 조회한다.")
void isExistMemberFindByEmail() {
    // given: Repository에서 이메일로 회원을 찾은 상태이고
    when(memberRepository.findMemberByEmail(any())).thenReturn(Optional.of(member));

    // when: Service에서 이메일로 회원으려고 할 때
    Member findByEmailMember = memberService.findMemberByEmail(member.getEmail());

    // then: 회원이 있어야 하고, Repository의 member와 ID와 Email이 같아야 한다.
    assertThat(findByEmailMember).isNotNull();
    assertThat(findByEmailMember.getId()).isEqualTo(member.getId());
    assertThat(findByEmailMember.getEmail()).isEqualTo(member.getEmail());
}
```

### JUnit5

JUnit이란, 자바용 유닛 테스트 프레임워크이다.

**JUnit5의 어노테이션**

- `@Test`
    
    테스트 메소드를 표시
    
- `@DisplayName(”테스트 설명”)`
    
    테스트 클래스나 메소드에 이름을 붙여줄 때 사용한다.
    
- `@BeforeEach`, `@AfterEach`
    
    각 테스트 메소드가 실행되기 전, 실행된 뒤에 실행되야 하는 메소드를 표시
    
    → `@BeforeEach`는 주로 테스트 하기전에 필요한 데이터를 미리 세팅해주기 위해 사용
    
- `@BeforeAll`, `@AfterAll`
    
    테스트가 처음 시작할 때, 테스트가 모두 끝났을 때 딱 한번만 실행되야하는 메소드를 표시
    
- `@Disabled`
    
    테스트를 비활성화 한다.
    
- `@ExtendWith`
    
    테스트 메소드에 특정 클래스를 확장하는 어노테이션
    
    → @ExtendWith(MokitoExtention.class): Mokito의 Mock객체를 사용하게 해줌
    

**Assertions**

- `assertEquals(A, B, “message(선택)”)`
    
    A와 B값을 비교하여 일치여부 판단. 실패 시 반환할 메세지 설정 가능
    
- `assertNull`, `assertNotNull`
    
    값의 null, notNull 여부 확인
    
- `assertTrue`, `assertFalse`
    
    값이 True인지 False인지 확인
    
- `assertNotEquals`
    
    두 값이 같지 않은지 확인, 같으면 실패
    
- `assertThrows(예외 클래스, 테스트 코드)`
    
    테스트 코드로 예외가 발생하였는지 확인
    
- `fail`
    
    실패 메세지와 함께 테스트 실패
    

### Mock

Mock Object: 흉내내는 오브젝트

![image](https://user-images.githubusercontent.com/108508730/199686397-aaa4469f-c86b-4387-8355-f7b4418fdf9a.png)

Service를 테스트 하기 위해서는 Repository가 필요하다. 하지만 서비스만 테스트 하기 위해서 Repository까지 불러오는 것은 비효율적이다.

→ Mock Repository를 만들어서 사용하면 실제 Repository를 불러올 필요가 없다

→ Mockito 활용

**Mokito 주요 어노테이션**

@Mock

- 스프링 컨테이너가 아닌 Mockito 환경에 가짜로 객체를 띄움

@InjectMocks

- 해당 객체가 만들어질 때 해당 테스트 코드에서 `@Mock`으로 등록된 객체를 주입받음

**예시**

```java
// memberService가 만들어질 때 해당 테스트 코드의 @Mock으로 등록된 객체를 주입받음
@InjectMocks
private GeneralMemberService memberService;

// 스프링 컨테이너가 아닌 Mockito 환경에 가짜로 객체를 띄움
@Mock
private MemberRepository memberRepository;
...
@Test
@DisplayName("해당 이메일로 가입된 회원이 존재하는 경우 정상적으로 사용자를 조회한다.")
void isExistMemberFindByEmail() {
    // given
		// 가짜 memberRepository에서 findMemberByEmail메소드를 실행하면
		// 미리 정해둔 member가 return된다고 가정한다.
    when(memberRepository.findMemberByEmail(any())).thenReturn(Optional.of(member));

    // when
		// memberService의 findMemberByEmail로 이메일을 통해 member 찾음
		// memberService의 findMemberByEmail은 memberRepository의
		// findMemberByEmail를 호출 -> given에서 member를 반환한다고 가정했음
    Member findByEmailMember = memberService.findMemberByEmail(member.getEmail());

    // then
		// 찾은 결과값이 비어있지 않고, id와 email이 원래 member와 같은지 확인
    assertThat(findByEmailMember).isNotNull();
    assertThat(findByEmailMember.getId()).isEqualTo(member.getId());
    assertThat(findByEmailMember.getEmail()).isEqualTo(member.getEmail());
}
```

1. 객체 생성, 해당 객체에 필요한 객체를 Mock 객체로 만들어 주입
2. 가짜 객체의 행동과 결과를 given절에서 정의하고
3. when절에서 테스트할 행동을 진행
4. then절에서 테스트한 행동이 예상한 결과와 일치하는지 확인
