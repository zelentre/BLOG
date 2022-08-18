# Spring Cloud Gateway如何实现限流？

## 一、问题回答

1. Spring Cloud Gateway使用**令牌桶算法**实现限流。
2. Spring Cloud Gateway默认使用**redis**的**RateLimiter**限流算法来实现。所以我们要使用需要引入redis的依赖。
3. 使用过程中，我们主要配置**令牌桶令牌填充的速率**，**令牌通的容量**，**指定限流的Key**。
4. 限流的Key，可以根据用户来做限流，IP来做限流，接口限流等等。

## 二、辅助理解

### 1. 漏桶算法和令牌桶算法

**漏桶(Leaky Bucket)算法**

思路很简单，水（请求）先进入到漏桶里，漏桶以一定的速度出水（接口有响应速率），当水流入速度过大会直接溢出（访问频率超过接口响应速率），然后就拒绝请求，可以看出漏桶算法能强制限制数据的传输速率。

**令牌桶算法(Token Bucket)**

和Leaky Bucket**效果一样但方向相反**的算法，更加容易理解。随着时间流逝，系统会按恒定1/QPS时间间隔（如果QPS=100，则间隔时间是10ms）往桶里加入Token（想象和漏洞漏水相反，有个水龙头在不断的加水），如果桶已经满了就不在加了。新的请求来临时，会各自拿走一个Token，如果没有Token可拿了就阻塞或者拒绝服务。

**两者的主要区别**

漏桶算法能够**强行限制数据的传输速率**，而令牌桶算法在能**够限制数据的平均传输速率外**，还**允许某种程度的突发传输**。在令牌桶算法中，**只要令牌桶中存在令牌**，那么就允许突发地传输数据直到达到用户配置的门限，所以它适合于具有**突发特性的流量**。

### 2. 通过ip限流使用步骤

引入redis

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

ip限流的keyResolver

```java
@Bean
public KeyResolver ipKeyResolver(){
    return exchange -> Mono.just(exchange.getRequest().getRemoteAddress().getHostName());
}
```

配置

```yaml
spring:
 gateway:
  routes:
  - id: fsh-house
  uri: lb://fsh-house
  predicates:
  - Path=/house/**
  filters:
  - name: RequestRateLimiter
  args:
  redis-rate-limiter.replenishRate: 10
  redis-rate-limiter.burstCapacity: 20
  key-resolver: "#{@ipKeyReso1ver}"
```

- filter名称必须是RequestRateLimiter
- redis-rate-limiter.replenishRate：允许用户每秒处理多少个请求
- redis-rate- imiter.burstCapacity：令牌桶的容量，允许在一秒内完成的最大请求数
-  key-resolver：使用SpEL按名称引用bean

### 3. 了解其他限流方式

用户限流

```java 
@Bean
public KeyResolver userKeyResolver(){
    return exchange -> Mono.just(exchange.getRequest().getQueryParams().getFirst("userId"));
}
```

接口限流

```java 
@Bean
public KeyResolver apiKeyResolver(){
    return exchange -> Mono.just(exchange.getRequest().getPath().value());
}
```

也可以通过重写RedisRateLimiter来实现自己的**限流策略**