

~~~java
/** 
     * 启用新的get方法，防止缓存被“击穿” 
     * <p> 
     * 击穿 ：缓存在某个时间点过期的时候，恰好在这个时间点对这个Key有大量的并发请求过来， 
     *      这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。 
     * 如何解决：业界比较常用的做法，是使用mutex。 
     *      简单地来说，就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db，而是先使用缓存工具的某些带成 
     *      功操作返回值的操作（比如Redis的SETNX或者Memcache的ADD）去set一个mutex key，当操作返回成功时，再进行 
     *      load db的操作并回设缓存；否则，就重试整个get缓存的方法。 
     * </p> 
     * @param key 
     * @param pjp 
     * @param cache 
     * @return 
     * @throws Throwable 
     */  
public  class  RedisUtils{
    public static Object get(final String key, final ProceedingJoinPoint pjp, final Cacheable cache) throws Throwable {  
        @SuppressWarnings("unchecked")  
        ValueOperations<String, Object> valueOper = redisTemplate.opsForValue();  
        Object value = valueOper.get(key); // 从缓存获取数据  
        if (value == null) { // 代表缓存值过期  
            // 设置2min的超时，防止del操作失败的时候，下次缓存过期一直不能load db  
            String keynx = key.concat(":nx");  
            if (CacheUtils.setnx(keynx, "1", ExpireTime.TWO_MIN)) { // 代表设置成功  
                value = pjp.proceed();  
                if (cache.expire().getTime() <= 0) { // 如果没有设置过期时间,则无限期缓存  
                    valueOper.set(key, value);  
                } else { // 否则设置缓存时间  
                    valueOper.set(key, value, cache.expire().getTime(), TimeUnit.SECONDS);  
                }  
                CacheUtils.del(keynx);  
                return value;  
            } else { // 这个时候代表同时候的其他线程已经load db并回设到缓存了，这时候重试获取缓存值即可  
                Thread.sleep(10);  
                return get(key, pjp, cache); // 重试  
            }  
        } else {  
            return value;  
        }  
    } 
 }   
~~~