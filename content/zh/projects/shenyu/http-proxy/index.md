---
title: Http 用户
keywords: shenyu
description: http 用户
---

## 说明

* 本文旨在帮助 http 用户。
* ShenYu 网关使用 divide 插件来处理 http 请求。请求在 shenyu-admin 后台开启它。
* 接入前，请正确的启动 `shenyu-admin`，以及 [搭建环境](../shenyu-set-up) OK。


## 引入网关对http的代理插件

* 在网关的 `pom.xml` 文件中增加如下依赖：

```xml
  <!--if you use http proxy start this-->
   <dependency>
       <groupId>org.apache.shenyu</groupId>
       <artifactId>shenyu-spring-boot-starter-plugin-divide</artifactId>
        <version>${last.version}</version>
   </dependency>

   <dependency>
       <groupId>org.apache.shenyu</groupId>
       <artifactId>shenyu-spring-boot-starter-plugin-httpclient</artifactId>
        <version>${last.version}</version>
   </dependency>
```

* 当然是要重新启动网关。

## Http请求接入网关（springMvc 体系用户）

* 首先要确保在 `shenyu-admin` 后台 divide 插件是否开启。

##### Shenyu-Client 接入方式。 （此方式针对 SpringMvc, SpringBoot 用户）

* `SpringBoot 用户`

   * 在你的真实服务的 `pom.xml` 新增如下依赖:

    ```xml
         <dependency>
             <groupId>org.apache.shenyu</groupId>
             <artifactId>shenyu-spring-boot-starter-client-springmvc</artifactId>
             <version>${last.version}</version>
         </dependency>
     ```

   * 注册中心详细接入配置请参考：[注册中心接入](../register-center-access).

* `SpringMvc 用户`

   * 在你的真实服务的 `pom.xml` 新增如下依赖：

    ```xml
           <dependency>
               <groupId>org.apache.shenyu</groupId>
               <artifactId>shenyu-client-springmvc</artifactId>
               <version>${last.version}</version>
           </dependency>
     ```
  * 在你的 bean 定义的 xml 文件中新增如下：

     ```xml
        <bean id ="springMvcClientBeanPostProcessor" class ="org.apache.shenyu.client.springmvc.init.SpringMvcClientBeanPostProcessor">
             <constructor-arg  ref="shenyuRegisterCenterConfig"/>
        </bean>

        <bean id="shenyuRegisterCenterConfig" class="org.apache.shenyu.register.common.config.ShenyuRegisterCenterConfig">
             <property name="registerType" value="http"/>
             <property name="serverList" value="http://localhost:9095"/>
             <property name="props">
                  <map>
                    <entry key="contextPath" value="/你的contextPath"/>
                    <entry key="appName" value="你的名字"/>
                    <entry key="port" value="你的端口"/>
                    <entry key="isFull" value="false"/>
                  </map>
             </property>
        </bean>
    ```
* 在你的 `controller` 的接口上加上 `@ShenyuSpringMvcClient` 注解。

   * 你可以把注解加到 `Controller` 类上面，里面的path属性则为前缀，如果含有 `/**` 代表你的整个接口需要被网关代理。

   * 举例子（1）： 代表 `/test/payment`，`/test/findByUserId` 都会被网关代理。

    ```java
      @RestController
      @RequestMapping("/test")
      @ShenyuSpringMvcClient(path = "/test/**")
      public class HttpTestController {

          @PostMapping("/payment")
          public UserDTO post(@RequestBody final UserDTO userDTO) {
              return userDTO;
          }

          @GetMapping("/findByUserId")
          public UserDTO findByUserId(@RequestParam("userId") final String userId) {
              UserDTO userDTO = new UserDTO();
              userDTO.setUserId(userId);
              userDTO.setUserName("hello world");
              return userDTO;
          }
       }
    ```
   * 举例子（2）：代表 `/order/save`，会被网关代理，而 `/order/findById` 则不会。

    ```java
      @RestController
      @RequestMapping("/order")
      @ShenyuSpringMvcClient(path = "/order")
      public class OrderController {

          @PostMapping("/save")
          @ShenyuSpringMvcClient(path = "/save")
          public OrderDTO save(@RequestBody final OrderDTO orderDTO) {
              orderDTO.setName("hello world save order");
              return orderDTO;
          }

          @GetMapping("/findById")
          public OrderDTO findById(@RequestParam("id") final String id) {
              OrderDTO orderDTO = new OrderDTO();
              orderDTO.setId(id);
              orderDTO.setName("hello world findById");
              return orderDTO;
          }
      }
    ```

* 启动你的项目，你的接口接入到了网关。

## Http请求接入网关（其他语言，非springMvc体系）

* 首先在 `shenyu-admin` 找到 divide 插件，进行选择器，和规则的添加，进行流量的匹配筛选。

* 如果不懂怎么配置，请看选择，规则介绍 [选择器规则介绍](../selector-and-rule)。

* 您也可以自定义开发属于你的 http-client，参考 [多语言 Http 客户端开发](../developer-shenyu-client)。

## 用户请求

* 说白了，你之前怎么请求就怎么请求，没有很大的变动，变动的地方有 2 点。

* 第一点，你之前请求的域名是你自己的服务，现在要换成网关的域名 （这个你听的懂？）

* 第二点，ShenYu 网关需要有一个路由前缀，这个路由前缀就是你接入项目进行配置 `contextPath`，如果熟的话，可以自由在 `shenyu-admin` 中的divide插件进行自由更改。
    * 比如你有一个 order 服务 它有一个接口，请求路径 http://localhost:8080/test/save

    * 现在就需要换成：http://localhost:9195/order/test/save

    * 其中 localhost:9195 为网关的ip端口，默认端口是9195 ，/order 是你接入网关配置的 contextPath

    * 其他参数，请求方式不变。

    * 我讲到这里还不懂？ 请加群问吧
* 然后你就可以进行访问了，如此的方便与简单。
