---
layout: post
title:  "WIKI - Spring Boot JPA ManyToMany Mapping"
tags:
  - springboot
  - wiki
  - jpa
hero: https://source.unsplash.com/random
overlay: seagreen
---
[환경]  
Spring Tool Suite(STS) 3.9.5 RELEASE  
Spring Boot 2.1.8.RELEASE  
jdk 1.8.0_181  
MySQL 8.0.17  

이번 포스터에서는 @ManyToMany 어노테이션을 사용하여 두 개의 Entity 사이에 다대다 관계를 mapping 하는 방법을 다뤄보겠습니다.  
DataBase에는 글의 정보를 담고 있는 Board 테이블과 태그 정보를 담고 있는 Tag 테이블이 있습니다. WIKI 프로젝트 특성상 하나의 Board는 여러 개의 Tag를 가질 수 있고, 하나의 Tag 또한 여러 개의 Board에 포함될 수 있습니다. 저는 **@JoinTable** 어노테이션을 사용해서 Board 테이블과 Tag 테이블 사이에 다대다 관계를 성립시켜주는 테이블을 생성하는 방법을 사용할 것입니다. RDB에서 정규화를 신경 쓰면서 다대다 관계를 테이블 두개로 표현하는 방법은 없다고 생각합니다. 물론 @JoinTable과 @ManyToMany 어노테이션을 사용하지 않고 수동으로 테이블들을 Create 해주고 Join 쿼리를 통해 튜플들에 접근해도 됩니다. 하지만 이렇게 할 경우 데이터 무결성에 위배될 수도 있고, Board를 저장하거나 수정할 때 원래면 작성하지 않아도 될 추가적인 코드가 발생할 것입니다.  

{: .lead}
<!–-break-–>

![Alt text](/uploads/wiki/wiki_4_manytomany.PNG)

# 1. Board 클래스

Join Table을 만들어주기 위해 우선 **@ManyToMany, @JoinTable, @JoinColumn** 어노테이션을 사용해서 Board 클래스의 내용을 수정하도록 하겠습니다. @JoinTable 어노테이션에는 **name, joinColumns, inverseJoinColumns**가 인자 값으로 들어가는데 각각 생성될 다대다 테이블의 테이블명, 현재 Entity를 참조할 Foreign Key, 반대 Entity를 참조할 Foreign Key를 의미합니다.
<pre><code>package com.hoyy.test.models;

import java.util.HashSet;
import java.util.Set;

import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.JoinTable;
import javax.persistence.ManyToMany;
import javax.persistence.Table;
import javax.validation.constraints.NotNull;

import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
@Table(name="board")
public class Board {
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	@Column(name="no", length=10)
	private int boardNo;
	
	@ManyToMany(cascade=CascadeType.PERSIST)
	@JoinTable(name="board_tag", joinColumns = {@JoinColumn(name="board_no")},
			inverseJoinColumns = {@JoinColumn(name="tag_no")})
	private Set&lt;Tag&gt; tags = new HashSet&lt;&gt;();
	
	@NotNull
	@Column(length=50)
	private String title;

	public void addTags(Tag tag) {
		tags.add(tag);
	}
}</code></pre>

위와 같이 Board 클래스의 내용을 수정해주면 다대다 관계를 mapping 해줄 board_tag 테이블이 생성되고, board_no와 tag_no Column들이 만들어질 것입니다.  
Primary Key인 boardNo는 **@GeneratedValue(strategy=GenerationType.AUTO)** 어노테이션을 사용하여 튜플이 저장됨에 따라 자동으로 증가하도록 만들었습니다.  
Board에 대한 Tag들을 담는 tags 객체를 HashSet 타입으로 한 이유는 하나의 Board에 같은 Tag가 중복으로 들어가는 것을 막기 위함입니다. List로도 Join 테이블을 만들 수 있습니다.

--------------------------------------------------------------

# 2. Tag 클래스

Board 클래스의 작성이 끝났으니 Tag 클래스의 내용을 수정하도록 하겠습니다.
<pre><code>package com.hoyy.test.models;

import java.util.HashSet;
import java.util.Set;

import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.ManyToMany;
import javax.persistence.Table;
import javax.persistence.UniqueConstraint;
import javax.validation.constraints.NotNull;

import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
@Table(uniqueConstraints={@UniqueConstraint(columnNames={"tag"})})
public class Tag {
	@Id
	@GeneratedValue(strategy=GenerationType.AUTO)
	private int no;
	
	@ManyToMany(cascade=CascadeType.PERSIST, mappedBy="tags")
	private Set&lt;Board&gt; boards = new HashSet&lt;&gt;();
	
	@NotNull
	@Column(name="tag", length=30)
	private String tag;
	
	public Tag() {}
	public Tag(String tag) {
		this.tag = tag;
	}
}</code></pre>

Board 클래스와 Tag 클래스 모두 CascadeType을 PERSIST로 설정했는데, 그 이유는 Board 튜플을 삭제할 때 Tag 테이블의 튜플이 같이 삭제되는 것을 막기 위함입니다.  
tag Column에 유니크 속성을 부여하기 위해 @Table 어노테이션에 **@UniqueConstraint** 어노테이션을 사용했습니다. @ManyToMany의 **mappedBy** 속성을 사용해 방향성을 설정해줬다. mappedBy가 없는 쪽이 관계의 주인이 됩니다.  
위와 같이 설정을 하고 프로젝트를 빌드 해주면 spring.jpa.hibernate.ddl-auto=update 설정으로 인해 자동으로 board, tag, board_tag 테이블이 생성되는 것을 확인할 수 있을 겁니다.  

---
[참고 사이트]  
<https://siyoon210.tistory.com/27>