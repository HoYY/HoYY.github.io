---
layout: post
title:  "WIKI - Spring Security 적용"
tags:
  - springboot
  - wiki
hero: https://source.unsplash.com/collection/145103/
overlay: navy
---
[환경]  
Spring Tool Suite(STS) 3.9.5 RELEASE  
Spring Boot 2.1.8.RELEASE  
jdk 1.8.0_181  
MySQL 8.0.17  

이번 포스터에서는 Spring Security를 적용하는 방법을 다뤄보도록 하겠습니다.  
Spring Security는 사용자 인증이나, 권한 체크, CSRF(Cross Site Request Forgery) 공격을 막아주는 등 보안에 관련된 여러 기능을 제공해주는 프레임 워크입니다.  

{: .lead}
<!–-break-–>

# 1. DB 설정

Spring Security를 설정하기 앞서 src/main/resources 아래에 있는 application.properties를 통해 손쉽게 DB 설정을 하도록 하겠습니다.  
<pre><code>spring.jpa.database=mysql
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
spring.datasource.url=jdbc:mysql://localhost:3306/wiki?useUnicode=true&characterEncoding=utf8&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=1234
</code></pre>

spring.jpa.hibernate.ddl-auto=update로 할 경우 프로젝트가 빌드될때 DB의 스키마와 @Entity 어노테이션이 명시된 Model 클래스들을 비교하여 update가 필요한 부분만 새롭게 추가해주게 됩니다.  
이러한 기능을 사용하면 Table들을 수동으로 만들지 않아도 빌드 시 자동으로 만들어지게 할 수 있습니다.  
다른 설정 값으로는 create, create-drop, none 등이 있다. none으로 설정할 경우 ddl-auto 기능을 비활성화 시키기 때문에 Table들을 수동으로 만들어주어야 합니다. create와 create-drop으로 설정하면 SessionFactory가 시작될때 drop을 실행하기 때문에 프로젝트가 빌드될때마다 DB가 초기화되고 다시 Table들이 만들어지게 됩니다. DB가 초기화되면 안되는 프로젝트의 경우 create로 설정하지 않는 것을 추천합니다.  
MySQL 버전이 5 버전대이면 spring.jpa.properties.hibernate.dialect의 설정 값을 org.hibernate.dialect.MySQL5Dialect로 하시기 바랍니다. 참고로 org.hibernate.dialect.MySQL8Dialect는 Hibernate 버전이 5.3.1 이상이여야 합니다.  
spring.jpa.show-sql=true로 설정할 경우 Hibernate의 쿼리 동작을 로그를 통해 확인할 수 있습니다.

--------------------------------------------------------------

# 2. Spring Security 설정 (WebSecurityConfig)

Controller에서 @RequestMapping 또는 @GetMapping 어노테이션을 사용하여 **"/"** URL을 View단으로 mapping해줘도 접속을 해보면 Spring Security로 인해 **"/login"** 로 redirect 되는것을 확인할 수 있습니다.  
**WebSecurityConfig** 클래스를 만들어 Spring Security에 대한 기본적인 설정을 하고 URL을 관리해보도록 하겠습니다.  
우선 **WebSecurityConfigurerAdapter**를 상속받는 **WebSecurityConfig** 클래스를 생성해주고 **@Configuration, @EnableWebSecurity** 어노테이션을 명시해 줍니다. 다음으로 HttpSecurity를 파라미터로 갖는 configure 메소드를 오버라이드 해줍니다.  

<pre><code>@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Autowired
	private UserDetailsService userDetailsService;
	
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.antMatchers("/login", "/signup", "/account", "/bootstrap/**", "/js/**", "/css/**")
				.permitAll()
				.antMatchers("/admin/**").hasRole("ADMIN")
				.anyRequest().authenticated()
				.and()
			.formLogin()
				.loginPage("/login")
				.loginProcessingUrl("/login")
				.defaultSuccessUrl("/")
				.usernameParameter("id")
				.passwordParameter("password")
				.failureUrl("/login?error")
				.permitAll()
				.and()
			.logout()
				.logoutUrl("/logout")
				.logoutSuccessUrl("/login")
				.invalidateHttpSession(true)
				.permitAll();
	}</code></pre>
    
configure 메소드를 살펴보면 먼저 로그인과 회원가입 기능은 모든 사용자가 접근이 가능해야하기 때문에 permitAll()을 사용해줍니다. 또한 모든 페이지에서 사용되는 정적 파일인 css, js에 대한 접근도 permitAll()을 사용해 모두 허용해줍니다.  
나중에 관리자 페이지를 만들기 위해 **hasRole()**을 사용하여 **"/admin"** URL에 대해서 **"ROLE_ADMIN"** 권한이 있어야만 접근이 가능하게 해줍니다. hasRole("a")을 사용하게 될 경우 기본적으로 앞에 "ROLE_"이 붙어 "ROLE_a" 권한으로 검사하게 됩니다.  
**formLogin()** 부분은 로그인기능을 설정하는 부분입니다. loginPage()는 로그인 페이지의 경로를 설정하는 부분이고, loginProcessingUrl()은 로그인을 백엔드에서 처리하는 경로입니다. **loginPage()를 통해 "/login"으로 로그인 페이지 경로를 설정해줬기 때문에 컨트롤러에서 "/login" request를 자신의 로그인 페이지로 mapping해주어야 합니다.** loginProcessingUrl()의 경로와 login 페이지에서 백엔드로 값을 넘겨주는 경로를(form 태그의 action 등) 같게 설정해야 합니다. defaultSuccessUrl()은 로그인 성공 시 이동할 경로를 뜻한다. usernameParameter()와 passwordParameter()는 로그인 시의 파라미터 name입니다. usernameParameter()의 default 값은 "username" 입니다. form 태그의 값들과 맞게 설정해주시면 됩니다. failureUrl()은 말그대로 로그인 실패시 redirect 될 URL 입니다.  
**logout()** 부분은 로그아웃 기능을 설정하는 부분입니다. logoutUrl()은 로그아웃을 수행할 경로이고 logoutSuccessUrl()은 로그아웃 시 redirect 될 경로입니다. 또한 invalidateHttpSession(true)를 통해 로그아웃시 세션을 없애줍니다.  
  
다음으로 AuthenticationManagerBuilder를 파라미터로 갖는 configure 메소드를 오버라이드 해줍니다. Spring Security에서 많이 사용되는 BCryptPasswordEncoder를 사용하기 위해 PasswordEncoder 메소드를 만들어주고 @Bean 어노테이션을 사용하여 Bean으로 등록해줍니다. 이렇게 Bean으로 등록해주면 나중에 회원가입이나 비밀번호 변경 등에서 @Autowired 어노테이션으로 객체를 주입받아 사용할 수 있습니다.  
**AuthenticationManagerBuilder.userDetailsService()를 통해 Spring Security에서 사용되는 유저 정보를 입맛에 맞게 커스터마이징 할 수 있습니다.**  
이렇게 만들어진 **WebSecurityConfig**의 전체적인 코드는 아래와 같습니다.
<pre><code>package com.hoyy.test.configs;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Autowired
	private UserDetailsService userDetailsService;
	
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.antMatchers("/login", "/signup", "/account", "/bootstrap/**", "/js/**", "/css/**")
				.permitAll()
				.antMatchers("/admin/**").hasRole("ADMIN")
				.anyRequest().authenticated()
				.and()
			.formLogin()
				.loginPage("/login")
				.loginProcessingUrl("/login")
				.defaultSuccessUrl("/")
				.usernameParameter("id")
				.passwordParameter("password")
				.failureUrl("/login?error")
				.permitAll()
				.and()
			.logout()
				.logoutUrl("/logout")
				.logoutSuccessUrl("/login")
				.invalidateHttpSession(true)
				.permitAll();
	}
	
	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth
			.userDetailsService(userDetailsService)
			.passwordEncoder(passwordEncoder());
	}
	
	@Bean
	protected PasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}
}</code></pre>

--------------------------------------------------------------

# 3. Create Account Entity
이제 userDetailsService에서 사용될 Account Entity 클래스를 만들어보도록 하겠습니다.  
@Entity 어노테이션을 명시한 Account 클래스를 생성해줍니다. 자신의 입맛에 맞게 Entity 캘래스를 구성해 주시면 되는데 저는 간단하게 id, password, name, authority로 구성을 하도록 하겠습니다. 소규모의 WIKI 페이지이기 때문에 authority는 Account 당 하나씩만 갖게끔 구성하겠습니다. Getter, Setter를 하나하나 코드로 작성하지 않고 Lombok 라이브러리를 사용할 것인데 자세한 내용은 다음 포스팅에서 다루도록 하겠습니다.  
<pre><code>package com.hoyy.test.models;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.EnumType;
import javax.persistence.Enumerated;
import javax.persistence.Id;
import javax.persistence.Table;
import javax.validation.constraints.NotNull;

import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
public class Account {
	@Id
	@NotNull
	@Column(length=100)
	private String id;
	
	@NotNull
	@Column(length=100)
	private String pw;
	
	@Column(length=20)
	private String name;
	
	@NotNull
	@Column(length=15)
	@Enumerated(EnumType.STRING)
	private Role authority;

	
	public Account() {}
	public Account(String id, String pw, String name) {
		this.id = id;
		this.pw = pw;
		this.name = name;
		authority = Role.ROLE_CLIENT;
	}


	public enum Role{
		ROLE_CLIENT,
		ROLE_ADMIN
	}
}</code></pre>

--------------------------------------------------------------

# 4. Create AccountRepository
Spring Data JPA의 JpaRepository를 사용하여 AccountRepository를 만들어준다. Spring Data Jpa의 JpaRepository를 사용할 경우 간단한 CRUD는 쿼리 작성 없이 사용이 가능하다.  
<pre><code>package com.hoyy.test.repositories;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

import com.hoyy.hpcwiki.models.Account;
import com.hoyy.hpcwiki.models.Account.Role;

@Repository
public interface AccountRepository extends JpaRepository<Account, String> {

}</code></pre>  

--------------------------------------------------------------
# 5. Create UserDetailsImpl
다음으로 User 클래스를 상속받는 UserDetailsImpl 클래스를 만들어줍니다.  
<pre><code>package com.hoyy.test.configs;

import org.springframework.security.core.authority.AuthorityUtils;
import org.springframework.security.core.userdetails.User;

public class UserDetailsImpl extends User {
	public UserDetailsImpl (String id, String password, String authority) {
		super(id, password, AuthorityUtils.createAuthorityList(authority));
	}
}</code></pre>

--------------------------------------------------------------

# 6. Create UserDetailsServiceImpl
마지막으로 UserDetailsService 인터페이스를 구현한 UserDetailsServiceImpl 클래스를 만들어준다.  
<pre><code>package com.hoyy.test.Services;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import com.hoyy.hpcwiki.configs.UserDetailsImpl;
import com.hoyy.hpcwiki.models.Account;
import com.hoyy.hpcwiki.repositories.AccountRepository;

@Service
public class UserDetailsServiceImpl implements UserDetailsService {
	@Autowired
	private AccountRepository accountRepository;
	
	@Override
	public UserDetails loadUserByUsername(String id) throws UsernameNotFoundException {
		Account account = accountRepository.findById(id);
		
		return (new UserDetailsImpl(account.getId(), account.getPw(), account.getAuthority().toString()));
	}
}</code></pre>

loadUserByUsername 메소드를 오버라이드해서 id에 맞는 Account를 select 하여 UserDetailsImpl 인스턴스로 return 해줍니다.  
이렇게 하면 기본적인 Spring Security 설정이 끝나게 됩니다.  
참고로 Post, Put, Delete 방식으로 데이터를 보낼때는 아래와 같이 CSRF 토큰을 같이 보내줘야 합니다.  
<pre><xmp><input th:name="${_csrf.parameterName}" type="hidden" th:value="${_csrf.token}"/></xmp></pre>

---
[참고 사이트]  
<https://pravusid.kr/java/2018/10/10/spring-database-initialization.html>
