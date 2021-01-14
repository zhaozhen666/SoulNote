## Soul 源码环境搭建

## 拉取代码,修改配置

首先访问https://github.com/dromara/soul 对该仓库进行star和watch，作为一个网关使用的新手。需要在后续关注soul的开发动向,把他更好的用在工作当中
随后将该仓库fork到自己的github中。方便自己后续进行代码的学习和注释。然后对自己fork的仓库进行clone
```
git clone git@github.com:zhendiao/soul.git
```

然后将自己的代码导入到IDEA当中，首先观察Soul项目的目录结构
![file](http://cdn.kobefan.cn/Fn01W_zO67h0tVmeuWoXcLyU42fJ)
我们可以很明显的猜到Soul-admin为该网关的管理控制台项目。打开配置文件，虽然可以提供内置的H2数据库作为数据源，但是为了使用方便，我们还是使用MySQL作为数据源，建议MySQL版本为mysql5。修改mysql的配置,无需手动创建数据库，启动之后自动创建了数据库。
![file](http://cdn.kobefan.cn/Fp_tlMiYbCa7bYgn2xFD4NHEXBhB)

可以看到启动之后的界面
![file](http://cdn.kobefan.cn/FmSm8Nz0z8Mw8TC-0_uIIO27qBmF)

## 网关核心层Soul-Bootstrap
soul-bootrap作为拦截外部请求的核心层，目前默认的数据同步方式为websocket配置，配置如下
```
soul :
    file:
      enabled: true
    corss:
      enabled: true
    dubbo :
      parameter: multi
    sync:
        websocket :
             urls: ws://localhost:9095/websocket
```
同时注释的代码中也包含了其他诸如Zookeeper，nacos，http等同步方式。在后面几天的尝试中，会尝试其他几种方式

## 测试项目
参考官网进行http(springboot)方式进行最简单的代理环境搭建，首先新建一个springboot项目,配置如下
```

soul:
  # Soul 针对 SpringMVC 的配置项，对应 SoulHttpConfig 配置类
  http:
    admin-url: http://127.0.0.1:9095 # Soul Admin 地址
    context-path: /soulboot # 设置在 Soul 网关的路由前缀，例如说 /order、/product 等等。
    # 后续，网关会根据该 context-path 来进行路由
    app-name: soulboot # 应用名。未配置情况下，默认使用 `spring.application.name` 配置项
    port: 7070 #你本项目的启动端口
    full: false   # 设置true 代表代理你的整个服务，false表示代理你其中某几个controller
server:
  port: 7070
```
新建一个测试controller
```
package org.dromara.soul.boot.controller;

import org.dromara.soul.client.springmvc.annotation.SoulSpringMvcClient;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/user")
public class HomeController {

    @GetMapping("/get")
    @SoulSpringMvcClient(path = "/user/get", desc = "获得用户详细")
    public String getUser(@RequestParam("id") Integer id) {
        return "DEMO:" + id;
    }


}
```
通过访问soul-boostrap的端口搭配本项目的路由地址即可实现成功访问
![file](http://cdn.kobefan.cn/FmMlU-y3RxZ05LW9XDeJHKieFveo)


## 目前疑问
* 配置的端口为admin的端口，soul-bootstrap如何同步数据并代理的 