**使用方法：**

这个版本在gitee上面维护。

https://gitee.com/fufu669/cache-fetch-later







============================================================================
Method to use:

**1. 引入jar包**

**import the jar with maven**

```
<dependency>
    <groupId>com.gitee.fufu669</groupId>
    <artifactId>cache-fetch-later</artifactId>
    <version>最新版本</version>
</dependency>
```

**2. 在bootstrap.yml增加配置**

**add configuration in bootstrap.yml**

```
redis:
  hostAndPorts: 166.66.6.166:6379,166.66.6.167:6379
  maxTotal: 200
  maxIdle: 200
  minIdle: 8
  maxWaitMillis: 3000
  testOnBorrow: true
  lifo: true
  testOnReturn: true
  password: zzzzzz
  keyPrefix: com.gitee.fufu669 
  #请更改上面这个keyPrefix，这个是在redis里面存入key时前面增加的前缀。

threadpooltaskexecutor:
  corepoolsize: 6
  maxpoolsize: 200
  queuecapacity: 100000
  keepaliveseconds: 60
  threadnameprefix: threadPoolTaskExecutor-
  #此框架用到了线程池。所以如果使用了自己的线程池，请在注入时，要用@Qualifier来区分.
  ```
**3. 在springboot的application上面加上一个注解**

**add annotation @EnableCacheFetchLater on application.java of springboot**
```
@EnableCacheFetchLater
```
**4. 然后就可以在@Service, @Controller, @Component上面使用 下面4个标签。**

**Then you can use below 4 annotation on @Service, @Controller, @Component**
```
@CacheFetchLater : redis里面有数据，就直接返回，并在加一把分布式锁后，起多线程更新数据。redis里面没有数据，就从方法里取数据，并存入redis。
                if redis has data, return value in redis and update value with multiple thread. if redis has no data, then get from method, set to redis. 
                有2个参数，
                refreshSeconds=30(默认是-1)，意味着分布式锁的过期时间是30秒，意思是30秒内如果数据没有抓完成，是不允许再次重入的。如果已经完成还是允许继续抓数据的。如果值是-1或小于0，则不加分布式锁。
                expireSeconds=666666(默认是666666)，意味着这个缓存过期时间是666666秒，也就是一周。
                lockSeconds=60(默认是60），更新缓存前加的分布式锁的过期时间,默认是60，意思是分布式锁60秒过期。
@CacheMockFetchLater : redis里面有数据，就直接返回，并起多线程更新数据。redis没有数据就返回一个新建对象，然后起多线程更新数据。
                if redis has data, return value in redis and update value with multiple thread. if redis has no data, return a new object of return type, then update value with backend thread.
                有3个参数：
                type=3(默认是3)，缓存没有值时的处理 1:抛出异常 2:返回一个空对象 3:如果是List返回[]，-1:其他值返回null。
                refreshSeconds=30(默认是-1)，意味着分布式锁的过期时间是30秒，意思是30秒内如果数据没有抓完成，是不允许再次重入的。如果已经完成还是允许继续抓数据的。如果值是-1或小于0，则不加分布式锁。
                expireSeconds=666666(默认是666666)，意味着这个缓存过期时间是666666秒，也就是一周。
                lockSeconds=60(默认是60），更新缓存前加的分布式锁的过期时间,默认是60，意思是分布式锁60秒过期。

@LockWithCache : 在方法上面加一把分布式锁。防止多线程多服务器重入。
                add a redis lock. to prevent multi thread or multi server to visit method at the same time.    
                有1个参数，
                expireSeconds=6(默认是6)，意味着这个缓存过期时间是6秒。

@MockExecuteLater : 立即返回null，并起多线程执行。
                   return null and execute with background thread.

redis的key算法：配置的keyPrefix+":cacheKey:"+函数的类+方法名+所有参数的Json（除去没法序列化的）算出来的一个64位hash值（用sha256算法）。

然后Bean里面生成了一个cacheService可以直接使用的
@Autowired
private CacheService cacheService;

新增一个功能：用Controller类 implements com.gitee.fufu669.aspect.Logging会自动打印日志。

注意：spring注解功能在同一个类里互相调用是无效的。必须要用其他类的方法来调用加了注解的另一个方法

```

**5. 更改日志**

```
6.6.689  增加了SimpleResponse，现在SimpleResponse从这个包里面获取了
6.6.692  增加了打印多线程开始时的日志
6.6.6002 增加CacheAesUtil加密
6.6.6003 更改CacheServerException的props的读写方式
6.6.6006 增加了CacheService中的结果打印
6.6.6007 增加了空值返回时的过期时间
6.6.6008 增加了对void返回类型的判断来防止抛空
6.6.6009 优化了获取ip地址的算法
```
