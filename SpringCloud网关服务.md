### Spring Cloud Gateway

API Gateway（APIGW / API 网关），系统对外的唯一入口

- 核心概念

路由（Route）：路由是网关最基础的部分，路由信息由 ID、目标 URI、一组断言和一组过滤器组成。**如果断言 路由为真，则说明请求的 URI 和配置匹配**。

断言（Predicate）：Java8 中的断言函数。 ServerWebExchange。允许开发者定义**匹配来自于 Http Request 中的任何信息，比如请求头和参数等**。

过滤器（Filter）：一个标准的 Spring Web Filter。Spring Cloud Gateway 中的 Filter 分为两种类型，分别是 Gateway Filter 和 Global Filter。**过滤器将会对请求和响应进行处理**。

- 提示：目前gateway网关服务，用于路由转发、异常处理、限流、降级、接口、鉴权等等


- 路由规则

路由断言工厂RoutePredicateFactory包含的主要实现类如图所示，包括Datetime、请求的远端地址、路由权重、请求头、Host 地址、请求方法、请求路径和请求参数等类型的路由断言。在**配置文件中添加参数**。

```yaml
spring: 
  application:
    name: ruoyi-gateway
  cloud:
    gateway:
      routes:
        - id: ruoyi-system
          uri: http://localhost:9201/
          predicates:
          	#匹配日期时间之后发生的请求
            - After=2021-02-23T14:20:00.000+08:00[Asia/Shanghai]
            #匹配指定名称且其值与正则表达式匹配的cookie
            - Cookie=loginname, ruoyi
            #匹配具有指定名称的请求头，\d+值匹配正则表达式
            - Header=X-Request-Id, \d+
            #匹配主机名的列表
            - Host=**.somehost.org,**.anotherhost.org
            #匹配请求methods的参数，它是一个或多个参数
            - Method=GET,POST
            #匹配请求路径
            - Path=/system/**
            #匹配查询参数
            - Query=username, abc.
            #匹配IP地址和子网掩码
            - RemoteAddr=192.168.10.1/0
            #匹配权重
            - Weight=group1, 2
```

- 路由配置

http地址配置方式

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: ruoyi-api
          uri: http://localhost:9090/
          predicates:
            - Path=/api/**
```

- 限流配置

通过限流，我们可以很好地控制系统的QPS，从而达到保护系统的目的。
Spring Cloud Gateway官方提供了RequestRateLimiterGatewayFilterFactory过滤器工厂，使用Redis 和Lua脚本实现了令牌桶的方式

1、添加依赖

```xml
<!-- spring data redis reactive 依赖 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

2、限流规则，根据URL限流（yaml编写时注意换行，格式要求严格）

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: 
  cloud:
    gateway:
      routes:
        # 系统模块
        - id: ruoyi-system
          uri: lb://ruoyi-system
          predicates:
            - Path=/system/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 1 # 令牌桶每秒填充速率
                redis-rate-limiter.burstCapacity: 2 # 令牌桶总容量
                key-resolver: "#{@pathKeyResolver}" # 使用 SpEL 表达式按名称引用 bean
```

3、编写URI限流规则配置类

```java
package com.ruoyi.gateway.config;

import org.springframework.cloud.gateway.filter.ratelimit.KeyResolver;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import reactor.core.publisher.Mono;

/**
 * 限流规则配置类
 */
@Configuration
public class KeyResolverConfiguration
{
    @Bean
    public KeyResolver pathKeyResolver()
    {
        return exchange -> Mono.just(exchange.getRequest().getURI().getPath());
    }
}
```

- Sentinel分组限流

**注意**：sentinel 的groupId不是org.springframework.cloud，而是属于alibaba自己的groupId

1、添加依赖（父级）

```xml
<dependencyManagement>
  <dependencies>
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-dependencies</artifactId>
          <version>${spring-cloud.version}</version>
          <type>pom</type>
          <scope>import</scope>
      </dependency>
      <!--整合Spring Cloud Alibaba-->
      <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-alibaba-dependencies</artifactId>
           <version>2.1.0.RELEASE</version>
           <type>pom</type>
           <scope>import</scope>
       </dependency>
  </dependencies>
</dependencyManagement>
```

2、添加依赖（子级）

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-spring-cloud-gateway-adapter</artifactId>
</dependency>
```

3、限流配置类

```java
package com.ruoyi.gateway.config;

import java.util.HashSet;
import java.util.Set;
import javax.annotation.PostConstruct;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.core.annotation.Order;
import com.alibaba.csp.sentinel.adapter.gateway.common.rule.GatewayFlowRule;
import com.alibaba.csp.sentinel.adapter.gateway.common.rule.GatewayRuleManager;
import com.alibaba.csp.sentinel.adapter.gateway.sc.SentinelGatewayFilter;
import com.ruoyi.gateway.handler.SentinelFallbackHandler;

/**
 * 网关限流配置
 * 
 * @author ruoyi
 */
@Configuration
public class GatewayConfig
{
    @Bean
    @Order(Ordered.HIGHEST_PRECEDENCE)
    public SentinelFallbackHandler sentinelGatewayExceptionHandler()
    {
        return new SentinelFallbackHandler();
    }

    @Bean
    @Order(-1)
    public GlobalFilter sentinelGatewayFilter()
    {
        return new SentinelGatewayFilter();
    }

    @PostConstruct
    public void doInit()
    {
        // 加载网关限流规则
        initGatewayRules();
    }

    /**
     * 网关限流规则   
     */
    private void initGatewayRules()
    {
        Set<GatewayFlowRule> rules = new HashSet<>();
        rules.add(new GatewayFlowRule("ruoyi-system")
                .setCount(3) // 限流阈值
                .setIntervalSec(60)); // 统计时间窗口，单位是秒，默认是 1 秒
        // 加载网关限流规则
        GatewayRuleManager.loadRules(rules);
    }
}
```

4、自定义异常信息

```java
public class SentinelFallbackHandler implements WebExceptionHandler{

    private Mono<Void> writeResponse(ServerResponse response, ServerWebExchange exchange)
    {
        ServerHttpResponse serverHttpResponse = exchange.getResponse();
        serverHttpResponse.getHeaders().add("Content-Type", "application/json;charset=UTF-8");
        byte[] datas = "{\"code\":429,\"msg\":\"请求超过最大数，请稍后再试\"}".getBytes(StandardCharsets.UTF_8);
        DataBuffer buffer = serverHttpResponse.bufferFactory().wrap(datas);
        return serverHttpResponse.writeWith(Mono.just(buffer));
    }

    @Override
    public Mono<Void> handle(ServerWebExchange exchange, Throwable ex)
    {
        if (exchange.getResponse().isCommitted())
        {
            return Mono.error(ex);
        }
        if (!BlockException.isBlockException(ex))
        {
            return Mono.error(ex);
        }
        return handleBlockedRequest(exchange, ex).flatMap(response -> writeResponse(response, exchange));
    }

    private Mono<ServerResponse> handleBlockedRequest(ServerWebExchange exchange, Throwable throwable)
    {
        return GatewayCallbackManager.getBlockHandler().handleRequest(exchange, throwable);
    }
}
```

