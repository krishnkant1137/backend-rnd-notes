1️⃣ What is Cache? (start from here)
Cache is a temporary storage used to keep frequently used data so that we do not need to fetch it from the database every time.
Why cache is needed:
Database calls are slow
Repeated reads increase DB load
Cache improves response time

2️⃣ What is Second Level Cache?
Second level cache is a cache maintained inside the application by frameworks like Hibernate.
How it works:
First request → data comes from DB
Data is stored in application memory
Next request → data comes from cache
Important point: Second level cache works per application instance.


3️⃣ Limitations of Second Level Cache (VERY IMPORTANT)
This is where the problem starts.
Limitation 1: Not shared
If we have multiple application instances, each has its own cache
Data becomes inconsistent
Limitation 2: Not suitable for microservices
In microservices, many services run independently
Local cache cannot be shared across services
Limitation 3: Memory duplication
Same data stored multiple times in different instances
Wastes memory
Important point: Second level cache works well for monoliths, but not for distributed systems.

4️⃣ Now introduce Redis (this is the transition)
In production, we usually use microservices and multiple backend instances.
So we need a shared cache that all services can use.
That is where Redis comes in.

4️⃣ Now introduce Redis (this is the transition)
In production, we usually use microservices and multiple backend instances.
So we need a shared cache that all services can use.
That is where Redis comes in.

6️⃣ How Redis fits in production architecture
Multiple Services
     ↓
 Shared Redis Cache
     ↓
   Database

imp point: Redis acts as a centralized cache layer for the entire system.

6️⃣ How Redis fits in production architecture
Multiple Services
     ↓
 Shared Redis Cache
     ↓
   Database

imp point: Redis acts as a centralized cache layer for the entire system.

8️⃣ Redis limitations (important to know)
Limitation 1: Uses RAM
RAM is costly
Cannot store very large datasets
Limitation 2: Not permanent storage
Data can be lost on restart (if persistence not configured)
Limitation 3: Cache consistency
Cached data can become outdated
Needs eviction or TTL strategy

9️⃣ Are Redis limitations solved in production?
Yes, partially, with strategies.
1. Persistence options
RDB snapshots
AOF logs
(Used when limited durability is needed)
2. Cache-aside pattern
DB remains source of truth
Redis is rebuildable
3. TTL + Eviction strategies
Prevent stale and unused data
4. Redis Cluster
For scalability and availability
imp point: Redis limitations are managed using proper architecture, not ignored.

🔟 summary: 
Cache improves performance.
Second level cache is limited to a single application instance.
In microservices, we need a shared and distributed cache.
Redis solves this problem by providing a fast, centralized cache layer, while the database remains the source of truth.
------------------------------------------------------------------------------------------------------------

1️⃣ How Many Ways Can We Implement Redis?
In Spring Boot, Redis is mainly implemented in two ways:
Manual way using RedisTemplate
Automatic way using @Cacheable annotations
Both are valid.
The choice depends on use case, not preference.

Way 1: RedisTemplate (Manual Implementation)
What this way means
In this approach:
Developer manually writes Redis logic
We decide:
Cache key
TTL
When to read/write cache

How it works (flow)
Service checks Redis
If data exists → return it
If not → fetch from DB
Store data in Redis with TTL
Return data

Example (conceptual)
Doctor cached = redisTemplate.opsForValue().get(key);

if (cached != null) {
    return cached;
}

Doctor doctor = repository.findById(id);
redisTemplate.opsForValue().set(key, doctor, 5, TimeUnit.MINUTES);
return doctor;

Requirements
Entity must implement Serializable
RedisConfig must define RedisTemplate bean

When to use this approach
OTP storage
Tokens
Rate limiting
Custom TTL per key
Complex logic

Pros
Full control
Flexible
Works for advanced cases

Cons
More code
Easy to make mistakes
Repeated logic


Way 2: @Cacheable (Annotation-Based Implementation)
What this way means:
In this approach:
Developer does not write Redis code
Spring handles caching automatically
Cleaner and simpler code
How it works (flow)
Method is called
Spring checks Redis automatically
If cache hit → method does not execute
If cache miss → method executes and result is cached

Example
@Cacheable(value = "doctors", key = "#id")
public Doctor getDoctor(Long id) {
    return repository.findById(id);
}

Requirements:
@EnableCaching in RedisConfig
Redis cache manager configured
Serializable is not required (JSON serialization)

When to use this approach:
Read-heavy APIs
Simple GET operations
Business data caching

Pros:
Very clean code
Less error-prone
Easy to maintain
Industry preferred for APIs

Cons:
Less fine-grained control
TTL usually global per cache

Which Way Is Best?
There is no single best way.
The best way depends on what you are caching.

Simple rule (important):
Use RedisTemplate for infrastructure logic
Use @Cacheable for business logic

Best for:
RedisTemplate Best for-OTP, tokens
@Cacheable  Best for-API caching

-------
3️⃣ High-Level Architecture
Client
  ↓
Controller
  ↓
Service  ──▶ Redis (Cache)
  ↓            ↓
Repository ──▶ Database

Pattern used: Cache-Aside Pattern

4️⃣ Environment Setup (Pre-requisites)
Tools used: Spring Boot
Redis (running on Ubuntu via WSL)
MySQL (optional / future)
Java 17+
Redis Verification:
redis-cli ping
PONG

5️⃣ Spring Boot Dependencies
Selected dependencies:
Spring Web
Spring Data JPA
Spring Data Redis (Access + Driver)
MySQL Driver
Spring Boot DevTools

6️⃣ Application Configuration
application.properties
spring.redis.host=localhost
spring.redis.port=6379

── config
│   └── RedisConfig.java

code of this class (it is a fixed code you don't have to lean this) if you use redis templet way
//@Configuration
//@EnableCaching
//public class RedisConfig {
//
//    @Bean
//    public RedisTemplate<String, Object> redisTemplate(
//            LettuceConnectionFactory connectionFactory) {
//
//        RedisTemplate<String, Object> template = new RedisTemplate<>();
//        template.setConnectionFactory(connectionFactory);
//        return template;
//    }
//}


in servise class you have create refrence for this templace calss
          // @Autowired
  this-> // private final RedisTemplate<String, Object> redisTemplate;

like this -> i service layer we can use this 
//	public Doctor getDoctor(Long id) {
//
//	    String key = "doctor:" + id;
//	    Doctor cached = (Doctor) redisTemplate.opsForValue().get(key);
//	    if (cached != null) {
//	        return cached;
//	    }
//
//	    Doctor doctor = doctorRepository
//	            .findById(id)
//	            .orElseThrow(() ->
//	                new RuntimeException("Doctor not found"));
//
//	    redisTemplate.opsForValue().set(key, doctor, 5, TimeUnit.MINUTES);
//
//	    return doctor;
//	}