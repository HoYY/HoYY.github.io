---
layout: post
title:  "WIKI - Thymeleaf Layout Dialect & Markdown js 오픈소스 적용"
tags:
  - springboot
  - wiki
  - thymeleaf
hero: https://source.unsplash.com/collection/427433/
overlay: gray
---
[환경]  
Spring Tool Suite(STS) 3.9.5 RELEASE  
Spring Boot 2.1.8.RELEASE  
jdk 1.8.0_181  
MySQL 8.0.17  

이번 포스터에서는 Thymeleaf Layout Dialect를 사용하여 Layout을 구성하는 법을 다루고, GitHub에 오픈소스로 공개되있는 Markdown js 프로젝트를 사용하여 WIKI 작성 페이지를 만들도록 하겠습니다.

{: .lead}
<!–-break-–>

# 1. Thymeleaf Layout Dialect
## 1-1. build.gradle dependencies 추가

Thymeleaf에서는 Thymeleaf Layout Dialect 라이브러리를 사용하여 쉽게 Layout을 구성할 수 있습니다.  
우선 build.gradle의 dependencies를 추가해줍니다.
<pre><code>dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	<u><strong>implementation 'nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect'</strong></u>
}
</code></pre>

dependencies를 추가했으면 Refresh Gradle Project를 통해 라이브러리를 최신화 해줍니다.

## 1-2. 공통 head 태그 작성

이제 본격적으로 Layout을 작성해보겠습니다. 우선 Layout의 head 부분을 작성해보겠습니다.  
**'cssLayout.html'** 파일을 만들어줍니다. **head** 태그 안에 **th:fragment** 속성을 사용하여 fragment 이름을 설정해줍니다. head 태그 안에 **th:block** 태그를 추가하여 페이지마다 달라지는 내용들을 정의할 수 있습니다.  
참고로 **'charset'**의 경우 charset을 적용하려는 페이지들에 모두 적어주어야 합니다. Layout 파일에 charset을 설정해도 다른 페이지들에는 적용되지 않습니다.  
<pre><code>&lt;!-- cssLayout.html --&gt;
&lt;head th:fragment="cssLayout"&gt;
	&lt;title&gt;HPCLab WIKI&lt;/title&gt;
	&lt;th:block layout:fragment="pageCustomCss"&gt;&lt;/th:block&gt;
	&lt;link rel="stylesheet" th:href="@{/bootstrap/css/bootstrap.min.css}"&gt;
	&lt;link rel="stylesheet" th:href="@{/css/index.css}"&gt;
	&lt;style&gt;
		html, body { height:100%; }
		body { background-color:#F2F2F2; }
	&lt;/style&gt;
&lt;/head&gt;
</code></pre>

## 1-3. mainLayout 작성

이제 'cssLayout.html' 파일을 이용하여 전체적인 Layout을 만들 것입니다. **'mainLayout.html'** 파일을 만들어줍니다. html 태그에 **xmlns:th="http://www.thymeleaf.org"** 속성과 **xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"** 속성을 추가해줍니다. **xmlns:th** 속성을 추가하지 않으면 'th:block'과 같은 Thymeleaf 문법들에 대해서 노란줄로 warning이 표시됩니다.  
head 태그에 **th:replace** 속성을 추가하여 'cssLayout.html' 파일을 불러와 줍니다. 여기서 중요한 점은 'css_layout_path :: fragment_name' 형태로 작성해야 합니다.  
이제 body 부분을 작성해야합니다. Layout의 공통된 body 부분을 적어주시고 아까와 같이 Custom 코드가 들어갈 위치에 **'th:block'** 태그를 추가해 줍니다. body의 아랫부분에 공통된 javascript를 작성해주시고 **'th:block'** 태그를 사용하여 페이지마다 사용할 javascript 코드를 추가할 수 있게 합니다.
<pre><code>&lt;!DOCTYPE html&gt;
<u><strong>&lt;html xmlns:th="http://www.thymeleaf.org" xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout"&gt;</strong></u>
&lt;meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/&gt;
<u><strong>&lt;head th:replace="/layout/cssLayout :: cssLayout"&gt;&lt;/head&gt;</strong></u>
&lt;body&gt;
    &lt;nav class="navbar navbar-expand-lg navbar-dark bg-dark"&gt;
        &lt;span class="navbar-brand pr-4"&gt;&lt;h3 class="text-white"&gt;HPCLab WIKI&lt;/h3&gt;&lt;/span&gt;
        &lt;button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation"&gt;
            &lt;span class="navbar-toggler-icon"&gt;&lt;/span&gt;
        &lt;/button&gt;
	
        &lt;div class="collapse navbar-collapse" id="navbarSupportedContent"&gt;
            &lt;ul class="navbar-nav mr-auto mt-2 mx-2 mb-1"&gt;
                &lt;li class="nav-item active pr-4"&gt;
                    &lt;a class="nav-link h5" th:href="@&#123;&#123;baseUrl&#125;/(baseUrl=$&#123;baseUrl&#125;)&#125;"&gt;Home &lt;/a&gt;
                &lt;/li&gt;
                &lt;li class="nav-item active pr-4"&gt;
                    &lt;a class="nav-link h5" th:href="@&#123;&#123;baseUrl&#125;/boards/new(baseUrl=$&#123;baseUrl&#125;)&#125;"&gt;Write&lt;/a&gt;
                &lt;/li&gt;
                &lt;li class="nav-item dropdown active"&gt;
                    &lt;a class="nav-link dropdown-toggle h5" href="" id="navbarDropdown" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false"&gt;
                        Account
                    &lt;/a&gt;
                    &lt;div class="dropdown-menu" aria-labelledby="navbarDropdown"&gt;
                        &lt;a class="dropdown-item" th:if="${message == 'admin'}" th:href="@&#123;&#123;baseUrl&#125;/accounts(baseUrl=$&#123;baseUrl&#125;)&#125;"&gt;Information&lt;/a&gt;
                        &lt;div class="dropdown-divider"&gt;&lt;/div&gt;
                        &lt;a class="dropdown-item" href="#" th:onclick="signout(&#91;&#91;$&#123;baseUrl&#125;&#93;&#93;);"&gt;Sign out&lt;/a&gt;
                    &lt;/div&gt;
                &lt;/li&gt;
            &lt;/ul&gt;
            &lt;form class="form-inline my-2 my-lg-0" id="keywordForm" th:action="@&#123;&#123;baseUrl&#125;/boards/search(baseUrl=$&#123;baseUrl&#125;)&#125;" style="width:450px;"&gt;
		      	&lt;input class="form-control mr-sm-2" name="keyword" style="width:360px;" type="search" placeholder="Search" aria-label="Search"&gt;
		      	&lt;button class="btn btn-outline-success my-2 my-sm-0" type="submit"&gt;Search&lt;/button&gt;
		    &lt;/form&gt;
	  	&lt;/div&gt;
	&lt;/nav&gt;

  	<u><strong>&lt;th:block layout:fragment="pageCustomBody"&gt;&lt;/th:block&gt;</strong></u>

  	&lt;script src="https://code.jquery.com/jquery-3.2.1.min.js"&gt;&lt;/script&gt;
  	&lt;script th:src="@{/bootstrap/js/bootstrap.bundle.min.js}"&gt;&lt;/script&gt;
  	&lt;script th:src="@{/js/layout.js}"&gt;&lt;/script&gt;
  	
  	<u><strong>&lt;th:block layout:fragment="pageCustomScript"&gt;&lt;/th:block&gt;</strong></u>

&lt;/body&gt;
&lt;/html&gt;
</code></pre>

## 1-4. Layout 적용

마지막으로 mainLayout을 적용할 페이지의 html 태그에 **'layout:decorator'** 속성을 추가하여 mainLayout을 불러와 줍니다. 그리고 페이지에 맞게 **'th:block'** 태그들을 채워주시면 됩니다. 채울 내용이 없으면 해당 위치의 **'th:block'** 태그를 작성하지 않으면 됩니다.  
<pre><code><u><strong>&lt;html xmlns:th="http://www.thymeleaf.org" xmlns:layout="http://www.ultraq.net.nz/web/thymeleaf/layout" layout:decorator="layout/mainLayout"&gt;</strong></u>
&lt;head&gt;
    &lt;meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/&gt;
&lt;/head&gt;
&lt;body&gt;
    <u><strong>&lt;th:block layout:fragment="pageCustomBody"&gt;</strong></u>
    &lt;div class="container-fluid"&gt;
        CustomBody
    &lt;/div&gt;
    <u><strong>&lt;/th:block&gt;</strong></u>
  	
    <u><strong>&lt;th:block layout:fragment="pageCustomScript"&gt;</strong></u>
    &lt;script&gt;
        $(window).scroll(function() &#123; 
            var height = $(document).scrollTop();
            if(height &gt;= 76) &#123; 
                $("#tagArea").addClass("position-fixed"); 
                $("#tagArea").css("top", "0px")
            &#125;
            else if(height &lt; 76) &#123; 
                $("#tagArea").removeClass("position-fixed"); 
            &#125; 
        &#125;);
    &lt;/script&gt;
    <u><strong>&lt;/th:block&gt;</strong></u>
&lt;/body&gt;
&lt;/html&gt;
</code></pre>

위와 같이 Layout에는 head 태그 안에 **&lt;th:block layout:fragment="pageCustomCss"&gt;&lt;/th:block&gt;** 태그가 있지만 딱히 추가할 내용이 없으면 해당 **'th:block'** 태그를 작성하지 않으면 됩니다.  
최종 WIKI 프로젝트의 루트 페이지는 아래와 같습니다.  

![Alt text](/uploads/wiki/wiki_7_homepage.JPG)

--------------------------------------------------------------

# 2. Markdown js 오픈소스 적용

이제 WIKI 게시글 작성 페이지를 만들어야 합니다. 모든 연구실원들이 Markdown에 친숙했기 때문에 Markdown javascript 오픈소스가 GitHub에 있는지 찾아보니 오픈소스가 존재해서 해당 프로젝트를 사용하여 개발하였습니다.  
<https://github.com/jbt/markdown-editor>  
중요하게 볼 부분은 id 속성이 'in', 'out'인 div 태그이다.
<pre><code>&lt;div class="col-6 h-100" id="in"&gt;
    &lt;textarea id="code" name="contents"&gt;# New Document&lt;/textarea&gt;
&lt;/div&gt;
&lt;div id="out" class="markdown-body col-6 h-100"&gt;&lt;/div&gt;
</code></pre>

id 값이 'in'인 div 태그 안의 id 값이 'code'인 textarea 태그에 Markdown 문법의 글을 작성하면 id 값이 'out'인 div 태그에 변환된 결과값이 나오게 됩니다.  
최종 WIKI 프로젝트의 글 작성 페이지와 게시글 조회 페이지는 아래와 같습니다.

![Alt text](/uploads/wiki/wiki_7_writepage.JPG)


----
[참고 사이트]  
<https://www.baeldung.com/thymeleaf-spring-layouts>  
<https://github.com/jbt/markdown-editor>