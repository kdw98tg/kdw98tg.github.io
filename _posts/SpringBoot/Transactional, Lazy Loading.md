---
title: "[SpringBoot] Transactional 과 Lazy Loading"

categories:
  - SpringBoot
  
tags:
  - [Java, Transactional, LazyLoading]

toc: true
toc_sticky: true

published: true

date: 2025-08-26
last_modified_at: 2025-08-26
---

```java
    // @Transactional
    // Transactional 은 무조건 public 메서드 앞에만 달게 되어 있음
    // private 메서드일 경우 그냥 Transactional 빼고 쓰면 되고,
    // 사용하는 public 메서드에서 Transactional 을 달아주면 됨
    // 그리고, Transactional 은 LazyLoading 체인을 타야 하는 함수에만 넣어주면 됨
    // 가령, GetProject().GetUser() 처럼 체인을 타는거는 넣어줘야 하는거임
    // 그게 아니면 EagerLoading 으로 가져오면 됨
    // 근데 위에꺼는 성능이 안좋아서 안씀
    // 결국 LazyLoading 은 Transactional 을 사용해야 함
    private boolean IsValidUser(Long _userId, Long _functionDetailId) {
        boolean result = false;

        FunctionDetail functionDetail = functionDetailRepository.findById(_functionDetailId)
                .orElseThrow(() -> new RuntimeException("FunctionDetail not found"));

        if (functionDetail.getFunction().getProject().getUser().getUserId().equals(_userId)) {
            result = true;
        }

        return result;
    }
```