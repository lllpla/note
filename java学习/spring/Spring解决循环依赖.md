# Spring解决循环依赖
# 前言
通常来说，如果问Spring内部如何解决循环依赖，一定是单默认的单例Bean中，属性互相引用的场景。
比如几个Bean之间的互相引用：
![title](https://raw.githubusercontent.com/lllpla/img/master/gitnote/2020/04/14/1586873862987-1586873863298.png)
甚至自己“循环”依赖自己：
![title](https://raw.githubusercontent.com/lllpla/img/master/gitnote/2020/04/14/1586874076407-1586874076413.png)
先说明前提：**原型(Prototype)的场景是不支持循环依赖的**，通常会走到`AbstractBeanFactory`类中下面的判断，抛出异常。
```java
if (isPrototypeCurrentlyInCreation(beanName)) {
  throw new BeanCurrentlyInCreationException(beanName);
}
```
原因很好理解，创建新的A时，发现要注入原型字段B，又创建新的B发现要注入原型字段A...
这就套娃了, 你猜是先`StackOverflow`还是`OutOfMemory？`
Spring怕你不好猜，就先抛出了`BeanCurrentlyInCreationException`
基于构造器的循环依赖，就更不用说了，官方文档说了了，你想让构造器注入支持循环依赖，是不存在的，不如把代码改了。

## 
