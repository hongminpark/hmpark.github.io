---
layout: post
title: "CaffeineCache를 잘 활용한 코드 개선"
author: "Hongmin Park"
---
팀 내에서 JVM에 내부에 캐시용도로 [`CaffeineCache`](https://github.com/ben-manes/caffeine/wiki)를 사용하고 있는데, 이를 개선하는 작업을 과제로 받았다.
#### 문제
- 현재 코드에서는 30분 후 캐시가 만료되도록 설정되어 있고, 만료 후 새 요청이 들어오면 새로 캐시를 갱신한다.
- 30분 후 만료되어 새로 갱신 작업이 오래걸려 Nginx에서 Timeout에 의해 500에러가 발생한다.

#### 요구사항
- 스케쥴러를 통해 30분 주기로 캐시를 갱신한다.
- 사용량 통계를 통해 사용되지 않는 쿼리는 캐시 생성 대상에서 제외하는 로직을 추가한다.

초기 문제와 요구사항은 위와 같았다. 처음에는 요구사항 대로 스프링 스케쥴러를 등록하고, 사용량 통계를 기록하려 했다. 그런데, 생각해보니 두 작업 모두 캐시에서 지원해야 하는 기능인 것 같았다. 캐시라이브러리를 사용하는데도 별도로 스케쥴러, 통계가 필요할 리가 없다고 생각되어 [CaffeineCache documentation](https://github.com/ben-manes/caffeine/wiki)을 정독했다.
<br><br>
정독결과 역시 캐시라이브러리는 다양한 기능을 지원하고 있었다. 특히, CaffeineCache(를 포함한 대부분의 캐시)에서는 주기적으로 값을 갱신([refresh](https://github.com/ben-manes/caffeine/wiki/Refresh))할 수 있는 기능과 메모리/시간에 기반하여 잘 사용하지 않는 값은 캐시 대상에서 제거([evict](https://github.com/ben-manes/caffeine/wiki/Eviction))하는 기능이 있는데, 이를 활용하여 코드를 아래와 같이 개선시키고 요구사항을 만족시킬 수 있었다.


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
기존의 코드는 위와 같이 key에 대한 value가 있는지 확인하고, 없으면(=30분 뒤 만료된 상황) update하는 로직이었다. 가장 단순한 [Manual](https://github.com/ben-manes/caffeine/wiki/Population#manual) get/put 방식이다.


### After
```java
    // 캐시 객체 생성 부분
    LoadingCache<String, String> cache = Caffeine.newBuilder()
                                          .refreshAfterWrite(30, TimeUnit.MINUTES)
                                          .recordStats()
                                          .build(key -> sampleService.getValue(key));
    // 캐시객체 초기화 시 warm-up
    cache.getAll(getKeyLists()); 

    // 캐시를 이용한 비즈니스 로직 부분
    doSomething(cache.get(key));
		cache.getAll(getKeyLists());
```
위와 같이 `LoadingCache`를 활용하여 비동기 방식으로 캐시를 로드할 수 있다. `refreshAfterWrite`을 통해 30분 뒤 갱신하도록 하고, build 내부에 load함수를 넘겨주면서 key에 대한 value를 동적으로 load할 수 있도록 하였다. null체크를 할 필요가 없어졌다.

### 주의사항
> *Refresh operations are executed asynchronously using an Executor. The default executor is ForkJoinPool.commonPool() and can be overridden via Caffeine.executor(Executor)*
-> Refresh는 별도의 스레드를 통해 비동기로 수행된다. 여기서 나는 특정 주기로 비동기 스레드가 refresh 작업을 수행한다고 생각했는데, 그건 아니다.

> *In contrast to expireAfterWrite, refreshAfterWrite will make a key eligible for refresh after the specified duration, but a refresh will only be actually initiated when the entry is queried.*
-> refreshAfterWrite에 명시된 주기 이후에 key가 호출되면, 해당 key에 대한 old value가 리턴되고 갱신작업은 비동기 스레드로 진행된다.
