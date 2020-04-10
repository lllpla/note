# guava实现springboot上的AOP

<a name="he9nm"></a>
# 1. 引入guava包的依赖
有没有遇到过这种情况：由于网速等原因，网页响应很慢，提交一次表单后发现服务久久没响应，然后你就疯狂点击提交按钮（12306就经常被这样怒怼），如果做过防重复提交还好，否则那是什么级别的灾难就不好说了。。。<br />今天主要是用 `自定义注解、` `AOP、`· `Guava` 包中Cache来生成一种本地锁，来达到的防重复提交效果，整体的实现比较简单，没有什么太大的难度，代码也是比较少，，由于是基于内存的缓存，**因此这种实现方式并不适用于分布式服务**。旨在给大家介绍一种实现防重复提交的方案。

```java
<dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>21.0</version>
</dependency>
```


<a name="AgSBp"></a>
# 2. 自定义注解
服务器端实现防止重复提交，一般都是利用 `AOP` 自定义注解的方式，作用于 `controller` 的入口方法。自定义一个 `LocalLock` 注解用于防止重复提交的方法上。<br />

```java
/**
* 本地锁
*
*/
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface LocalLock{
	String key() default "";
}
```
注解定义好以后就需要做AOP拦截器切面的具体实现，在 `interceptor()` 方法上采用的是 `Around` (环绕增强),因此所有带 `LocalLock` 注解的都将被当作切面处理；<br />

```java
@Around("execution(public * *(..)) && @annotation(com.xxx.annotation.LocalLock)"
```

<br />既然是缓存，那紧跟着的属性一定要有过期时间，通过 `expireAfterWrite` 设置缓存的过期时间， `maximumSize` 设置缓存的个数。<br />通过在内存中查询key是否存在来判断是否允许再次提交，和 `Redis` 的 `setNX` 方法比较像。<br />这里我们设置同一个方法，5秒钟内相同参数的请求只允许执行一次。<br />**<br />**那么这个注解应该怎么用呢?** <br />

```java
@Aspect
@Configuration
public class LockMethodInterceptor {

    private static final Cache<String, Object> CACHES = CacheBuilder.newBuilder()
        // 最大缓存 100 个
        .maximumSize(1000)
        // 缓存5秒后过期
        .expireAfterWrite(5, TimeUnit.SECONDS)
        .build();
    
    @Around("execution(public * *(..)) && @annotation(com.xxx.annotation.LocalLock)"
    public Object interceptor(ProceedingJoinPoint pjp) {
    	MethodSignature signature (MethodSignature) pjp.getSignature();
        Method method = signature.getMethod();
        LocalLock localLock = method.getAnnotation(LocalLock.class);
        String key getKey(localLock.key(),pjp.getArgs());
        if (!StringUtils.isEmpty(key)) {
        	if (CACHES.getIfPresent(key) != null) {
            	throw new RuntimeException("请勿重复提交请求");
            }
            // 如果是第一次请求 就将
            CACHES.put(key,key);
        }
        try {
            return pjp.proceed();
        }catch (Throwable throwable) {
            throw new RuntimeException("服务器异常")
        }finally{
            // TODO 为了演示效果， 这里就不调用 CACHES.invalidate(key);代码了
        }
        
    }
}
```


<a name="1OcLN"></a>
# 3. 控制层的实现
我们将注解加在控制层方法上， `key="city:arg[0]"`  key自己定义， `arg[0]` 这个匹配规格表示替换成第一个参数。那么就是先了city:token在一定时间内不可以重复提交了。<br />

```java
@RestController
@RequestMapping("/city")
public class BookController {
 
    @LocalLock( key="city:arg[0]")
    @GetMapping
    public String query(@RequestParam String token) {
    	return "ok- " + token;
    }
}
```
<a name="AJ4TG"></a>
# 4. 测试
接下来我们就测试一下，预期结果：5秒内只有第一次提交生效,，其余的显示`“请勿重复提交”`，看看执行结果
