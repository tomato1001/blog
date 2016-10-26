---
layout: post
title: "JPA notes"
categories: jpa
date: 2015-06-16 22:19:09
---
> * 使用junit测试时，为单元测试加上事务，避免不必要的错误

### JPA one to many

* 双向映射

{% highlight java %}
public class User{
	
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;

	// mappedBy指定关系持有者(owner)端
	@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true, mappedBy = "user" )
	private List<Address> addressItems;

}

public class Address{
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;

	@ManyToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "user_id")
	private User user;	
}
{% endhighlight %}

* 单向映射

{% highlight java %}
public class User{
	
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;

	@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
	@JoinColumn(name="user_id") //开启自动建表后，将会在Address中建立user_id字段
	private List<Address> addressItems;

}

public class Address{
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;
}
{% endhighlight %}
> * 单向映射会先插入到子表，然后更新对应的OneToMany字段

