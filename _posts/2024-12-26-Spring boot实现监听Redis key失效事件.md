---
layout: post
title: SpringBoot实现监听Redis key失效事件
subtitle:
author: Shiayanga
categories: [SpringBoot]
banner:
tags: [SpringBoot,Redis]
sidebar: []
---

# 一. 开启Redis key过期提醒
- 方式一：修改配置文件

  `redis.conf`

    ```shell
    # 默认 notify-keyspace-events ""
    notify-keyspace-events Ex
    ```

- 方式二：命令行开启

    ```sql
    CONFIG SET notify-keyspace-events Ex
    CONFIG GET notify-keyspace-events
    ```


# 二. notify-keyspace-events

**notify-keyspace-events 选项的默认值为空**

notify-keyspace-events 的参数可以是以下字符的**任意组合**， 它指定了服务器该发送哪些类型的通知。

| 字符  | 发送的通知                                  |
| --- | -------------------------------------- |
| K   | 键空间通知，所有通知以 **keyspace@** 为前缀          |
| E   | 键事件通知，所有通知以 **keyevent@** 为前缀          |
| g   | DEL 、 EXPIRE 、 RENAME 等类型无关的通用命令的通知    |
| $   | 字符串命令的通知                               |
| l   | 列表命令的通知                                |
| s   | 集合命令的通知                                |
| h   | 哈希命令的通知                                |
| z   | 有序集合命令的通知                              |
| x   | 过期事件：每当有过期键被删除时发送                      |
| e   | 驱逐(evict)事件：每当有键因为 maxmemory 政策而被删除时发送 |
| A   | 参数 g$lshzxe 的别名                        |

# 三. Coding

1. 初始化一个`Spring Boot`项目
2. `pom.xml`
    ```xml
    <dependencies>
    		<dependency>
    			<groupId>org.springframework.boot</groupId>
    			<artifactId>spring-boot-starter-data-redis</artifactId>
    		</dependency>
    		<dependency>
    			<groupId>org.projectlombok</groupId>
    			<artifactId>lombok</artifactId>
    		</dependency>
    	</dependencies>
    ```

3. 定义配置类`RedisListenerConfig
    ```java
    @Configuration
    public class RedisListenerConfig {
    
    	@Bean
    	RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory) {
    		RedisMessageListenerContainer container = new RedisMessageListenerContainer();
    		container.setConnectionFactory(connectionFactory);
    		return container;
    	}
    
    }
    ```

4. 定义数据生产类`ProviderDataToRedis`
    ```java
    @Slf4j
    @Component
    public class ProviderDataToRedis implements CommandLineRunner {
    
    	@Autowired
    	private StringRedisTemplate stringRedisTemplate;
    
    	@Override
    	public void run(String... args) throws Exception {
    		int[] num = new int[]{1};
    		Random random = new Random();
    		while (true) {
    			int max = random.nextInt(5);
    			IntStream.range(0, max).forEach(n -> stringRedisTemplate.opsForValue().set(String.format("mq:s1:%s", ++num[0]), "已预订", 5, TimeUnit.SECONDS));
    			log.info("放了 {} 条数据到redis...", max);
    			TimeUnit.SECONDS.sleep(3);
    		}
    	}
    }
    ```

5. 定义监听器 实现`KeyExpirationEventMessageListener`接口
   查看源码发现，该接口监听所有db的过期事件`keyevent@*:expired"`
   定义`Status1ExpirationListener`监听状态1到期
    ```java
    @Slf4j
    @Component
    public class Status1ExpirationListener extends KeyExpirationEventMessageListener {
    
    	public Status1ExpirationListener(RedisMessageListenerContainer listenerContainer) {
    		super(listenerContainer);
    	}
    
    	@Autowired
    	private StringRedisTemplate stringRedisTemplate;
    
    	@Override
    	public void onMessage(Message message, byte[] pattern) {
    		// 用户做自己的业务处理即可,注意message.toString()可以获取失效的key
    		String expiredKey = message.toString();
    		if (expiredKey.startsWith("mq:s1:")) {
    			log.info("-----------------------------------");
    			log.info(String.format("过期key[%s]", expiredKey));
    			String newKey = String.format("mq:s2:%s", expiredKey.substring(6));
    			String newValue = "行程中";
    			stringRedisTemplate.opsForValue().set(newKey, newValue, 3, TimeUnit.SECONDS);
    			log.info(String.format("%s: %s", newKey, newValue));
    			log.info("-----------------------------------");
    		}
    	}
    
    }
    ```

   定义`Status2ExpirationListener`监听状态2到期
    ```java
    @Slf4j
    @Component
    public class Status2ExpirationListener extends KeyExpirationEventMessageListener {
    
    	public Status2ExpirationListener(RedisMessageListenerContainer listenerContainer) {
    		super(listenerContainer);
    	}
    
    	@Override
    	public void onMessage(Message message, byte[] pattern) {
    		// 用户做自己的业务处理即可,注意message.toString()可以获取失效的key
    		String expiredKey = message.toString();
    		if (expiredKey.startsWith("mq:s2:")) {
    			log.info("***********************************");
    			log.info(String.format("过期key[%s]", expiredKey));
    			log.info("[{}]行程已完成，修改数据库状态。", newKey);
    			log.info("***********************************");
    		}
    	}
    
    }
    ```
# 四. 参考
[Redis keyspace notifications](https://redis.io/docs/manual/keyspace-notifications/)


