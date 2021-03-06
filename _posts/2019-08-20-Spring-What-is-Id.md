---
layout: post
title: "Spring Project: GroupId, ArtifactId, Version"
author: "Hongmin Park"
tags: spring
---
Spring 프로젝트를 만들다 보면, 초기생성할 때에 Spring Framework이건 Spring Boot이건 아래와 같이 항상 'GroupId', 'ArtifactId, 'Version'을 물어본다.<br><br>
![무엇을 입력하는 것인고](https://user-images.githubusercontent.com/21957275/63358312-cbef7980-c3a5-11e9-8827-eab65523d4e7.png)<br><br>

매번 남들따라.. 예제따라 아무거나 입력(com.example, com.hm, ...) 했었는데 이번에는 넘어가지 않고 찾아보기로 하였다.<br>
그런데, 구글에 Spring groupid, artifactid, version 이라고 치면 잘 안나온다.(~~당연히 잘 나올 줄 알았는데?~~)<br>
연관검색을 잘 보면, 
- What is groupId and artifactId?
- What is difference between groupId and artifactId in Maven?
- What should be the groupId and artifactId in Maven?
<br><br>
이런 질문들이 있다.<br>
여기서 나는 groupid, artifactid, version이 Maven project에서 필요한 것임을 처음 알게되었다.<br>
검색어를 수정해서 다시 찾아보자.<br>
> What is maven groupid, artifactid, version?

나는 공식다큐멘테이션을 좋아해서 [메이븐 공식 다큐멘테이션](https://maven.apache.org/guides/getting-started) 여기서 정의를 찾아보았다.<br>
다큐멘테이션에 따르면 groupid, artifactid, version은 Maven 프로젝트의 기본 단위인 POM(Project Object Model)를 이루는 요소 중 하나이다.<br>
세 개만 찾아보려고 했는데, pom.xml 정의에 필요한 다른 요소들도 같이 나와있어 읽어보았다.<br>
다큐멘테이션에 따르면 pom.xml은 메이븐 프로젝트에 필요한 모든 정보들이 다 들어있어서 이걸 이해하는게 굉장히 굉장히 중요하다고 되어있다.(~~그냥 버전정보인줄알았는데..~~)
<br>

pom.xml에서 쓰이는 간단한 아래 9가지 요소만 오늘은 보겠다.<br>
**pom.xml(sample)**
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>Maven Quick Start Archetype</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

### project

> This is the top-level element in all Maven pom.xml files.
<br> pom.xml에서 최상위 요소에 해당하는 태그. 뭔가 메이븐 버전정보, 스키마 로케이션 등이 담겨있다. 

### modelVersion
> This element indicates what version of the object model this POM is using. The version of the model itself changes very infrequently but it is mandatory in order to ensure stability of use if and when the Maven developers deem it necessary to change the model.
<br> Maven 버전이라고 생각하면 될 것 같다. 여기서 object model은 pom의 OM을 뜻하는 것 같은데, 느낌상 프로젝트에 사용되는 모든 것들..즉, pom.xml에 있는 모든 것들이 오브젝트가 아닐까.. 라는 추측을 해본다.

### groupId
> This element indicates the unique identifier of the organization or group that created the project. The groupId is one of the key identifiers of a project and is typically based on the fully qualified domain name of your organization. For example org.apache.maven.plugins is the designated groupId for all Maven plugins.
<br> groupId는 프로젝트의 고유한 '도메인'이라고 한다. `org.apache.Maven`처럼 package명 스럽게 지어야 한다고 한다. -> 추후 의존관계와 관련이 있을 것 같다.

### artifactId
> This element indicates the unique base name of the primary artifact being generated by this project. The primary artifact for a project is typically a JAR file. Secondary artifacts like source bundles also use the artifactId as part of their final name. A typical artifact produced by Maven would have the form `<artifactId>-<version>.<extension>` (for example, myapp-1.0.jar).
<br> 생성될 jar 파일의 이름이라고 생각하면 될 것 같다. 앱을 실행하게 될 파일이니 unique 하고 알아보기 쉽도록 하면 좋을 것 같다. `<artifactId>-<version>.<extension>` 추후에 이렇게 파일이름이 된다.

### packaging 
> This element indicates the package type to be used by this artifact (e.g. JAR, WAR, EAR, etc.). This not only means if the artifact produced is JAR, WAR, or EAR but can also indicate a specific lifecycle to use as part of the build process. (The lifecycle is a topic we will deal with further on in the guide. For now, just keep in mind that the indicated packaging of a project can play a part in customizing the build lifecycle.) The default value for the packaging element is JAR so you do not have to specify this for most projects.
<br> 나중에 이 프로젝트를 어떻게 묶어서 제공할 것인가의 단위이다. WAR, JAR, EAR 등이 있는데, 쉽게 생각하면 이 프로젝트를 압축해서 하나로 만드는 것이다. zip, tar 처럼 애플리케이션을 war, jar 등의 규격으로 압축한다.

### version 
> This element indicates the version of the artifact generated by the project. Maven goes a long way to help you with version management and you will often see the SNAPSHOT designator in a version, which indicates that a project is in a state of development. We will discuss the use of snapshots and how they work further on in this guide.
<br> 말그대로 버전인데, SANPSHOT이란건 앱이 개발단계라는 것을 뜻한다. 

### name 
> This element indicates the display name used for the project. This is often used in Maven's generated documentation.
<br> 메이븐 패키징 후에 maven documentation에 들어갈 프로젝트의 name 이라는 것 같다. artifactId랑 비슷하게 가면 될 것 같다. 

### url 
> This element indicates where the project's site can be found. This is often used in Maven's generated documentation.
<br> 역시 패키징 후 documentation에 들어갈 값이라고 한다. 

### description 
> This element provides a basic description of your project. This is often used in Maven's generated documentation.
<br> 역시 패키징 후 documentation에 들어갈 프로젝트에 대한 설명인 것 같다.

<br><br>

다큐멘테이션을 읽으면 항상 만든 사람의 의도를 이해하려 노력하게 된다.(~~물론 이해한건아니다.~~)<br>
artifactid, groupid, version 이외에도 다른 요소가 많고 생각보다 pom.xml이 매우 중요함을 인지하게 되었다. <br>

