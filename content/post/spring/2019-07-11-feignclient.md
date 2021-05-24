---
author: ymkNK
categories: Spring
date: "2019-07-11T16:31:53Z"
img: 8.jpg
subtitle: fallback
tag: Spring
title: FeignClient的FallBack处理
---
# 前言
在使用FeignClient的时候，我们很难保证第三方的调用不会出问题，这个时候，可以写一个FallBack类，来对出错的时候进行处理，可以异常上报，也可以进行打点处理，可以不再单独地去做这些事情。

# FeignClient的编写
1. 添加maven依赖。
```
		<dependency>
		    <groupId>org.springframework.cloud</groupId>
		    <artifactId>spring-cloud-starter-openfeign</artifactId>
		    <version>2.0.2.RELEASE</version>
		</dependency>
		<dependency>
		    <groupId>io.github.openfeign</groupId>
		    <artifactId>feign-core</artifactId>
		    <version>9.7.0</version>
		</dependency>
		<dependency>
		    <groupId>io.github.openfeign</groupId>
		    <artifactId>feign-slf4j</artifactId>
		    <version>9.7.0</version>
		</dependency>
```

2. 新建一个接口，在上方添加FeignClient注解，在value中使用占位符的方式，对参数进行调用。
```
		@FeignClient(name = "gitApi", url = "${api.server.gitlab.url}",fallback = GitApiFeignFallBack.class)
		public interface GitApiFeign {

			@GetMapping(value = "/{namespace}/-/jobs/{jobId}/trace")    
		    JSONObject getTrace(
		            @PathVariable("namespace") String namespace,
		            @PathVariable("jobId") Integer jobId,
		            @RequestParam("private_token") String accessToken);
		}
```

3. @Autowired 注解就能对FeignClient进行使用。
```
@Autowired
GitApiFeign gitApiFeign;
```

# FeignClient的Fallback实现
在上方的代码中，可以看见注解中有fallback的选项，这个就是新建这个Client对应的fallback类来实现FeignClient。  
在fallback类的实现代码大致如下，最简单的实现就是在fallBack类中重写一遍Client中的所有方法。
```
@Component
@Slf4j
public class GitApiFeignFallBack implements GitApiFeign {

    @Override
    public JSONObject getTrace(String namespace, Integer jobId, String accessToken) {
        log.error("调用gitlab api 失败");
        return null;
    }

}

```
