# Soul源码中dubbo和sofa的执行过程
## Soul源码中dubbo的执行过程

* 首先在 soul-examples-apache-dubbo-service 中依赖的soul-client中ApacheDubboServiceBeanPostProcessor对注解SoulDubboClient了向soul-admin中的 http://localhost:9095/soul-client/dubbo-register 进行了注册。而此处我们注意到ApacheDubboServiceBeanPostProcessor并不是实现的BeanPostProcessor接口而是实现的ApplicationListener<ContextRefreshedEvent> 即Spring上下文刷新的事件
* 在该对应的接口中执行的是与前文相同的利用Spring的事件处理机制进行的数据同步操作，将数据同步到了网关soul-bootstrap，这一部分，可参考zookeeper的数据同步机制进行了解
* 紧接着是在客户端发起对dubbo接口的请求的代理时，soul是如何处理的，处理的逻辑主要是

![file](http://cdn.kobefan.cn/Fqzr6q7sU4w-mbAAFhGcuZ_2rgMJ)

通过泛化调用，调用的实际上是zk内的某个service然后异步获取结果。将结果，封装到具体的字段中，然后返回给前端

整个过程中对于接口的缓存的处理与http的请求基本相同，从这里就可以看出soul的设计的情况非常好

## Soul源码中Sofa项目的执行过程

* 同样的，第一步还是通过扫描注解通过依赖的soul-client中SofaServiceBeanPostProcessor对注解SoulDubboClient了向soul-admin中的 http://localhost:9095//soul-client/sofa-register 进行了注册/soul-client/sofa-register，而这个类中使用的并不是与dubbo相同的事件处理器接口，而是与http请求相同的实现了BeanPostProcessor的接口对注解和参数等进行的处理，这是一个需要考虑的点，目前我还不是太清楚，后续需要再深入了解。
* 随后的流程与Http和dubbo的流程基本相同，即在该对应的接口中执行的是与前文相同的利用Spring的事件处理机制进行的数据同步操作，将选择器规则数据同步到了网关soul-bootstrap，这一部分几乎所有插件都一样。
* 后面同样的是和dubbo相同的泛化调用的过程，不同的是dubbo的泛化调用是异步的调用，而sofa的泛化调用是异步的调用，从这一点上来说，在soul中使用dubbo或许性能更好。

```java
        genericService.$invoke(metaData.getMethodName(), pair.getLeft(), pair.getRight());
        return Mono.fromFuture(future.thenApply(ret -> {
            if (Objects.isNull(ret)) {
                ret = Constants.SOFA_RPC_RESULT_EMPTY;
            }
            exchange.getAttributes().put(Constants.SOFA_RPC_RESULT, ret);
            exchange.getAttributes().put(Constants.CLIENT_RESPONSE_RESULT_TYPE, ResultEnum.SUCCESS.getName());
            return ret;
        })).onErrorMap(SoulException::new);
```





## 关于ApacheDubboServiceBeanPostProcessor并不是实现的BeanPostProcessor接口而是实现的ApplicationListener<ContextRefreshedEvent> 的问题

在类中的onApplicationEvent中有对应issue中有对应的https://github.com/dromara/soul/issues/415答疑，主要是上传的顺序跟dubbo暴露的顺序问题导致的。使用beanPostProcessor可能导致no provider的情况。

