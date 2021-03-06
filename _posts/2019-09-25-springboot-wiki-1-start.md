---
layout: post
title:  "WIKI - 정보 공유용 WIKI 제작"
tags:
  - springboot
  - wiki
hero: https://source.unsplash.com/collection/228632/
overlay: indigo
---
[환경]  
Spring Tool Suite(STS) 3.9.5 RELEASE  
Spring Boot 2.1.8.RELEASE  
jdk 1.8.0_181  
MySQL 8.0.17  

이번 포스터에서는 연구실에서 사용할 WIKI의 제작 과정을 담으려고 합니다.  
WIKI는 많은 분들이 아시겠지만 지식과 정보를 다른 사람들과 공유하기 위해 사용되는 웹페이지입니다. 이번에 소속 연구실에서 인수인계 등 연구실 내부의 정보를 공유하기 위해 게시판형 WIKI를 제작하게 되었습니다.  
{: .lead}
<!–-break-–>

# 1. 프로젝트 구성

프로젝트에 어떤 기능이 포함되어야 하는지 생각해 보면 WIKI 프로젝트기 때문에 기본적으로 글의 작성, 수정, 삭제가 가능해야 합니다.  
WIKI에서 원하는 정보를 찾아내기 위해 검색 기능도 있어야 합니다. 검색 엔진에 대해서 Elasticsearch와 Hibernate Search를 고려해보겠습니다.  연구실 내에서만 사용하는 WIKI이기 때문에 Elasticsearch는 과한 느낌이 있습니다. 그리고 무엇보다도 Hibernate와 Spring Data Jpa를 사용하고 있다면 Hibernate Search는 비교적 쉽게 구성할 수가 있습니다.  
ORM을 사용하기 위해 Hibernate와 Spring Data JPA를 사용할 것이기 때문에 이것들과 잘 어우러지는 Hibernate Search와 Lucene을 사용하도록 하겠습니다.  
Spring Security를 사용하여 관리자 패이지도 추가하도록 하겠습니다. Build Tool로는 Gradle을 사용하도록 하겠습니다. DB는 운영 서버에서 기존에 사용 중인 MySQL을 사용하겠습니다.  
Spring에서 많이 사용하는 MVC 패턴을 사용할 것인데 View단을 Spring에서 사용되는 템플릿 중 하나인 Thymeleaf를 사용하도록 하겠습니다.  
Thymeleaf는 Java 기반으로 만들어진 server-side 템플릿입니다. Thymeleaf는 Thymeleaf Layout Dialect를 사용해 Layout을 쉽게 구성할 수 있습니다.  

--------------------------------------------------------------

# 2. Spring Boot 프로젝트 생성

New Project에서 Spring Starter Project를 선택해줍니다.  
아래와 같이 Type 부분을 Gradle로 선택해주고 나중에 운영 서버에 War 형태로 배포할 것이기 때문에 Packaging 부분을 War로 선택해줍니다.

![Alt text](/uploads/wiki/wiki_1_new_project.PNG)

New Spring Starter Project Dependencies 단계에서 기본적으로 추가할 수 있는 라이브러리들을 선택해줍니다.

![Alt text](/uploads/wiki/wiki_1_new_dependencies.PNG)

프로젝트 생성이 완료되면 아래와 같은 구조의 프로젝트가 생성될 것입니다.
<pre><code>src/main/java
    com/hoyy/test
        ServletInitializer.java
        TestApplication.java
</code></pre>

**ServletInitializer**는 프로젝트를 생성 시 Packaging 방법을 War로 선택했을 때 생기는 파일로 나중에 배포 시에 사용되게 됩니다. 일단은 무시하도록 하겠습니다.  
참고로 mysql-connector-java는 버전을 명시하지 않을 경우 자동으로 8.0.17 버전으로 생성된다. 8.0.17 버전은 MySQL 5.6 버전부터 지원이 되니 MySQL 버전이 낮을 경우 수동으로 추가해주어야 합니다.  
다음 포스팅에서는 Spring Security를 통한 보안 기능을 다뤄보도록 하겠습니다.  
