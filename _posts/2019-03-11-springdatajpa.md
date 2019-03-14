---
layout: post
title: 源码阅读SpringDataJPA
subtitle: 源码阅读SpringDataJPA
author: ymkNK
date: 2019-03-11 19:28:16 +0800
categories: Tech
tag: Spring
img: 5.jpg
---
# 前言
CRUD是平常业务开发过程中最常接触到的，因此想通过阅读这个最常接触模块的代码，使得自己能够更加深入的了解Spring，提升自己的技术能力，而不是只会CRUD，却都不知道它是怎么实现的。

#### Hibernate

>Hibernate是一个开放源代码的对象关系映射框架，它对JDBC进行了非常轻量级的对象封装，它将POJO与数据库表建立映射关系，是一个全自动的orm（Object Relational Mapping）框架，hibernate可以自动生成SQL语句，自动执行，使得Java程序员可以随心所欲的使用对象编程思维来操纵数据库。Hibernate可以应用在任何使用JDBC的场合，既可以在Java的客户端程序使用，也可以在Servlet/JSP的Web应用中使用，完成数据持久化的重任。[Hibernate百科](https://baike.baidu.com/item/Hibernate/206989?fr=aladdin)

#### JPA
>JPA诞生的缘由是为了整合第三方ORM框架，建立一种标准的方式，百度百科说是JDK为了实现ORM的天下归一，目前也是在按照这个方向发展，但是还没能完全实现。在ORM框架中，Hibernate是一支很大的部队，使用很广泛，也很方便，能力也很强，同时Hibernate也是和JPA整合的比较良好，我们可以认为JPA是标准，事实上也是，JPA几乎都是接口，实现都是Hibernate在做，宏观上面看，在JPA的统一之下Hibernate很良好的运行。  
我们都知道，在使用持久化工具的时候，一般都有一个对象来操作数据库，在原生的Hibernate中叫做Session，在JPA中叫做EntityManager，在MyBatis中叫做SqlSession，通过这个对象来操作数据库。我们一般按照三层结构来看的话，Service层做业务逻辑处理，Dao层和数据库打交道，在Dao中，就存在着上面的对象。那么ORM框架本身提供的功能有什么呢？答案是基本的CRUD，所有的基础CRUD框架都提供，我们使用起来感觉很方便，很给力，业务逻辑层面的处理ORM是没有提供的，如果使用原生的框架，业务逻辑代码我们一般会自定义，会自己去写SQL语句，然后执行。在这个时候，Spring-data-jpa的威力就体现出来了，ORM提供的能力他都提供，ORM框架没有提供的业务逻辑功能Spring-data-jpa也提供，全方位的解决用户的需求。使用Spring-data-jpa进行开发的过程中，常用的功能，我们几乎不需要写一条sql语句。  
>[博文地址](http.csdn.net/qq_22172133/article/details/81192040)  

# 应用实例
在大家的日常开发中都用到了很多，再次就不细致列举了，详情可以查看另外一篇博文
>[Spring开发的小细节（六）](https://lllovol.com/spring6/)  

# 源码解析
 
## Repository

		@Indexed
		public interface Repository<T, ID> {
		}


## CrudRepository


#### 主要接口

		public interface CrudRepository<T, ID> extends Repository<T, ID> {
		    <S extends T> S save(S var1);

		    <S extends T> Iterable<S> saveAll(Iterable<S> var1);

		    Optional<T> findById(ID var1);

		    boolean existsById(ID var1);

		    Iterable<T> findAll();

		    Iterable<T> findAllById(Iterable<ID> var1);

		    long count();

		    void deleteById(ID var1);

		    void delete(T var1);

		    void deleteAll(Iterable<? extends T> var1);

		    void deleteAll();
		}


#### 接口实现

除了SimpleKeyValueRepository的实现之外，还有SimpleMongoRepository、SimpleLdapRepository等的实现  
这里主要是看的是最基础的SimpleKeyValueRepository的接口实现


		public class SimpleKeyValueRepository<T, ID> implements KeyValueRepository<T, ID> {
			//这个类主要定义了各种各样的操作的方法，在SimpleMongoRepository、SimpleLdapRepository里面就是MongoOperations、LdapOperation
			//?成员变量可以定义为函数？
		    private final KeyValueOperations operations;
		    //定义了实体的详细，
		    private final EntityInformation<T, ID> entityInformation;

		    public SimpleKeyValueRepository(EntityInformation<T, ID> metadata, KeyValueOperations operations) {
		        Assert.notNull(metadata, "EntityInformation must not be null!");
		        Assert.notNull(operations, "KeyValueOperations must not be null!");
		        this.entityInformation = metadata;
		        this.operations = operations;
		    }

		    public Iterable<T> findAll(Sort sort) {
		        return this.operations.findAll(sort, this.entityInformation.getJavaType());
		    }

		    public Page<T> findAll(Pageable pageable) {
		        Assert.notNull(pageable, "Pageable must not be null!");
		        if(pageable.isUnpaged()) {
		            List content1 = this.findAll();
		            return new PageImpl(content1, Pageable.unpaged(), (long)content1.size());
		        } else {
		            Iterable content = this.operations.findInRange(pageable.getOffset(), pageable.getPageSize(), pageable.getSort(), this.entityInformation.getJavaType());
		            return new PageImpl(IterableConverter.toList(content), pageable, this.operations.count(this.entityInformation.getJavaType()));
		        }
		    }

		    public <S extends T> S save(S entity) {
		        Assert.notNull(entity, "Entity must not be null!");
		        if(this.entityInformation.isNew(entity)) {
		            this.operations.insert(entity);
		        } else {
		            this.operations.update(this.entityInformation.getRequiredId(entity), entity);
		        }

		        return entity;
		    }

		    public <S extends T> Iterable<S> saveAll(Iterable<S> entities) {
		        entities.forEach(this::save);
		        return entities;
		    }

		    public Optional<T> findById(ID id) {
		        return this.operations.findById(id, this.entityInformation.getJavaType());
		    }

		    public boolean existsById(ID id) {
		        return this.findById(id).isPresent();
		    }

		    public List<T> findAll() {
		        return IterableConverter.toList(this.operations.findAll(this.entityInformation.getJavaType()));
		    }

		    public Iterable<T> findAllById(Iterable<ID> ids) {
		        ArrayList result = new ArrayList();
		        ids.forEach((id) -> {
		            this.findById(id).ifPresent(result::add);
		        });
		        return result;
		    }

		    public long count() {
		        return this.operations.count(this.entityInformation.getJavaType());
		    }

		    public void deleteById(ID id) {
		        this.operations.delete(id, this.entityInformation.getJavaType());
		    }

		    public void delete(T entity) {
		        this.deleteById(this.entityInformation.getRequiredId(entity));
		    }

		    public void deleteAll(Iterable<? extends T> entities) {
		        entities.forEach(this::delete);
		    }

		    public void deleteAll() {
		        this.operations.delete(this.entityInformation.getJavaType());
		    }
		}




