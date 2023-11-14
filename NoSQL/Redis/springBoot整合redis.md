

# springBoot整合redis

> 整合入门以及错误记录

## 0、安装过程

```shell
#安装redis 默认是稳定版 静默安装完成
brew install redis

#redis使用
#启动Redis
方式一: brew services start redis@3.2 
方式二: redis-server /usr/local/etc/redis.conf
#查看ps(或者其他应用进程)
ps axu | grep redis(其他进程名)
isole            84721   0.0  0.0  4267768    900 s000  S+   10:08上午   0:00.00 grep redis
isole            84501   0.0  0.0  4309180   1568   ??  Ss   10:06上午   0:00.07 redis-server 127.0.0.1:6379
#关闭redis服务(直接kill +进程号)
方法一：kill 84721   
方法二：brew services stop redis
#连接redis客户端
redis-cli -h 127.0.0.1 -p 6379 如下: 127.0.0.1:6379> get("123")
#关闭redis客户端
redis-cli shutdown
#关闭服务(杀死进程)
sudo pkill redis-server
```



## 1、整合步骤

### 1.1导包

```xml
				<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

```

### 1.2 yml配置

```yaml
redis:
    password:
    host: 127.0.0.1
    port: 6379
    database: 0
    #连接池最大连接数（使用负值表示没有限制） 还可以用lettuce
    jedis:
      pool:
        max-active: 8
        max-wait: -1  # 连接池最大阻塞等待时间（使用负值表示没有限制）
        max-idle: 8   # 连接池中的最大空闲连接
        min-idle: 0   # 连接池中的最小空闲连接
    timeout: 0        # 连接超时时间（毫秒）
```

### 1.3 Test测试

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class redisTest {

		//操作对象的Template
    @Autowired
    private RedisTemplate redisTemplate;
  	//操作字符的Template
    @Autowired
    private StringRedisTemplate stringRedisTemplate;
    @Autowired
    private EmployeeService employeeService;

    @Test
    public void test01(){
      	//添加字符串
        stringRedisTemplate.opsForValue().append("gender","男性1");
        Employee employee = employeeService.getEmployeeById(3);
        //获取value
        String name = stringRedisTemplate.opsForValue().get("name");
        System.out.println(name);
      	//添加对象
        redisTemplate.opsForValue().set("emp",employee);

    }
}
```

### 1.4 去rdm中查看数据

> 发现有乱码问题，因为序列化的时候默认是用jdk序列化的
>
> 解决办法：自定义RedisTemplate 重新定义自己需要的序列化类型

```java

package org.springframework.data.redis.serializer;
import org.springframework.lang.Nullable;

public interface RedisSerializer<T> {

	/**
	 * 用的是这个序列化 导致乱码
	 */
	static RedisSerializer<Object> java(@Nullable ClassLoader classLoader) {
		return new JdkSerializationRedisSerializer(classLoader);
	}

	static RedisSerializer<Object> json() {
		return new GenericJackson2JsonRedisSerializer();
	}

}

```

### 1.5 RedisConfig配置

> 去RedisAutoConfiguration中复制一份RedisTemplate 配置自己的RedisTemplate
>
> 自定义序列化配置
>
> StringRedisTemplate同理配置

```java
/**
 * @author JackGao
 * @date 2019-07-11
 * @Description: TODO redis序列化配置
 **/
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
            throws UnknownHostException {
        RedisTemplate<Object, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
      	//设置key的序列化
        template.setKeySerializer(new StringRedisSerializer());
        //设置value的序列化 可以用下列的setDefaultSerializer 设置所有的序列化问题
        template.setValueSerializer(new Jackson2JsonRedisSerializer<Object>(Object.class));
      //template.setDefaultSerializer(new Jackson2JsonRedisSerializer<Object(Object.class));
        //设置默认序列化的类型
				template.setDefaultSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }


}
```

##### 1.5.1 使用 **new GenericJackson2JsonRedisSerializer()** 序列化方法

```json
{
  "@class": "com.manulife.cache.study.bean.Employee",
  "id": 2,
  "lastName": "李四",
  "email": "a@163.com",
  "gender": 0,
  "did": 2
}
```

##### 1.5.2 使用 **new Jackson2JsonRedisSerializer<Object(Object.class)** 序列化方法

```
{
  "id": 2,
  "lastName": "李四",
  "email": "a@163.com",
  "gender": 0,
  "did": 2
}
```

