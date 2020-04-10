# springboot + profile不同环境读取不同配置

## **一、具体做法**

 

不同环境的配置设置一个配置文件，例如：dev环境下的配置配置在application-dev.properties中；prod环境下的配置配置在application-prod.properties中。在application.properties中指定使用哪一个文件：

```properties
1. application-dev.properties（dev环境下的配置）profile = dev_envrimont 
2. application-prod.properties（prod环境下的配置）profile = prod_envrimont
3. application.properties
```

```properties
spring.data.mongodb.uri=mongodb://192.168.22.110:27017/myfirstMongodb      #spring.profiles.active      
spring.profiles.active=dev  
```



其中：Controller代码为

```java
   @Autowired  
   private Environment env;  
           
   @RequestMapping("/testProfile")  
   public String testProfile(){  
         return env.getProperty("profile");  
   }  
```

## **二、测试**

1. 上述代码执行后的结果是：

> **dev_envrimont和mongodb://192.168.22.110:27017/myfirstMongodb**

2. 如果application.properties的配置改为：spring.profiles.active=prod，则结果是：     

> **prod_envrimont**

3. 如果application.properties的配置改为：spring.profiles.active=prod，而application.properties中也配置了profile=xxx（不管该配置配置在spring.profiles.active=prod的上方还是下方），这个时候结果是：

> **prod_envrimont**

4. 如果application.properties的配置改为：spring.profiles.active=prod，而application.properties中也配置了profile=xxx（不管该配置配置在spring.profiles.active=prod的上方还是下方），但是application-prod.properties删掉了profile     = prod_envrimont，这个时候结果是：

> **xxx**

## **三、结论**

各个环境公共的配置写在application.properties中各个模块独有的配置配置在自己的application-{xxx}.properties文件中程序读取的时候优先读取application.properties中选中的profile的配置，若读不到才会application.properties去读

## **四、指定外部的配置文件**

有些系统，关于一些**数据库**或其他第三方账户等信息，由于安全问题，其配置并不会提前配置在项目中暴露给开发人员。 

对于这种情况，我们在运行程序的时候，可以通过参数指定一个外部配置文件。 

以 demo.jar 为例，方法如下：

```shell
  java -jar demo.jar --spring.config.location=/opt/config/application.properties  
```

其中文件名随便定义，无固定要求。