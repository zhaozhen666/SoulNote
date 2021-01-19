# Soul插件之SpringCloud

##  com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server

出现这个错误的主要原因是因为。soul-examples里面的springcloud插件项目使用的默认注册中心是eureka,pom文件中也是eureka。即使yml中配置文件修改了也没用。因此需要讲示例中心中的pom依赖修改为nacos的依赖。重启即可解决问题

## 正常启动情况下，被代理接口的访问
注册成功后，被成功代理的接口列表
![file](http://cdn.kobefan.cn/Fg0d5JWXtS1dGuGWcbE5qxuSCLrC)
正常代理的单个接口的访问
![file](http://cdn.kobefan.cn/FgHjDfYQIvOvZDCUkxeTLgPZNv8A)
通配符代理的接口的访问，主要是要符合通配的格式要求
* 如果含有 /** 代表你的整个接口需要被网关代理
另外，在配置文件中如果增加了soul.springcloud.full=true即可代理全部接口
![file](http://cdn.kobefan.cn/FsEMd14znSTZwpRjrFlnU5LojDMt)
可以看到被代理接口增加了超时时间的选择

随后启动多个端口不同的示例项目，发现选择器中并没有注入这些项目，怀疑并没有在网关层做负载均衡，后续源码阶段可以一探究竟

## 另一个可以关注的点：元数据
其他的许多中间件和插件也都有元数据的概念，掌握Soul里的元数据的概念对我们理解其他中间件的元数据的使用应该也会很有帮助。同时，学习使用Soul在元数据操作上的经验
![file](http://cdn.kobefan.cn/Fnt6qEGIu3qhaNbfCR8ANeW7LKpS)

## 另一个可以关注的点：插件处理管理
![file](http://cdn.kobefan.cn/FmKt65Czd4wLBQO6nYqdnBy6bSu7)
看列表数据可以看到这里的字段值对应的某种规则。后续源码解读可以理解这一块的逻辑和处理思路。可以尝试着自己自定义一个这样的插件处理管理的规则

## 问题
* 代理Spring Cloud时没有做负载均衡的操作。负载均衡由springCloud组件来完成