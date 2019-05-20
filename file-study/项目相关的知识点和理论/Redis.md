# Redis单机
## RedisPool的一些配置
RedisPool 封装了JedisPool。
```java
private static JedisPool pool;//jedis连接池
    private static Integer maxTotal = Integer.parseInt(PropertiesUtil.getProperty("redis.max.total","20")); //最大连接数
    private static Integer maxIdle = Integer.parseInt(PropertiesUtil.getProperty("redis.max.idle","20"));//在jedispool中最大的idle状态(空闲的)的jedis实例的个数
    private static Integer minIdle = Integer.parseInt(PropertiesUtil.getProperty("redis.min.idle","20"));//在jedispool中最小的idle状态(空闲的)的jedis实例的个数

    private static Boolean testOnBorrow = Boolean.parseBoolean(PropertiesUtil.getProperty("redis.test.borrow","true"));//在borrow一个jedis实例的时候，是否要进行验证操作，如果赋值true。则得到的jedis实例肯定是可以用的。
    private static Boolean testOnReturn = Boolean.parseBoolean(PropertiesUtil.getProperty("redis.test.return","true"));//在return一个jedis实例的时候，是否要进行验证操作，如果赋值true。则放回jedispool的jedis实例肯定是可以用的。

    private static String redisIp = PropertiesUtil.getProperty("redis.ip");
    private static Integer redisPort = Integer.parseInt(PropertiesUtil.getProperty("redis.port"));
```
JedisPoolConfig -> GenericObjectPoolConfig -> BaseObjectPoolConfig
* JedisPool-连接池
* maxTotal-最大连接数,默认8
* maxIdle-在jedispool中最大的idle状态(空闲的)的jedis实例的个数,默认8
* minIdle-在jedispool中最小的idle状态(空闲的)的jedis实例的个数,默认0
* testOnBorrow-在borrow一个jedis实例的时候，是否要进行验证操作，如果赋值true。则得到的jedis实例肯定是可以用的。默认false
* testOnReturn-在return一个jedis实例的时候，是否要进行验证操作，如果赋值true。则放回jedispool的jedis实例肯定是可以用的。默认false
* redisIp-连接的ip
* redisPort-连接的port

## 封装一个RedisPoolUtil的工具类
<img src="https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/RedisPoolUtil.PNG" width=50% height=50%/>

## 更新代码
### 封装一个CookieUtil类
<img src="https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/CookieUtil.PNG" width=50% height=50%/>

#### readLoginToken(HttpServletRequest request)
这个方法其实没啥，就是从cookie里面拿出来我们的token的string。

#### writeLoginToken(HttpServletResponse response,String token)
```java
public static void writeLoginToken(HttpServletResponse response,String token){
        Cookie ck = new Cookie(COOKIE_NAME,token);
        ck.setDomain(COOKIE_DOMAIN);
        ck.setPath("/");//代表设置在根目录
        ck.setHttpOnly(true);
        //单位是秒。
        //如果这个maxage不设置的话，cookie就不会写入硬盘，而是写在内存。只在当前页面有效。
        ck.setMaxAge(60 * 60 * 24 * 365);//如果是-1，代表永久
        log.info("write cookieName:{},cookieValue:{}",ck.getName(),ck.getValue());
        response.addCookie(ck);
    }
```
//X:domain=".happymmall.com" <br>
//a:A.happymmall.com            cookie:domain=A.happymmall.com;path="/"<br>
//b:B.happymmall.com            cookie:domain=B.happymmall.com;path="/"<br>
//c:A.happymmall.com/test/cc    cookie:domain=A.happymmall.com;path="/test/cc"<br>
//d:A.happymmall.com/test/dd    cookie:domain=A.happymmall.com;path="/test/dd"<br>
//e:A.happymmall.com/test       cookie:domain=A.happymmall.com;path="/test"<br>

另外ck.setHttpOnly(true);表示对于脚本隐藏此cookie

#### delLoginToken(HttpServletRequest request,HttpServletResponse response)
通过ck.setMaxAge(0);来删除cookie

### 还需要一个JsonUtil类
通过jackson封装一个util类，其主要配置如下：
```java
static{
    objectMapper.setSerializationInclusion(Inclusion.ALWAYS);
    objectMapper.configure(SerializationConfig.Feature.WRITE_DATES_AS_TIMESTAMPS,false);
    objectMapper.configure(SerializationConfig.Feature.FAIL_ON_EMPTY_BEANS,false);
    objectMapper.setDateFormat(new SimpleDateFormat(DateTimeUtil.STANDARD_FORMAT));
    objectMapper.configure(DeserializationConfig.Feature.FAIL_ON_UNKNOWN_PROPERTIES,false);
}
```
1. objectMapper.setSerializationInclusion:设置序列化与反序列化包括哪些内容，有以下参数可选
    1. ALWAYS
    2. NON_NULL
    3. NON_DEFAULT
    4. NON_EMPTY
2. objectMapper.configure(SerializationConfig.Feature.WRITE_DATES_AS_TIMESTAMPS,false)：取消默认转换timestamps形式
3. objectMapper.configure(SerializationConfig.Feature.FAIL_ON_EMPTY_BEANS,false)：忽略空Bean转json的错误
4. objectMapper.setDateFormat(new SimpleDateFormat(DateTimeUtil.STANDARD_FORMAT))：统一日期格式为yyyy-MM-dd HH:mm:ss
5. objectMapper.configure(DeserializationConfig.Feature.FAIL_ON_UNKNOWN_PROPERTIES,false)：忽略 在json字符串中存在，但是在java对象中不存在对应属性的情况。防止错误

### 更新user的login等

# Redist分布式
配置几乎一样，这里需要注意的是使用的hash算法。本项目使用的是一致性hash算法。
```java
pool = new ShardedJedisPool(config,jedisShardInfoList, Hashing.MURMUR_HASH, Sharded.DEFAULT_KEY_TAG_PATTERN);
```
1. Hashing.MURMUR_HASH：使用一致性hash算法
2. Sharded.DEFAULT_KEY_TAG_PATTERN：
```java
public static final Pattern DEFAULT_KEY_TAG_PATTERN = Pattern
	    .compile("\\{(.+?)\\}");
```

## hash一致性算法
### 一致性hash算法
1. 环形hash空间
2. 把对象映射到hash空间上
3. 把cache映射到hash空间，使用相同的hash算法
<br><img src="https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/chash1.PNG" width=50% height=50%/>
4. 移除添加cache影响较小
### 一致性hash算法的改进
>> 有可能cache节点分布不均匀，hash倾斜性
1. 引入虚拟节点
<br><img src="https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/chash2.PNG" width=50% height=50%/>
2. 对虚拟节点再hash
<br><img src="https://raw.githubusercontent.com/dnhua/mall/v2.0/file-study/img/chash3.PNG" width=50% height=50%/>
3. 命中率计算：(1-n/(n+m))*100%