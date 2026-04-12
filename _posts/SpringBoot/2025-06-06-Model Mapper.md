---
title: "[SpringBoot] Model "

categories:
  - SpringBoot
  
tags:
  - [Java, Spring Boot]

toc: true
toc_sticky: true

published: false

date: 2025-06-06
last_modified_at: 2025-06-06
---

```java
package com.bst.code_sequence.config;  
  
import org.modelmapper.ModelMapper;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
@Configuration  
public class ModelMapperConfig {  
    @Bean  
    public ModelMapper modelMapper() {  
        ModelMapper modelMapper = new ModelMapper();  
        modelMapper.getConfiguration().setFieldMatchingEnabled(true).setFieldAccessLevel(org.modelmapper.config.Configuration.AccessLevel.PRIVATE);  
        return modelMapper;  
    }  
  
}
```

```java
package com.bst.code_sequence;  
  
import com.bst.code_sequence.dto.UserLoginDto;  
import com.bst.code_sequence.entitiy.User;  
import jakarta.transaction.Transactional;  
import org.junit.jupiter.api.Assertions;  
import org.junit.jupiter.api.Test;  
import org.junit.jupiter.api.extension.ExtendWith;  
import org.modelmapper.ModelMapper;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.boot.test.context.SpringBootTest;  
import org.springframework.test.context.junit.jupiter.SpringExtension;  
  
@SpringBootTest  
@Transactional  
@ExtendWith(SpringExtension.class)  
public class ModelMapperTest {  
  
    @Autowired  
    private ModelMapper modelMapper;  
  
    @Test  
    public void MapperTest() {  
        //given  
        User user = new User("1", "2", "ko");  
  
        //when  
        UserLoginDto dto = modelMapper.map(user, UserLoginDto.class);  
  
        //then  
        Assertions.assertEquals(dto.getUserName(), user.getUserName());  
        Assertions.assertEquals(dto.getUserProfileImage(), user.getUserProfileImage());  
    }  
}
```