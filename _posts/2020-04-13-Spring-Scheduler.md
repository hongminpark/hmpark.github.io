---
layout: post
title: "Scheduler - fixedRate, fixedDelay, cron 동작방식"
author: "Hongmin Park"
tags: java spring scheduler
---
사내 코드에서 스케쥴러를 이용한 작업 처리가 많이 있다. 메소드에 `@Scheduled` 어노테이션을 달아서 많이 사용하는데, 이 때 **fixedRate, fixedDelay, cron** 세 가지 동작방식을 지정할 수 있다.
직관적으로 생각했을 때에는 fixedDelay의 경우 이전 작업이 정상적으로 끝나야 다음 작업이 진행될 것 같은데, fixedRate과 cron은 어떻게 동작할까 ?<br>
테스트 전 나의 예상은 fixedRate은 이전 작업이 종료되지 않아도 다음 작업은 별도의 스레드에서 수행될 것이고 cron역시 마찬가지라고 생각했다.<br>
결과는 전혀 달랐다. **결론**부터 이야기하면, *`fixedRate`, `fixedDelay`, `cron` **모두 동시에 하나의 작업이 수행됨을 보장한다.***<br>
각 경우에 대한 설명은 아래에 CASE 테스트코드와 실제 로그로 대체한다. 각 케이스는 `Thread.sleep()`을 통해 작업을 지연시키며 테스트 해보았다.

## 스케쥴러 동작방식
### [CASE1] fixedRate
작업이 rate보다 오래걸리면 다음 작업은 queing되고 현 작업이 종료된 후 바로 실행됨.
#### 테스트코드
```java
  @SneakyThrows
	@Scheduled(fixedRate = 30 * 1000) // 30s 주기
	public void printCacheStats() {
		log.info("CacheStats : {}", cache.stats());
		Thread.sleep(1 * 60 * 1000); // 쓰레드 1분 sleep
	}
```
#### 결과로그
2020-04-13 **15:20:54.779** INFO 18675 --- [ulingExecutor-1] c.n.a.p.s.service.Cache : CacheStats : CacheStats{hitCount=0, missCount=0, loadSuccessCount=0, loadFailureCount=0, totalLoadTime=0, evictionCount=0, evictionWeight=0}<br>
2020-04-13 **15:21:54.785** INFO 18675 --- [ulingExecutor-2] c.n.a.p.s.service.Cache : CacheStats : CacheStats{hitCount=0, missCount=0, loadSuccessCount=0, loadFailureCount=0, totalLoadTime=0, evictionCount=0, evictionWeight=0}<br>
### [CASE2] fixedDelay
이전 작업이 종료되면 delay만큼 지난 후 다음 작업 실행

#### 테스트코드
```java
  @SneakyThrows
	@Scheduled(fixedDelay = 30 * 1000) // 30s Delay
	public void printCacheStats() {
		log.info("CacheStats : {}", cache.stats());
		Thread.sleep(1 * 60 * 1000); // 쓰레드 1분 sleep
	}
```
#### 결과로그
2020-04-13 **15:27:23.960** INFO 19039 --- [ulingExecutor-1] c.n.a.p.s.service.Cache : CacheStats : CacheStats{hitCount=0, missCount=0, loadSuccessCount=0, loadFailureCount=0, totalLoadTime=0, evictionCount=0, evictionWeight=0}<br>
2020-04-13 **15:28:53.971** INFO 19039 --- [ulingExecutor-2] c.n.a.p.s.service.Cache : CacheStats : CacheStats{hitCount=0, missCount=0, loadSuccessCount=0, loadFailureCount=0, totalLoadTime=0, evictionCount=0, evictionWeight=0}<br>

### [CASE3] cron
이전 작업이 설정된 cron 시간까지 종료되지 않으면 작업을 실행하지 않고 다음 cron 시간을 기다림.
#### 테스트코드
```java
  @SneakyThrows
	@Scheduled(cron = "0,10,20,30,40,50 * * * * *") // 매 00,10,20,30,40,50s
	public void printCacheStats() {
		log.info("CacheStats : {}", cache.stats());
		Thread.sleep(1 * 60 * 1000); // 쓰레드 1분 sleep
	}
```
#### 결과로그
2020-04-13 **15:24:40.018** INFO 18904 --- [ulingExecutor-1] c.n.a.p.s.service.Cache : CacheStats : CacheStats{hitCount=0, missCount=0, loadSuccessCount=0, loadFailureCount=0, totalLoadTime=0, evictionCount=0, evictionWeight=0}<br>
2020-04-13 **15:25:50.001** INFO 18904 --- [ulingExecutor-3] c.n.a.p.s.service.Cache : CacheStats : CacheStats{hitCount=0, missCount=0, loadSuccessCount=0, loadFailureCount=0, totalLoadTime=0, evictionCount=0, evictionWeight=0}<br>
<br>
*참고 : [ScheduledExecutorService](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html?is-external=true)
