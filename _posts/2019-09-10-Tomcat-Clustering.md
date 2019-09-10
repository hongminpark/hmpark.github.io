---
layout: post
title: "Tomcat Clustering - 1편"
author: "Hongmin Park"
---
사내에서 WAS를 상용에서 오픈소스로 전환하려 하고 있다. 톰캣이 레퍼런스가 많다고 하지만, 
실제로 설정하면서 일어나는 이슈들에 대해서 아직 알려지지 않거나 찾기 힘들거나 그냥 나 스스로 정리가 안 된 부분이 아주 많다.
특히 추후에 이슈가 될 것 같은 부분은 **클러스터링, DB커넥션**과 같은 부분들이다. 
클러스터링으로 보통 Redis 같은 IMDB(In-Memory DB)를 사용하는 경우가 많은데, 우리의 경우에는 여건이 되지 않아
Tomcat 내의 자체 클러스터링을 사용하려고 한다. 톰캣에서도 클러스터링을 지원하지만, 안정성이 확보되지 않아서 
다른 서비스에서는 IMDB를 보통 사용한다고 한다. 
실제로 설정해보면 클러스터링이 되긴 된다. 세션 백업이 되긴 되는데, 왜 안정성 확보가 되지 않은것인가? 
그럼 우리는 안정성을 확보하기 위해서 어떻게 해야하는가?<br><br>
아직 운영환경에서 사용하기 위해서 검증되지 않은 측면이 너무 많다. 언젠간 한 번 파야한다고 생각했는데 그게 지금인 것 같다.
오늘은 톰캣에서 클러스터링 documentation을 낱낱이 정리할 것이다. 
사실 documentation의 해석에 지나지 않는다고 생각될 수 있지만, 운영자는 이런 설정 하나하나가 나중에 
어떻게 무엇이 되어 돌아올지 모르기 때문에 다양한 설정의 존재와 기능을 적어도 '인지'는 하고있어야 한다. 
그런 측면에서 documentation을 정리해보았다.
Tomcat 8 기준으로 [공식문서](https://tomcat.apache.org/tomcat-8.0-doc/cluster-howto.html)를 참조하였다.
<br><br>

# QuickStart
`server.xml`에 아래처럼 설정하면 클러스터 설정이 된다. 아래와 같이 클러스터링할 노드의 `server.xml`에 동일하게 넣어준다.
```xml
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
         channelSendOptions="8">

  <Manager className="org.apache.catalina.ha.session.DeltaManager"
           expireSessionsOnShutdown="false"
           notifyListenersOnReplication="true"/>

  <Channel className="org.apache.catalina.tribes.group.GroupChannel">
    <Membership className="org.apache.catalina.tribes.membership.McastService"
                address="228.0.0.4"
                port="45564"
                frequency="500"
                dropTime="3000"/>
    <Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
              address="auto"
              port="4000"
              autoBind="100"
              selectorTimeout="5000"
              maxThreads="6"/>

    <Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
      <Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
    </Sender>
    <Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
    <Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatch15Interceptor"/>
  </Channel>

  <Valve className="org.apache.catalina.ha.tcp.ReplicationValve"
         filter=""/>
  <Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>

  <Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
            tempDir="/tmp/war-temp/"
            deployDir="/tmp/war-deploy/"
            watchDir="/tmp/war-listen/"
            watchEnabled="false"/>

  <ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener"/>
</Cluster>
```
각 요소들에 대해서는 2편에서 더 자세히 살펴볼 것이고, 이번 편에서는 급한 사람들을 위해 대-강 보겠다.<br>
기본 클러스터링 설정은 아래와 같다. 
1. Multicast addresss는 228.0.0.4
2. Multicast port는 45564
3. Broadcast 되는 IP는 `java.net.InetAddress.getLocalHost().getHostAddress()`이다. (무슨말인지 잘..)
4. 백업받는 서버가 listen 하는 TCP port는 4000-4100대역에서 가능한 서버소켓이다.
5. Listener는 `ClusterSessionLister`로 설정되어있다.
6. `TcpFailureDetector`, `MessageDispatch15Interceptor` 라는 두 가지 interceptor가 있다.
<br><br>
클러스터링을 구현하려면 기본적으로
- 세션 속성들은 모두 `java.io.Serializable` 을 implement 해야한다. 
(Serializable의 개념은 [우아한형제들의 블로그](http://woowabros.github.io/experience/2017/10/17/java-serialize.html)에 매우 잘 설명되어있으니 한번쯤 아니 꼭 읽어볼만 하다.)
- `web.xml`에 `<distributable/>` 이 있어야 한다.
- mod_jk를 쓴다면, `server.xml`의 jvmRoute `<Engine name="Catalina" jvmRoute="node01" >` 와 `workers.properties`의 worker name이 일치하도록 해야 한다.
- session의 상태는 cookie로 트랙된다.
<br><br>
기본적으로 톰캣 클러스터링은 위와 같이 `server.xml`에 클러스터링 관련 속성을 설정하여 구현할 수 있다. 
하지만, 실제로는 저 설정만을 복붙한다고 성공하진 않을 수도 있다. 클러스터링은 노드간 통신이 필수이기 때문에
통신과정 중에 문제가 생길 수도 있다. 그러한 문제를 해결하고 통신을 성공시키기 위해서는 위의 설정들의 값들을 살펴볼 필요가 매우 있다.
2편에서는 클러스터링 관련 속성들을 더 자세히 살펴보겠다. 

