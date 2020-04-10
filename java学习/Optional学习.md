

# 使用Optional摆脱NullPointerException的折磨  

在目前的工作中，我对Java中的Stream和Lambda表达式都使用得很多，之前也写了两篇文章来总结对应的知识。 

不过对于Optional这个特性，一直没有很好地使用起来，所以最近又开始阅读《Java 8实战》这本书，本文是针对其中第10章的一个学习总结。 

## **一、背景** 

在Java中，如果你尝试对null做函数调用，就会引发NullPointerException（NPE），NPE是Java程序开发中的最典型的异常，对于Java开发者来说，无论你是初出茅庐的新人和还工作多年的老司机，NPE经常让他们翻车。为了避免NPE，他们会加很多if判断语句，使得代码的可读性变得很差。 

从软件设计的角度来看，null本身是没有意义的语义，这是一种对缺失变量值的错误的建模。 

从Java类型系统的角度看，null可以被赋值给任何类型的变量，并且不断被传递，知道最后谁也不知道它是从哪里引入的。 

## **二、Optional的引入** 

Java设计者从Haskell和Scala中获取灵感，在Java 8中引入了一个新的类java.util.Optional<T>。如果一个接口返回Optional，可以表示一个人可能有车也可能没有车，这个比简单的返回Car要更明确，阅读代码的人不需要提前准备业务知识。 

Optional的目的就在于此：通过类型系统让你的领域模型中隐藏的知识显式地体现在你的代码中。 

## **三、Optional的使用** 

| **方法**        | **描述**                                                     |
| --------------- | ------------------------------------------------------------ |
| **empty**       | 返回一个空的Optional实例                                     |
| **filter**      | 如果值存在并且满足提供的过滤条件，则返回包含该值的Optional对象；否则就返回一个空的Optional对象 |
| **map**         | 如果值存在，就对该值执行提供的mapping函数调用                |
| **flatMap**     | 如果值存在，就对该值执行提供的mapping函数调用，返回一个Optional类型的值，否则就返回一个空的Optional对象 |
| **ifPresent**   | 如果值存在，就执行使用该值的方法调用，否则什么也不做         |
| **of**          | 将指定值用Optional封装之后返回，如果该值为null，则抛出一个NPE |
| **ofNullable**  | 将指定值用Optional封装之后返回，如果该值为null，则返回一个空的Optional对象 |
| **orElse**      | 如果有值则返回，否则返回一个默认值                           |
| **orElseGet**   | 如果有值则返回，否则返回一个由指定的Supplier接口生成的值(如果默认值的生成代价比较高的话，则适合使用orElseGet方法) |
| **orElseThrow** | 如果有值则返回，否则返回一个由指定的Supplier接口抛出的异常   |
| **get**         | 如果值存在，则返回该值，否则抛出一个NoSuchElementException异常 |
| **isPresent**   | 如果值存在则返回true，否则返回false                          |

上面这张表里列举了Optional的基础API，我这里列举了一些使用的tips： 

- 你可以用**ofNullable**将一个可能为null的对象封装为Optional对象，然后获取值的时候使用**orElse**方法提供默认值； 

```java
//ofNullable方法的使用  
Optional<Car> optCar = Optional.ofNullable(car); 
```

 

- 可以使用**empty**方法创建一个空的Optional对象； 

```java
//empty方法的使用  
Optional<Car> optCar = Optional.empty(); 
```

 

- **of**方法一般不用，不过如果你知道某个值不可能为null，则可以用Optional封装该值，这样它一旦为null就会抛出异常。 

//of方法的使用  

Optional<Car> optCar = Optional.of(car); 

 

- 你可以使用**map**方法从Optional对象中它封装的值中的某个字段的值； 

Optional<Insurance> optInsurance = Optional.ofNullable(insurance);  Optional<String> name = optInsurance.map(Insurance::getName); 

 

- 如果需要连续、层层递进的从某个对象链的末端获取字段的值，则不能全部使用map方法，需要先使用**flatMap**，最后再使用map方法； 

//转换之前  public String getCarInsuranceName(Person person) {   return person.getCar().getInsurance().getName();  }    //转换后  public String getCarInsuranceName(Optional<Person> person) {   return person.flatMap(Person::getCar)          .flatMap(Car::Insurance)          .map(Insurance::getName)          .orElse("Unknown");  } 

 

- Optional中的map、flatMap和filter方法，在概念是与Stream中对应的方法都很类似，区别就在于Optional中的元素至多有一个，算是Stream的一种特殊情况——一种特殊的集合。 

 

- 不要使用**ifPresent**和**get**方法，它们本质上和不适用Optional对象之前的模式相同，都是臃肿的if-then-else判断语句； 

 

- **由于Optional无法序列化，所以在领域模型中，无法将某个字段定义为Optional的**，原因是：Optional的设计初衷仅仅是要支持能返回Optional对象的语法，如果我们希望在域模型中引入Optional，则可以用下面这种替代的方法： 

public class Person {   private Car car;   public Optional<Car> getCarAsOptional() {    return Optional.ofNullable(car);   }  } 

 

- 不要使用基础类型的Optional对象，原因是：基础类型的Optional对象不支持map、flatMap和filter方法，而这些方法是Optional中非常强大的方法。 

 

**四、实战案例** 

 

案例1：使用工具类方法改良可能抛出异常的API 

Java方法处理异常结果的方式有两种： 

- 返回null（或错误码）； 
- 抛出异常， 

例如：Integer.parseInt(String)这个方法——如果无法解析到对应的整型，该方法就抛出一个NumberFormationException，这种情况下我们一般会使用try/catch语句处理异常情况。 

 

一般我们建议将try/catch块单独提取到一个方法中，在这里使用Optional设计这个方法，代码如下。在开发中，可以尝试构建一个OptionalUtility工具类，将这些复杂的try/catch逻辑封装起来。 

public static Optional<Integer> stringToInt(String a) {   try{    return Optional.of(Integer.parseInt(s));   } catch (NumberFormationException e) {    return Optional.empty();   }  } 

 

案例2：综合案例 

现在有个方法，是尝试从一个属性映射中获取某个关键词对应的值，例子代码如下： 

 public static int readDuration(Properties properties, String name) {      String value = properties.getProperty(name);      if (value != null) {        try {          int i = Integer.parseInt(value);          if (i > 0) {            return i;          }        } catch (NumberFormatException e) {          }      }      return 0;    } 

使用Optional的写法后，代码如下所示： 

  public static int readDurationWithOptional(Properties properties, String name) {      return Optional.ofNullable(properties.getProperty(name))        .flatMap(OptionalUtility::stringToInt)        .filter(integer -> integer > 0)        .orElse(0);    } 

程序的步骤逻辑如下： 

1. 如果需要访问的属性值不存在，Properites.getProperty(String)方法的返回值就是一个null，使用noNullable工厂方法就可以将该值转换为Optional对象； 
2. 接下来，可以使用flatMap将一个Optional转换为Optional对象； 
3. 最后使用filter过滤掉负数，然后就可以使用orElse获取属性值，如果拿不到则返回默认值0。 

 

**五、总结** 

使用Optional的思路和Stream相同，都是链式思路，跟数据库查询似的，表达力很强，而且省去了哪些复杂的try/catch和if-then-else方法。在后面的开发中，可以使用Optional设计API，这样可以设计出更安全的接口和方法。 