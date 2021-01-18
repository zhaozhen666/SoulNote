## Soul的Sofa插件

关于sofa代理的插件，从开始就碰到了问题。

* 以为可以类似于之前的http一样直接启动就可以了，没想到还是有问题。需要在soul-bootstrap中加入 sofa插件的依赖
```
<dependency>
           <groupId>com.alipay.sofa</groupId>
           <artifactId>sofa-rpc-all</artifactId>
           <version>5.7.6</version>
       </dependency>
       <dependency>
           <groupId>org.apache.curator</groupId>
           <artifactId>curator-client</artifactId>
           <version>4.0.1</version>
       </dependency>
       <dependency>
           <groupId>org.apache.curator</groupId>
           <artifactId>curator-framework</artifactId>
           <version>4.0.1</version>
       </dependency>
       <dependency>
           <groupId>org.apache.curator</groupId>
           <artifactId>curator-recipes</artifactId>
           <version>4.0.1</version>
       </dependency>
       <dependency>
           <groupId>org.dromara</groupId>
           <artifactId>soul-spring-boot-starter-plugin-sofa</artifactId>
           <version>${last.version}</version>
       </dependency>
```
添加了之后，开启sofa插件，后台仍然出现错误
![file](http://cdn.kobefan.cn/FmlX5qZxBHBmAZH_Ie7_UqoOisk9)
![file](http://cdn.kobefan.cn/FvpPxwJAREIxpmVWpXk-7KIINlbK)
断点调试之后，发现
![file](http://cdn.kobefan.cn/FscnAaezxzqs-TzPBAQv452wH-2p)
检查选择器无法选择到的错误是在检查divide,springcloud和dubbo中产生的。
在soul-bootstrap中已引入了许多插件的代码为什么soul插件需要单独引入。
在多次尝试没有成功之后，重新拉取代码。删除原有的创建的soul数据库，按照先启动soul-admin，开启sofa插件。soul-bootstrap启动，sofa插件示例项目启动的顺序进行操作，sofa插件成功注册到soul网关中。在浏览器中直接访问端口测试正常
![file](http://cdn.kobefan.cn/FoezXOq5HQdFSzbY3RJBlb2E76Ng)

推测之前产生问题的原因
* 数据库因为某项操作产生了脏数据。
* 更改了配置的问题。

使用拉取最新代码之前的代码使用新库重新操作。正常执行。目前推测还是数据库操作有脏数据
所以给自己作为一个初学者的建议还是要规范操作，按顺序来，多参阅文档和其他人的经验，实在不行尽快转换思路使用基础的配置