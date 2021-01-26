# Soul的SPI以及负载均衡策略研究

上一节留下的几个问题在之后进行的研究

* 如何从abstractSoulPlugin执行完之后到WebClientPlugin的相同方法，是责任链模式还是其他的加载过程

> 各个插件执行的时候实际上是责任链模式。请求分发执行的这个方法主要是SoulWebHandler 继承了Spring webflux的WebHandler的handle方法。handle方法中的参数正好就是请求的相关参数，然后我们就可以在插件的执行逻辑内转发和做操作

* abstractSoulPlugin是如何加载注册或修改后的选择器等数据 

> 可以看到在数据同步的配置中,是由zkClient.subscribeDataChanges来订阅数据的改变操作，从感觉上来说，可能没有websocket那么明显

* plugin 中的执行方法是如何获取到ServerWebExchange的相关请求数据

> SoulWebHandler 继承了Spring webflux的WebHandler的handle方法,springwebflux内部获取了请求的相关属性放入了ServerWebExchange中



## Soul的SPI以及引申出的负载均衡

针对负载均衡的研究，其实我在一开始的使用的文章中就已经提到过了。因此我今天继续尝试了负载均衡的相关代码的查看，首先，在上一节的基础上，我们可以知道，插件是通过责任链模式进行执行的。而我们负载均衡这一部分是使用的 默认的divide插件来执行的。在divide插件的doExecute方法中，首先进行了SPI的类加载，然后根据类加载的情况获取了具体的负载均衡的策略。用来执行了具体的负载均衡策略，通过如下在SPI代码插入的日志语句

![file](http://cdn.kobefan.cn/Fujfh-vpvCCLnv11UHvF0NXBlsKg)

可以看到。SPI不仅加载了负载均衡的具体方式。而且在此之前，还加载了多次匹配策略，全局搜索后发现不仅base插件和divide插件会加载SPI类，监控相关的MetricsTrackerFacade中也同样使用了SPI类。同时，在SPI的第一次加载某个SPI实现时，SPI会将相关类实例在缓存中缓存起来。这样不用在后续的代码中继续执行加载资源解析spi文件等具体的耗时操作，另外 负载均衡中内置了hash，轮询roundrobin,随机random三种负载均衡方式，由于对算法还没有进行深入研究，在此不做过多说明，后续进行了一些研究再在此基础上进行说明
![file](http://cdn.kobefan.cn/FiDj9pN4QlVYP0vmyZsBjBM6KkGm)
![file](http://cdn.kobefan.cn/FrlwfgifgrMa8pFiRelpToDzG_i6)
![file](http://cdn.kobefan.cn/FhFCdUGhoiTWZFd1qCQwgw-dAHVI)
下面，想探讨下，当选择器负载均衡和选择器规则负载均衡不相同且选择器规则负载均衡为random时为什么选择的是选择器的负载均衡
![file](http://cdn.kobefan.cn/Fk83disQuhmZgnO6HnjCiE2LgARm)

通过在插件执行的断点中执行可以看出。插件同时获取了规则和选择器的负载均衡策略。而执行那个负载均衡策略在具体的执行器中进行选择

![file](http://cdn.kobefan.cn/FnGP52U3LG6HnxrjR9GtVGiBRgsn)
可以看到在random的负载均衡策略中仍然进行了选择器权重的区分。通过类的继承关系可以看到getWeight方法被定义在模板抽象类中，同时被roundrobin和random使用，从而实现了上述的所说的在选择器规则存在的情况下依然选择了roundrobin负载均衡