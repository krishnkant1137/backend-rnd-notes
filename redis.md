1️⃣ Problem Statement (R&D starts here)
In a normal backend application, every client request fetches data directly from the database.
Problems:
Database hit on every request
Slow response time
High DB load
Poor scalability
Goal: Reduce database calls and improve performance using Redis caching.

2️⃣ What is Redis?
Redis is an in-memory data store used mainly as a cache.
Key points:
Stores data in RAM
Extremely fast
Used with database (not a replacement)

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


🟢 WAY 1: RedisTemplate + Serializable (Manual way)
Entity / POJO classes

📄 Class
model → Doctor.java
👉 Redis me object store karna hai to:
implements Serializable

🔹 Why Serializable?
Redis object ko byte stream me convert karke store karta hai.

🟢 WAY 2: @Cacheable + Annotations (Automatic way ⭐)
Ye latest + recommended approach hai
ex-
@Cacheable(value = "doctors", key = "#id")
public Doctor getDoctor(Long id) {
    System.out.println("👉 Fetching from DATABASE");
    return doctorRepository.findById(id)
            .orElseThrow(() -> new RuntimeException("Not found"));
}

in RedisConfig you have add this anotaion
@Configuration
@EnableCaching
public class RedisConfig {
}


7️⃣ Redis Configuration
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(
            LettuceConnectionFactory connectionFactory) {

        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        return template;
    }
}

8️⃣ Service Layer (CORE REDIS LOGIC)
    private final RedisTemplate<String, Object> redisTemplate;

public Doctor getDoctor(Long id) {
    String key = "doctor:" + id;
    Doctor cached = (Doctor) redisTemplate.opsForValue().get(key);
    if (cached != null) {
        return cached;
    }
    Doctor doctor = doctorRepository
            .findById(id)
            .orElseThrow(() ->
                new RuntimeException("Doctor not found"));
    redisTemplate.opsForValue().set(key, doctor, 5, TimeUnit.MINUTES);
    return doctor;
}


First API Call:
GET /doctors

Console:
👉 Fetching from DATABASE
💾 Saved to REDIS

Second API Call:
GET /doctors

Console:
⚡ Data from REDIS

9️⃣ Why TTL is used?
TTL ensures:
Old data expires automatically
No stale cache
Memory management
Example:
5, TimeUnit.MINUTES


🔟 Advantages of Redis in this Project
Faster API response
Reduced DB load
Scalable architecture
Clean separation of layers

1️⃣1️⃣Limitations
Redis uses RAM (costly)
Not suitable for permanent storage
Requires cache invalidation strategy

1️⃣2️⃣ Redis is used as a cache layer to improve application performance.
The database remains the source of truth, while Redis optimizes read operations.
This implementation demonstrates real-world Redis usage in a layered Spring Boot application.