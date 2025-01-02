---
title: "[SpringBoot] Controller 계층 이란?"

categories:
  - SpringBoot
  
tags:
  - [Java, Head-First Design Pattern]

toc: true
toc_sticky: true

published: false

date: 2025-01-02
last_modified_at: 2025-01-02
---

## Spring Boot의 Controller 계층이란?

Controller 계층은 사용자의 요청을 직접적으로 받는 계층입니다. 개발자가 지정한 라우팅 경로를 사용자가 입력하였을 때, Controller가 입력을 받아서, Service 계층의 함수를 호출하여 작업을 수행하게 됩니다. 이러한 과정을 Handle 이라고 합니다. Controller 역시 `@Controller`어노테이션을 달아서 `제어 역전의 원칙`에 따라서 Spring Boot 프레임워크가 자동으로 만들어주게 합니다.

```java
@Controller
@RequiredArgsConstructor
public class MemberController {
    private final MemberService memberService;
}
```

`@Controller`어노테이션을 달아주고 컨트롤러는 Service 계층의 함수를 사용해야 하기 때문에 Lombok 라이브러리의 `@RequriedArgsConstructor`를 달아주어 Service 를 DI 받습니다.

그리고 컨트롤러 메서드는 다음과 같이 생겼습니다.

```java
@Controller
@RequiredArgsConstructor
public class MemberController {
    private final MemberService memberService;

    @GetMapping("/members/new")
    public String createForm(Model model) {
        model.addAttribute("memberForm", new MemberForm());
        return "members/createMemberForm";
    }
}
```

메서드 위에 달려있는 `@GetMapping`은 아래에서 설명하겠습니다. 그 외에 특이한 점은 String을 반환하는 메서드이고, return 값에는 경로가 적혀있다는 것입니다.

return 값 뒤에는 화면에 렌더링 해야 할 html 파일의 경로를 나타냅니다. 뒤에 `.html` 이 생략된 형식 입니다.

## Restful 요청

Controller에서 주목해야 할 점은 바로 Restful한 요청 입니다. Restful API 는 GET, POST, PUT, PATCH, DELETE 등의 행동을 명시적으로 나타내는 API 프로토콜을 이야기 합니다.

Controller에서 해당 동작에 따라 메서드명 위에 어노테이션을 달아주게 됩니다.

- @GetMapping : 데이터를 조회하거나 리소스 상태를 가져오는 요청일 때 사용, 서버의 상태를 변경하지 않기에, 주로 읽기 전용 작업에 사용함

```java
@GetMapping("/users")
public List<User> getAllUsers() {
    return userService.getAllUsers();
}
```

- @PostMapping : 서버에 새로운 데이터를 생성하는 요청

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@RequestBody UserForm user) {
    User createdUser = userService.createUser(user);
    return "redirect:/";
}
```

여기서, `@RequestBody`는 클라이언트로부터 전달받은 HTTP 요청 본문을 Java 객체로 변환하는데 사용됩니다. 그리고, DTO(Data Transfer Object)를 생성하여 작업을 처리합니다.

- @PutMapping : 기존 리소스를 완전히 대체하거나 수정함. 만약 해당하는 리소스가 없다면, 생성 가능

```java
@PutMapping("/users/{id}")
public ResponseEntity<User> updateUser(@PathVariable Long id, @RequestBody User user) {
    User updatedUser = userService.updateUser(id, user);
    return "redirect:/";
}
```

여기서, `@PathVariable`은 라우팅 경로에서 들어오는 값을 매핑하는데 사용합니다.

- @PatchMapping : 기존 리소스의 일부를 수정함

```java
@PatchMapping("/users/{id}")
public ResponseEntity<User> partialUpdateUser(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
    User updatedUser = userService.partialUpdateUser(id, updates);
    return "redirect:/";
}
```

- @DeleteMapping : 서버에서 특정 리소스를 삭제하는 요청

```java
@DeleteMapping("/users/{id}")
public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    userService.deleteUser(id);
    return "redirect:/";
}
```


##  Form 객체 사용

위의 Restful API를 설명할 때, `Form`객체를 사용한 것을 볼 수 있습니다. Entity 클래스를 바로 사용하는것이 아닌 Form 객체를 사용하면, 불필요한 데이터 필드의 사용을 줄일 수 있고, 화면 계층과 서비스 계층을 명확하게 분리한다는 장점이 있습니다. 또한, 화면 요구사항이 복잡해지기 시작하면, 엔티티에 화면을 처리하기 위한 기능이 점점 증가합니다. 결과적으로 엔티티는 점점 화면에 종속적으로 변하고, 이렇게 화면 기능 때문에 지저분해진 엔티티는 결국 유지보수하기가 어려워지게 됩니다. 결국, 단일 책임의 원칙을 잘 지켜야 한다는 뜻입니다.

다음은, 회원가입할 때 사용하는 `MemberForm` 객체입니다.
```java
@Getter @Setter
public class MemberForm {
	@NotEmpty(message =
	private String name;
	"회원 이름은 필수 입니다")
	private String city;
	private String street;
	private String zipcode;
}
```

해당 객체를 사용하여 데이터를 받아 처리를 합니다. DTO 의 역할이라고 생각하면 됩니다.

## Model 객체 사용

Controller의 입력을 받아서 특정 데이터를 조회하는 로직이 있다고 생각해 봅시다. 조회가 끝나면, View 에게 해당 정보를 뿌려주는 과정이 필요합니다. 그러게 위해서 Spring MVC 가 제공하는 `Model` 객체에 보관하고 뿌려줍니다. Model 객체는 `addAttribute`라는 함수를 통해서 key값과 value 값을 전달하게 됩니다.

```java
    @GetMapping("/members")
    public String list(Model model) {
        List<Member> members = memberService.findMembers();
        model.addAttribute("members",members);
        return "members/memberList";
    }
```




