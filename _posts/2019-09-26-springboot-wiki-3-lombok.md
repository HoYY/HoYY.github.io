---
layout: post
title:  "WIKI - STS Lombok 적용"
tags:
  - springboot
  - wiki
  - sts
hero: https://source.unsplash.com/collection/430471/
overlay: yellow
---
[환경]  
Spring Tool Suite(STS) 3.9.5 RELEASE  
Spring Boot 2.1.8.RELEASE  
jdk 1.8.0_181  
MySQL 8.0.17  

이번 포스터에서는 STS(Spring Tool Suite)에 Lombok을 적용하는 방법을 다뤄보도록 하겠습니다.  
Lombok은 Getter, Setter와 같은 고정적으로 반복되는 코드들을 어노테이션을 통해 손쉽게 생성해주는 라이브러리입니다.  

{: .lead}
<!–-break-–>

# 1. Gradle Dependency 추가

우선 Lombok을 사용하기 위해 gradle.build에 Lombok을 추가해줍니다.  
<pre><code>plugins {
	id 'org.springframework.boot' version '2.1.8.RELEASE'
	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
	id 'java'
	id 'war'
}

group = 'com.hoyy'
version = '0.0.1'
sourceCompatibility = '1.8'
[compileJava, compileTestJava]*.options*.encoding = 'UTF-8'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-security'
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	<u><strong>implementation 'org.projectlombok:lombok'</strong></u>
	runtimeOnly 'mysql:mysql-connector-java'
	providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'org.springframework.security:spring-security-test'
}
</code></pre>

gradle.build에 Lombok을 추가했으면 프로젝트를 우클릭하여 Refresh Gradle Project를 클릭하여 프로젝트의 라이브러리들을 최신화 시켜줍니다.  
이제 Getter, Setter를 추가하고 싶은 클래스에 @Getter, @Setter 어노테이션을 명시해줍니다. 이렇게 해주면 정상적으로 Getter, Setter가 생성돼야 하지만 막상 하게 되면 Getter, Setter가 생성되지 않아 오류가 발생하게 됩니다.  

--------------------------------------------------------------

# 2. STS에 Lombok 적용시키기

1번과 같이 Lombok을 추가하게 되면 정상적으로 동작해야 하지만 이상하게 STS에서는 동작을 안 하는 문제가 있었습니다.  
이 문제를 해결하기 위해선 Lombok을 수동으로 다운받고 STS에 적용을 해야 합니다. 이 부분에 대해서는 아래 블로그를 참고하시기 바랍니다.  
<https://duzi077.tistory.com/142>  
이 글에 나온 대로 따라 하게 되면 Getter, Setter가 정상적으로 생성된 것을 확인할 수 있을 겁니다.

--------------------------------------------------------------

# 3. Gradlew build 시 Lombok 오류
2번을 통해 Lombok을 STS에 성공적으로 추가했지만 아직 문제가 남아있습니다.  
프로젝트를 완성하고 운영 서버에 배포하기 위해선 War 형태로 build 해야 합니다. 하지만 이상하게도 gradlew로 build 시 Getter, Setter를 생성하지 못하며 **Task :compileJava FAILED** 에러가 발생하게 됩니다.  
이 문제를 해결하기 위해선 build.gradle의 dependencies에 **implementation 'org.projectlombok:lombok'** 외에도 **annotationProcessor**를 추가해줘야 합니다. 많은 구글링을 하며 삽질을 했는데 Lombok 홈페이지에 아주 간단하게 나와있었습니다. 역시 공홈이 좋습니다.
<pre><code>plugins {
	id 'org.springframework.boot' version '2.1.8.RELEASE'
	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
	id 'java'
	id 'war'
}

group = 'com.hoyy'
version = '0.0.1'
sourceCompatibility = '1.8'
[compileJava, compileTestJava]*.options*.encoding = 'UTF-8'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-security'
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	<u><strong>compileOnly 'org.projectlombok:lombok'</strong></u>
	<u><strong>annotationProcessor 'org.projectlombok:lombok'</strong></u>
	runtimeOnly 'mysql:mysql-connector-java'
	providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'org.springframework.security:spring-security-test'
}
</code></pre>


---
[참고 사이트]  
<https://duzi077.tistory.com/142>  
<https://projectlombok.org/setup/gradle>