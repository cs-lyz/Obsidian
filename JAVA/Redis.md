E:\java_code\Redis>redis-server.exe

使用redis实现短信验证登录
![[7266973aa80e32616fe3340cb8ec2f7.jpg]]





### 缓存穿透
：请求的数据在缓存和数据库中都不存在，缓存不会生效，请求都会打到数据库里面
缓存空对象
布隆过滤
```java  title:缓存空对象
//来吧，从这开始,怎么加缓存空对象,加热点key问题解决，高并发访问的热点key失效了  
    @Override  
    public emp selectById(String id) throws IllegalAccessException {  
        emp e=null;  
        Map<String, Object> temp2=null;  
        // 从Redis查出来不为空  需要注意的是，如果 Redis 中根本不存在这个键，  
        // entries() 方法会返回一个空的 Map，而不是 null，所以这里用 isEmpty() 检查是合适的  
        // isEmpty() 是 Map 的实例方法，必须在非 null 对象上调用  
        Boolean exists = stringRedisTemplate.hasKey(id);  
        Map<Object, Object> temp;  
        if (exists != null && exists) {//如果key存在  
             temp= stringRedisTemplate.opsForHash().entries(id);  
//             且不是空对象，转化一下返回就行，最后返回了  
            if(!temp.isEmpty()) {//从Map<Object,Object>到Map<String, Object>  
                System.out.println("temp:" + temp);  
                temp2= temp.entrySet()  
                        .stream()  
                        .collect(Collectors.toMap(  
                                ee -> ee.getKey().toString(),  
                                Map.Entry::getValue  
                        ));  
                System.out.println("temp2:" + temp2);//值给temp2了，从Redis里面出去是可以正常显示的  
                e=ObjectToMapConverter.mapToemp(temp2);  
                System.out.println("e:"+e);  
                return e;  
            }  
//            如果是空对象，照样返回null  
            else  return e;  
        } else {  
            // key 不存在  
            e=empMapper.selectById(id);  
            if(e==null){  
                System.out.println("查无此人");  
//              把这个Id 对应null缓存到Redis里面去  
                stringRedisTemplate.opsForHash().putAll(id, Map.of("__nil__", "1"));  
                stringRedisTemplate.expire(id, 30, TimeUnit.MINUTES);  
                return null;  
            }  
            Map<String,Object> temp1=ObjectToMapConverter.convertToMap(e);  
            System.out.println("temp1:"+temp1);//从数据库查出来是好的  
            //把e放到Redis里面去  
            stringRedisTemplate.opsForHash().putAll(id,temp1);  
            e=ObjectToMapConverter.mapToemp(temp1);  
            System.out.println("e:"+e);  
            return e;  
        }  
    }
```

### 缓存击穿：
热点key失效了，本来就热点，失效以后一堆相同请求过来了，打到数据库里面了
#### 解决方法一：互斥锁
```java 
//来吧，从这开始,怎么加缓存空对象,加互斥锁，热点key问题解决，高并发访问的热点key失效了  
    @Override  
    public emp selectById(String id) throws IllegalAccessException {  
        SimpleILock lock=new SimpleILock("lock",stringRedisTemplate);  
        emp e=null;  
        Map<String, Object> temp2=null;  
        // 从Redis查出来不为空  需要注意的是，如果 Redis 中根本不存在这个键，  
        // entries() 方法会返回一个空的 Map，而不是 null，所以这里用 isEmpty() 检查是合适的  
        // isEmpty() 是 Map 的实例方法，必须在非 null 对象上调用  
  
        Map<Object, Object> temp;  
        while(true){  
            Boolean exists = stringRedisTemplate.hasKey(id);  
            if (exists != null && exists) {//如果key存在  
                temp= stringRedisTemplate.opsForHash().entries(id);  
//             且不是空对象，转化一下返回就行，最后返回了  
                if(!temp.isEmpty()) {//从Map<Object,Object>到Map<String, Object>  
                    temp2= temp.entrySet()  
                            .stream()  
                            .collect(Collectors.toMap(  
                                    ee -> ee.getKey().toString(),  
                                    Map.Entry::getValue  
                            ));  
                    e=ObjectToMapConverter.mapToemp(temp2);  
                    return e;  
                }  
//            如果是空对象，照样返回null  
                else  return e;  
            } else {  
                // key 不存在  
//            先获取互斥锁，如果不成功，休眠一会儿重试  
                if(lock.tryLock(30)){  
                    try {  
                        // 双重检查（防止其他线程已经写入 Redis）  
                        exists = stringRedisTemplate.hasKey(id);  
                        if (exists != null && exists) {  
                            continue; // 其他线程已写入，重新循环  
                        }  
  
                        e = empMapper.selectById(id);  
                        Thread.sleep(200);  
                        if (e == null) {  
                            // 缓存空值标记，防止缓存穿透  
                            stringRedisTemplate.opsForHash().putAll(id, Map.of("__nil__", "1"));  
                            stringRedisTemplate.expire(id, 30, TimeUnit.MINUTES);  
                            return null;  
                        }  
  
                        // 写入 Redis                        
                        Map<String, Object> temp1 = ObjectToMapConverter.convertToMap(e);  
                        stringRedisTemplate.opsForHash().putAll(id, temp1);  
                        stringRedisTemplate.expire(id, 30, TimeUnit.MINUTES);  
                        return e;  
                    } catch (InterruptedException ex) {  
                        throw new RuntimeException(ex);  
                    } finally {  
                        lock.unlock();  
                    }  
                }  
                else{//休眠一会儿  
                    try {  
                        Thread.sleep(100); // 100ms 后重试  
                    } catch (InterruptedException ex) {  
                        Thread.currentThread().interrupt();  
                        throw new IllegalAccessException("线程中断");  
                    }  
                }  
            }  
        }  
    }
```

#### 方法二：逻辑过期，设置永远不会过期的数据
![[Pasted image 20250609213153.png]]

```java
@Override  
    public RedisEmp selectById(String id) throws IllegalAccessException {  
        SimpleILock lock=new SimpleILock("lock",stringRedisTemplate);  
        RedisEmp re=new RedisEmp();  
  
  
//        逻辑过期，假定key一定存在在Redis里面  
        Map<Object, Object> rempMapoo= stringRedisTemplate.opsForHash().entries(id);  
        Map<String, Object> rempMapso= rempMapoo.entrySet()  
                .stream()  
                .collect(Collectors.toMap(  
                        ee -> ee.getKey().toString(),  
                        Map.Entry::getValue  
                ));  
//             且不是空对象，转化一下返回就行，最后返回了  
        if(!rempMapso.isEmpty()) {//从Map<Object,Object>到Map<String, Object>  
            re.setExpireTime((String) rempMapso.get("expireTime"));  
            re.setData((String) rempMapso.get("data"));  
            if(re.isExpired())   {  
//                      过期后尝试获取锁，成功后开启独立线程  
                if(lock.tryLock(30)){  
                try{  
//                            开启独立线程  查询数据库重建Redis里面的信息  
                    new Thread(() -> {  
                        try{  
                            System.out.println("Lambda线程: " + Thread.currentThread().getName());  
                            emp e = empMapper.selectById(id);  
                            Thread.sleep(200);  
                            RedisEmp redisEmp=new RedisEmp();  
                            redisEmp.setExpireTime(String.valueOf(LocalDateTime.now().plusSeconds(30)));  
                            redisEmp.setData(String.valueOf(e));  
                            Map<String,Object> rempMap=ObjectToMapConverter.convertToMap(redisEmp);  
                            stringRedisTemplate.opsForHash().putAll(id, rempMap);  
                        } catch (Exception ex) {  
                            log.error("缓存重建失败", ex);  
                        } finally {  
                            lock.unlock(); // 确保锁释放  
                        }  
                    }).start();  
                } catch (Exception e) {  
                    lock.unlock(); // 主线程异常时释放锁  
                    throw e;  
                }  
                }  
            }  
            return re;  
        }  
//            如果是空对象，照样返回null  
        else {  
            return null;  
        }  
    }
```



缓存雪崩：同一时段大量的缓存key同时失效或者Redis服务宕机，导致大量请求到达数据库
给不同key的TTL添加随机值
利用Redis集群
<mark style="background: #BBFABBA6;">更新数据</mark>
 **避免“脏读”问题**
- **场景**：如果先删除缓存再更新数据库，在并发请求下可能出现以下问题：
    1. 线程A删除缓存。
    2. 线程B读取缓存未命中，从数据库读取**旧值**并写入缓存。
    3. 线程A更新数据库。  
        **结果**：缓存中残留旧数据，导致后续请求读到脏数据。
- **先更新数据库再删除缓存**可以极大降低这种概率，因为即使缓存删除失败，旧数据的存在时间也仅限于下一次缓存更新前的短暂窗口。

### 抢库存  如何避免超库存
解决方法：乐观锁，悲观锁，只有更新才能用乐观锁，观察前后状态是否一致
#### 乐观锁版本一
```java
//正常情况，扣减库存   乐观锁：判断更新时的库存值与查询到的库存值是否相等.eq("stock",voucher.getStock())  乐观锁还是不行,锁的太多了，库存一百个才卖出去二十多
@Transactional  
    @Override    public Result killCoupon(String voucherName) {  
//      判断库存是否充足，查数据库吧  
        RedisIdWorker idWorker=new RedisIdWorker(stringRedisTemplate);  
        int num;  
        if(( num=voucherMapper.selectNum(voucherName))>0)  
        {  
            if(voucherMapper.updateByname(voucherName,num)>0){  
                //这个写法是我改了这一时间段就是我的进程，别人就不能改，锁太大了，导致库存只少了20个  
                //因为有很多线程可能同时进来，在数据库层加锁，更新前和更新后的库存数要一样，更新时同时要判断一下  
                long temp=idWorker.nextId();  
                userorderMapper.insert(temp);  
                return Result.success("订单编号："+temp);  
            }  
            throw new RuntimeException("抢购失败，请重试");  
        }  
        throw new RuntimeException("库存不足");  
//        return Result.error("库存不足");  
    }

<update id="updateByname">  
    update voucher set numbers=numbers-1 where VoucherName=#{VoucherName}  and numbers=#{numbers}
    </update>
```
#### 乐观锁版本二
```java title:完美达到，200个线程正好有一百个成功实现
@Transactional  
    @Override    public Result killCoupon(String voucherName) {  
//      判断库存是否充足，查数据库吧  
        RedisIdWorker idWorker=new RedisIdWorker(stringRedisTemplate);  
        int num;  
        if(( num=voucherMapper.selectNum(voucherName))>0)  
        {  
            if(voucherMapper.updateByname(voucherName,num)>0){  
                //更新时判断库存是否大于0  
                long temp=idWorker.nextId();  
                userorderMapper.insert(temp);  
                return Result.success("订单编号："+temp);  
            }  
            throw new RuntimeException("抢购失败，请重试");  
        }  
        throw new RuntimeException("库存不足");  
//        return Result.error("库存不足");  
    }
```
### 如何实现一人一单
![[1749796273355_d.png]]
悲观锁，每个用户id加一个锁
#### 版本一
```java
//    同一时间只允许 一个线程 执行该方法 相当于串行执行  
//    注意这个版本仍然有线程安全问题  这个块synchronized (Integer.toString(userId).intern()) {  
//    执行完以后锁释放，事务未提交，还没写到数据库里面，会不会有第二个线程进来获取锁然后查询为空然后下单  
//    应该事务提交以后再释放锁  
    public  Result OneOrder(String voucherName, int userId, int num, RedisIdWorker idWorker) {  
//        注意这个toString方法，底层是创建了一个新的字符串对象 intern()先去字符串常量池里去找  
        synchronized (Integer.toString(userId).intern()) {  
  
            if (userorderMapper.selectByVnameId(voucherName, userId) != null)  
                throw new RuntimeException("已有订单");  
//            往下走，这是没下过单的  
            if (voucherMapper.updateByname(voucherName, num) > 0) {  
                //更新时判断库存是否大于0  
                long OrderNumber = idWorker.nextId();  
                userorderMapper.insert(OrderNumber, voucherName, userId);  
                return Result.success("订单编号：" + OrderNumber);  
            }  
            return null;  
        }  
    }
```


#### 版本二
```java
@Slf4j  
@Service  
public class voucherOrderServiceImpl implements voucherOrderService {  
    @Autowired  
    voucherMapper voucherMapper;  
    @Autowired  
    StringRedisTemplate stringRedisTemplate;  
    @Autowired  
    private OrderInnerService orderInnerService;  
  
    @Override  
    public Result killCoupon(String voucherName) {  
//      判断库存是否充足，查数据库吧  
        RedisIdWorker idWorker=new RedisIdWorker(stringRedisTemplate);  
        int num;  
        int userId=0001;  
        if(( num=voucherMapper.selectNum(voucherName))>0)  
        {  
            Result OrderNumber;  
//            根据优惠券名字和用户Id查询订单，看是否存在,简单点，也不用本地存储传了，直接写上得了  
//            有多个请求直接突破了这个查询，判断这个订单插入过没有，已经没法用之前乐观锁的方法了，要用悲观锁  
//            应该加在整个函数外面，把整个函数锁起来,这样事务执行完肯定是提交了的，数据库里有订单了  
//            注意要新增Service层，同类调用@Transactional会失效  
            synchronized (Integer.toString(userId).intern()) {  
                OrderNumber =orderInnerService.createOrder(voucherName,userId,num,idWorker.nextId()) ;  
            }  
            if (OrderNumber != null) return OrderNumber;  
            throw new RuntimeException("抢购失败，请重试");  
        }  
        throw new RuntimeException("库存不足");  
//        return Result.error("库存不足");  
    }
```

```java
// 新增服务类  
@Service  
public class OrderInnerService {  
    @Autowired  
    private userorderMapper userorderMapper;  
    @Autowired  
    private voucherMapper voucherMapper;  
  
    @Transactional(rollbackFor = Exception.class)  
    public Result createOrder(String voucherName, int userId, int num, long orderId) {  
        if (userorderMapper.selectByVnameId(voucherName, userId) != null) {  
            throw new RuntimeException("已有订单");  
        }  
        if (voucherMapper.updateByname(voucherName, num) > 0) {  
            userorderMapper.insert(orderId, voucherName, userId);  
            return Result.success("订单编号：" + orderId);  
        }  
        return null;  
    }  
}
```

#### 分布式锁
