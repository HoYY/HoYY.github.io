---
layout: post
title:  "WIKI - Thymeleaf 문법 알아보기"
tags:
  - springboot
  - wiki
  - thymeleaf
hero: https://source.unsplash.com/collection/582860/
overlay: darkmagenta
---
[환경]  
Spring Tool Suite(STS) 3.9.5 RELEASE  
Spring Boot 2.1.8.RELEASE  
jdk 1.8.0_181  
MySQL 8.0.17  

이번 포스터에서는 Spring Boot에서 공식 지원하는 템플릿 엔진 중 하나인 Thymeleaf의 문법 중에 WIKI 프로젝트에서 사용한 문법들을 다뤄보도록 하겠습니다. Thymeleaf는 서버에서 템플릿에 따라 html을 그려서 클라이언트에 전달해주는 서버 사이드 템플릿 엔진입니다.

{: .lead}
<!–-break-–>

# 1. 동적 Path 표기

Thymeleaf에서는 'script 태그의 src 속성', 'link 태그의 href 속성', 'a 태그의 href 속성', 'form 태그의 action 속성' 등 경로가 들어가는 속성에 Controller에서 보내준 데이터를 포함시킬 수 있습니다. 방법은 아래와 같습니다.  
<pre><code>&lt;a th:href="@{/boards/{boardNo}(boardNo=${board.boardNo})}"&gt;
&gt;&gt;&gt; &lt;a href="/boards/3">
</code></pre>

--------------------------------------------------------------

# 2. html 태그 value에 데이터 적용

페이지를 만들다 보면 input 태그나 td 태그 등 여러 가지 태그에 Controller에서 보내준 데이터들을 넣어야 하는 경우가 있습니다.  
Thymeleaf에서는 각각의 태그에 맞게 이러한 경우를 처리하는 방법이 있습니다.  
<pre><code>td 태그 (th:text)
&gt;&gt;&gt; &lt;td th:text="${board.title}"&gt;&lt;/td&gt;

input 태그 (th:value)
&gt;&gt;&gt; &lt;input type="text" name="tilte" th:value="${board.title}"/&gt;

div 태그 (th:utext)
&gt;&gt;&gt; &lt;div th:utext="${board.htmlcode}"&gt;&lt;/div&gt;

h 태그 (th:text)
&gt;&gt;&gt; &lt;h3 th:text="${board.title}"&gt;&lt;/h3&gt;

select 태그의 option 태그 (th:text)
&gt;&gt;&gt; &lt;option selected th:text="${board.title}"&gt;&lt;/option&gt;
</code></pre>

--------------------------------------------------------------

# 3. foreach 문 (반복문)

반복문은 **'th:each'**를 사용한다. index도 사용이 가능합니다.
<pre><code>&lt;tr th:each="board,index : ${boards}"&gt;
	&lt;td scope="row" th:text="${index.index + 1}"&gt;&lt;/td&gt;
	&lt;td th:text="${board.title}"&gt;&lt;/td&gt;
	&lt;td th:text="${board.writer}"&gt;&lt;/td&gt;
	&lt;td th:text="${board.date}"&gt;&lt;/td&gt;
&lt;/tr&gt;
</code></pre>

--------------------------------------------------------------

# 4. if, else 문 (조건문)

Thymeleaf에서는 조건문 또한 존재한다. **여기서 주의해야 할 점은 if 문과 else 문의 조건식이 같아야 한다는 점이다.**
<pre><code>&lt;div th:if="${#strings.isEmpty(baseUrl) || message == 'true'}"&gt;
	참
&lt;/div&gt;
&lt;div th:unless="${#strings.isEmpty(baseUrl) || message == 'true'}"&gt;
	거짓
&lt;/div&gt;
</code></pre>

--------------------------------------------------------------

# 5. Enum 비교

Thymeleaf에서 enum을 비교하느라 꽤 골치가 아팠습니다. 다른 분들이 삽질을 덜 하게끔 글을 적어봅니다.  
Controller에서 보낸 enum 타입을 사용하기 위해선 다음의 형태로 코드를 작성해야 합니다.  
**T(package를 포함한 enum 객체 경로).enum_value**
<pre><code>&lt;select&gt;
	&lt;option selected th:text="${authority}"&gt;&lt;/option&gt;
	&lt;option th:text="${authority == T(com.hoyy.test.models.Account.Role).ROLE_CLIENT ? 'ROLE_ADMIN' : 'ROLE_CLIENT'}"&gt;&lt;/option&gt;
&lt;/select&gt;
</code></pre>

--------------------------------------------------------------

# 6. html 태그 js 속성에 데이터 적용

여러 html 태그의 onclick 속성에는 js가 들어갑니다. 이런 onclick 속성에서 Controller가 보내준 데이터를 사용하는 방법은 아래와 같습니다.  
<pre><code>&lt;button type="button" th:onclick="'location.href=\'' + @&#123;&#123;baseUrl}/boards/edit/{boardNo}(baseUrl=${baseUrl}, boardNo=${board.boardNo})} + '\';'"&gt;클릭&lt;/button&gt;
또는
&lt;a href="#" th:onclick="signout(&#91;&#91;${baseUrl}&#93;&#93;);"&gt;Sign out&lt;/a&gt;
</code></pre>

위와 같이 js에 Thymeleaf 변수를 사용하려면 [[]] 이렇게 대괄호 두개로 감싸줘야 합니다. 대괄호 두개로 감싸지 않으면 "${baseUrl}"과 같이 변수명이 text로 박히는 걸 확인할 수 있을 것입니다.

--------------------------------------------------------------

# 7. 자바스크립트 태그 안에서 Thymeleaf 변수 사용하기

이 부분에 대해서는 아래 블로그 글을 참고하시기 바랍니다.
<https://cnpnote.tistory.com/entry/SPRING-Thymeleaf%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-Spring-%EB%AA%A8%EB%8D%B8%EC%97%90%EC%84%9C-javascript-%EB%B3%80%EC%88%98-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0>  

----
[참고 사이트]  
<https://shlee0882.tistory.com/134>  
<https://cnpnote.tistory.com/entry/SPRING-Thymeleaf%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-Spring-%EB%AA%A8%EB%8D%B8%EC%97%90%EC%84%9C-javascript-%EB%B3%80%EC%88%98-%EC%84%A4%EC%A0%95%ED%95%98%EA%B8%B0>