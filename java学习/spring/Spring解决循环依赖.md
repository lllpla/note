# Spring解决循环依赖

通常来说，如果问Spring内部如何解决循环依赖，一定是单默认的单例Bean中，属性互相引用的场景。
比如几个Bean之间的互相引用：
![title](https://raw.githubusercontent.com/lllpla/img/master/gitnote/2020/04/14/1586873862987-1586873863298.png)