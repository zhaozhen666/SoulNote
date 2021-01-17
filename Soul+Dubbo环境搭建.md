## Soul+Dubbo环境搭建
今天一下午，试了几个小时如何搭建环境，发现了如下几个 问题
### 版本不同，无法注册
首先参考芋道源码http://www.iocoder.cn/Soul/install/ 实现了一下dubbo+nacos。但是发现自己复制的2.1.2版本与下载的源码的soul-admin和soul-boostrap的版本不对。项目无法被注册到网关上，这个是个问题。后续希望可以通过看源码能了解甚至解决这个问题
### dubbo版本配置无法读取到it's not a valid config! Please add <dubbo:application name="..." /> to your application config
参考soul-example的xml配置修改为使用yml搭配注解配置出现了上述错误。调整spring版本和dubbo版本均无效果,但观察soul-admin后台可以发现还是注册成功了。应该是dubbo的校验出了问题，（soul开发者群中说是数据库验证的问题，还未完全验证，后续可尝试下）而注册到soul-boostrap的信息并没有问题
![file](http://cdn.kobefan.cn/FlDk5iIPwgQJ_IMonCvC6eLs6_RX)
### 成功版本--完全使用soul-example
soul-example采用的是dubbo+zookeeper,与nacos的方案略有不同
![file](http://cdn.kobefan.cn/FkwhaDHD2m2PzzKQE76DmVk1-5ot)
通过查看规则发现，基本的匹配规则和均衡规则与http的并无不同，这个引发我另一个想法，如果dubbo本省的负载均衡规则和soul的均衡规则同时配置，那么该遵守哪一个规则呢？我们可以后续通过尝试和源码解读来了解
启动之后，我们，可以通过网关代理的接口来访问到dubbo的服务了。
![file](http://cdn.kobefan.cn/FvQOD_6NYceQC7iSuQO_vaYP-s_s)
另外当插件中的zookeeper的端口配置错误时，错误是这样的java.io.IOException: Packet len1213486160 is out of range! 很明显上下文中的端口是nacos的8848我却没有意识到。
![file](http://cdn.kobefan.cn/FqfI3ZJUf9sVnWJoQO9dUaNouBnA)


## 问题
* 版本兼容问题，低版本Soul无法注册到高版本的soul-admin/soulboostrap中
* dubbo的负载均衡规则和soul集成的负载均衡规则的优先级