## 로컬(windows) 도커 기반 레디스 설치
docker windows 설치
WSL 설치, ubuntu 또는 centos 설치

테스트는 wsl에서 centos 사용하면 systemctl, service 명령어가 안먹혀서 ubuntu로 진행함  
ubuntu 설치 후 windows desktop docker 설치하면 ubuntu에서 도커 접속 가능함  
`docker pull redis`  
`docker run -d -p 6379:6379 --name redis1 redis`  
`docker ps`  
`docker logs redis1`  

redis cli 사용  
`docker exec -it redis1 redis-cli` --> ping 날려보기

## docker-cli 명령어
Key / Value Serializer를 설정해주는 이유는 RedisTemplate에서 Spring ~ Redis간 데이터 직, 역직렬화 시 사용하는 방식이 Jdk 직렬화 방식이기 때문이다.  
동작에는 문제가 없지만 redis-cli를 통해 직접 데이터를 보려고 할 때 알아볼수 없는 형태로 출력되기 때문에 Serializer를 변경해준 것이다.  
한글의 경우 이 설정으로도 깨질수가 있는데, cli 접속 시 redis-cli --raw 이렇게 옵션을 주고 접속하면 정상적으로 볼 수 있다.  

```java
@Bean
public RedisConnectionFactory redisConnectionFactory(RedisConfig redisConfig) {
    RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
    redisStandaloneConfiguration.setHostName(redisConfig.getHost());
    redisStandaloneConfiguration.setPort(redisConfig.getPort());
    return new LettuceConnectionFactory(redisStandaloneConfiguration);
}

@Bean
public RedisTemplate<?, ?> redisTemplate(RedisConnectionFactory connectionFactory) {
    RedisTemplate<?, ?> redisTemplate = new RedisTemplate<>();
    redisTemplate.setConnectionFactory(connectionFactory);

    redisTemplate.setKeySerializer(new StringRedisSerializer());
    redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());

    // Hash Operation 사용 시
    redisTemplate.setHashKeySerializer(new StringRedisSerializer());
    redisTemplate.setHashValueSerializer(new StringRedisSerializer());
    return redisTemplate;
}

@Bean
public CacheManager redisCacheManager(RedisConnectionFactory connectionFactory, RedisConfig redisConfig) {
    RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration
            .defaultCacheConfig()
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
            .entryTtl(Duration.ofSeconds(redisConfig.getTimeout()));

    return RedisCacheManager.RedisCacheManagerBuilder.fromConnectionFactory(connectionFactory)
            .cacheDefaults(redisCacheConfiguration)
            .build();
}
```

https://stackoverflow.com/questions/37953019/wrongtype-operation-against-a-key-holding-the-wrong-kind-of-value-php
