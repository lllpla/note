# 什么是适配器模式

定义：将一个系统的接口转换成另外一种形式，从而使原来不能直接调用的接口变得可以调用。

#适配器模式中的角色

1. 源（Adaptee）：需要被适配的对象或者类型，相当于插头。
2. 适配器（Adapter）：连接目标和源的中间对象，相当于插头转换器。
3. 目标（Target）：期待得到的目标，相当于插座。

# 适配器的三种形式

1. 对象适配器
2. 类适配器
3. 接口适配器

# 适配器的应用场景

1. 新老版本接口的兼容
2. Mybatis多种日志框架的整合。

#适配器的创建方式

1. 对象适配器（组合模式）
2. 类适配器（继承模式）

#一个例子

比如早期的时候V1版本的订单接口的入参都为Map类型，随着业务的更新和迭代，在V2版本的时候该订单接口支持传入List类型。在不改变接口代码的情况下，如何支持List类型。

## 1.源

```java
public void froOrderMap(Map map){
    for(int i=0; i<map.size(); i++){
        String value = (String) map.get(i);
        System.out.println("value:"+value);
    }
}
```

## 2.目标

```java
public interface List<E> extends Collection<E>{
    ……
    int size();
    E get(int index);
    E set(int index; E element);
}
```

## 3.适配器

```java
pul
```

