---
title: "[SpringBoot] Model Struct 사용하여 Model, Dto간 매핑하기"

categories:
  - SpringBoot
  
tags:
  - [Java, Spring Boot, Model Struct]

toc: true
toc_sticky: true

published: false

date: 2025-08-26
last_modified_at: 2025-08-26
---

Spring Boot와 같은 백엔드 프로젝트에서 클라이언트와 서버 간 데이터를 주고받을 때는 보통 순수 Entity를 그대로 사용하지 않고 DTO(Data Transfer Object)를 활용합니다.  
이는 관심사의 분리, 보안 강화, 서버 호출 최소화 등의 이유 때문으로, 제대로 된 프로젝트에서는 DTO 사용이 사실상 필수적입니다.

하지만 프로젝트를 진행하다 보면 Model과 DTO를 서로 변환하는 과정에서 코드가 복잡해지는 경우가 많습니다. Setter 메서드나 다수의 생성자 오버로딩이 등장해 코드 가독성이 떨어지곤 합니다.

이러한 문제를 해결하기 위해 Model Struct를 활용하면 변환 과정을 단순화하고 코드의 깔끔함을 유지할 수 있습니다.


## Map Struct 의존성 추가

Map Struct 라이브러리를 사용하기 위해서는 다음의 의존성을 추가해야 합니다. (MapStruct 는 Lombok의 Getter, Setter, Builder를 사용해서 생성되기 때문에, Lombok 의존성을 먼저 넣어주신 후에 MapStruct의 의존성을 넣어야 합니다.)

- build.gradle
```groovy
dependency{
implementation 'org.mapstruct:mapstruct:1.5.3.Final'//map struct  
annotationProcessor 'org.projectlombok:lombok-mapstruct-binding:0.2.0'//map struct  
annotationProcessor 'org.mapstruct:mapstruct-processor:1.5.3.Final'//map struct
}
```

## MapStruct 사용 예시

우선, `User`라는 Entity와 User의 로그인 기능이 수행될 때 데이터를 옮길 `UserLoginDto`를 사용해서 예시를 만들어 보았습니다.

 **<span style="color: #c03a92">class User</span>**
```java
package com.bst.code_sequence.user.entity;  
  
import com.bst.code_sequence.project.entity.Project;  
import com.bst.code_sequence.security.entity.Role;  
import jakarta.persistence.*;  
import lombok.Getter;  
import lombok.Setter;  
  
import java.util.ArrayList;  
import java.util.List;  
  
@Getter  
@Setter  
@Entity  
public class User {  
    @Id  
    @GeneratedValue    @Column(name = "user_id")  
    private Long userId;  
  
    @Column(name = "user_name")  
    private String userName;  
  
    @Column(name = "user_email")  
    private String userEmail;  
  
    @Column(name = "user_profile_image")  
    private String userProfileImage;  
  
    @Column(name = "user_locale")  
    private String userLocale;  
  
    @Enumerated(EnumType.STRING)  
    @Column(name = "user_role")  
    private Role role;  
  
    //필드는 가급적 변경 x, 필드 초기화 권장  
    @OneToMany(mappedBy = "user")  
    private List<Project> projects = new ArrayList<Project>();  
}
```

**<span style="color: #c03a92">class UserLoginDto</span>**
```java
package com.bst.code_sequence.user.dto;  
  
import lombok.Getter;  
import lombok.Setter;  
  
@Getter  
@Setter  
public class UserLoginDto {  
    private String userName;  
    private String userProfileImage;  
}
```


다음으로는 Mapper Interface를 작성합니다. 해당 Interface를 작성하면 MapStruct 라이브러리에서 자동으로 세부 구현 코드를 작성해 줍니다. 그래서, 해당 클래스가 Mapper 클래스라는것을 알리기 위해 어노테이션 `@Mapper`를 선언 합니다.

그리고, 매퍼 클래스에서 해당 인터페이스를 찾게 하기 위해서 INSTANCE를 정의 해줍니다.

마지막으로, 현재 Entity 1개, DTO 1개가 있으므로, Entity를 DTO로 바꿔주는 매서드 하나, DTO를 Entity로 바꿔주는 메서드 하나를 정의합니다. 네이밍은 반환형을 기준으로 접미사 `to`를 붙였습니다.

```java
@Mapper
public interface IUserMapper {
    IUserMapper INSTANCE = Mappers.getMapper(IUserMapper.class);

    public User toLoginDto(UserLoginDto _dto);

    public UserLoginDto toUserLoginDto(User _user);
}
```

이렇게 설정해주면, ModelStruct 라이브러리에서 컴파일을 할 때, 자동으로 `IUserMapperImpl`클래스를 생성해 줍니다.

그리고 사용할때는 다음과 같이 사용하면 됩니다.

```java
public void SomeMethod() {  
    User user = new User();  
    UserLoginDto userLoginDto = userMapper.toUserLoginDto(user);  
    User userEntity = userMapper.toLoginDto(userLoginDto);  
}
```

이렇게하면 수많은 setter를 사용할 필요도 없고, 수많은 생성자 오버로딩을 할 필요도 없고 코드 한줄로 Entity와 DTO를 변환할 수 있습니다.