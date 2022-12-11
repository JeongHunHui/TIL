## DTO란?

Data Transfar Object의 약자로, 계층 간 데이터 교환 역할을 합니다.

![image](https://user-images.githubusercontent.com/108508730/206895651-a9413d22-7f50-40c5-b4ba-38ca3d84a79b.png)

위와 같이 **DB에 저장할 때는 Entity**를 저장하지만, **계층간 데이터가 이동할 때는 DTO**를 이용하여 데이터를 교환합니다.

DTO는 **계층간 데이터 교환**만을 위해서 만든 객체이므로 특별한 로직을 가지지 않는 순수한 데이터 객체여야 합니다.

## Entity와 DTO를 분리하는 이유

1. Entity의 모든 속성이 외부에 노출된다.
    - 비즈니스 로직 등 민감한 정보가 외부에 노출되는 보안 문제와도 직결됩니다.
2. 요청과 응답 객체가 항상 엔티티와 같지는 않다.
    - UI 화면마다 필요한 정보는 다르지만, Entity는 UI에서 사용하지 않을 불필요한 데이터까지 보유하고 있습니다.
3. UI 계층에서 엔티티의 메서드를 호출하거나 상태를 변경시킬 위험이 존재한다.
    - Controller에서 실수로 Entity의 Setter를 사용하여 Entity의 상태가 변경될 수 있습니다.
4. Model과 View가 강하게 결합되어 View의 요구사항 변화가 Model에 영향을 끼치기 쉽다.
    - DB에 저장하기 위한 Entity가 화면의 구성까지 신경쓰는 것은 관심사의 분리가 안된 것입니다.
5. Validation 코드와 Entity 속성 코드를 분리할 수 있다.
    - Entity class에는 @Column, @OneToMany 등 모델링을 위한 어노테이션들이 사용됩니다. 이 Entity영역에 @NotNull, @Min, @Legnth 등의 validation에 대한 어노테이션이 들어간다면, Entity class는 모델링과 검증을 함께 담당하며 class가 복잡해집니다.

등등.. Entity와 DTO를 분리하는 데에는 여러 이유가 있습니다.

## 스프링부트에서의 DTO 사용

클라이언트에서 User의 정보를 받아 User를 등록하는 API를 만들어보겠습니다.

**User.java**

```java
@Getter
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class User {
	@Id
  @GeneratedValue(strategy = GenerationType.SEQUENCE)
  private Long id;

  private String username;
  private String password;
	private Integer age;
	private String bio;

  @Builder
  private User(String username, String password, Integer age, String bio) {
    this.username = username;
    this.password = password;
		this.age = age;
    this.bio = bio;
  }
}
```

평범한 User Entity입니다. id를 제외하고 Builder 패턴을 적용시켜 놓았습니다.

**MemberRegistrationRequest.java**

```java
@Getter
@Builder
@AllArgsConstructor(access = AccessLevel.PROTECTED)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class UserRegistrationRequest {
  private String username;
  private String password;
	private Integer age;

	public static User toEntity(UserRegistrationRequest request) {
		return User.builder()
				.username(request.getUsername())
				.password(request.getPassword())
				.age(request.getAge())
				.build()
	}
}
```

사용자 생성 요청 정보를 담을 DTO 클래스입니다.

id 같은 경우는 DB에 저장할 때 자동으로 생성되기 때문에, DTO에 포함하지 않았고, 자기소개(bio)의 경우도 회원 가입 이후에 등록하도록 하였기 때문에 DTO에 포함하지 않았습니다.

**UserRepository.java**

```java
public interface UserRepository extends JpaRepository<User, Long> {}
```

SpringDataJPA를 사용하였습니다.

**UserService.java**

```java
@Service
@RequiredArgsConstructor(access = AccessLevel.PROTECTED)
public class UserService {
  private final UserRepository userRepository;

  public void registrationUser(UserRegistrationRequest request) {
		User user = UserRegistrationRequest.toEntity(request);
    userRepository.save(user);
  }
}
```

사용자의 요청을 담은 DTO를 받아서 User Entity로 변환한 뒤 이를 Repository를 통해 DB에 저장합니다.

**UserController.java**

```java
@RestController
@RequiredArgsConstructor(access = AccessLevel.PROTECTED)
@RequestMapping("/api/users")
public class UserController {
  private final UserService userService;

  @PostMapping
  public ResponseEntity<ResultResponse> registration(@RequestBody UserRegistrationRequest request) {
    userService.registrationUser(request);
    return ResponseEntity.ok(ResultResponse.of(USER_REGISTRATION_SUCCESS));
  }
}
```

사용자 등록 API 입니다. 사용자 등록 요청 DTO를 받아서 이를 service 계층에 전달합니다.

이렇게 사용자 정보를 저장하는 API를 DTO를 사용하여 구현해보았습니다.

## DTO를 공부하며 든 의문점

1. **DTO를 어느 계층까지 사용할 것인가? (Controller VS Service)**
    - 위에 작성한 예제 코드에서는 Service 계층에서 DTO를 Entity로 변환하였습니다. 하지만 문득 Controller 계층에서 변환을 해도 상관이 없지 않나? 라는 생각이 들었습니다.
        
        이에 대해 많이 찾아보았는데 정말 의견이 다양한 주제였습니다.
        
    - 일단, Controller 계층에서 DTO를 Entity로 변화시켜 사용하게 되면 Controller와 Entity사이의 **결합도가 상승**하고 이는 **Entity의 변경이 Controller의 변경을 촉발**하게될 수 있습니다.
        
        ```java
        @GetMapping
        public ResponseEntity<ResultResponse> getUserInfo(@RequestParam Long userId) {
          User user = userService.findById(userId);
        	Tag userInterestaTag = tagService.findByInterest(user.getInterest());
        	UserInfoResponse response = UserInfoResponse.toDto(user, userInterestaTag);
          return ResponseEntity.ok(ResultResponse.of(response));
        }
        ```
        
        만약 DTO를 API의 응답 값으로 반환해야하는 경우 Controller에서 Entity를 DTO로 변환하게되면 **View에 반환할 필요가 없는 데이터 까지 Controller**까지 넘어오게 됩니다.
        
        위의 코드 같은 경우, 보여줄 필요없는 유저의 생성시간, 유저의 비밀번호 등이 Controller까지 넘어오게 됩니다.
        
        또한, Controller가 여러 도메인 객체들의 정보를 조합해서 DTO를 생성해야 하는 경우 **Service에서 처리해야 할 비즈니스 로직이 Controller**까지 오게됩니다.
        
        위의 코드 같은 경우 UserController에서 tagService까지 사용하게 됩니다.
        
        이로 인해서 **하나의 Controller가 여러 Service에 의존**하게 될 수 있습니다.
        
    - Service가 DTO를 받아서 사용하는 경우에는 DTO를 Entity로 변환할 때 Repository를 통해 여러 정보를 조회하여 Entity를 쉽게 구성하여 사용할 수 있습니다.
        
        ```java
        @PostMapping
        public ResponseEntity<ResultResponse> registration(@RequestBody UserRegistrationRequest request) {
        	Tag userInterestaTag = tagService.findByInterest(request.getInterest());
        	User user = new User(request.getUsername(), request.getPassword(), userInterestaTag);
          userService.registrationUser(request);
          return ResponseEntity.ok(ResultResponse.of(USER_REGISTRATION_SUCCESS));
        }
        ```
        
        하지만 Controller에서 DTO를 Entity로 변환해야 하는 경우에는 위와 같은 코드처럼 Controller가 여러 Service에 의존하게됩니다.
        
    - 하지만 반대로 Service에서 DTO를 사용하게 되었을때 생기는 문제도 있습니다.
        
        만약 Service에서 DTO를 사용하게되면 **해당 Service의 메소드는 해당 컨트롤러만 사용가능**하게 됩니다.
        
        ```java
        public List<Tag> getTagsByTagInfo(TagInfoRequest tagInfo) {
        	// 태그의 정보(이름, 분류 등)를 통해 DB에서 조건에 맞는 tag목록을 가져온다.
        	return tagRepository.findAllByTagInfo(
        			tagInfo.getName(), tagInfo.getCategory());
        }
        ```
        
        위와 같은 코드 처럼 Service에서 DTO를 사용하게되면 getTagsByTagInfo 메소드는 TagInfoRequest DTO를 요청으로 받는 TagController 밖에 사용할 수 없게됩니다.
        
        다른 Controller에서 태그의 이름과 분류를 통해서 태그의 목록을 가져오고 싶어도 TagService의 getTagsByTagInfo는 TagInfoRequest DTO를 Parameter로 받기에 재사용이 불가능합니다.
        
    
    → 결국 정답은 없고, **프로젝트의 규모와 아키텍쳐 등을 고려해서 결정**해야 할 것 같습니다.
    
2. **DTO를 Entity로 변환하는 Mapper 메소드는 어디에 두어야 하는가?**
    - 위의 예시에서는 DTO 클래스 안에 toEntity라는 static Mapper 메소드를 만들어서 사용하였습니다. 하지만 위에서 DTO는 특별한 로직을 가지지 않아야 한다고 했습니다. 그러면 어떻게 하는 것이 좋을까요?
    - 만약 예시 처럼 DTO안에 toEntity 메서드를 만들거나 Entity안에 toDto 메서드를 만들어서 변환을 한다면 둘 중 하나가 바뀌어도 서로를 수정해야하고, 이는 Controller와 Service의 의존을 만들게 됩니다.
        
        ```java
        @Component
        public class UserMapper {
          public CreateUserResponse toDto(User user) {
            return CreateUserResponse.builder().userName(user.getUserName()).build();
          }
        
          public User toEntity(CreateUserRequest dto) {
            User user =
                User.builder()
                    .userName(dto.getStudyName())
                    .build();
            return user;
          }
        }
        ```
        
        이를 해결하기 위해서 Mapper라는 클래스를 만드는 방법이 있습니다. Mapper를 사용하면 DTO와 Entity 사이의 의존관계를 줄일 수 있고, 한 쪽에 수정이 발생해도 Mapper만 수정하면 됩니다.
        
    
    → 결론은 Mapper 클래스를 따로 만들어서 toEntity, toDto를 넣는 방법으로 결정했습니다.
    
3. **각 API의 요청, 응답 값을 전부 DTO로 만들어야 하는가?**
    - 위의 예시에서는 사용자 등록 API를 만들었습니다. 하지만, 사용자 정보 변경, 사용자 정보 반환 등의 API가 새로 만들어 질 때마다 각 API의 요청, 응답에 맞게 DTO를 새로 만들어야 할까요?
    - 만약에 같은 필드를 가지는 요청, 응답의 경우라고 해도 지금은 같을 수 있지만, 이후 수정을 하게되면 같은 DTO를 사용하는 곳에 영향을 줄 수 있습니다. 또한, 같은 필드인데 어떤 경우에는 null이고, 어떤 경우에는 값이 있고 하게 되면 유지보수가 어려워집니다.
        
        그래서 중요한점은 클래스를 여러개 만들고, 코드가 중복되는 것 처럼 보일지라도, API 응답 스펙이 정해지면 그 필드에 값은 **항상 같은 원칙으로 반환되도록 명확하게 설계**하는 것이 훨씬 더 나은 선택입니다.
        
    - 하지만 그래도 모든 요청과 응답에 DTO를 만들게되면 아래와 같이 수많은 DTO를 볼 수 있습니다..
        
        ![image](https://user-images.githubusercontent.com/108508730/206895666-859f139c-fe38-40bc-9f29-2558f0f65b79.png)
        
        이러한 문제를 해결하기 위해서 DTO를 Inner Class로 관리하는 방법이 있습니다.
        
        ```java
        public class UserDto {
          @Getter
          @AllArgsConstructor
          @Builder
          public static class Info {
            private int id;
            private String name;
            private int age;
          }
        
          @Getter
          @Setter
          public static class Request {
            private String name;
            private int age;
          }
        
          @Getter
          @AllArgsConstructor
          public static class Response {
            private Info info;
            private int returnCode;
            private String returnMessage;
          }
        }
        ```
        
        위와 같이 도메인 별로 관련되는 DTO들을 Class 하나에 묶게되면 클래스의 수도 줄어들고, DTO의 ClassName을 정하는 것도 수월해질 것입니다.
        

## 📕 References

- [https://tecoble.techcourse.co.kr/post/2021-04-25-dto-layer-scope/](https://tecoble.techcourse.co.kr/post/2021-04-25-dto-layer-scope/)
- [https://kafcamus.tistory.com/12?category=912020](https://kafcamus.tistory.com/12?category=912020)
- [https://www.inflearn.com/questions/72423/dto](https://www.inflearn.com/questions/72423/dto)
- [https://velog.io/@p4rksh/Spring-Boot에서-깔끔하게-DTO-관리하기](https://velog.io/@p4rksh/Spring-Boot%EC%97%90%EC%84%9C-%EA%B9%94%EB%81%94%ED%95%98%EA%B2%8C-DTO-%EA%B4%80%EB%A6%AC%ED%95%98%EA%B8%B0)
