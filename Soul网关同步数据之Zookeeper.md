# Soul网关同步数据之Zookeeper
## 调整配置
pom文件中注释掉原来默认的websocket同步方式，改为zookeeper同步。
```
        <!--soul data sync start use zookeeper-->
        <dependency>
            <groupId>org.dromara</groupId>
            <artifactId>soul-spring-boot-starter-sync-data-zookeeper</artifactId>
            <version>${project.version}</version>
        </dependency>

        <!--soul data sync start use websocket-->
        <!--<dependency>-->
            <!--<groupId>org.dromara</groupId>-->
            <!--<artifactId>soul-spring-boot-starter-sync-data-websocket</artifactId>-->
            <!--<version>${project.version}</version>-->
        <!--</dependency>-->
```
同理注释掉yml问价中的默认的websocket配置修改为zookeeper配置
```
    zookeeper:
        url: localhost:2181
        sessionTimeout: 5000
        connectionTimeout: 2000
```
对应的网关soul-bootstrap层根据官方文档做对应的设置

根据上一文的调试过程。我们同样找到了zookeeper对应的配置文件和加载的代码
![file](http://cdn.kobefan.cn/Fp9HY3FINyDK8C-q-_FAU4QSfEVT)
从传递格式可以看到与websocket的同步的配置基本一样，但是同样的，我在这里断点调试时 final ObjectProvider<PluginDataSubscriber> pluginSubscriber组件的数据来源有一些疑问。因此我全局搜索了PluginDataSubscriber，发现了实现了PluginDataSubscriber接口的类CommonPluginDataSubscriber，发现在这个类中有事件订阅和刷新相关的方法。毫无疑问，这些就是后续再具体操作时实现数据变化刷新时的关键逻辑
紧接着。我发现CommonPluginDataSubscriber类上并没有Spring中相关的注解。那么他是如何被注入到spring中的呢。于是我又进行了全局搜索，发现这个类在soul-web模块中的SoulConfiguration模块中进行了注入 
>    @Bean
>      public PluginDataSubscriber pluginDataSubscriber(final ObjectProvider<List<PluginDataHandler>> pluginDataHandlerList) {
>          return new CommonPluginDataSubscriber(pluginDataHandlerList.getIfAvailable(Collections::emptyList));
>      }

	 同时可以发现这个类中有其他转发和插件处理等相关逻辑。这个后续的分析可能会用到，值得关注一下
	 另外可以看到上述列表中同样适用了ObjectProvider来注入插件数据处理类。利用这个线索。我们可以全局搜索一下PluginDataHandler相关的类，搜索后发现，基本每一个插件都是类似的这种做法Configuration-->PluginDataHandler
同时，可以知道数据流向RateLimiterConfiguration-->RateLimiterPluginDataHandler-->SoulConfiguration-->pluginDataSubscriber-->ZookeeperSyncDataConfiguration

同时在ZookeeperSyncDataService我们可以看到基于zookeeper的客户端进行了选择器和规则，元数据和权限相关的数据同步操作。关于这一部分的内容，后续我们继续查看如何进行的数据同步
同时，后面我们会讲解一些关于响应式编程的相关内容,例如Mono和Flux