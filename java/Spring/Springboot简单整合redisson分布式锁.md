## 目录

*   [依赖](#依赖)

*   [配置文件](#配置文件)

*   [配置类](#配置类)

# Springboot简单整合redisson分布式锁

## 依赖

```xml
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson</artifactId>
            <version>3.6.5</version>
        </dependency>

```

## 配置文件

```yaml
spring:
  redis:
    host: 192.168.1.1
    port: 6379
    password: 
```

## 配置类

```java
import org.redisson.Redisson;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * redisson单节点配置
 */
@Configuration
public class RedissonConfig {

    @Value("${spring.redis.host}")
    private String host;

    @Value("${spring.redis.port}")
    private String port;

    @Bean
    public RedissonClient getRedisson() {
        Config config = new Config();
        config.useSingleServer().setAddress("redis://" + host + ":" + port);
        return Redisson.create(config);
    }
}

```
