# Spring的事务传播机制

![title](https://raw.githubusercontent.com/lllpla/img/master/gitnote/2020/04/10/1586502982434-1586502982465.png?token=ACTJ35SRZMLEYUBACF6VYLS6SAOY4)

---

<a name="anyFc"></a>
## 经典案例
<a name="AcBms"></a>
### 案例一
这也是大家最常见的情况 即PROPAGATION_REQUIRED

```java
@Transactional
@Override
public void Example1(User user){
	userMapper.insert(user);
    propagationService.required();
}
```

```java
@Transactional
@Override
public void required(){
	throw new NullPointerException("假装抛出了异常");
}

```

单元测试

```java
@Test
public void testExample1(){
    User user = new User();
    user.setName("hello");
    userService.Example1(user);
}
```

---

<a name="wKFlw"></a>
### 案例二 try-required

```java
@Transactional
@Override
public void Example2(User user) {
    userMapper.insert(user);
    try {
        propagationService.required();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

```java
@Transactional
@Override
public void required() {
    throw new NullPointerException("假装抛出了异常");
}
```
单元测试

```java
@Test
public void testExample2() throws Exception {
    User user = new User();
    user.setName("hello");
    userService.Example2(user);
}

```
<a name="0jsas"></a>
### 案例三：try-requiresNew
这个案例和上面很像，区别在于这里的隔离级别是REQUIRES_NEW

```java
@Transactional
@Override
public void Example3(User user){
    userMapper.insert(user);
    try{
    	propagationService.requiresNew();
    }cache(Exception e){
    	e.printStackTrace();
    }
}
```

```java
@Transactional(propagation=Propagation.REQUIRES_NEW)
@Override
public void requiresNew(){
    throw new NullPointerException("一个假异常");
}
```

单元测试
```java
@Test
public void testExample3() throws Exception{
    User user = =new User();
    user.setName("Hello");
    userService.Example3(user);
```
<a name="0WqIR"></a>
### 案例四：常规情况
这个和案例一很像

```java
@Transactional
@Override
public void Example4(User user) {
    userMapper.insert(user);
    propagationService.requiresNew();
}
```

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
@Override
public void requiresNew() {
    throw new NullPointerException("假装抛出了异常");
}
```

单元测试

```java
@Test
public void testExample4() throws Exception {
    User user = new User();
    user.setName("Hello");
    userService.Example4(user);
}
```

---

<a name="QL67Q"></a>
## 原理解析
<a name="t40Tu"></a>
### 案例一插入失败
异常必然会回滚，数据库不会插入数据。
<a name="xAoFE"></a>
### 案例二插入失败
由于抛出了异常，不会插入数据。。

```
org.springframework.transaction.UnexpectedRollbackException: 
Transaction rolled back because it has been marked as rollback-only

```
<a name="pn1hy"></a>
### 案例三插入成功
能正常插入数据
<a name="NCZyv"></a>
### 案例四插入失败
不能插入数据
<a name="2Gl7t"></a>
### 原理
先来看看`@Transactional`的核心方法

```java
	if (txAttr == null || !(tm instanceof CallbackPreferringPlatformTransactionManager)) {
		// 开启事务
		TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
		Object retVal = null;
		try {
			// 执行业务方法
			retVal = invocation.proceedWithInvocation();
		}
		catch (Throwable ex) {
			// 回滚事务
			completeTransactionAfterThrowing(txInfo, ex);
			throw ex;
		}
		finally {
			cleanupTransactionInfo(txInfo);
		}
		// 提交事务
		commitTransactionAfterReturning(txInfo);
		return retVal;
	}
```
我们来重点看看案例二和案例三的情况：<br />首先案例二，在执行 `Example2` 方法的时候，由上下文得知，即将开启一个事务，再执行到 `required` 方法时，此时，因为用的是默认的隔离级别，因此这个时候会加入到刚才开启的事务之中，然后在 `requied` 方法中出现了异常。
