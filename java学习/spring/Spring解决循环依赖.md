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

## Spring如何解决循环依赖
首先，**Spring内部维护了三个Map，也就是我们通常说的三级缓存。**
在Spring的`DefaultSingletonBeanRegistry`类中，你会赫然发现类上方挂着这三个Map：

- **singletonObjects** 它是我们最熟悉的朋友，俗称“**单例池**” “**容器**”，缓存创建完成单例Bean的地方。
- **singletonFactories** 映射创建Bean的原始工厂
- **earlySingletonObjects** 映射Bean的**早期引用**，也就是说在这个Map里的Bean不是完整的，甚至还不能称之为“Bean”，只是一个Instance.

后两个Map其实是“垫脚石”级别的，只是创建Bean的时候，用来借助了一下，创建完成就清掉了。

所以笔者前文对“三级缓存”这个词有些迷惑，可能是因为注释都是以Cache of开头吧。

为什么成为后两个Map为垫脚石，假设最终放在singletonObjects的Bean是你想要的一杯“凉白开”。
那么Spring准备了两个杯子，即singletonFactories和earlySingletonObjects来回“倒腾”几番，把热水晾成“凉白开”放到singletonObjects中。


