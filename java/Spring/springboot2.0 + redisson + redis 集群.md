## 目录

*   [springboot2.0 + redisson + redis 集群](#springboot20--redisson--redis-集群)

    *   [1：redis集群搭建](#1redis集群搭建)

    *   [2：引入redssion 依赖](#2引入redssion-依赖)

    *   [3：redssion 集群配置](#3redssion-集群配置)

    *   [4：redssion 初始化+ RedisTemplate](#4redssion-初始化-redistemplate)

    *   [5：测试](#5测试)

# springboot2.0 + redisson + redis 集群

## 1：redis集群搭建

## 2：引入redssion 依赖

```xml
<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson-spring-boot-starter</artifactId>
   <version>3.11.0</version>
</dependency>
```

## 3：redssion 集群配置

在resource下创建 redssion.yml文件 ，名称随便起，后缀可以是yml，properties，xml，当前使用yml文件进行配置

配置说明：

```yaml
clusterServersConfig:
  # 连接空闲超时，单位：毫秒 默认10000
  idleConnectionTimeout: 10000
  pingTimeout: 1000
  # 同任何节点建立连接时的等待超时。时间单位是毫秒 默认10000
  connectTimeout: 10000
  # 等待节点回复命令的时间。该时间从命令发送成功时开始计时。默认3000
  timeout: 3000
  # 命令失败重试次数
  retryAttempts: 3
  # 命令重试发送时间间隔，单位：毫秒
  retryInterval: 1500
  # 重新连接时间间隔，单位：毫秒
  reconnectionTimeout: 3000
  # 执行失败最大次数
  failedAttempts: 3
  # 密码
  password: test1234
  # 单个连接最大订阅数量
  subscriptionsPerConnection: 5
  clientName: null
  # loadBalancer 负载均衡算法类的选择
  loadBalancer: !<org.redisson.connection.balancer.RoundRobinLoadBalancer> {}
  #从节点发布和订阅连接的最小空闲连接数
  slaveSubscriptionConnectionMinimumIdleSize: 1
  #从节点发布和订阅连接池大小 默认值50
  slaveSubscriptionConnectionPoolSize: 50
  # 从节点最小空闲连接数 默认值32
  slaveConnectionMinimumIdleSize: 32
  # 从节点连接池大小 默认64
  slaveConnectionPoolSize: 64
  # 主节点最小空闲连接数 默认32
  masterConnectionMinimumIdleSize: 32
  # 主节点连接池大小 默认64
  masterConnectionPoolSize: 64
  # 订阅操作的负载均衡模式
  subscriptionMode: SLAVE
  # 只在从服务器读取
  readMode: SLAVE
  # 集群地址
  nodeAddresses:
    - "redis://192.168.184.128:30001"
    - "redis://192.168.184.128:30002"
    - "redis://192.168.184.128:30003"
    - "redis://192.168.184.128:30004"
    - "redis://192.168.184.128:30005"
    - "redis://192.168.184.128:30006"
  # 对Redis集群节点状态扫描的时间间隔。单位是毫秒。默认1000
  scanInterval: 1000
  #这个线程池数量被所有RTopic对象监听器，RRemoteService调用者和RExecutorService任务共同共享。默认2
threads: 0
#这个线程池数量是在一个Redisson实例内，被其创建的所有分布式数据类型和服务，以及底层客户端所一同共享的线程池里保存的线程数量。默认2
nettyThreads: 0
# 编码方式 默认org.redisson.codec.JsonJacksonCodec
codec: !<org.redisson.codec.JsonJacksonCodec> {}
#传输模式
transportMode: NIO
# 分布式锁自动过期时间，防止死锁，默认30000
lockWatchdogTimeout: 30000
# 通过该参数来修改是否按订阅发布消息的接收顺序出来消息，如果选否将对消息实行并行处理，该参数只适用于订阅发布消息的情况, 默认true
keepPubSubOrder: true
# 用来指定高性能引擎的行为。由于该变量值的选用与使用场景息息相关（NORMAL除外）我们建议对每个参数值都进行尝试。
#
#该参数仅限于Redisson PRO版本。
#performanceMode: HIGHER_THROUGHPUT
```

## 4：redssion 初始化+ RedisTemplate

```java
@Configuration
public class RedissonConfig {

    @Bean
    public RedissonClient redisson() throws IOException {
        Config config = Config.fromYAML(new ClassPathResource("redisson.yml").getInputStream());
        RedissonClient redisson = Redisson.create(config);
        return redisson;
    }

    @Bean
    public RedissonConnectionFactory redissonConnectionFactory(RedissonClient redisson) {
        return new RedissonConnectionFactory(redisson);
    }
    
    @Bean("redisTemplate")
    public RedisTemplate getRedisTemplate(RedisConnectionFactory redissonConnectionFactory) {
        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate();
        redisTemplate.setConnectionFactory(redissonConnectionFactory);
        redisTemplate.setValueSerializer(valueSerializer());
        redisTemplate.setKeySerializer(keySerializer());
        redisTemplate.setHashKeySerializer(keySerializer());
        redisTemplate.setHashValueSerializer(valueSerializer());
        return redisTemplate;
    }

    @Bean
    public RedisSerializer keySerializer() {
        return new StringRedisSerializer();
    }

    @Bean
    public RedisSerializer valueSerializer() {
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        return jackson2JsonRedisSerializer;
    }
}
```

## 5：测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(classes = {RedissonConfig.class})
//开启自动配置，排除springjdbc自动配置
@EnableAutoConfiguration(exclude = DataSourceAutoConfiguration.class)
public class RedissonTests {

    @Autowired
    RedisTemplate redisTemplate;

    @Test
    public void test() {
        ValueOperations valueOperations = redisTemplate.opsForValue();
        valueOperations.set("redisson", "hello word");
    }

}
```
