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
- >[博文地址](http.csdn.net/qq_22172133/article/details/81192040) 
- >[JPA官方文档](https://docs.spring.io/spring-data/jpa/docs/2.1.5.RELEASE/reference/html/) 

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

		
CurdRepository	

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


MongoRepository

		@NoRepositoryBean
		public interface MongoRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
		    <S extends T> List<S> saveAll(Iterable<S> var1);

		    List<T> findAll();

		    List<T> findAll(Sort var1);

		    <S extends T> S insert(S var1);

		    <S extends T> List<S> insert(Iterable<S> var1);

		    <S extends T> List<S> findAll(Example<S> var1);

		    <S extends T> List<S> findAll(Example<S> var1, Sort var2);
		}



#### 接口实现

除了SimpleKeyValueRepository的实现之外，还有SimpleMongoRepository、SimpleLdapRepository等的实现  
这里主要是看的是最Mongo的SimpleMongoRepository的接口实现


		public class SimpleMongoRepository<T, ID> implements MongoRepository<T, ID> {
			//定义了Mongo里面的各种操作的方法
		    private final MongoOperations mongoOperations;
		    private final MongoEntityInformation<T, ID> entityInformation;

		    public SimpleMongoRepository(MongoEntityInformation<T, ID> metadata, MongoOperations mongoOperations) {
		        Assert.notNull(metadata, "MongoEntityInformation must not be null!");
		        Assert.notNull(mongoOperations, "MongoOperations must not be null!");
		        this.entityInformation = metadata;
		        this.mongoOperations = mongoOperations;
		    }

		    public <S extends T> S save(S entity) {
		        Assert.notNull(entity, "Entity must not be null!");
		        if(this.entityInformation.isNew(entity)) {
		            this.mongoOperations.insert(entity, this.entityInformation.getCollectionName());
		        } else {
		            this.mongoOperations.save(entity, this.entityInformation.getCollectionName());
		        }

		        return entity;
		    }

		    public <S extends T> List<S> saveAll(Iterable<S> entities) {
		        Assert.notNull(entities, "The given Iterable of entities not be null!");
		        Streamable source = Streamable.of(entities);
		        boolean allNew = source.stream().allMatch((it) -> {
		            return this.entityInformation.isNew(it);
		        });
		        if(allNew) {
		            List result = (List)source.stream().collect(Collectors.toList());
		            this.mongoOperations.insert(result, this.entityInformation.getCollectionName());
		            return result;
		        } else {
		            return (List)source.stream().map(this::save).collect(Collectors.toList());
		        }
		    }

		    public Optional<T> findById(ID id) {
		        Assert.notNull(id, "The given id must not be null!");
		        return Optional.ofNullable(this.mongoOperations.findById(id, this.entityInformation.getJavaType(), this.entityInformation.getCollectionName()));
		    }

		    public boolean existsById(ID id) {
		        Assert.notNull(id, "The given id must not be null!");
		        return this.mongoOperations.exists(this.getIdQuery(id), this.entityInformation.getJavaType(), this.entityInformation.getCollectionName());
		    }

		    public long count() {
		        return this.mongoOperations.getCollection(this.entityInformation.getCollectionName()).count();
		    }

		    public void deleteById(ID id) {
		        Assert.notNull(id, "The given id must not be null!");
		        this.mongoOperations.remove(this.getIdQuery(id), this.entityInformation.getJavaType(), this.entityInformation.getCollectionName());
		    }

		    public void delete(T entity) {
		        Assert.notNull(entity, "The given entity must not be null!");
		        this.deleteById(this.entityInformation.getRequiredId(entity));
		    }

		    public void deleteAll(Iterable<? extends T> entities) {
		        Assert.notNull(entities, "The given Iterable of entities not be null!");
		        entities.forEach(this::delete);
		    }

		    public void deleteAll() {
		        this.mongoOperations.remove(new Query(), this.entityInformation.getCollectionName());
		    }

		    public List<T> findAll() {
		        return this.findAll(new Query());
		    }

		    public Iterable<T> findAllById(Iterable<ID> ids) {
		        return this.findAll(new Query((new Criteria(this.entityInformation.getIdAttribute())).in((Collection)Streamable.of(ids).stream().collect(StreamUtils.toUnmodifiableList()))));
		    }

		    public Page<T> findAll(Pageable pageable) {
		        Assert.notNull(pageable, "Pageable must not be null!");
		        Long count = Long.valueOf(this.count());
		        List list = this.findAll((new Query()).with(pageable));
		        return new PageImpl(list, pageable, count.longValue());
		    }

		    public List<T> findAll(Sort sort) {
		        Assert.notNull(sort, "Sort must not be null!");
		        return this.findAll((new Query()).with(sort));
		    }

		    public <S extends T> S insert(S entity) {
		        Assert.notNull(entity, "Entity must not be null!");
		        this.mongoOperations.insert(entity, this.entityInformation.getCollectionName());
		        return entity;
		    }

		    public <S extends T> List<S> insert(Iterable<S> entities) {
		        Assert.notNull(entities, "The given Iterable of entities not be null!");
		        List list = (List)Streamable.of(entities).stream().collect(StreamUtils.toUnmodifiableList());
		        if(list.isEmpty()) {
		            return list;
		        } else {
		            this.mongoOperations.insertAll(list);
		            return list;
		        }
		    }

		    public <S extends T> Page<S> findAll(Example<S> example, Pageable pageable) {
		        Assert.notNull(example, "Sample must not be null!");
		        Assert.notNull(pageable, "Pageable must not be null!");
		        Query q = (new Query((new Criteria()).alike(example))).with(pageable);
		        List list = this.mongoOperations.find(q, example.getProbeType(), this.entityInformation.getCollectionName());
		        return PageableExecutionUtils.getPage(list, pageable, () -> {
		            return this.mongoOperations.count(q, example.getProbeType(), this.entityInformation.getCollectionName());
		        });
		    }

		    public <S extends T> List<S> findAll(Example<S> example, Sort sort) {
		        Assert.notNull(example, "Sample must not be null!");
		        Assert.notNull(sort, "Sort must not be null!");
		        Query q = (new Query((new Criteria()).alike(example))).with(sort);
		        return this.mongoOperations.find(q, example.getProbeType(), this.entityInformation.getCollectionName());
		    }

		    public <S extends T> List<S> findAll(Example<S> example) {
		        return this.findAll(example, Sort.unsorted());
		    }

		    public <S extends T> Optional<S> findOne(Example<S> example) {
		        Assert.notNull(example, "Sample must not be null!");
		        Query q = new Query((new Criteria()).alike(example));
		        return Optional.ofNullable(this.mongoOperations.findOne(q, example.getProbeType(), this.entityInformation.getCollectionName()));
		    }

		    public <S extends T> long count(Example<S> example) {
		        Assert.notNull(example, "Sample must not be null!");
		        Query q = new Query((new Criteria()).alike(example));
		        return this.mongoOperations.count(q, example.getProbeType(), this.entityInformation.getCollectionName());
		    }

		    public <S extends T> boolean exists(Example<S> example) {
		        Assert.notNull(example, "Sample must not be null!");
		        Query q = new Query((new Criteria()).alike(example));
		        return this.mongoOperations.exists(q, example.getProbeType(), this.entityInformation.getCollectionName());
		    }

		    private Query getIdQuery(Object id) {
		        return new Query(this.getIdCriteria(id));
		    }

		    private Criteria getIdCriteria(Object id) {
		        return Criteria.where(this.entityInformation.getIdAttribute()).is(id);
		    }findInRange(pageable.getOffset(), 
		    pageable.getPageSize(), 
		    pageable.getSort(), 
		    this.entityInformation.getJavaType());

		    private List<T> findAll(@Nullable Query query) {
		        return query == null?Collections.emptyList():this.mongoOperations.find(query, this.entityInformation.getJavaType(), this.entityInformation.getCollectionName());
		    }
		}
