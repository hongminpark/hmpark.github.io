---
layout: post
title: "CaffeineCache를 활용한 코드 개선"
author: "Hongmin Park"
---
팀 내에서 JVM에 내부에 캐시용도로 [`CaffeineCache`](https://github.com/ben-manes/caffeine/wiki)를 사용하고 있는데, 이를 개선하는 작업을 과제로 받았다.
과제 자체는 간단했다. 

#### 문제
- 현재 코드에서는 30분 후 캐시가 만료되도록 설정되어 있고, 만료 후 새 요청이 들어오면 새로 캐시를 로드하도록 되어있다.
- 30분 후 만료되어 새로 갱신 시 너무 오래걸려 Nginx에서 Timeout에 의해 500에러가 발생한다.
- 사용자가 많이 없는 시간대에 요청을 보내는 사람은 꼭 500에러를 보게되어있다.

#### 요구사항
- 스케쥴러를 통해 30분 주기로 만료되도록 한다.
- 사용량 통계를 통해 사용되지 않는 쿼리는 캐시 생성 대상에서 제외하는 로직 필요

초기 문제와 요구사항은 위와 같았다. 요구사항 대로 스케쥴러를 등록하고, 사용량 통계를 기록하려고 하였는데 무언가 이상했다. 
두 요구사항을 만족하기 위해 캐시를 사용하는게 아닌가? 라는 의문이 들었고, 그래서 [CaffeineCache documentation](https://github.com/ben-manes/caffeine/wiki)을 정독했다.
<br><br>
정독결과, CaffeineCache(를 포함한 대부분의 캐시)에서는 주기적으로 값을 갱신([refresh](https://github.com/ben-manes/caffeine/wiki/Refresh))할 수 있는 기능과 메모리/시간에 기반하여 잘 사용하지 않는 값은 캐시 대상에서 제거([evict](https://github.com/ben-manes/caffeine/wiki/Eviction))하는 기능이 있다.
CaffeineCache의 기본 기능을 잘 활용하여 코드를 아래와 같이 개선시키고 요구사항을 만족시킬 수 있었다.


### Before
```java
    ...
    // 캐시 객체 생성 부분
    Cache<String, String> cache = Caffeine.newBuilder().expireAfterWrite(30, TimeUnit.MINUTES).build();
    String value = cache.getIfPresent(key);

    // 캐시를 이용한 비즈니스 로직 부분
    if (value == null) {
        cache.put(sampleService.getValue(key));
    }
    
    doSomething(value);
    ...
```
기존의 코드는 위와 같이 key에 대한 value가 있는지 확인하고, 없으면(=30분 뒤 만료된 상황) update하는 로직이었다.

### After
```java
    // 캐시 객체 생성 부분
		Cache<String, String> cache = Caffeine.newBuilder()
                                          .refreshAfterWrite(30, TimeUnit.MINUTES)
                                          .recordStats()
                                          .build(key -> sampleService.getValue(key));
    // 캐시객체 초기화 시 warm-up
		cache.getAll(getKeyLists()); 

    // 캐시를 이용한 비즈니스 로직 부분
    doSomething(cache.get(key));
		cache.getAll(getKeyLists());
```
위와 같이 `refreshAfterWrite`을 통해 30분 뒤 갱신하도록 하였다. 또한, build 내부에 load함수를 넘겨주면서 key에 대한 value를 동적으로 load할 수 있도록 하였다.
