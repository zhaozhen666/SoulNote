# Soul网关限流插件Sentinel和Resilience4J扫盲
## Soul网关限流插件Sentinel扫盲
首先看Soul中Sebtinel可以配置的项目
![file](http://cdn.kobefan.cn/FoZR4nmSmLGdDel0lb8oprMTGi7m)
对应的配置的含义
* degrade count：熔断阈值

* whether to open the degrade (1 or 0)：是否开启熔断，1开启 0关闭

* degrade type：熔断类型、熔断策略,slow call ratio(秒级RT) 、 exception ratio（异常比例）、 exception number strategy（分钟级异常数）

* degrade window size：降级窗口期大小，单位s

* control behavior： warm up（预热/冷启动方式，流量缓慢增加）、 constant speed queuing （匀速排队）、 preheating uniformly queued

* grade count：限流阈值

* whether control behavior is enabled (1 or 0)：是否开启限流，1开启 0关闭

* grade type：限流阈值类型 ，QPS 、number of concurrent threads（当前线程数）


但是从界面和参数代表的意思可以看到，实际上熔断和限流的开关开启对其他参数的设置是有影响的但是界面弹窗却没有体现这一点，按照理想状态应该是开启了熔断开关再去显示其他的熔断参数，同样的限流也是这样的。另外对于界面上使用数字调整 whether to open the degrade 的值且并未对值得大小做限制的情况。我觉得还是需要调整的。
![file](http://cdn.kobefan.cn/FordLdACpLRYtWlwKeMwOr8U2ond)。可以看到代码中即是开启熔断或者限流的开关之后才会去加载相关配置

## Soul网关限流插件Resilience4J扫盲
同样的首先看Soul网关中Resilience4J可以配置的项目
![file](http://cdn.kobefan.cn/FpvqWwGfvq_zo1kQ9tik938N1HNi)
* token filling period (ms)：刷新令牌的时间间隔，单位ms，默认值：500。

* token filling number：每次刷新令牌的数量，默认值：50。

* control behavior timeout (ms)：熔断服务，对外停止服务持续时间，单位ms

*  circuit enable：是否开启熔断，0：关闭，1：开启，默认值：0。

* circuit timeout (ms)：熔断超时时间，请求服务响应超过此时间，则触发熔断，单位ms，默认值：30000。

* fallback uri：降级处理的uri。

* sliding window size：滑动窗口大小，默认值：100。

* sliding window type：滑动窗口类型，0：基于计数，1：基于时间，默认值：0。

* enabled error minimum calculation threshold： 开启熔断的最小请求数，超过这个请求数才开启熔断统计，默认值：100。

* degrade opening duration：熔断器开启持续时间，单位ms，默认值：10

*  half open threshold：半开状态下的环形缓冲区大小，必须达到此数量才会计算失败率，默认值：10。

* degrade failure rate：错误率百分比，达到这个阈值，熔断器才会开启，默认值50。

![file](http://cdn.kobefan.cn/FiYNIzqsut36E3XcEVS1qnYHS1qp)
![file](http://cdn.kobefan.cn/Fm6G3L4-QWLBgEBfh-pN1RRV5qgb)
可以看到这里也是熔断器开启了才会去加载某些配置，可以看到，这里的两个1的魔数，实际上可以被修改为固定的参数值类似Sentinel中设置的。当然我感觉可能设置成true或者false更好，期待有序优化。另外还有一个参数可以关注的就是半开状态这个参数，因为不理解这个半开状态的意思。所以在网上具体的查了一下，描述如下

> CircuitBreaker的状态转换通过一个有限状态机来实现的，有3种常用状态： 关闭(CLOSED)、打开(OPEN)、半开(HALF_OPEN)和2种特定状态：不可用(DISABLED)、强制打开(FORCE_OPEN)
> * CLOSED ==> OPEN：单向转换。当请求失败率超过阈值时，熔断器的状态由关闭状态转换到打开状态。失败率的阈值默认50%，可以通过设置CircuitBreakerConfig实例的failureRateThreshold属性值进行改变。
> * OPEN <==> HALF_OPEN：双向转换。打开状态的持续时间结束，熔断器的状态由打开状态转换到半开状态。这时允许一定数量的请求通过，当这些请求的失败率超过阈值，熔断器的状态由半开状态转换回打开状态。半开时请求的数量是由CircuitBreakerConfig实例的ringBufferSizeInHalfOpenState属性值设置的。
> * HALF_OPEN ==> CLOSED：如果请求失败率小于阈值，则熔断器的状态由半开状态转换到关闭状态。
> * DISABLED和FORCE_OPEN这2种状态仅仅是表示退出上面3种状态时的临界状态标识，这2种状态不会被记录到统计指标中，也不会发送状态转换事件。
> 如下图所示转换关系
> ![file](http://cdn.kobefan.cn/FvJ2I4fPLeuaWX5M5z-qEA94s5uf)

## 我认为的可优化项

* 根据熔断或限流开关的设置再展示其他的配置项
* 去掉魔数1调整开关项的配置为true或false。