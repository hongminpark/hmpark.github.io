---
layout: post
title: "[Spring.3-2] Business Layer - Transaction"
author: "Hongmin Park"
---

![Business Layer](https://mail.google.com/mail/u/0?ui=2&ik=e26376f5e4&view=fimg&th=16b690ff6a49d2e9&attid=0.12&disp=emb&attbid=ANGjdJ9BuqnXHFctMzgwFEuBvjZP16IXM-IP-8JM96NirSOfoNPPvSJ7_B3biXB3FWN_gmagntVZiV9HWzwsy-7sI27j06CTYu2xN2aBytUbz-8DsitZ-ghXAGD2uF8&sz=s0-l75-ft&ats=1560951710471&rm=16b690ff6a49d2e9&zw&atsh=1 "Business Layer")
_Business Layer_

<br>

# 정의
비즈니스 로직을 묶어서 처리하는 단위이다. 데이터베이스 기준으로는 connection을 open하고 rollback/commit, close 하는 단위이다.<br><br>

# 속성/특성
1) ACID
- Atomicity(원자성)<br>
하나의 transaction 내에 있는 비즈니스 로직들이 모두 정상 처리되어야 transaction 처리가 완료(commit)된다.<br>
즉 어느 하나의 비즈니스 로직이 정상적으로 처리되지 않는다면 rollback한다.<br>
- Consistency(정합성)<br>
transaction이 처리되는 과정에서 모든 로직은 동일한 데이터로 작업해야 한다.<br> 
(데이터의 정합성 의미)<br>
- Isolation(독립성)<br>
여러 transaction은 독립적으로 처리된다. <br>
병렬적으로 여러 transaction 요청이 있을 때에, 한 transaction 종료 후에 다음 transaction이 진행된다.
- Durability(영속성)<br>
트랜잭션이 완료(commit)되면 데이터는 그 상태 그대로 유지된다.<br><br>

2) 트랜잭션의 경계
트랜잭션은 Presentation Layer와 Business Layer 사이에 위치한다. <br>
즉, Service 클래스의 메소드가 실행되고 종료되는 시점이 트랜잭션의 경계이다. 

3) Global/Local 트랜잭션
트랜잭션을 2가지 종류로 분류하기도 한다.<br>
- Local Transaction : 단일 리소스(데이터소스)에 대한 트랜잭션, 하나의 데이터소스에 대해서 작업이 끝나면 바로 commit한다.
- Global Tranaction : 여러 리소스(데이터소스)에 대한 트랜잭션, 여러 데이터소스(이기종 DB)에 대하여 작업을 한 뒤에 commit/rollback 해야 하는 경우에 각 트랜잭션을 묶어서 Global Transaction이라고 한다.<br>
즉, 어떤 작업/비즈니스 로직(Service)가 단일 DB 작업이라면 로컬트랜잭션, 물리적으로 다중 DB에 대한 작업이라면 글로벌 트랜잭션이라고 한다.<br><br>


# 기술
스프링의 Transaction은 PlatformTransactionManager(Interface)를 중심으로 구성되어 있다. PlatformTransactionManager를 구현한 클래스를 사용하여 관리한다.<br>
또한, 트랜잭션 매니저를 구현하는 방법에는 'Programmatic/Declarative' 두 가지가 있다.<br><br>

#### [트랜잭션 구현 방법]
###### 1) 명시적 트랜잭션(Programmatic Transaction)
비즈니스 로직 각각에 직접 트랜잭션 코드를 구현하여 관리하는 방법이다.<br>
###### 2) 선언적 트랜잭션(Declarative Transaction)
AOP/Annotation(@Transactional) 방식 존재<br>
**2-1) AOP방식**<br>
xml 설정파일에 AOP 설정을 하는 방식. 한 곳에서 Transaction 관리한다.
{% highlight xml %}
<bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <property name="dataSource" ref="dataSource"/>
</bean>
<tx:advice id="txAdvice" transaction-manager="txManager">
   <tx:attributes>
       <tx:method name="select*" read-only="true"/>
       <tx:method name="insert*" rollback-for="Exception"/>
       <tx:method name="update*" rollback-for="Exception"/>
       <tx:method name="delete*" rollback-for="Exception"/>
       <tx:method name="multi*" rollback-for="Exception"/>
   </tx:attributes>
</tx:advice>
{% endhighlight %}

**2-2) Annotation방식**<br>
- 스프링 트랜잭션매니저를 사용할 클래스/메소드 등 위에 **@Transactional**을 단다.
{% highlight java %}
@Transactional
@Service("basicSampleService")
public class BasicSampleServiceImpl extends HService implements BasicSampleService{
  ...
}
{% endhighlight %}
- 단, 주의할 점은 **@Transactional**을 사용할 Service 레이어 또는 class가 Spring bean으로 등록되어 있어야 한다.<br>
- 또한, xml 설정파일에 
{% highlight java %}<tx:annotaion-driven transaction-manager="txManager">{% endhighlight %}
를 기입하여야 한다. <br>
- 마지막으로, 메소드가 'public' method 여야 한다. (private, protected, package-visible method 의 경우에는 AspectJ를 고려한다.) 
<br>이는 모두 Spring Transaction이 **AOP** 를 기반으로 작동하며, Spring AOP는 기본적으로 **Proxy** 기반이기 때문이다.
<br><br>

# 기타
###### 1) AOP는 트랜잭션(connection)을 한 곳에서 한 번에 관리.<br>
AOP는 Datasource가 여러 개 있을 경우에, 한 번에 Connection open하고 close한다. <br>
만약 어떤 Datasource(DB Server)가 물리적으로 먼 곳에 있어서 open/close에 오래 걸리는 경우에 전체 Transaction의 속도에 영향을 주는 문제점이 있다.

###### 2) 2 Phase Commit
한 트랜잭션에서 이기종 DB 즉, 물리적으로 다른 DB를 참조하고 있을 때. <br>
이기종 DB간 커밋할 일이 있을 때를 2 Phase Commit이라고 한다. <br>
예를 들어 DB1 커밋 -> DB2 조희 -> DB1 커밋 에는 transaction 구현이 복잡해 진다.

###### 3) transaction 장애발생 대표 유형<br>
**3-1)** Exception처리 (try/catch문)<br>
트랜잭션에 roll-back 설정을 할 수 있는데 default는 RuntimeException이다. <br>
또한, 스프링 트랜잭션은 Checked Exception의 경우는 롤백하지 않는다.<br>
transaction의 경계는 프레젠테이션 레이어와 비즈니스 레이어 사이. 즉, Service 클래스에서 메소드의 실행과 종료가 transaction의 경계이다.<br>
비즈니스레이어(Service 클래스)에서 Exception처리를 통해 메소드들이 정상 종료되면 transaction은 정상적으로 commit된다. 비즈니스레이어(Service)에서 try-catch문으로 예외를 처리하면 해당 메소드 종료 시에는 예외처리로 인해 정상상태이고 따라서 transaction은 commit된다.<br>
즉, 예외를 잡기 위해 예외처리를 하였는데 transaction이 정상 commit 되는 상황이 발생한다.<br>
-> 이러한 경우에는 try/catch를 빼거나, 프레젠테이션레이어(Controller) 에서 예외 처리를 해야 한다.<br><br>

**3-2)** PL(Procedural Language)/SQL 사용 시<br>
PL/SQL에서 내부적으로 예외 처리를 할 시에도 transaction은 정상적으로 commit되기 때문에 transaction이 처리됨.<br><br>
**3-3)** package 명시 에러<br>
AOP에서 pointcut으로 package를 잘못 등록했을 때

 

 
