---
title: "[SpringBoot] Repository 계층 이란?"

categories:
  - SpringBoot
  
tags:
  - [Java, Spring Boot]

toc: true
toc_sticky: true

published: true

date: 2025-01-02
last_modified_at: 2025-01-02
---

## Spring Boot의 Repository 계층

Spring Boot 프레임워크에서 사용하는 Repository계층 이란 데이터베이스에 접근해서 특정 동작을 수행하는 게층의 로직을 구성하는 계층을 말합니다. Repository는 하나의 엔티티의 CRUD(Create, Read, Update, Delete)를 담당하기 때문에, 엔티티 1개당 Repository 1개가 필요하게 됩니다.

Java 클래스를 Repository로 만드는 방법은 아주 간단합니다. 클래스 위에 `@Repository`라는 어노테이션을 달게 되면, Spring Boot 프레임워크가 `Component Scan`기능을 통해서 해당 클래스를 Repository로 등록하게 됩니다.

```java
package jpabook.jpashop.repository;

//Component Scan 에 의해서 자동으로 SprinBean 으로 관리가 됨
@Repository
public class MemberRepository {

}
```

Repository 패턴에서 데이터베이스에 접근할 수 있게 해주는 인터페이스가 있습니다. `EntityManager`라는 녀석 입니다. 이녀석을 DI 해줘야 합니다. 

스프링에서는 기본적으로 `제어 역전의 원칙`에 따라 개발자가 DI를 하는게 아닌 프레임워크에서 코드를 분석해서 DI를 해주게 됩니다. DI를 해주는 방법에는 몇가지가 있지만, 해당 포스팅에서는 `Lombok`라이브러리의 `@RequireArgsConstructor`를 사용하여 DI를 해주도록 합니다. 그리고 EntityManager를 final 로 선언해서, 혹시 DI가 되지 않았을 때의 오류를 체크할 수 있습니다.

```java
package jpabook.jpashop.repository;

//Component Scan 에 의해서 자동으로 SprinBean 으로 관리가 됨
@Repository
@RequireArgsConstructor
public class MemberRepository {
	private final EntityManager entityManager;
}
```

이렇게 하고, EntityManager 인터페이스의 메서드를 호출하여 데이터베이스에 접근해 작업을 수행하는 코드를 만들 수 있습니다.

EntityManager의 주요 메서드들은 다음과 같습니다. (자세한 설명은 다른 포스팅에서 하겠습니다.)
- persist() : 엔티티를 영속성 컨텍스트에 저장합니다. 트랜잭션이 커밋될 때 데이터베이스 반영합니다.
- find() : 주어진 엔티티 클래스와 기본 키를 사용하여 데이터베이스에서 엔티티를 찾습니다. (영속성 컨텍스트에 있다면 해당 엔티티를 반환함)
- merge : 엔티티의 상태를 full update 합니다.
- createQuery() : 조인등의 복잡한 쿼리가 필요할 때 사용합니다. (jpql 사용)

위의 정보들을 이용하여 회원정보를 저장하고, 찾는 기능을 개발 합니다.

```java
package jpabook.jpashop.repository;

//Component Scan 에 의해서 자동으로 SprinBean 으로 관리가 됨
@Repository
@RequireArgsConstructor
public class MemberRepository {
	private final EntityManager entityManager;

	    public void save(Member member) {
        // persist 한다고 insert가 나가는게 아님 기본적으로 안나감
        // commit될 때 insert 쿼리가 쭉 날라감
        em.persist(member);
    }

    public Member findOne(Long id) {
        return em.find(Member.class, id);
    }

    public List<Member> findAll() {
        // jpql 이라는건데 sql이랑 조금 다름
        // 근데 이제 차이는 sql은 테이블을 대상으로 쿼리를 날리는데 jpql은 엔티티쪽으로 쿼리를 날림
        return em.createQuery("select m from Member m", Member.class).getResultList();
    }
}
```

이제 데이터베이스에 접근하는 계층인 Repository가 완성되었습니다. 해당 코드를 `Service` 계층에서 사용합니다. `Service` 계층이란, 웹앱의 비지니스 로직들을 처리하는 계층 입니다.

여기까지 Repository 계층의 구현이었습니다.