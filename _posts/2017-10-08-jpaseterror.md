---
layout: post
title: "[JPA]对象在set属性时不提交但更新数据库问题"
date: 2018-10-08 
tag: JPA
---

　使用JPA(Hibernate)遇到的一个问题。从数据库中查出数据进行set操作并没有执行save但是数据库中的数据做了改变。查询资料做出如下修改可解决此问题。

```python
	HibernateEntityManager hEntityManager = (HibernateEntityManager) entityManager;
    Session session = hEntityManager.getSession();
	//在对对象进行操作是先evict
	session.evict(poItem);

```

### 分析

Hibernate有三种基本状态：
 `(1)`自由态（临时状态）：直接new出来的对象，既没有被保存到数据库中，也不处于session缓存中
 `(2)`游离态：已经被保存到数据库中但不处于session缓存中
 `(3)`持久态：已经被保存到数据库中并且加入到session缓存中

上述代码中的对象是持久化状态的对象，对其进行set操作时session缓存中的数据发生了改变，数据库也会跟着进行相应的改变，所以执行了update的更新操作

最简单也是最容易想到的方式就是重新new一个对象然后再去set属性，这个时候因为不是session中的数据，不会因为对象属性发生改变而同步到数据库中

```python
	PoItem poItem = new PoItem();
	BeanUtils.copyProperties(poItem,poItemCopy);
	//数量
	poItemCopy.setNum(num);
	//总价
	poItemCopy.setPrice(price);

```

但如果这个对象要用的到，那么在set之前可以先将其转为游离态，session中提供了几个方法：
close方法：关闭session这样这个对象肯定是游离态了，因为session已经关闭了，但是往往我们实际的开发过程中，session在后面是要用的到的，所以这个方法虽然可行，但也要分场景
clear方法：将session中的所有的对象全部清除出缓存，虽然session清除了全部的对象之后自然就会变为游离态了，但这样做不太合适
evict方法：将某一个对象清除出缓存session，这个方法是很好的实现方式，推荐使用
